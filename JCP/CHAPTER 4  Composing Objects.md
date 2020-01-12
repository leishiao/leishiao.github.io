## CHAPTER 4 : Composing Objects

### 4.1 Designing a thread-safe class

Encapsulation makes is possible to determine that a class is thread-safe without examining the entire program.

The design process for a thread-safe class should include these three basic elements.

> - Identify the variables that form the object's state;
> - Identify the invariants that constrain the state variables;
> - Establish a policy for managing concurrent access to the object's state.

​	An *object's state* starts with its fields. If they are all of primitive type, the fields comprise the entire state. If the object has fields that are references to other objects, its state will encompass fields from the referenced objects as well.

​	The *synchronization policy* defines how an object coordinates access to its state without violating its invariants ant postconditions. It specifies what combination of immutability, thread confinement, and locking is used to maintain thread safety and witch variables are guarded by witch locks .

#### 4.1.1 Gathering synchronization requirements

> You cannot ensure thread safety without understanding an object's invariants and postconditions. Constraints on the valid values or state transitions for state variables can create atomicity and encapsulation requirements.
>

#### 4.1.2 State-dependent Operations

Operations with state-based preconditions are called state-dependent.

What is the consequence if a precondition does not hold.

What are the build-in mechanisms for efficiently waiting for condition to become true.  

#### 4.1.3 State ownership

When defining which variables form an object's state, we want to consider only the data that the object owns.

In many cases, ownership and encapsulation goes together -- the object encapsulates the state it owns and owns the state it encapsulate. It is the owner of a given state variable that gets to decide on the locking protocol used to maintain the integrity of that variable's state. Ownership implies control, but once you publish a reference to a mutable object, you no longer have exclusive control; at best, you might have "shared ownership". A class usually does not own the objects passed to its method and constructors, unless the method is designed to explicitly transfer ownership of objects passed in.

Collection classes often exhibit a form of "spilt ownership", in which the collection owns the state of the infrastructure, but client code owns the objects stored in the collection.

### 4.2 Instance confinement

Encapsulation simplifies making classes thread-safe by promoting *instance confinement*, often just called *confinement*. Combining confinement with an appropriate  locking discipline can ensure that otherwise  non-thread-safe objects are used in a thread-safe manner.

> Encapsulating data with an object confines access to the data to the object's methods, making it easier to ensure that the data is always accessed with the appropriate lock held.

Confined object must not escape their intended scope.

Instance confinement allows different state variables to be guarded by different locks.

> Confinement makes it easier to build thread-safe classes because a class that confines its state can be analyzed for thread-safety without examine the whole program.

#### 4.2.1 The Java monitor Pattern

An object following the Java monitor pattern encapsulates all its mutable state and guards it with its intrinsic lock.

There are advantages to using a private lock object instead of an object's intrinsic lock (or any other publicly accessible lock).

### 4.3 Delegating Thread safety

The Java monitor pattern is useful when building classes from scratch or composing classes out of objects that are not thread safe. But what if the components of our classes are already thread-safe? Do we need to add an additional layer of thread safety? 

We could say that CountingFactorizer delegates its thread safety responsibilities to the AtomicLong: CountingFactorizer is thread-safe because AtomicLong is. 

#### 4.3.2 Independent state variables

The delegation example so far delegate to a *single*, thread-safe state variable. We can also delegate thread safety to more than one underlying state variable as long as those underlying variables are independent, meaning that the composite class does not impose any invariants involving the multiple state variables.

#### 4.3.3 When delegation fails

Most composite classes have invariants that relate their component state variables. If a class has compound actions, delegation alone is again not a suitable approach for thread safety. In these cases, the class must provide its own locking to ensure that the compound actions are atomic, unless the entire compound action can also be delegated to the underlying state variables.

> If a class is composed of multiple *independent* thread-safe state variables and has no operations that have any invalid state transitions, then it can delegate thread safety to the underlying state variables.

#### 4.3.4 Publishing underlying state variables

When you delegate thread safety to an object's underlying state variables, under what conditions can you publish those underlying variables so that other classes can modify them as well? Again, the answer depends on what invariants your class imposes on those state variables.

> If a state variable is thread-safe, dose not participate in any invariants that constrains its value, and has  no prohibited state transitions for any of its operations, then it can safely be published.

Publishing underlying state variables may not be a good idea, since publishing mutable variables constrains future development and opportunities for subclassing.

### 4.4 Adding functionality to existing thread-safe classes

Sometimes a thread-safe class that supports all of the operations we want already exists, but often the best we can find is a class that supports *almost* all the operations we want, and then we need to add a new operation to it without undermining its thread safety.

 The safest way to add a new atomic operation is to modify the original classes to support the desired operation, but this is not always possible because you may not have access to the source code or may not be free to modify it.

If you can modify the original class, you need to understand the implementation's synchronization policy so that you can enhance it in a manner consistent with its original design. 

Another approach is to extend the class, assuming it was designed for extension. But not all classes expose enough of their state to subclass to admit this approach.

Extension is more fragile than directly adding code to a class, because of the implementation of synchronization policy is now distributed over multiple, separately maintained source files.

#### 4.4.1 Client-side locking

Sometimes, neither of above two approach adding method to the original class or extending the class works.

A third strategy is to extend the functionality of a class without extending the class itself by placing extension code in a "helper "class.

Client-side locking entails guarding client code that use some object X with the lock X uses to guard its own state. In order to use client-side locking, you must know what lock X uses. 

Client-side locking is even more fragile than extending a class, because it entails putting locking code for class C into classes that are totally unrelated to C.

#### 4.4.2 Composition

There is a less fragile alternative for adding an atomic operation to an existing class: *composition*.

This pattern creates a new class  that implements the interfaces of the underlying class by delegate those operations to the underlying class and add new atomic operations with own intrinsic lock hold. 

This strategy  does not care whether the underlying class is thread-safe, because it provides its own consistent locking that provides thread safety even if the underlying class changes its locking implementation.