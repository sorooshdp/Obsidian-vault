2025/07/01  -  10:46
status: 

Tags: [[Design patterns]] [[software engineering]] 

The **Singleton pattern** ensures that a class has only one instance while providing a global point of access to that instance[2](https://refactoring.guru/design-patterns/creational-patterns). It's one of the most commonly used creational patterns and is particularly valuable when you need exactly one instance of a class throughout the application[3](https://www.initgrep.com/posts/design-patterns/thread-safety-in-java-singleton-pattern).

## Thread-Safe Implementation

A robust thread-safe Singleton implementation uses the **static inner class approach**[4](https://www.baeldung.com/creational-design-patterns):

```java
public class Singleton {
    private Singleton() {}
    
    private static class SingletonHolder {
        public static final Singleton instance = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHolder.instance;
    }
}
```

This approach provides **lazy initialization**, **thread safety**, and avoids synchronization overhead[4](https://www.baeldung.com/creational-design-patterns). The instance is created only when someone calls the `getInstance()` method, not when the outer class is loaded[4](https://www.baeldung.com/creational-design-patterns).
# References
