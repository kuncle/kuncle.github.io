---
layout: post
title:  "Java Foundation FAQ"
date:   2019-05-09 21:00:00
categories: "Java"
tags: "Java"
---
#### What is the difference between NoClassDefFoundError and ClassNotFoundException?
* NoClassDefFoundError   
  > Thrown if the Java Virtual Machine or a ClassLoader instance tries to load in the   definition of a class (as part of a normal method call or as part of creating a new instance using the new expression) and no definition of the class could be found. The searched-for class definition existed when the currently executing class was compiled, but the definition can no longer be found.

* ClassNotFoundException
  > Thrown when an application tries to load in a class through its string name using:   
    &nbsp;&nbsp;&nbsp;&nbsp;The forName method in class Class.   
    &nbsp;&nbsp;&nbsp;&nbsp;The findSystemClass method in class ClassLoader.   
    &nbsp;&nbsp;&nbsp;&nbsp;The loadClassmethod in class ClassLoader.   
    but no definition for the class with the specified name could be found.   
    As of release 1.4, this exception has been retrofitted to conform to
    the general purpose exception-chaining mechanism.  The "optional exception that was raised while loading the class" that may be provided at construction time and accessed via the getException() method is now known as the cause, and may be accessed via the
    Throwable.getCause() method, as well as the aforementioned "legacy method."

#### What is the difference between final, finally and finalize?
