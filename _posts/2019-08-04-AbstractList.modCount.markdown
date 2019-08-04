---
layout: post
title:  "AbstractList.modCount"
date:   2019-08-04 11:00:00
categories: "Java"
tags: "Java"
---
#### What
Below details is copy from AbstractList.java source code.
``` java
/**
  * The number of times this list has been <i>structurally modified</i>.
  * Structural modifications are those that change the size of the
  * list, or otherwise perturb it in such a fashion that iterations in
  * progress may yield incorrect results.
  *
  * <p>This field is used by the iterator and list iterator implementation
  * returned by the {@code iterator} and {@code listIterator} methods.
  * If the value of this field changes unexpectedly, the iterator (or list
  * iterator) will throw a {@code ConcurrentModificationException} in
  * response to the {@code next}, {@code remove}, {@code previous},
  * {@code set} or {@code add} operations.  This provides
  * <i>fail-fast</i> behavior, rather than non-deterministic behavior in
  * the face of concurrent modification during iteration.
  *
  * <p><b>Use of this field by subclasses is optional.</b> If a subclass
  * wishes to provide fail-fast iterators (and list iterators), then it
  * merely has to increment this field in its {@code add(int, E)} and
  * {@code remove(int)} methods (and any other methods that it overrides
  * that result in structural modifications to the list).  A single call to
  * {@code add(int, E)} or {@code remove(int)} must add no more than
  * one to this field, or the iterators (and list iterators) will throw
  * bogus {@code ConcurrentModificationExceptions}.  If an implementation
  * does not wish to provide fail-fast iterators, this field may be
  * ignored.
  */
 protected transient int modCount = 0;
```
#### Why
This field is used by the iterator and list iterator implementation returned by the iterator and listIterator methods. If the value of this field changes unexpectedly, the iterator (or list iterator) will throw a ConcurrentModificationException in response to the next, remove, previous, set or add operations. This provides fail-fast behavior, rather than non-deterministic behavior in the face of concurrent modification during iteration.

#### How
``` java
/**
    * Returns a list-iterator of the elements in this list (in proper
    * sequence), starting at the specified position in the list.
    * Obeys the general contract of {@code List.listIterator(int)}.<p>
    *
    * The list-iterator is <i>fail-fast</i>: if the list is structurally
    * modified at any time after the Iterator is created, in any way except
    * through the list-iterator's own {@code remove} or {@code add}
    * methods, the list-iterator will throw a
    * {@code ConcurrentModificationException}.  Thus, in the face of
    * concurrent modification, the iterator fails quickly and cleanly, rather
    * than risking arbitrary, non-deterministic behavior at an undetermined
    * time in the future.
    *
    * @param index index of the first element to be returned from the
    *              list-iterator (by a call to {@code next})
    * @return a ListIterator of the elements in this list (in proper
    *         sequence), starting at the specified position in the list
    * @throws IndexOutOfBoundsException {@inheritDoc}
    * @see List#listIterator(int)
    */
   public ListIterator<E> listIterator(int index) {
       checkPositionIndex(index);
       return new ListItr(index);
   }

   private class ListItr implements ListIterator<E> {
       private Node<E> lastReturned;
       private Node<E> next;
       private int nextIndex;
       private int expectedModCount = modCount;

       ......

       final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
   }
```
You will find that they will check the value (modCount != expectedModCount ?).

#### Fail Fast & Fail Safe
> Iterators in java are used to iterate over the Collection objects.

* Fail-Fast iterators immediately throw ConcurrentModificationException if there is structural modification of the collection. Structural modification means adding, removing or updating any element from collection while a thread is iterating over that collection. Iterator on ArrayList, HashMap classes are some examples of fail-fast Iterator.
* Fail-Safe iterators don’t throw any exceptions if a collection is structurally modified while iterating over it. This is because, they operate on the clone of the collection, not on the original collection and that’s why they are called fail-safe iterators. Iterator on CopyOnWriteArrayList, ConcurrentHashMap classes are examples of fail-safe Iterator.
