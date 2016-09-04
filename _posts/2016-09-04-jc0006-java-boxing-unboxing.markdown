---
layout:     post
title:      "Autoboxing - Unboxing"
date:       2016-09-09 18:36:00 +0200
modified:   2016-09-09 18:36:00 +0200
categories: java
keywords:   java, boxing, unboxing, NullpointerException
video:      A0rkiy3gLeQ
abstract:   "Here's a funny issue that I ran over a while ago. We had some code and experienced a NullPointerException, but there could not be a NullPointerException, all involved Objects where tested to not be null. A mystery!"
author:     "Rainer Jung"
---
Autoboxing - Unboxing
=====================

Here's a funny issue that I ran over a while ago.  
We had some code and experienced a `NullPointerException`, but there could not
be a `NullPointerException`, all involved Objects where tested to not be
`null`. See for yourself, here's some kind of similar code:

``` java
@RequestMapping("/document/{id}")
public void showPublicDocument(@PathVariable Long id) {
    Document document = documents.find(id);
    if (document != null && document.isPublic()) { // <- in this row
        return document;
    } else {
        throw new DocumentNotAccessibleException("document is not public");
    }
}
```

So this could be a method in a Spring Rest-Controller. The
`DocumentNotAccessibleException` has the annotation to return the status `404`.
There's a `DocumentService` in the variable `documents`. The bad thing is that
there can only be a `NullPointerException` if `documents` is `null`, right?
There's no other call on a object that can be `null`, but it's checked before
that also the `document` is not `null`, so where does this exception come from?

If you know about this topic and have checked with the subject of this JavaCast,
you'll know that this is a topic on autoboxing or unboxing. Autoboxing/unboxing
was
[introduced with java 1.5](http://docs.oracle.com/javase/1.5.0/docs/relnotes/features.html#boxing).
It's about primitive and their object wrapper types.

So here's an example that you can also find on
[github](https://github.com/javacasts/jc0006-autoboxing-unboxing).

``` java
Document document = documents.find(123l);
if (document != null && document.isPublic()) {
    LOGGER.info("Found document: " + document.getName());
} else {
    LOGGER.info("Could not find document");
}
```

The method `isPublic()` on the `Document`-class does return a `Boolean`. This is
not a `boolean` (the primitive type). If this code is compiled, the
java-compiler "unboxes" the Boolean class to a primitive `boolean`.  
You can use a java-decompiler to see, what actually was created when this code
was compiled:

``` java
Document document = this.documents.find(Long.valueOf(123L));
if (document != null && document.isPublic().booleanValue()) {
// ...
```

Now you see what has happened. The method `isPublic()` returned a `null`-value.
On this value the method `booleanValue()` is called and this caused the
`NullPointerException`. But you don't only see the unboxing of the boolean
value, the `long`-value that we used as the `id` of the document is actually
a `Long`. And this is also converted with `Long.valueOf(123L)`.

So here's the list of primitive types and their corresponding wrapper-class:

| Primitive type | Wrapper class |
|----------------|---------------|
| boolean        | Boolean       |
| byte           | Byte          |
| char           | Character     |
| float          | Float         |
| int            | Integer       |
| long           | Long          |
| short          | Short         |
| double         | Double        |
