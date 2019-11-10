---
layout:     post
title:      "Java Basics - Concurrent Callable"
date:       2019-11-10 16:10:00 +0200
modified:   2019-11-10 16:10:00 +0100
categories: basics
keywords:   java, basics, concurrent, callable
video:      r8U05bJIDQo
abstract:   "Callable, a simple class from `java.utils.concurrent`"
author:     "Rainer Jung"
---
Java Basics - Concurrent Callable
=================================

One of the most frequent used packages of java is the `java.util` package. One
of the inner packages is the `java.util.concurrent` package. As there's lot of
change going around with reactive programming, let's take a look at this
package and the classes within.

This time, it's the simple interface `Callable`.

`java.util.concurrent.Callable`
-------------------------------

The class is an interface. So it defines methods to be implemented. This one
has only one method to be implemented, `V call() throws Exception;`. Besides
this simple method that can be implemented, this interface has two additional
specialities.  
The class uses "generics", so you can define, which type of response this
method shall have. You can see the generics definition by the capital `V`
between the lower than and greater than characters. It defines that `V` can be
replaced by any `Class` (I shortly described a class in javacast about the very
[basics]({% link _posts/2017-02-11-basics-hello-world.markdown %}) on java).  
The class is also annotated with `@FunctionalInterface`, which does not give
any functionality, but marks that this class has only one method. We get there
later, what that can be used for.

In the documentation, it says:
> A task that returns a result and may throw an exception. Implementors define
> a single method with no arguments called `call`.

Let's write a test about this.

Call a `Callable`
-----------------

The test is pretty simple, we need to create a class of `Callable`. We will
just create an inline class for this (so there's no class definition in it's
own file). The `Callable` has the generics definition to return a `String`,
we implement the `call` method and return the string `Hello Callable`. As the
method can throw an `Exception`, the test needs to catch and handle it, but for
now, I will just have the test method defined to throw an `Exception` itself.

```java
@Test
public void verifyCallingCallableReturnsAValue() throws Exception {
  Callable<String> callable = new Callable<String>() {
    public String call() throws Exception {
      return "Hello callable";
    }
  };

  String result = callable.call();

  assertThat(result, equalTo("Hello callable"));
}
```

Now we can execute the `call` method, and see that the result is what we
return.

`Callable` throws an `Exception`
--------------------------------

As we've seen in the definition of the `Callable`, the method `call` can also
throw an exception. To demonstrate this, here's a very simple test. This time,
I create a `Callable` with the generics `Void`, which is used, if there's no
class expected. Instead of returning something, the `Callable` throws an
`Exception` with some message.  
If I call the `Callable` now, an `Exception` is thrown and I can verify the
message of it.

```java
@Test
public void verifyCallableCanThrowAnException() throws Exception {
  Callable<Void> callable = new Callable<Void>() {

    @Override
    public Void call() throws Exception {
      throw new Exception("Error message");
    }
  };

  assertThrows(Exception.class, () -> {
    callable.call();
  });
}
```

That's, what the `Callable` is doing, returning a value. Of course there can be
implementations that are way more complicated, they can be dependent on other
data, they could run a query against a database, load something from the
harddrive or query some data from the internet, but the interface itself is
this simple.

Cleanup
-------

I mentioned previously that `Callable` is annotated with
`@FunctionalInterface`. Like this, the code can be simplified. Instead of the
class definition, we can use a lambda definition. Like this, there's not that
much code around, but you need to understand lambdas and functional interfaces.

```java
@Test
public void verifyCallingCallableReturnsAValue() throws Exception {
  Callable<String> callable = () -> "Hello callable";

  String result = callable.call();

  assertThat(result, equalTo("Hello callable"));
}

@Test
public void verifyCallableCanThrowAnException() throws Exception {
  Callable<Void> callable = () -> {
    throw new Exception("Error message");
  };

  assertThrows(Exception.class, () -> {
    callable.call();
  });
}
```
