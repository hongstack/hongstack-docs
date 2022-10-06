---
sidebar_label: Sealed Classes
---

# Java Sealed Classes
Sealed Classes and Interfaces in Java 15

## Overview
The release of Java SE 15 introduces sealed classes (JEP 360) as a preview feature.

**This feature is about enabling more fine-grained inheritance control in Java. Sealing allows classes and interfaces to define their permitted subtypes**.

In other words, a class or an interface can now define which classes can implement or extend it. It is a useful feature for domain modeling and increasing the security of libraries.

## Motivation
A class hierarchy enables us to reuse code via inheritance. However, the class hierarchy can also have other purposes. Code reuse is great but is not always our primary goal.

### Modeling Possibilities
An alternative purpose of a class hierarchy can be to model various possibilities that exist in a domain.

As an example, imagine a business domain that only works with cars and trucks, not motorcycles. When creating the Vehicle abstract class in Java, we should be able to allow only Car and Truck classes to extend it. In that way, we want to ensure that there will be no misuse of the Vehicle abstract class within our domain.

**In this example, we are more interested in the clarity of code handling known subclasses than defending against all unknown subclasses**.

Before version 15, Java assumed that code reuse is always a goal. Every class was extendable by any number of subclasses.

### The Package-Private Approach
In earlier versions, Java provided limited options in the area of inheritance control.

A [final class][] can have no subclasses. A [package-private class][] can only have subclasses in the same package.

[final class]: https://www.baeldung.com/java-final
[package-private class]: https://www.baeldung.com/java-access-modifiers

Using the package-private approach, users cannot access the abstract class without also allowing them to extend it:

```java
public class Vehicles {
    abstract static class Vehicle {
        private final String registrationNumber;

        public Vehicle(String registrationNumber) {
            this.registrationNumber = registrationNumber;
        }

        public String getRegistrationNumber() {
            return registrationNumber;
        }
    }

    public static final class Car extends Vehicle {
        private final int numberOfSeats;

        public Car(int numberOfSeats, String registrationNumber) {
            super(registrationNumber);
            this.numberOfSeats = numberOfSeats;
        }

        public int getNumberOfSeats() {
            return numberOfSeats;
        }
    }

    public static final class Truck extends Vehicle {
        private final int loadCapacity;

        public Truck(int loadCapacity, String registrationNumber) {
            super(registrationNumber);
            this.loadCapacity = loadCapacity;
        }

        public int getLoadCapacity() {
            return loadCapacity;
        }
    }
}
```

### Superclass Accessible, Not Extensible
A superclass that is developed with a set of its subclasses should be able to document its intended usage, not constrain its subclasses. Also, having restricted subclasses should not limit the accessibility of its superclass.

**Thus, the main motivation behind sealed classes is to have the possibility for a superclass to be widely accessible but not widely extensible**.

## Creation
The sealed feature introduces a couple of new modifiers and clauses in Java: *sealed*, *non-sealed*, and *permits*.

### Sealed Interfaces
To seal an interface, we can apply the sealed modifier to its declaration. The *permits* clause then specifies the classes that are permitted to implement the sealed interface:

```java {1}
public sealed interface Service permits Car, Truck {
    int getMaxServiceIntervalInMonths();

    default int getMaxDistanceBetweenServicesInKilometers() {
        return 100000;
    }
}
```

### Sealed Classes
Similar to interfaces, we can seal classes by applying the same *sealed* modifier. The *permits* clause should be defined after any *extends* or *implements* clauses:

```java
public abstract sealed class Vehicle permits Car, Truck {
    protected final String registrationNumber;

    public Vehicle(String registrationNumber) {
        this.registrationNumber = registrationNumber;
    }

    public String getRegistrationNumber() {
        return registrationNumber;
    }

}
```

A permitted subclass must define a modifier. It may be [declared final][] to prevent any further extensions:

[declared final]: https://www.baeldung.com/java-final

```java
public final class Truck extends Vehicle implements Service {
    private final int loadCapacity;

    public Truck(int loadCapacity, String registrationNumber) {
        super(registrationNumber);
        this.loadCapacity = loadCapacity;
    }

    public int getLoadCapacity() {
        return loadCapacity;
    }

    @Override
    public int getMaxServiceIntervalInMonths() {
        return 18;
    }
}
```
A permitted subclass may also be declared *sealed*. However, if we declare it *non-sealed*, then it is open for extension:

```java
public non-sealed class Car extends Vehicle implements Service {
    private final int numberOfSeats;

    public Car(int numberOfSeats, String registrationNumber) {
        super(registrationNumber);
        this.numberOfSeats = numberOfSeats;
    }

    public int getNumberOfSeats() {
        return numberOfSeats;
    }

    @Override
    public int getMaxServiceIntervalInMonths() {
        return 12;
    }
}
```

## Constraints
A sealed class imposes three important constraints on its permitted subclasses:
1. All permitted subclasses must belong to the same module as the sealed class.
1. Every permitted subclass must explicitly extend the sealed class.
1. Every permitted subclass must define a modifier: *final*, *sealed*, or *non-sealed*.