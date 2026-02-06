# Mutable, Immutable, and Pass-by-Value in Java

Java is strictly **pass-by-value**.
- Primitives: the value is copied
- Objects: the value of the reference is copied

---

## Mutable Object (No Return Needed)

Mutable objects can be changed inside a method, and the change is visible to the caller even without returning. This is due to refernce shared instead of copy of object. 

    class Counter {
        int value;
    }

    static void increment(Counter counter) {
        counter.value++; // mutates same object
    }

    public static void main(String[] args) {
        Counter counter = new Counter();
        counter.value = 1;

        increment(counter);

        System.out.println(counter.value); // 2
    }

---

## Immutable Object (Return Required)

Immutable objects cannot be modified. A new value must be returned to calling function.

    static String addExclamation(String text) {
        return text + "!";
    }

    public static void main(String[] args) {
        String text = "hello";

        text = addExclamation(text);

        System.out.println(text); // hello!
    }

---

## Reference Reassignment Does Not Affect Caller

Reassigning a reference inside a method does not change the caller’s reference.

    static void replaceCounter(Counter counter) {
        counter = new Counter(); // local reference only
        counter.value = 100;
    }

    public static void main(String[] args) {
        Counter counter = new Counter();
        counter.value = 1;

        replaceCounter(counter);

        System.out.println(counter.value); // 1
    }

---

## Chained Sorting Using Comparator

    class Person {
        String name;
        int age;
        // getters
    }

    people.sort(
        Comparator.comparing(Person::getName)
                  .thenComparing(Person::getAge)
    );

---
