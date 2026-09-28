2026/09/15  -  13:59
status: 

Tags: [[js]]  

### Overview & Core Theme

Chapter 4 explores **Class Orientation** as an optional software design pattern and contrasts classical class concepts with JavaScript’s actual object mechanics. While languages like Java or C++ naturally implement class hierarchies through copies, JavaScript **does not natively have classes**. Instead, JavaScript only provides objects that are linked together. To overcome this gap, developers often use **`mixins`** to simulate class copying and inheritance, which can lead to complex and brittle code.

---

### 1. Class Theory & Design Patterns

#### **Data Structures and Encapsulation**

Class-oriented programming centers on packaging data together with the behaviors that operate on it—a concept known as **encapsulation**. For instance, a string packages character data with methods to search, append, or calculate its length.

#### **The Building / Blueprint Metaphor**

- **Class (Blueprint):** An architect’s blueprint defines the structure and characteristics of a building, but it is not a physical building itself.
- **Instance (Building):** To interact with a building (e.g., opening a door), it must be constructed out of thin air from the blueprint.
- **Instantiation as Copying:** Constructing an instance copies all characteristics from the class definition into a concrete object.

```js
// Pseudo-code illustrating Class & Constructor instantiation
class CoolGuy {
    specialTrick = nothing
    CoolGuy( trick ) {
        specialTrick = trick
    }
    showOff() {
        output( "Here's my trick: ", specialTrick )
    }
}

Joe = new CoolGuy( "jumping rope" )
Joe.showOff() // Here's my trick: jumping rope
```


#### **Class Inheritance & Polymorphism**

Child classes can specialize parent classes by inheriting base functionality and overriding methods. **Polymorphism** describes how a child method can override a parent method while still relatively referencing the base behavior.

```js
// Pseudo-code illustrating Class Inheritance and Polymorphism
class Vehicle {
    engines = 1
    ignition() {
        output( "Turning on my engine." )
    }
    drive() {
        ignition()
        output( "Steering and moving forward!" )
    }
}

class Car inherits Vehicle {
    wheels = 4
    drive() {
        inherited:drive() // Relative polymorphic call
        output( "Rolling on all ", wheels, " wheels!" )
    }
}

class SpeedBoat inherits Vehicle {
    engines = 2
    ignition() {
        output( "Turning on my ", engines, " engines." )
    }
    pilot() {
        inherited:drive()
        output( "Speeding through the water with ease!" )
    }
}
```


---

### 2. Does JavaScript Have Classes?

Plain and simple: **No**.

Although JavaScript features class-like keywords such as `new`, `instanceof`, and ES6's `class` syntax, these are merely syntactic sugar. In traditional class languages, instantiation and inheritance mean **copying** behaviors from parent to child and from class to instance. In JavaScript, no automatic copy operations occur; objects are simply **linked together**.

#### **Multiple Inheritance & The Diamond Problem**

Some languages allow a child class to inherit from multiple parents. However, this leads to complications like the **Diamond Problem** (where a child inherits conflicting overridden methods from two parent classes that share a common ancestor). JavaScript does **not** provide a native mechanism for multiple inheritance.

---

### 3. Mixins: Faking Class Copy Behavior in JS

Because JavaScript does not automatically copy properties during "inheritance," developers manually simulate copy operations using **mixins**.

#### **1. Explicit Mixins**

An explicit mixin utility iterates over the properties of a source object and manually copies them onto a target object.

```js
// Vastly simplified mixin() utility
function mixin( sourceObj, targetObj ) {
    for (var key in sourceObj) {
        // Only copy if not already present
        if (!(key in targetObj)) {
            targetObj[key] = sourceObj[key];
        }
    }
    return targetObj;
}

var Vehicle = {
    engines: 1,
    ignition: function() {
        console.log( "Turning on my engine." );
    },
    drive: function() {
        this.ignition();
        console.log( "Steering and moving forward!" );
    }
};

var Car = mixin( Vehicle, {
    wheels: 4,
    drive: function() {
        // Explicit pseudo-polymorphism
        Vehicle.drive.call( this );
        console.log( "Rolling on all " + this.wheels + " wheels!" );
    }
} );
```

##### Key Takeaways on Explicit Mixins:

- **Explicit Pseudo-Polymorphism (`Vehicle.drive.call( this )`):** Because JavaScript lacks relative polymorphism support prior to ES6, methods with identical names must be explicitly called by name bound to the child’s `this` context. This creates manual, brittle bindings throughout your code.
- **Mixins Copy References, Not Functions:** Functions in JavaScript cannot be duplicated in a standard way. A mixin copies a _reference_ to a shared function object. If that function object is modified, both the source and target objects are affected.

---

#### **2. Parasitic Inheritance**

Popularized by Douglas Crockford, parasitic inheritance is a pattern that combines explicit mixin techniques with constructor functions. It instantiates a parent object internally, augments it with child properties, and returns that customized instance.

```js
// "Traditional JS Class"
function Vehicle() {
    this.engines = 1;
}
Vehicle.prototype.ignition = function() {
    console.log( "Turning on my engine." );
};
Vehicle.prototype.drive = function() {
    this.ignition();
    console.log( "Steering and moving forward!" );
};

// "Parasitic Class"
function Car() {
    // First, `car` is a `Vehicle` instance
    var car = new Vehicle();

    // Specialize `car`
    car.wheels = 4;

    // Save a privileged reference to Vehicle's drive()
    var vehDrive = car.drive;

    // Override Vehicle::drive()
    car.drive = function() {
        vehDrive.call( this );
        console.log( "Rolling on all " + this.wheels + " wheels!" );
    };

    return car;
}

var myCar = new Car();
myCar.drive();
// Output:
// Turning on my engine.
// Steering and moving forward!
// Rolling on all 4 wheels!
```

---

#### **3. Implicit Mixins**

Implicit mixins involve borrowing functions at runtime using `.call( this )` inside a method.

```js
var Something = {
    cool: function() {
        this.greeting = "Hello World";
        this.count = this.count ? this.count + 1 : 1;
    }
};

Something.cool();
Something.greeting; // "Hello World"
Something.count;    // 1

var Another = {
    cool: function() {
        // Implicit mixin borrowing Something.cool
        Something.cool.call( this );
    }
};

Another.cool();
Another.greeting; // "Hello World"
Another.count;    // 1 (state is not shared with `Something`)
```

---

### Summary

1. **Classes Mean Copies:** Real class orientation requires parent-to-child and class-to-instance copying.
2. **JavaScript Only Links Objects:** JavaScript has no classes—only objects linked via prototype linkages.
3. **Mixins Are Imperfect:** Explicit and implicit mixins attempt to emulate class copies, but they lead to fragile syntax (explicit pseudo-polymorphism) and only copy function references rather than duplicating objects.
4. **Faking Classes Causes Friction:** Attempting to force class design patterns onto JavaScript creates landmines and unnecessary complexity.

---

# References
