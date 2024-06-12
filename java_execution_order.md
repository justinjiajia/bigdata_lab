
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

- The public class or the 1st class with a static block or a static function definition, which comes first, is loaded first.

- If the loaded class is a derived one, all its superclasses are loaded first according to hierarchical order.

- Other classes are skipped.




class `LoadTest` is public, should be declared in a file named *LoadTest.java*


#####  public class LoadTest -> class Parent  -> class Grandparent  -> class Child ->class IAmAClassThatIsNeverUsed

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

class IAmAClassThatIsNeverUsed {
    // Constructor
    public IAmAClassThatIsNeverUsed() {
        System.out.println("constructor - IAACTINU");
    }

    // Instance init block
    {
        System.out.println("instance - IAACTINU");
    }

    // Static init block
    static {
        System.out.println("static - IAACTINU");
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


#####

```java

public class LoadTest {


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
```
OUTPUT:

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


```

class Child extends Parent {
    // Constructor
    public Child() {
        System.out.println("constructor - child");
    }


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

static - grandparent
static - parent
static - child


```java

class Child extends Parent {
    // Constructor
    public Child() {
        System.out.println("constructor - child");
    }


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


static - grandparent
static - parent
error: can't find main(String[]) method in class: Child

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


##### class Child -> class Grandparent -> class Parent -> public class LoadTest 

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







https://medium.com/@mk8961052/different-ways-of-the-order-of-constructor-execution-in-java-308815a6fabf

1) Order of execution of constructor in Single inheritance

In single level inheritance, the constructor of the base class is executed first.

OrderofExecution1.java

/* Parent Class */
class ParentClass {
    /* Constructor */
    ParentClass(){
    System.out.println("ParentClass constructor executed.");
    }
}
/* Child Class */
class ChildClass extends ParentClass{
    /* Constructor */
    ChildClass(){
    System.out.println("ChildClass constructor executed.");
    }
}
public class OrderofExecution1{
    /* Driver Code */
    public static void main(String ar[]){
        /* Create instance of ChildClass */
        System.out.println("Order of constructor execution…");
        new ChildClass();
    }
}
Output:

Order of constructor execution…
ParentClass constructor executed.
ChildClass constructor executed.
In the above code, after creating an instance of ChildClass the ParentClass constructor is invoked first and then the ChildClass.

2) Order of execution of constructor in Multilevel inheritance

In multilevel inheritance, all the upper class constructors are executed when an instance of bottom most child class is created.

OrderofExecution2.java

class College{
    /* Constructor */
    College(){
    System.out.println("College constructor executed");
    }
}
class Department extends College{
    /* Constructor */
    Department(){
    System.out.println("Department constructor executed");
    }
}
class Student extends Department{
    /* Constructor */
    Student(){
    System.out.println("Student constructor executed");
    }
}
public class OrderofExecution2{
    /* Driver Code */
    public static void main(String ar[]){
        /* Create instance of Student class */
        System.out.println("Order of constructor execution in Multilevel inheritance…");
        new Student();
    }
}
Output:

Order of constructor execution in Multilevel inheritance…
College constructor executed
Department constructor executed
Student constructor executed
In the above code, an instance of Student class is created and it invokes the constructors of College, Department and Student accordingly.

3) Calling same class constructor using this keyword

Here, inheritance is not implemented. But there can be multiple constructors of a single class and those constructors can be accessed using this keyword.

OrderofExecution3.java

public class OrderofExecution3
{
    /* Default constructor */
    OrderofExecution3()
    {
        this("CallParam");
        System.out.println("Default constructor executed.");
    }
    /* Parameterized constructor */
    OrderofExecution3(String str)
    {
        System.out.println("Parameterized constructor executed.");
    }
    /* Driver Code */
    public static void main(String ar[])
    {
        /* Create instance of the class */
        System.out.println("Order of constructor execution…");
        OrderofExecution3 obj = new OrderofExecution3();
    }
}
Output:

Order of constructor execution…
Parameterized constructor executed.
Default constructor executed.
In the above code, the parameterized constructor is called first even when the default constructor is called while object creation. It happens because this keyword is used as the first line of the default constructor.

4) Calling superclass constructor using super keyword

A child class constructor or method can access the base class constructor or method using the super keyword.

OrderofExecution4.java

/* Parent Class */
class ParentClass
{
    int a;
    ParentClass(int x)
    {
        a = x;
    }
}
/* Child Class */
class ChildClass extends ParentClass
{
    int b;
    ChildClass(int x, int y)
    {
        /* Accessing ParentClass Constructor */
        super(x);
        b = y;
    }
    /* Method to show value of a and b */
    void Show()
    {
        System.out.println("Value of a : "+a+"\nValue of b : "+b);
    }
}
public class OrderofExecution4
{
    /* Driver Code */
    public static void main(String ar[])
    {
        System.out.println("Order of constructor execution…");
        ChildClass d = new ChildClass(79, 89);
        d.Show();
    }
}
Output:

Order of constructor execution…
Value of a : 79
Value of b : 89
In the above code, the ChildClass calls the ParentClass constructor using a super keyword that determines the order of execution of constructors.
