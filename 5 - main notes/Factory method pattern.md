2025/07/01  -  10:51
status: 

Tags: [[Design patterns]] [[software engineering]]


The **Factory Method pattern** provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created[5](https://refactoring.guru/design-patterns/factory-method). It centralizes creation of objects of a specific type, choosing one of several implementations[1](https://en.wikipedia.org/wiki/Creational_pattern).

```java
// Product interface
interface Transport {
    void deliver();
}

// Concrete products
class Truck implements Transport {
    public void deliver() {
        System.out.println("Deliver by land in a box");
    }
}

class Ship implements Transport {
    public void deliver() {
        System.out.println("Deliver by sea in a container");
    }
}

// Creator
abstract class Logistics {
    abstract Transport createTransport();
    
    public void planDelivery() {
        Transport transport = createTransport();
        transport.deliver();
    }
}

// Concrete creators
class RoadLogistics extends Logistics {
    Transport createTransport() {
        return new Truck();
    }
}

class SeaLogistics extends Logistics {
    Transport createTransport() {
        return new Ship();
    }
}
```
This pattern is widely used in Java's core libraries, including `Calendar.getInstance()`, `ResourceBundle.getBundle()`, and `NumberFormat.getInstance()`[6](https://refactoring.guru/design-patterns/factory-method/java/example).

## 3. Abstract Factory Pattern

The **Abstract Factory pattern** provides an interface for creating families of related or dependent objects without specifying their concrete classes[2](https://refactoring.guru/design-patterns/creational-patterns). It's essentially a "factory of factories" that encapsulates a group of individual factories with a common theme[7](https://java-design-patterns.com/patterns/abstract-factory/).

## Real-World Example
Consider a furniture manufacturing system[8](https://dev.to/jps27cse/software-design-pattern-abstract-factory-pattern-5f3b):

```java 
// Abstract factory
interface AbstractFurnitureFactory {
    Chair createChair();
    Table createTable();
    Sofa createSofa();
}

// Abstract products
interface Chair {
    void sitOn();
}

interface Table {
    void use();
}

interface Sofa {
    void lieOn();
}

// Concrete products for Modern style
class ModernChair implements Chair {
    public void sitOn() {
        System.out.println("Sitting on a modern chair");
    }
}

class ModernTable implements Table {
    public void use() {
        System.out.println("Using a modern table");
    }
}

// Concrete factory
class ModernFurnitureFactory implements AbstractFurnitureFactory {
    public Chair createChair() {
        return new ModernChair();
    }
    
    public Table createTable() {
        return new ModernTable();
    }
    
    public Sofa createSofa() {
        return new ModernSofa();
    }
}

```
# References
