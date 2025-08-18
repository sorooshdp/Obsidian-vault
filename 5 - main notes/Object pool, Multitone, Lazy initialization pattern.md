2025/07/01  -  11:13
status: 

Tags: [[Design patterns]] [[software engineering]]


The **Object Pool pattern** uses a set of initialized objects kept ready for use rather than allocating and destroying them on demand[13](https://en.wikipedia.org/wiki/Object_pool_pattern). It's particularly effective when object creation is expensive and objects can be reused[14](https://designpatternsphp.readthedocs.io/en/latest/Creational/Pool/README.html).
## Key Benefits and Use Cases
- **Performance improvement** for expensive object creation[13](https://en.wikipedia.org/wiki/Object_pool_pattern)
- **Resource management** for limited resources like database connections[15](https://www.oodesign.com/object-pool-pattern)
- **Memory optimization** by reusing existing objects

```java
public class WorkerPool {
    private List<StringReverseWorker> freeWorkers = new ArrayList<>();
    private List<StringReverseWorker> occupiedWorkers = new ArrayList<>();
    
    public StringReverseWorker get() {
        if (freeWorkers.isEmpty()) {
            return new StringReverseWorker();
        }
        
        StringReverseWorker worker = freeWorkers.remove(freeWorkers.size() - 1);
        occupiedWorkers.add(worker);
        return worker;
    }
    
    public void dispose(StringReverseWorker worker) {
        occupiedWorkers.remove(worker);
        freeWorkers.add(worker);
    }
}

```

## Multiton Pattern

The **Multiton pattern** generalizes the Singleton pattern by allowing controlled creation of multiple instances, managed through a map with unique keys[16](https://en.wikipedia.org/wiki/Multiton_pattern). Each key corresponds to exactly one instance[17](https://java-design-patterns.com/patterns/multiton/).

```java
public class Multiton {
    private static final Map<String, Multiton> instances = new HashMap<>();
    private String key;
    
    private Multiton(String key) {
        this.key = key;
    }
    
    public static synchronized Multiton getInstance(String key) {
        if (!instances.containsKey(key)) {
            instances.put(key, new Multiton(key));
        }
        return instances.get(key);
    }
}

```
## 8. Lazy Initialization

**Lazy initialization** delays object creation until the first time it's needed[18](https://en.wikipedia.org/wiki/Lazy_initialization). This technique improves startup performance and reduces memory usage when objects might not be used[19](https://learn.microsoft.com/en-us/dotnet/framework/performance/lazy-initialization).
```java
public class LazyInitialization {
    private ExpensiveObject expensiveObject;
    
    public ExpensiveObject getExpensiveObject() {
        if (expensiveObject == null) {
            expensiveObject = new ExpensiveObject();
        }
        return expensiveObject;
    }
}

```
# References
