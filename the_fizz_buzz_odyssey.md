# The FizzBuzz Odyssey: Exploring Strategy Design Pattern and Modern Java Features

FizzBuzz is often considered a simple exercise. However, it can be a powerful tool for demonstrating various coding techniques and design patterns. In this article, we'll dive deep into an implementation that goes far beyond the basic solution, showcasing advanced Java features and object-oriented design principles.

## The Classic Problem
For those unfamiliar, FizzBuzz is a programming task where you print numbers from 1 to 100, but:
- For multiples of 3, print "Fizz" instead of the number
- For multiples of 5, print "Buzz" instead of the number
- For numbers which are multiples of both 3 and 5, print "FizzBuzz"

## Beyond the Basics: Three Approaches

Our implementation presents three distinct approaches to solving FizzBuzz:
- Traditional if-else statements: (Siren's Call Of Simplicity)
- Modern switch expressions (Java 21): (Switch Case Quest)
- Object-oriented design with the Strategy pattern: (Strategy Epihany)

Let's break down each approach and discuss its merits.

### The Classical If-Else: A Good start
We'll begin with the most straightforward way to solve FizzBuzz using if-else statements. It's easy to read and understand, making it an excellent choice for beginners or when simplicity is key.

```java
private void usingIfElse(final int i) {
    if (i % 5 == 0 && i % 3 == 0){
        System.out.println("FizzBuzz");
    } else if (i % 3 == 0) {
        System.out.println("Fizz");
    } else if (i % 5 == 0) {
        System.out.println("Buzz");
    } else {
        System.out.println(i);
    }
}
```

### The Switch Case Quest: Utilizing Java Modern Switch Expressions

This approach leverages Java's modern switch expressions, introduced in Java 14 and refined in subsequent versions. It demonstrates a more concise syntax and pattern matching capabilities, showcasing how language evolution can lead to cleaner code.

```java
private void usingSwitchCase(final Integer i) {
        switch (i) {
            case Integer j when j % 5 == 0 && j % 3 == 0 -> System.out.println("FizzBuzz");
            case Integer j when j % 3 == 0 -> System.out.println("Fizz");
            case Integer j when j % 5 == 0 -> System.out.println("Buzz");
            default -> System.out.println(i);
        }
    }
```

This approach includes two new features introduced Java 14+:
- Pattern Matching: case Integer j checks if i can be cast to an Integer.
- Guards: The when clause adds an additional condition to be met for that particular case to execute.

### The Strategy Epiphany: Implementing the Strategy Design Pattern

This implementation is the most sophisticated, employing the Strategy design pattern. It demonstrates:

- Interface-based design with `FizzBuzzStrategy`
- Composition over inheritance
- Use of Java streams and functional programming concepts
- Sorting strategies based on priority

#### Deep Dive: The Strategy Pattern Implementation
The Strategy pattern allows us to define a family of algorithms, encapsulate each one, and make them interchangeable. Let's break down its components:

##### The Strategy Interface

This interface defines the contract for our strategies, including a priority method for sorting.

```java
interface FizzBuzzStrategy extends Comparable<FizzBuzzStrategy> {
    boolean appliesTo(Integer i);
    String output();
    int priority();

    default int compareTo(FizzBuzzStrategy o) {
        return Integer.compare(this.priority(), o.priority());
    }
}
```

##### Concrete Strategies (Implementation)

We have three concrete implementations: FizzBuzz, Fizz, and Buzz. Each defines its own logic for when it applies and what output to produce.
1. `FizzBuzz` implementation
```java
class FizzBuzz implements FizzBuzzStrategy {
        @Override
        public boolean appliesTo(final Integer i) {
            return i % 5 == 0 && i % 3 == 0;
        }

        @Override
        public String output() {
            return "FizzBuzz";
        }

        @Override
        public int priority() {
            return 1;
        }
    }
```
2. `Fizz` Implementation
```java
class Fizz implements FizzBuzzStrategy {
        @Override
        public boolean appliesTo(final Integer i) {
            return i % 3 == 0;
        }

        @Override
        public String output() {
            return "Fizz";
        }

        @Override
        public int priority() {
            return 2;
        }
    }
```
3. `Buzz` Implementation 
```java
class Buzz implements FizzBuzzStrategy {
        @Override
        public boolean appliesTo(final Integer i) {
            return i % 5 == 0;
        }

        @Override
        public String output() {
            return "Buzz";
        }

        @Override
        public int priority() {
            return 3;
        }
    }
```

##### Strategy Selection
This creates a sorted set of strategies and applies them in order of `priority`.

```java
private void usingStrategyOOP(final Integer i) {
        Set<FizzBuzzStrategy> strategies = Stream.of(
                new Fizz(),
                new FizzBuzz(),
                new Buzz()
        ).collect(Collectors.toCollection(TreeSet::new));
        for (FizzBuzzStrategy strategy : strategies) {
            if (strategy.appliesTo(i)) {
                System.out.println(strategy.output());
                return;
            }
        }

        System.out.println(i);
    }
```

The strategy approach allows us to add or remove rules easily making it a great solution for complex problems.

## What we can learn.

By compariing this 3 approaches, we can learn about the importance of:

- Keeping our code simple and easy to understand
- Using the right tools for the job
- Making our code flexible and adaptable

The FizzBuzz problem may seem simple, but it can teach us valuable lessons about programming and problem-solving. By exploring different approaches, we can become better coders and develop a deeper understanding of software development principles.

## Conclusion 

Each approach to solving the FizzBuzz problem has its own merits. The classic if-else method is straightforward and easy to understand, the modern switch expressions offer a cleaner and more concise syntax, and the Strategy pattern provides a flexible and extensible solution. By understanding and utilizing these different techniques, developers can improve their coding skills and write more robust and maintainable code.

You can find the [Link to code](https://github.com/ifeanyichukwuOtiwa-sports/the-fizz-buzz-odyssey)