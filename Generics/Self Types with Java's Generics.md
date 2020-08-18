# Self Types with Java's Generics

In some situations, particularly when implementing the **builder pattern** or creating other **fluent APIs**, methods return `this`. The method’s return type is likely to be the **same as the class** in which the method is defined, but sometimes that doesn’t cut it! If we want to inherit methods and their return type should be the inheriting type (instead of the declaring type), then we’re fresh out of luck. We would need the return type to be something like “**the type of this**”, often called a ***self type*** but there is no such thing in Java.

Or is there?

## Some Examples

Before we go any further let’s look at some situations where self types would come in handy.

### `Object::clone`

A very good example is hiding in plain sight: [`Object::clone`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#clone--). With some [black magic incantations](https://books.google.de/books?id=ka2VUBqHiWkC&lpg=PA55&ots=yZIkJjn4Q3&dq=effective java clone&pg=PA54#v=onepage&q&f=false) it creates a copy of the object on which it is called. Barring [an exception](https://docs.oracle.com/javase/8/docs/api/java/lang/CloneNotSupportedException.html) that we can ignore here, it has the following signature:

```java
protected Object clone();
```

Note the return type: `Object`. Why so general? Let’s say we have a `Person` and would like to expose `clone`:

```java
public class Person {

    // ...

    @Override
    public Person clone() {
        return (Person) super.clone();
    }

}
```

We have to override the method to make it publicly visible. But we also have to cast the result of `super.clone()`, which is an `Object`, to `Person`. Mentally freeing ourselves from Java’s type system for a moment, we can see why that is weird. What else could `clone` return but an instance of the same class on which the method is called?

### Builder

Another example arises when employing the [builder pattern](http://www.informit.com/articles/article.aspx?p=1216151&seqNum=2). Let’s say our `Person` needs a `PersonBuilder`:

```java
public class PersonBuilder {

    private String name;

    public PersonBuilder withName(String name) {
        this.name = name;
        return this;
    }

    public Person build() {
        return new Person(name);
    }

}
```

We can now create a person as follows:

```java
Person doe = new PersonBuilder()
    .withName("John Doe")
    .build();
```

So far, so good.

Now let’s say we do not only have persons in our system, we also have employees and contractors. Of course they are also persons (right?) so `Employee extends Person` and `Contractor extends Person`. And because it went so well with `Person` we decide to create builders for them as well.

And here our journey begins. How do we set the name on our `EmployeeBuilder`?

We could just implement a method with the same name and code as in `PersonBuilder`, thus duplicating it except that it returns `EmployeeBuilder` instead of `PersonBuilder`. And then we do the same for `ContractorBuilder`? And then whenever we add a field to `Person` we add three fields and three methods to our builders? That doesn’t sound right.

Let’s try a different approach. What’s our favorite tool for code reuse? Right, [inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance). (Ok, that was a bad joke. But in this case I’d say that inheritance is ok.) So we have `EmployeeBuilder extend PersonBuilder` and we get `withName` for free.

But while the inherited `withName` indeed returns an `EmployeeBuilder`, the compiler does not know that–the inherited method’s return type is declared as `PersonBuilder`. That’s no good either! Assuming our `EmployeeBuilder` does some employee-specific stuff (like `withHiringDate`) we can’t access those methods if we call in the wrong order:

```java
Employee doe = new EmployeeBuilder()
    // now we have an EmployeeBuilder
    .withName("John Doe")
    // now we have a PersonBuilder
    .withHiringDate(LocalDateTime.now()) // compile error :(
    .build();
```

We could override `withName` in `EmployeeBuilder`:

```java
public class EmployeeBuilder {

    public EmployeeBuilder withName(String name) {
        return (EmployeeBuilder) super.withName(name);
    }

}
```

But that requires almost as many lines as the original implementation. Repeating such snippets in every subtype of `PersonBuilder` for each inherited method, is clearly not ideal.

Taking a step back, let’s see how we ended up here. **The problem is** that the return type of `withName` is explicitly fixed to the class that declares the method: `PersonBuilder`.

```java
public class PersonBuilder {

    public PersonBuilder withName(String name) {
        this.name = name;
        return this;
    }

}
```

So if the method is inherited, e.g. by `EmployeeBuilder`, the return type remains the same. But it shouldn’t! It should be the type on which the method was called instead of the one that declares it.

This problem quickly rears its head in fluent APIs, like the builder pattern, where the return type is of critical importance to make the whole API work.

### Recursive Containers

Finally let’s say we want to build a graph:

```java
public class Node {

    private final List<Node> children;

    public Stream<? extends Node> children() {
        return children.stream();
    }

}
```

Down the road we realize that we need different kinds of nodes and that trees can only contain nodes of one kind. As soon as we use inheritance to model the different kinds of nodes we end up in a very similar situation:

```java
public class SpecialNode extends Node {

    @Override
    public Stream<? extends SpecialNode> children() {
        return super.stream(); // compile error :(
    }

}
```

A `Stream` is no `Stream` so this doesn’t even compile. Again we have the problem that we would like to say “we return a stream of nodes of the type on which this method was called”.

## Self Types to the Rescue

Some languages have the concept of self types:

**A \*self type\* refers to the type on which a method is called (more formally called the \*receiver\*).**

If a self type is used in an inherited method, it represents a different type in each class that declares or inherits that method, namely *that specific class*, no matter whether it declared or inherited the method. Casually speaking it is the compile-time equivalent of `this.getClass()` or “**the type of this**”. Self type simply means the type of the receiver of instance methods.

In the remainder of this post I will notate it as `[this]` (`[self]` would be good, too, but with `[this]` we get some syntax highlighting).



## The Examples with Self Types

To be perfectly clear: **Java has no self types**. But what if it had? How would our examples look then?

### `Object::clone`

`Object::clone` was supposed to return a copy of the instance it was called on. That instance should of course be of the same type, so the signature could look as follows:

```java
protected [this] clone();
```

Subclasses can then directly get an instance of their own type:

```java
public class Person {

    // ...

    @Override
    public [this] clone() {
        // no cast required
        // because in this class [this] means Person
        Person clone = super.clone();
        return clone;
    }

}
```

### Builder

For the builders we discussed above, the solution is similarly obvious. We simply declare all `with...` methods to return the type `[this]`:

```java
public class PersonBuilder {

    public [this] withName(String name) {
        this.name = name;
        return this;
    }

}
```

As before `EmployeeBuilder::withName` returns an instance of `EmployeeBuilder` but this time the compiler knows that and what we wanted before works now:

```java
Employee doe = new EmployeeBuilder()
    // now we have an EmployeeBuilder
    .withName("John Doe")
    // still an EmployeeBuilder thanks to [this]
    .withHiringDate(LocalDateTime.now()) // works! :)
    .build();
```

### Recursive Containers

Last but not least, let’s see how our graph works out:

```java
public class Node {

    private final List<[this]> children;

    public Stream<? extends [this]> children() {
        return children.stream();
    }

}
```

Now `SpecialNode::children` returns a `Stream`, which is exactly what we wanted.

## Emulating Self Types with Generics

While Java doesn’t have self types, there is a way to emulate them with generics. This is limited and a little convoluted, though.

If we had a generic type parameter referring to this class, say `THIS`, we could simply use it wherever we used `[this]` above. But how do we get `THIS`? Simple (almost), we just add `THIS` as a type parameter and have inheriting classes specify their own type as `THIS`.

Wait, what? I think an example clears this up.

```java
public class Object<THIS> {

    protected THIS clone();

}

public class Person extends Object<Person> {

    @Override
    public Person clone() {
        // no cast required because
        // in this class THIS was specified as Person
        Person clone = super.clone();
        return clone;
    }

}
```

If this looks fishy (what does `Object` even mean?), you already stumbled upon one of the weaknesses of this approach but we’ll cover that in a minute. First, let’s explore it a little more and check our other examples.

```java
public class PersonBuilder<THIS> {

    private String name;

    public THIS withName(String name) {
        this.name = name;
        return (THIS) this;
        // if we do this more often, '(THIS) this'
        // should become its own method
    }
}

public class EmployeeBuilder
        extends PersonBuilder<EmployeeBuilder> { }
```

Now `EmployeeBuilder::withName` returns an `EmployeeBuilder` without us having to do anything.

Similarly our problems with `Node` goes away:

```java
public class Node<THIS> {

    private final List<THIS> children;

    public Stream<? extends THIS> children() {
        return children.stream();
    }

}

public class SpecialNode extends Node<SpecialNode> { }
```

Same as with `[this]`, we need no additional code in `SpecialNode`.

## Limitations and Weaknesses

That doesn’t look too bad, right? But there are some limitations and weaknesses that we have to iron out.

### Confusing Abstraction

As I said above, what does `Object` even mean? **Is it an object that holds, creates, processes or otherwise deals with a person?** Because that is **how we usually understand a generic type argument**. But it is not, it’s just an “Object of Person”, which is rather strange.

### Convoluted Types

It is also unclear how to declare the supertypes now. Is it `Node`, `Node`, or even more `Node`s? Consider the following:

```java
Node<Node> node = new Node<>();
Stream<Node> children = node.children();
Stream grandchildren = children
    .flatMap(child -> child.children());
```

Calling `children` twice, we “used up” the generic types we declared for `node` and now we get a raw stream. Note that thanks to the recursive declaration of `SpecialNode extends Node` we don’t have that problem there:

```java
SpecialNode node = new SpecialNode();
Stream<SpecialNode> children = node.children();
Stream<SpecialNode> grandchildren = children
    .flatMap(child -> child.children());
```

All of this will confuse and ultimately alienate users of such types.

### Single Level Inheritance

The trick to get `SpecialNode` to behave as expected on repeated calls to `children`meant that it had no self-referencing type parameter of its own. Now, a type extending it can not specify itself anywhere, so its methods return special nodes instead:

```java
public class VerySpecialNode extends SpecialNode { }

VerySpecialNode node = new VerySpecialNode();
// we a want Stream<VerySpecialNode>
Stream<SpecialNode> children = node.children(); // damn :(
```

### `THIS` Is Too Generic

As it stands, `THIS` can be any type, which means we must treat it as an `Object`. But what if we wanted to do something with our instances of it that was specific to our current class?

```java
public class Node<THIS> {

    // as before, especially `children`

    public Stream<THIS> grandchildren() {
        // doesn't compile because `child` is no `Node`
        // and hence has no `children` method
        return children.flatMap(child -> child.children());
    }

}
```

Well, that’s just stupid.

## Refining the Approach

Let’s see if we can’t do a little better than before and tackle some of those weaknesses. And there is indeed something we can do that addresses all the problems we just discussed.

### Recursive Generics

Last things first, let’s look at `THIS` being too generic. This can easily be fixed with **recursive generics**:

```java
public class Node<THIS extends Node<THIS>> {

    // as before, especially `children`

    public Stream<THIS> grandChildren() {
        // hah, now `child` is a `Node`
        return children.flatMap(child -> child.children());
    }

}
```

(I said “easily” not “simply”.)

But this exacerbates the problem of convoluted types considerably. Now it’s not even possible to declare a `Node` because the compiler always expects *another* type parameter:

```java
// doesn't compile
// the fourth `Node` is not
// within its type bounds of `Node<Node>`
Node<Node<Node<Node>>> node = new Node<>();
```

### Private Hierarchy

We can fix this, though, and the remaining problems with another detour.

**We can create a hierarchy of abstract classes that contain all the code we talked about so far. The concrete implementations our clients will use are then offshoots of that hierarchy. Because nobody will ever directly use the abstract classes we can use `THIS` on every level of inheritance–the implementations will then specify it.**

Ideally the abstract classes are package visible so their hideousness is hidden from the outside world.

```java
abstract class NodeScaffold<THIS extends NodeScaffold<THIS>> {

    private final List<THIS> children;

    public Stream<THIS> children() {
        return children.stream();
    }

    public Stream<THIS> grandChildren() {
        return children.stream()
            .flatMap(child -> child.children());
    }

}

abstract class SpecialNodeScaffold<THIS extends SpecialNodeScaffold<THIS>>
        extends NodeScaffold<THIS> {
    // special methods
}

abstract class VerySpecialNodeScaffold<THIS extends VerySpecialNodeScaffold<THIS>>
        extends SpecialNodeScaffold<THIS> {
    // more special methods
}

public class Node
        extends NodeScaffold<Node> { }

public class SpecialNode
        extends SpecialNodeScaffold<SpecialNode> { }

public class VerySpecialNode
        extends VerySpecialNodeScaffold<VerySpecialNode> { }
```

A small detail: The publicly visible classes do not inherit from one another so a `SpecialNode` is no `Node`. If we did the same with the builders that might be ok but it is awkward with nodes. To fix this, we need yet another layer of abstraction, namely some interfaces that extend each other and are implemented by the public classes.

Now the users of the public classes see clear abstractions and no convoluted types. The approach works across arbitrary many levels of inheritance and `THIS` always refers to the most specific type.

## `THIS` As Argument Type

You might have noticed that all the examples use `[this]` and `THIS` as a return type. Can’t we also use it as a type for arguments? Unfortunately not because while return types are covariant, argument types are contravariant, and `[this]`/`THIS` is inherently covariant. (Check out [this StackOverflow question](http://stackoverflow.com/q/2501023/2525313) for a brief explanation of these terms.)

If you try to do it (e.g. by adding `void addChild(THIS node)`) following the approach above, it will seem to work out until you try to create the interfaces that bring `Node`, `SpecialNode`, and `VerySpecialNode` into an inheritance relationship. Then the type of `node` can not become more specific as you go down the inheritance tree.

This post is already long enough so I will leave the details as an exercise to the curious reader.

## Summary

We have seen why we would sometimes need to reference “the type of this” and how a language feature could look like that does that. But Java doesn’t have that feature, so we had to come up with some tricks to do it ourselves:

- we defined the methods we want to inherit in an abstract classes (`A`)
- we gave them a recursive generic type parameter (`A>`)
- we created publicly visible concrete implementations that specify themselves as that type (`C extends A`)
- if required we create an interface inheritance tree that our concrete classes implement

Whether this was worth all the effort and trickery is up to you to decide and depends on the use case you have. Generally speaking, the more methods you can inherit this way the more you’ll feel the benefits (look at [AssertJ’s API implementation](https://github.com/joel-costigliola/assertj-core/tree/2c5f011d3c99d86f5d42a743a28238440729ae7f/src/main/java/org/assertj/core/api) for an example of how many this can be). Conversely the friction from understanding this pattern decreases when the classes you create this way are fundamental for your code base, in line with “if it hurts, do it more often”. If it remains a fringe solution, developers will not know it’s there and stumble into it inadvertently and unexpectedly, being more easily confused.

What do you think? Do you see a use case in your code base? I’m interested to hear about it, so leave a comment.