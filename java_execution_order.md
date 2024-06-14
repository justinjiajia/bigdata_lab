

```java
public class Test { 
    static {
        System.out.println("static block executed");
    }
 
    {
        System.out.println("block executed");
    }
 
    public Test() {
        System.out.println("constructor executed");
    }
 
    public void fun() {
        System.out.println("fun executed");
    }
 
    public static void main(String args[ ])  {
        System.out.println("main started");
        new Test().fun();
    } 
} 
```

OUTPUT:

> static block executed
> 
> main started
> 
> block executed
>
> constructor executed
> 
> fun executed
>
> 
##  Initialization of Classes and Interfaces

https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html#jls-12.4

Initialization of a class consists of executing its static initializers and the initializers for static fields (class variables) declared in the class.

Initialization of an interface consists of executing the initializers for fields (constants) declared in the interface.

###  When Initialization Occurs in Java

A class or interface type T will be initialized immediately before the first occurrence of any one of the following:

- T is a class and an instance of T is created.

- A static method declared by T is invoked.

- A static field declared by T is assigned.

- A static field declared by T is used and the field is not a constant variable (§4.12.4).

- T is a top level class (§7.6) and an assert statement (§14.10) lexically nested within T (§8.1.3) is executed.

When a class is initialized, its superclasses are initialized (if they have not been previously initialized), as well as any superinterfaces (§8.1.5) that declare any default methods (§9.4.3) (if they have not been previously initialized). 
Initialization of an interface does not, of itself, cause initialization of any of its superinterfaces.


## Experiments


```shell
 % java --version
java 17.0.11 2024-04-16 LTS
Java(TM) SE Runtime Environment (build 17.0.11+7-LTS-207)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.11+7-LTS-207, mixed mode, sharing)
```

### Observations

- When multiple class definition exists in the same file, only the 1st class is loaded.

- If the loaded class is a derived one, all its superclasses are loaded first according to hierarchical order.

- For loaded classes, Java runs their static definitions and blocks sequentially.

- All other classes are ignored.

- Finally, Java invokes `main()` (must be declared with the `static` keyword).




#####  public class LoadTest -> class Parent  -> class Grandparent  -> class Child -> class ClassNeverUsed

```java
public class LoadTest {

    // Static init block
    static {
        System.out.println("static - loadtest");
    }

    public static void main(String[] args) {
        System.out.println("START");
        new Child();
        System.out.println("END");
    }
}


class Child extends Parent {
    // Constructor
    public Child() {
        System.out.println("constructor - child");
    }

    // Static init block
    static {
        System.out.println("static - child");
    }

    // Instance init block
    {
        System.out.println("instance - child");
    }
}

class Grandparent {
    // Static init block
    static {
        System.out.println("static - grandparent");
    }

    // Instance init block
    {
        System.out.println("instance - grandparent");
    }

    // Constructor
    public Grandparent() {
        System.out.println("constructor - grandparent");
    }
}

class Parent extends Grandparent {
    // Instance init block
    {
        System.out.println("instance - parent");
    }

    // Constructor
    public Parent() {
        System.out.println("constructor - parent");
    }

    // Static init block
    static {
        System.out.println("static - parent");
    }
}

class ClassNeverUsed {
    // Constructor
    public ClassNeverUsed() {
        System.out.println("constructor - NeverUsed");
    }

    // Instance init block
    {
        System.out.println("instance - NeverUsed");
    }

    // Static init block
    static {
        System.out.println("static - NeverUsed");
    }
}
```

OUTPUT:

> static - loadtest
> 
> START
> 
> static - grandparent
> 
> static - parent
> 
> static - child
> 
> instance - grandparent
> 
> constructor - grandparent
> 
> instance - parent
> 
> constructor - parent
> 
> instance - child
> 
> constructor - child
> 
> END


#####  public class LoadTest (empty) -> class Child  -> class Grandparent  -> class Parent 

```java
public class LoadTest {

}

class Child extends Parent {
    // Constructor
    public Child() {
        System.out.println("constructor - child");
    }

    // Static init block
    static {
        System.out.println("static - child");
    }

    // Instance init block
    {
        System.out.println("instance - child");
    }
}

class Grandparent {
    // Static init block
    static {
        System.out.println("static - grandparent");
    }

    // Instance init block
    {
        System.out.println("instance - grandparent");
    }

    // Constructor
    public Grandparent() {
        System.out.println("constructor - grandparent");
    }
}

class Parent extends Grandparent {
    // Instance init block
    {
        System.out.println("instance - parent");
    }

    // Constructor
    public Parent() {
        System.out.println("constructor - parent");
    }

    // Static init block
    static {
        System.out.println("static - parent");
    }
}
```

OUTPUT:

> error: can't find main(String[]) method in class: LoadTest


#####  class Child (with static `main()` but without static block) -> public class LoadTest -> class Grandparent  -> class Parent 

