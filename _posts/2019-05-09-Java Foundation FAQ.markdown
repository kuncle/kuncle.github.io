---
layout: post
title:  "Java Foundation FAQ"
date:   2019-05-09 21:00:00
categories: "Java"
tags: "Java"
---
#### What is the difference between NoClassDefFoundError and ClassNotFoundException?
* NoClassDefFoundError   
  > &nbsp;&nbsp;&nbsp;&nbsp;Thrown if the Java Virtual Machine or a ClassLoader instance tries to load in the definition of a class (as part of a normal method call or as part of creating a new instance using the new expression) and no definition of the class could be found. The searched-for class definition existed when the currently executing class was compiled, but the definition can no longer be found.

* ClassNotFoundException
  > &nbsp;&nbsp;&nbsp;&nbsp;Thrown when an application tries to load in a class through its string name using:   
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**The forName method in class Class.**   
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**The findSystemClass method in class ClassLoader.**   
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**The loadClassmethod in class ClassLoader.**   
    but no definition for the class with the specified name could be found.   
    &nbsp;&nbsp;&nbsp;&nbsp;As of release 1.4, this exception has been retrofitted to conform to
    the general purpose exception-chaining mechanism.  The "optional exception that was raised while loading the class" that may be provided at construction time and accessed via the getException() method is now known as the cause, and may be accessed via the
    Throwable.getCause() method, as well as the aforementioned "legacy method."

#### What is the difference between final, finally and finalize?
* final
 > &nbsp;&nbsp;&nbsp;&nbsp;final is a reserved keyword in java. We can’t use it as an identifier as it is reserved. We can use this keyword with variables, methods and also with classes. The final keyword in java has different meaning depending upon it is applied to variable, class or method.   
 + final with Variables   
  > &nbsp;&nbsp;&nbsp;&nbsp;The value of variable cannot be changed once initialized. If we declare any variable as final, we can’t modify its contents since it is final, and if we modify it then we get Compile Time Error.
 + final with Class   
  > &nbsp;&nbsp;&nbsp;&nbsp;The class cannot be subclassed. Whenever we declare any class as final, it means that we can’t extend that class or that class can’t be extended or we can’t make subclass of that class.
 + final with Method   
  > &nbsp;&nbsp;&nbsp;&nbsp;The method cannot be overridden by a subclass. Whenever we declare any method as final, then it means that we can’t override that method.

* finally   
  > &nbsp;&nbsp;&nbsp;&nbsp;The finally keyword is used in association with a try/catch block and guarantees that a section of code will be executed, even if an exception is thrown. The finally block will be executed after the try and catch blocks, but before control transfers back to its origin.

* finalize   
  > &nbsp;&nbsp;&nbsp;&nbsp;It is a method that the Garbage Collector always calls just before the deletion/destroying the object which is eligible for Garbage Collection, so as to perform clean-up activity. Clean-up activity means closing the resources associated with that object like Database Connection, Network Connection or we can say resource de-allocation. Remember it is not a reserved keyword.   
  &nbsp;&nbsp;&nbsp;&nbsp;Once finalize method completes immediately Garbage Collector destroy that object. finalize method is present in Object class and its syntax is:   
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**protected void finalize throws Throwable{} **    
  &nbsp;&nbsp;&nbsp;&nbsp;Since Object class contains finalize method hence finalize method is available for every java class since Object is superclass of all java classes. Since it is available for every java class hence Garbage Collector can call finalize method on **any java object**
  Now, the finalize method which is present in Object class, has empty implementation, in our class clean-up activities are there, then we have to override this method to define our own clean-up activities.
