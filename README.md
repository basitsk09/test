Infosys Java + Spring Boot Interview Questions & Answers


---

✅ Core Java

1. Explain OOP principles with examples.

Encapsulation: Wrapping data and methods together. Example: private fields + public getters/setters.

Inheritance: Child class inherits parent class properties. Example: class Dog extends Animal.

Polymorphism: Same method name behaves differently.

Compile-time (method overloading)

Runtime (method overriding)


Abstraction: Hiding complex implementation. Achieved using interfaces or abstract classes.


2. Difference between HashMap, Hashtable, and ConcurrentHashMap.

HashMap: Not synchronized, allows one null key.

Hashtable: Synchronized (legacy), no null keys.

ConcurrentHashMap: Thread-safe, better concurrency than Hashtable, allows concurrent reads.


3. What are functional interfaces in Java 8?

An interface with only one abstract method. Used in lambda expressions.

@FunctionalInterface
interface MyFunc { void execute(); }

Examples: Runnable, Comparator, Consumer


4. How does equals() and hashCode() work?

equals() compares object contents.

hashCode() returns integer hash. Used in hashing collections (e.g., HashMap).

If equals() returns true, hashCode() must be same.



5. Final, finally, and finalize

final: Keyword for constants, methods, classes.

finally: Code block always executed (try-catch-finally).

`finalize