```java
class Child extends Parent {

    public static void main(String[] args){
        System.out.println("static - child");
    }

}

public class LoadTest {

    // Static init block
    static {
        System.out.println("static - loadtest");
    }

    public static void main(String[] args) {
        System.out.println("START");
        new Child();
        System.out.println("END");
    }
}

class Grandparent {
    // Static init block
    static {
        System.out.println("static - grandparent");
    }

    // Instance init block
    {
        System.out.println("instance - grandparent");
    }

    // Constructor
    public Grandparent() {
        System.out.println("constructor - grandparent");
    }
}

class Parent extends Grandparent {
    // Instance init block
    {
        System.out.println("instance - parent");
    }

    // Constructor
    public Parent() {
        System.out.println("constructor - parent");
    }

    // Static init block
    static {
        System.out.println("static - parent");
    }
}
```

OUTPUT:

> static - grandparent
>
> static - parent
>
> static - child


#####  class Child (with a static method but not `main()`) -> public class LoadTest -> class Grandparent  -> class Parent 

```java

class Child extends Parent {

    public static void test(String[] args){
        System.out.println("static - child");
    }

}

public class LoadTest {

    // Static init block
    static {
        System.out.println("static - loadtest");
    }

    public static void main(String[] args) {
        System.out.println("START");
        new Child();
        System.out.println("END");
    }
}

class Grandparent {
    // Static init block
    static {
        System.out.println("static - grandparent");
    }

    // Instance init block
    {
        System.out.println("instance - grandparent");
    }

    // Constructor
    public Grandparent() {
        System.out.println("constructor - grandparent");
    }
}

class Parent extends Grandparent {
    // Instance init block
    {
        System.out.println("instance - parent");
    }

    // Constructor
    public Parent() {
        System.out.println("constructor - parent");
    }

    // Static init block
    static {
        System.out.println("static - parent");
    }
}
```

OUTPUT:

> static - grandparent
>
> static - parent
>
> error: can't find main(String[]) method in class: Child


#####  class Child (no static definitions and blocks) -> public class LoadTest -> class Grandparent  -> class Parent 

```java

class Child extends Parent {
    String a = null;
}

public class LoadTest {

    // Static init block
    static {
        System.out.println("static - loadtest");
    }

    public static void main(String[] args) {
        System.out.println("START");
        new Child();
        System.out.println("END");
    }
}

class Grandparent {
    // Static init block
    static {
        System.out.println("static - grandparent");
    }

    // Instance init block
    {
        System.out.println("instance - grandparent");
    }

    // Constructor
    public Grandparent() {
        System.out.println("constructor - grandparent");
    }
}

class Parent extends Grandparent {
    // Instance init block
    {
        System.out.println("instance - parent");
    }

    // Constructor
    public Parent() {
        System.out.println("constructor - parent");
    }

    // Static init block
    static {
        System.out.println("static - parent");
    }
}
```

OUTPUT:

> static - grandparent
>
> static - parent
>
> error: can't find main(String[]) method in class: Child


#####    class Parent  -> public class LoadTest -> class Grandparent   -> class Child

```java

class Parent extends Grandparent {
    // Instance init block
    {
        System.out.println("instance - parent");
    }

    // Constructor
    public Parent() {
        System.out.println("constructor - parent");
    }

    // Static init block
    static {
        System.out.println("static - parent");
    }
}

public class LoadTest {

    // Static init block
    static {
        System.out.println("static - loadtest");
    }

    public static void main(String[] args) {
        System.out.println("START");
        new Child();
        System.out.println("END");
    }
}

class Grandparent {
    // Static init block
    static {
        System.out.println("static - grandparent");
    }

    // Instance init block
    {
        System.out.println("instance - grandparent");
    }

    // Constructor
    public Grandparent() {
        System.out.println("constructor - grandparent");
    }
}

class Child extends Parent {
    // Constructor
    public Child() {
        System.out.println("constructor - child");
    }

    // Static init block
    static {
        System.out.println("static - child");
    }

    // Instance init block
    {
        System.out.println("instance - child");
    }
}
```


OUTPUT:

> static - grandparent
>
> static - parent
>
> error: can't find main(String[]) method in class: Parent


##### class Child -> class Parent -> public class LoadTest -> class Grandparent

```java
class Child extends Parent {
    // Constructor
    public Child() {
        System.out.println("constructor - child");
    }

    // Static init block
    static {
        System.out.println("static - child");
    }

    // Instance init block
    {
        System.out.println("instance - child");
    }
}

class Parent extends Grandparent {
    // Instance init block
    {
        System.out.println("instance - parent");
    }

    // Constructor
    public Parent() {
        System.out.println("constructor - parent");
    }

    // Static init block
    static {
        System.out.println("static - parent");
    }
}

public class LoadTest {

    // Static init block
    static {
        System.out.println("static - loadtest");
    }

    public static void main(String[] args) {
        System.out.println("START");
        new Child();
        System.out.println("END");
    }
}

class Grandparent {
    // Static init block
    static {
        System.out.println("static - grandparent");
    }

    // Instance init block
    {
        System.out.println("instance - grandparent");
    }

    // Constructor
    public Grandparent() {
        System.out.println("constructor - grandparent");
    }
}
```

