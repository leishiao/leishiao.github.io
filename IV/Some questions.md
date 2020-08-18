#### Some questions

1、String， StringBuffer， StringBuilder 的故事？

Refer to EJ3.item17:minimize mutability

```
StringBuilder 
1.A mutable sequence of characters. 
2.As a drop-in replacement for StringBuffer when used by a single thread (as is generally the case).
3.Prefer StringBuilder to Stringbuffer,as it is faster.
4.It is recommended that StringBuffer can be used by multiple threads.
```

