# NoDots Solution Explanation
## Avoiding Backslashes
Avoiding backslashes isn't a problem at all, you don't really need them.

## Avoiding Semicolons
Avoiding semicolons is a little trickier, but you can generally avoid
them by structuring any statement which is a valid expression as an
if-statement instead:

Illegal:
```java
produceSomeValue();
```

Legal:
```java
if (produceSomeValue() == null) {}
```

It's a little trickier when code doesn't return a value, because you
can't include an expression which returns `void` inside a conditional.

However, java *does* have a control structure which will happily
execute code with a void signature: try-with-resources.

The try-with-resources control accepts an AutoCloseable instance, which
is in interface that helpfully looks like:

```java
interface AutoCloseable {
  void close() throws Exception;
}
```

This means we can coerce any arbitrary piece of code that does not
return a value into an `AutoCloseable`, regardless of any
`Exception`s it might throw. And, we can get java to execute the
`close()` method for us by sticking it in a try-with-resources block,
which will call `close()` when it completes.

Illegal:
```java
doTheThing();
```

Legal:
```java
try (AutoCloseable ignored = this::doTheThing) {}
```

## Avoiding Dots
This is the hard part.

With the above, we can get java to call any method or constructor
which we have access to, as long as we don't have to use any `.`'s to
reference it. Which is hard, because we need to access `System.out::println`.

How do you access methods without using `.`? Well, in java you can call a method
directly if it's defined by the same class you're in, or a *superclass*. I.e.:

Illegal:
```java
Foobar.doTheThing()
```

Legal:
```java
class Solution extends Foobar {
  ...
  doTheThing()
}
```

E.g., this can also be done with an anonymous inner class and an initializer block:
```java
if (new Foobar() {{ try (AutoCloseable c = doTheThing()) {} }} == null) {}
```

Unfortunately, `System` is a final class, so we can't subclass it to access `out`. And
because we can't use `.` or `;`, we can't import anything fancy: we're stuck with what's
in `java.lang.*`.

If you peruse the classes in `java.lang`, there's one very, very powerful one in
particular that isn't final: `ClassLoader`.

In particular, `ClassLoader` has a `defineClass(..)` method which takes in a raw
byte array and returns a `Class<?>` instance. This means we can write arbitrary code
in a separate class to print `"Hello world"` when it is loaded. Then we just need to
force it to load.

So write this code that would be entirely illegal:
```java
public class YesDots {
  static {
    System.out.println("Hello world");
  }
}
```

And compile it down to a byte array:
```java
new byte[] { (byte) 0xca, (byte) 0xfe, (byte) 0xba, (byte) 0xbe, ... , (byte) 0xd }
```

Then we can call `defineClass(..)` with that byte array. Unfortunately, that's not
quite enough: we need to access a member of the class (or create an instance of it)
to force java to actually load it and run the static initializer with our payload.

To do this, we can get a method reference to `newInstance`, and then coerce that
into an `AutoCloseable` which we can make java call with a try-with-resources
statement, producing the final output:


```java
    try (AutoCloseable c = defineClass("YesDots",
        new byte[] { (byte) 0xca, (byte) 0xfe, (byte) 0xba, (byte) 0xbe, ... , (byte) 0x0, (byte) 0xd }, 0, 398
    )::newInstance) {}
```