OUTPUT:

> static - grandparent
>
> static - parent
>
> static - child
>
> error: can't find main(String[]) method in class: Child




*hello.java*

```java
class Super {
    static { System.out.println("Super "); }
}

class One {
    static { System.out.println("One "); }
}

class Two extends Super {
    static { System.out.println("Two "); }
}

class Test {
    public static void main(String[] args) {
        One o = null;
        Two t = new Two();
        System.out.println((Object)o == (Object)t);
    }
}
```

OUTPUT:

> Super
>
> error: can't find main(String[]) method in class: Super


```java

class Test {
    public static void main(String[] args) {
        One o = null;
        Two t = new Two();
        System.out.println((Object)o == (Object)t);
    }
}

class Super {
    static { System.out.println("Super "); }
}

class One {
    static { System.out.println("One "); }
}

class Two extends Super {
    static { System.out.println("Two "); }
}

```

OUTPUT:

> Super
>
> Two
>
> false


```java
class Test {
    public static void main(String[] args) {
        One o = new One();
        Two t = new Two();
        System.out.println((Object)o == (Object)t);
    }
}

class Super {
    static { System.out.println("Super "); }
}

class One {
    static { System.out.println("One "); }
}

class Two extends Super {
    static { System.out.println("Two "); }
}
```


OUTPUT:

> One
>
> Super
>
> Two
>
>false




```java
public class ExampleSubclass extends Example {

    {
        step(11);
    }

    public static int step_3 = step(3);
    public int step_12 = step(12);

    static {
        step(4);
    }

    public ExampleSubclass(int unused) {
        super(step(7));
        step(13);
    }

    public static void main(String[] args) {
        step(5);
        new ExampleSubclass(step(6));
        step(14);
    }
}

public class Example {

    static {
        step(1);
    }

    public static int step_2 = step(2);
    public int step_8 = step(8);

    public Example(int unused) {
        super();
        step(10);
    }

    {
        step(9);
    }

    // Just for demonstration purposes:
    public static int step(int step) {
        System.out.println("Step " + step);
        return step;
    }
}
```


OUTPUT:

> Step 1
>
> Step 2
>
> Step 3
>
> Step 4
> 
> Step 5
>
> Step 6
>
> Step 7
>
> Step 8
>
> Step 9
>
> Step 10
>
> Step 11
>
> Step 12
>
> Step 13
>
> Step 14

Remarks on `super()`

> If a constructor does not explicitly invoke a superclass constructor, the Java compiler automatically inserts a call to the no-argument constructor of the superclass. If the super class does not have a no-argument constructor, you will get a compile-time error.


```java
public class Example {

    static {
        step(1);
    }

    public static int step_2 = step(2);
    public int step_8 = step(8);

    public Example(int unused) {
        super();
        step(10);
    }

    {
        step(9);
    }

    // Just for demonstration purposes:
    public static int step(int step) {
        System.out.println("Step " + step);
        return step;
    }
}


public class ExampleSubclass extends Example {

    {
        step(11);
    }

    public static int step_3 = step(3);
    public int step_12 = step(12);

    static {
        step(4);
    }

    public ExampleSubclass(int unused) {
        super(step(7));
        step(13);
    }

    public static void main(String[] args) {
        step(5);
        new ExampleSubclass(step(6));
        step(14);
    }
}
```

OUTPUT:

> Step 1
>
> Step 2
>
> error: can't find main(String[]) method in class: Example



If a constructor in the child class does not explicitly call a parent class constructor using super, the Java compiler automatically inserts a call to the no-argument constructor of the parent class.
No No-Argument Constructor: If the parent class does not have a no-argument constructor and the child class does not explicitly call another constructor of the parent class using super, the code will not compile.

```
public class Main {
    public static void main(String[] args) {
        Child child1 = new Child();
        Child child2 = new Child("Hello");
        Child child3 = new Child(42);
        Child child4 = new Child("Hello", 42);
    }
}

public class Child extends Parent {
    public Child() {
        System.out.println("Child no-arg constructor");
    }

    public Child(String message) {
        System.out.println("Child constructor with message: " + message);
    }

    public Child(int number) {
        System.out.println("Child constructor with number: " + number);
    }

    public Child(String message, int number) {
        System.out.println("Child constructor with message: " + message + " and number: " + number);
    }
}

public class Parent {
    public Parent() {
        System.out.println("Parent no-arg constructor");
    }

    public Parent(String message) {
        System.out.println("Parent constructor with message: " + message);
    }

    public Parent(int number) {
        System.out.println("Parent constructor with number: " + number);
    }
}
```

Output:

> Parent no-arg constructor
>
> Child no-arg constructor
>
> Parent no-arg constructor
>
> Child constructor with message: Hello
>
> Parent no-arg constructor
>
> Child constructor with number: 42
>
> Parent no-arg constructor
>
> Child constructor with message: Hello and number: 4
