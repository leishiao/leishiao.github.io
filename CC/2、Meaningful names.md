## 2、Meaningful names

We name and name and name. Because we do so much of it, we'd better do it well.

**Use Intention-revealing Names**

Choosing good names takes time but it saves more than it take. So take care of your names and change them when you find better ones. 
The name of a variable, function, or class should answer all the big questions. It should tell you why it exists, what it does and how it is used. If a name requires a comment, if does not reveal its intent.

**Avoid Disinformation**

We should avoid words whose entrenched meanings vary from our intended meaning. 
Do not refer to a grouping of accounts as an `accountList` unless it's actually a `List`.
Beware of using names which vary from in small ways.
Spelling similar concepts similarly is *information*. Using inconsistent spelling is *disinformation.*

**Make Meaningful Distinctions**

If name must be different, then they should mean something different.
Number-series naming (a1, a2, .. aN) is the opposite if intentional naming, they are noninformative.
Noise words are another meaningless distinction. *Info* and *Data* are indistinct noise words like *a*, an, and *the*.

**Use Pronounceable Names**

A significant part of our brains is dedicated to the concept of words.
If you can't pronounce it, you can't discuss it without sound like a idiot.
This matters because programing is a social activity.

**Use Searchable Names**

Single-letter names and numeric constants have a particular problem in that they are not easy to locate across a body of text.
In this regard, longer names trump short names, searchable name trumps a constant in code.
Single-letter names can ONLY be used as local variables inside short methods.

**Avoid Encoding**

Encoding type or scope information into names simply adds an extra burden of deciphering.
Nowadays HN and other forms of type encoding are simply impediments.

**Member Prefix**

You also don't need to prefix member variables with *m_* anymore. 
Your classes and methods should be small enough that you don't need them.
Eventually prefixes become unseen clutter and a marker of older code.

**Avoid Mental Mapping**

Readers shouldn't have to mentally translate your names into other names they already know. This problem generally arises from a choice to use neither problem domain terms nor solution domain terms.

One difference between a smart programmer and a professional programmer is that the professional understands that *clarity is king*. Professionals use their powers for good and write code that others can understand.

**Class Names**

Classes and objects should have none or noun phrase names like *Customer*, *WikiPage*, Account, and *AddressPaser*. Avoid words like *Manager*, Processor, *Data*, or *Info* in the name of a class. A class name should not be a verb.

**Method Names**

Methods should have *verb* or *verb* phrase names like *postPayment*, *deletePage*, or *save*. Accessors, mutators, and predicates should be named for their value and prefixed with get, set, and is according to the javabean standard.
When constructors are overloaded, use static factory methods with names that describe the arguments. 

**Don't Be Cute**

Choose clarity over entertainment value.

**Pick One Word per Concept**

Pick one word for one abstract and stick with it. For instance, it is confusing to have *fetch*, *retrieve*, and *get* as equivalent methods of different classes. Likewise, it is confusing to have a *controller* and a *manager* and a *driver* in same code base.

**Don't Pun**

Avoid using the same word for two purpose. Using the same term for two different ideas is essentially a pun. Our goal, as authors, is to make our code as easy as possible to understand. We want our code to be a quick skim, not an intense study.

**Use Solution Domain Names**

There are lots of very technical things that programmers have to do. Choosing technical names for those things is usually the most appropriate course.

**Add Meaningful Context**

There are few names which are meaningful in and of themselves, but most are not. Instead, you need to place names in context for your reader and enclosing them in well-named classes, functions, and namespaces. When all else fail, then prefixing may be necessary as a last resort.

**Don't Add Gratuitous Context**

Shorter names are generally better than longer ones, so long as they are clear. Add no more context to a name than is necessary.







