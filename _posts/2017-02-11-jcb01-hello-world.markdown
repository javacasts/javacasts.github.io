---
layout:     post
title:      "Java Basics - HelloWorld"
date:       2017-02-11 16:10:00 +0200
modified:   2017-02-11 16:10:00 +0200
categories: basics
keywords:   java, basics, HelloWorld
video:      r8U05bJIDQo
abstract:   "What do you need to start programming java? Only the Java Development Kit and an text-editor, and you can start"
author:     "Rainer Jung"
---
Java Basics - First Java Application
====================================

> Java is a general-purpose computer programming language that is concurrent,
> class-based, object-oriented, and specifically designed to have as few
> implementation dependencies as possible.
> - [Wikipedia](https://en.wikipedia.org/wiki/Java_%28programming_language%29)

Setup
-----

So what do we need to program anything in Java?

 - Java Development Kit
 - Editor / IDE

How to install this depends on the system you are using. You can get the JDK
from [oracle](http://www.oracle.com/technetwork/java/javase/downloads/index.html).

Once installed, you should be able to call `java -version` and `javac -version`,
which would print out the version you've installed.

![`java -version`]({{ site.url }}/assets/jcb01-java-version.png)

Then you need an editor. Source files are plain text-files, so you could use the
standard notepad in windows, but there are Integrated Development Environments
(IDE) available that help you write your code. Usually you would choose from
[Eclipse](https://eclipse.org/), [NetBeans](https://netbeans.org/), and
[IntelliJ IDEA](https://www.jetbrains.com/idea/), but you also can use
[Visual Studio Code](https://code.visualstudio.com/),
[Sublime](https://www.sublimetext.com/),
[emacs](https://www.gnu.org/software/emacs/), [vim](http://vim.org/) or any
other editor that can edit text-files.

Hello world
-----------

The first program in every language is usually a `hello world`-application. It
just prints out the message `hello world`. So, let's start wil the simpliest
`hello world` in java. Create a file with the name `HelloWorld.java`, and put
the following contents in there (using any editor).

``` java
class HelloWorld {

  public static void main(String[] args) {
    System.out.println("Hello world");
  }

}
```

So this is the source-code for the application. This first needs to be compiled
and then it can be executed.  
Java does not compile a standalone program, that would run on any machine, Java
uses a runtime that interprets the compiled Program. So here's what you need to
call:

``` sh
javac HelloWorld.java
java HelloWorld
```

This is how the outcome should look like:

![`hello world`]({{ site.url }}/assets/jcb01-hello-world.png)

So what has really happened here?  
We compiled a java source file (`HelloWorld.java`) with the java compiler
(`javac`). The java compiler created a java class file (`HelloWorld.class`) that
contains the compiled sources. Now java can run this compiled class file.

The Source-Code
---------------

So, what did the source-code of the `HelloWorld` contain?  
The first line says something about a class (`class HelloWorld`). This does tell
the java compiler that now a `class` definition starts. The curly bracket
defines the beginning of the class, and the last curly brackets define the end
of the class. Brackets always need to be opened and closed in pairs. The java
compiler would complain, if the opening and closing brackets do not match.

*What is a class?*

Java is an object-oriented language. The java language uses the concept of
objects. These objects are representation of entities. An entity can be a fruit,
a vehicle or just simply a list of characters or a number. To distinguish
between different objects, there are classes that define how objects behave.
In this example we define the class of HelloWorld. The class of HelloWorld
defines an application that can write `hello world` to the console.

Inside the class definition, there's now another context with curly braches. It
starts with `public static void main(String[] args)`, then comes a block again,
as for the `HelloWorld` class. This declaration is called a *method*.

*What is a method?*

Every code that is executed (mostly) within methods. Methods are some kind of
funktionality that can be called. If java tries to run a class, it is defined
that it tries to call a static method with the name `main`.  
Nut besides `main`, there are several other things in that line, they all have
a meaning. let's start with `public`.

*What does public mean?*

Every method that is defined with `public` can be called by anyone else. It's
like a button on a traffic light. anybody can press that button.

*What does static mean?*

`static` is means that the method can be called directly from the class. Don't
bother too much about this, once we get deeper into objects, you will understand
this.

*What does void mean?*

Every method must return something. It's like a result. Usually a method can
return a value. Like if you calculate the sum of two values, the method would
return the sum of those values. As this method does not return anything, java
uses the token `void` to identify that nothing is returned by the method.

*What's the `String[] args` for?*

If java runs a class like we did (`java Helloworld`), you can provide further
arguments in the commandline. Those will be available in a array of `String`s.
`String` is the class of a list of characters and `[]` indicates that the
variable `args` is a array of `String`s.

*What is a property?*

Another thing is a property. When we take a look at the command, we see this
part: `System.out`. System is a class that has a property with the name `out`.
We just need this to understand what actually is happening.

*How is the message written to the console?*

Within the `main`-method, there's the command
`System.out.println("Hello World");`. First of all, every command always ends
with a semicolon. There can be as many whitespaces including newlines as wanted.
The semicolon marks the end of each command. So here, the command is
`System.out.println("Hello World")`. `System` is a class that is defined by
java. it contains many possibillities to communicate with the system.
`System.out` for example asks for a `out` property of the `System` class. This
is defined by java, that this property exists. The property contains an Object
that can be used to write something to the console. The class of this Object is
`PrintStream`m and provides the method `println`. This method then takes one
argument (`"Hello World"`) and writes it to the console.
