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

You cannot ensure thread safety without understanding an object's invariants and postconditions. Constraints on the valid values or state transitions for state variables can create atomicity and encapsulation requirements.

