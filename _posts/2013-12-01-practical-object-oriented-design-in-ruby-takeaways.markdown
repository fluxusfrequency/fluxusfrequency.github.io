---
title: Practical Object-Oriented Design in Ruby - Takeaways
date: 2013-12-01 06:10:00 Z
categories:
- source
layout: post
---

{% img right http://s5.postimg.org/yh888oh3b/POODR.jpg 250 327 'Practical Object-Oriented Design in Ruby' 'The cover of a programming book' %}

I just finished reading [Practical Object-Oriented Design in Ruby](http://www.poodr.com/) by Sandi Metz.

Here were my takeaways.

## Object-Oriented Design

- Plan your application structure around the messages, rather than the objects.
- Objects should be seen as black boxes that can receive and send specific messages.
- Use [UML sequence diagrams](http://www.tracemodeler.com/articles/a_quick_introduction_to_uml_sequence_diagrams/) to help clarify architectural decisions before writing code.
- Defer design decisions as long as you can, waiting for more information. Having three concrete examples of an object before commiting to a solution is best.



### Embracing Change
- Write code that is: transparent, reasonable, usable, exemplary (TRUE).
- Write classes and methods that have only a single responsibility (Single Reponsibility Principle).
- Subtypes should be substitutable for their supertypes (Liskov Substitution Principle).



### Dependencies
- Dependencies are best managed through Injection, Isolation, and Reliance on Abstractions.
- Dependency Injection: when you have a dependency, inject it into your object as an attribute, without calling another class directly. Then your object can use any other (duck-typed) object that responds to the needed method call.
- Isolation: if you must keep a dependency, isolate it from other code to make it easy to spot and refactor.
- Reliance on Abstractions: objects should depend on things that change less than they do (abstractions).



### Interfaces
- Minimize an object's need for context to get its job done.
- Write public interfaces that are explicitly identified, are about *what* more than *how*, have names that will not change, and take initialization attributes in a hash (so that order and presence are not required).
- Law of Demeter: don’t reach through a class to get at another class’s data or methods.



### Duck-Types
- Sometimes an object expects many different subjects to all respond to the same method. Even if they are of different classes, they all respond. These are duck-types.



### Three Approaches to OO Architecture

1. Use inheritance for _is-a_ relationships. This means using classical inheritance and module mix-ins. With this approach, big changes can be made with little code, which is a double-edged sword. “Inheritance is specialization”. - Bertrand Meyer

2. Use duck-types for _behaves-like-a_ relationships. Use when there is a *role* that needs to be filled (e.g. *schedulable*).

3. Use composition for _has-a_ relationships. Composition is modular and flexible, but harder to comprehend all at once. “Use composition when the behavior is more than the sum of its parts”. - Grady Booch


### Testing

- Don’t write tightly coupled tests.
- Test the right thing only once, in the right place.
- Concentrate on incoming and outgoing messages that cross an object’s boundaries. That is, test its most stable part: its interface.
- Incoming messages should be tested for the state they return. Use *object doubles* for this, when appropriate.
- Outgoing command messages should be tested to ensure they are sent. Use *mocks* for this.
- Outgoing query essays should not be tested.
- Testing duck-types: write a module and mix it into the test for each of the duck-types.

## Conclusion

I really liked this book. It clarified some of the little parts of writing code that had previously eluded me. I can't wait to put some of its principles into practice!
