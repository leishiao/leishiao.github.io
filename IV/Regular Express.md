### Regular Express

Tutorials: https://docs.oracle.com/javase/tutorial/essential/regex/index.html

#### String Literals

The most basic form of pattern matching supported by this API is the match of a string literal. 

#### Metacharacters

This API also supports a number of `special characters` that affect the way a pattern is matched.

The metacharacters supported by this API are: `<([{\^-=$!|]})?*+.>`

There are two ways to force a metacharacter to be treated as an ordinary character:

- precede the metacharacter with a backslash, or
- enclose it within `\Q` (which starts the quote) and `\E` (which ends it).

When using this technique, the `\Q` and `\E` can be placed at any location within the expression, provided that the `\Q` comes first.

#### Character Classes

Tutorials: https://docs.oracle.com/javase/tutorial/essential/regex/char_classes.html

| Construct       | Description                                              |
| --------------- | -------------------------------------------------------- |
| `[abc]`         | a, b, or c (simple class)                                |
| `[^abc]`        | Any character except a, b, or c (negation)               |
| `[a-zA-Z]`      | a through z, or A through Z, inclusive (range)           |
| `[a-d[m-p]]`    | a through d, or m through p: [a-dm-p] (union)            |
| `[a-z&&[def]]`  | d, e, or f (intersection)                                |
| `[a-z&&[^bc]]`  | a through z, except for b and c: [ad-z] (subtraction)    |
| `[a-z&&[^m-p]]` | a through z, and not m through p: [a-lq-z] (subtraction) |

The left-hand column specifies the regular expression constructs, while the right-hand column describes the conditions under which each construct will match.

------

**Note:** The word "class" in the phrase "character class" does not refer to a `.class` file. In the context of regular expressions, a *character class* is a set of characters enclosed within square brackets. **It specifies the characters** that will successfully match **a single character** from a given input string.

##### Simple Classes

The most basic form of a character class is to simply place **a set of characters side-by-side** within square brackets. For example, the regular expression `[bcr]at` will match the words "bat", "cat", or "rat" because **it defines a character class** (accepting either "b", "c", or "r") as its first character.

##### Negation

To match all characters *except* those listed, insert the "`^`" metacharacter at the beginning of the character class. This technique is known as *negation*.

##### Ranges

Sometimes you'll want to define a character class that **includes a range of values**, such as the letters "a through h" or the numbers "1 through 5". To specify a range, simply insert the "`-`" metacharacter between the first and last character to be matched, such as `[1-5]` or `[a-h]`. You can also place different ranges beside each other within the class to further expand the match possibilities. For example, `[a-zA-Z]` will match any letter of the alphabet: a to z (lowercase) or A to Z (uppercase).

##### Unions

You can also use *unions* to create a single character class comprised of **two or more** separate character classes. To create a union, simply **nest one class inside the other**, such as `[0-4[6-8]]`.

##### Intersections

To create a single character class matching only the characters common to **all of its nested classes**, use `&&`, as in `[0-9&&[345]]`. This particular intersection **creates a single character class** matching only the numbers **common to both character classes**: 3, 4, and 5.

##### Subtraction

Finally, you can use *subtraction* to **negate one or more nested character classes**, such as `[0-9&&[^345]]`. This example creates a single character class that matches everything from 0 to 9, *except* the numbers 3, 4, and 5.

#### Predefined Character Classes

Tutorials: https://docs.oracle.com/javase/tutorial/essential/regex/pre_char_classes.html

The [`Pattern`](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html) API contains a number of useful ***predefined character classes***, which offer convenient shorthands for commonly used regular expressions:

| Construct | Description                                           |
| --------- | ----------------------------------------------------- |
| `.`       | Any character (may or may not match line terminators) |
| `\d`      | A digit: `[0-9]`                                      |
| `\D`      | A non-digit: `[^0-9]`                                 |
| `\s`      | A whitespace character: `[ \t\n\x0B\f\r]`             |
| `\S`      | A non-whitespace character: `[^\s]`                   |
| `\w`      | A word character: `[a-zA-Z_0-9]`                      |
| `\W`      | A non-word character: `[^\w]`                         |

In the table above, each construct in the left-hand column is shorthand for the character class in the right-hand column. For example, `\d` means a range of digits (0-9), and `\w` means a word character (any lowercase letter, any uppercase letter, the underscore character, or any digit). Use the predefined classes whenever possible. 

Constructs beginning with a backslash are called ***escaped constructs***.  If you are using an escaped construct within a string literal, you must precede the backslash with another backslash for the string to compile. For example:

```java
private final String REGEX = "\\d"; // a single digit
```

In this example `\d` is the regular expression; the extra backslash is required for the code to compile. The test harness reads the expressions directly from the `Console`, however, so the extra backslash is unnecessary.

You can refer to this table to figure out the logic behind each match:

- `\d` matches all digits
- `\s` matches spaces
- `\w` matches word characters

Alternatively, a capital letter means the opposite:

- `\D` matches non-digits
- `\S` matches non-spaces
- `\W` matches non-word characters

#### Quantifiers

Tutorials: https://docs.oracle.com/javase/tutorial/essential/regex/quant.html

*Quantifiers* allow you to specify the number of occurrences to match against. For convenience, the three sections of the Pattern API specification describing greedy, reluctant, and possessive quantifiers are presented below. At first glance it may appear that the quantifiers `X?`, `X??` and `X?+` do exactly the same thing, since they all promise to match "`X`, once or not at all". There are subtle implementation differences which will be explained near the end of this section.

| Greedy   | Reluctant | Possessive | Meaning                                           |
| -------- | --------- | ---------- | ------------------------------------------------- |
| `X?`     | `X??`     | `X?+`      | `X`, once or not at all                           |
| `X*`     | `X*?`     | `X*+`      | `X`, zero or more times                           |
| `X+`     | `X+?`     | `X++`      | `X`, one or more times                            |
| `X{n}`   | `X{n}?`   | `X{n}+`    | `X`, exactly *`n`* times                          |
| `X{n,}`  | `X{n,}?`  | `X{n,}+`   | `X`, at least *`n`* times                         |
| `X{n,m}` | `X{n,m}?` | `X{n,m}+`  | `X`, at least *`n`* but not more than *`m`* times |

##### Zero-Length Matches ?

##### Capturing Groups and Character Classes with Quantifiers ?

##### Differences Among Greedy, Reluctant, and Possessive Quantifiers ?

#### Capturing Groups

Tutorials: https://docs.oracle.com/javase/tutorial/essential/regex/groups.html

*Capturing groups* are a way to **treat multiple characters as a single unit**. 

#### Boundary Matchers

Tutorials: https://docs.oracle.com/javase/tutorial/essential/regex/bounds.html

