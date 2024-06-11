

```java
package pkg;

public class LoadTest {
    public static void main(String[] args) {
        System.out.println("START");
        new Child();
        System.out.println("END");
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
and got this output:


>START

> static - grandparent

> static - parent

> static - child

> instance - grandparent

> constructor - grandparent
> instance - parent
> constructor - parent
> instance - child
> constructor - child
> END

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
OUTPUT
======
static block executed 
main started
block executed
constructor executed
fun executed

A class or interface type T will be initialized immediately before the first occurrence of any one of the following:

T is a class and an instance of T is created.

A static method declared by T is invoked.

A static field declared by T is assigned.

A static field declared by T is used and the field is not a constant variable (§4.12.4).

T is a top level class (§7.6) and an assert statement (§14.10) lexically nested within T (§8.1.3) is executed.


https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html#jls-12.4
When a class is initialized, its superclasses are initialized (if they have not been previously initialized), as well as any superinterfaces (§8.1.5) that declare any default methods (§9.4.3) (if they have not been previously initialized). 
Initialization of an interface does not, of itself, cause initialization of any of its superinterfaces.


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
