2025/07/01  -  11:05
status: 

Tags: [[Design patterns]] [[software engineering]]

The **Prototype pattern** allows copying existing objects without making your code dependent on their classes[11](https://refactoring.guru/design-patterns/prototype). It's particularly useful when object creation is expensive and you have similar objects already existing[12](https://www.digitalocean.com/community/tutorials/prototype-design-pattern-in-java).

## Implementation Example

```java
// Prototype interface
interface Shape extends Cloneable {
    void draw();
    Shape clone();
}

// Concrete prototype
class Rectangle implements Shape {
    private String color;
    private int width, height;
    
    public Rectangle(String color, int width, int height) {
        this.color = color;
        this.width = width;
        this.height = height;
    }
    
    public void draw() {
        System.out.println("Drawing a " + color + " rectangle");
    }
    
    public Shape clone() {
        return new Rectangle(this.color, this.width, this.height);
    }
}

// Prototype registry
class ShapeRegistry {
    private static Map<String, Shape> shapeMap = new HashMap<>();
    
    static {
        shapeMap.put("rectangle", new Rectangle("blue", 10, 20));
        shapeMap.put("circle", new Circle("red", 5));
    }
    
    public static Shape getShape(String type) {
        Shape shape = shapeMap.get(type);
        return shape != null ? shape.clone() : null;
    }
}

```
The pattern is particularly beneficial when you need to **avoid expensive object creation**, **create objects with specific configurations**, or **add/remove products at runtime**[9](http://st.inf.tu-dresden.de/files/teaching/dpf/caisin-CreationalPatterns.pdf).

# References
