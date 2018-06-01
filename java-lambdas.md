# Lambda fun!

These are my notes on JAVA 8's lambdas.

**Topics**

1. [Anonymous inner class](#anonymous-inner-class)
2. [What's in a lambda](#whats-in-a-lambda)
3. [Type inference](#simplifying-further-with-type-inference)
4. [Java ready interfaces](#using-java-interfaces)
5. [Exception handling](#exception-handling)
6. [Closures](#closures)
7. [Method References :: ](#method-references)
8. [Resources](#resources)

**Examples**
- [Example 1 - Simple lambda expression](#example-1)
- [Example 2 - Using our own interface](#example-2)
- [Example 3 - Using a Java interface](#example-3)


## Anonymous inner class
An inner class declared without a class name is known as an **anonymous inner class**.
Generally, they are used whenever you need to override the method of a class or an interface.

Syntax:
```java
AnonymousInner an_inner = new AnonymousInner() {
   public void my_method() {
      ........
      ........
   }   
};
```

For example:
```java
abstract class Animal {
   public abstract void talk();
}

public class AnonymousExample {

   public static void main(String args[]) {

      Animal puppy = new Animal() {

         public void talk() {
            System.out.println("WOOF");
         }

      };

      puppy.talk();
   }
}
```


## Whats in a lambda
Java lambda expressions are Java's first step into **functional programming**. A Java lambda expression is thus a function which can be created without belonging to any class. A lambda expression can be passed around as if it was an object and executed on demand.

An example:

If we start with a function
```java
public int doubleNumber(int x) {
  returns x * 2;
}
```

We can assign it to a variable so that it can be passed around:
```java
myDoubleLambda = public int doubleNumber(int x) {
  returns x * 2;
}
```

We can simplify this by removing the `public` keyword as it's not needed, as it's inferred by the left side of the assignment.
```java
myDoubleLambda = int doubleNumber(int x) {
  returns x * 2;
}
```

We also don't need the function name `doubleNumber` as we are now assigning it to a variable.
```java
myDoubleLambda = int (int x) {
  returns x * 2;
}
```

The type `int` is not needed as Java can use type inference and work out that the return value will result in an `int`.
```java
myDoubleLambda = (int x) {
  returns x * 2;
}
```

Add the arrow symbol and now we have a lambda expression:
```java
myDoubleLambda = (int x) -> {
  returns x * 2;
}
```

We can move all of this onto one line as there is only one line of code and we can also remove the `return` keyword and the curly braces in this case.
```java
myDoubleLambda = (int x) -> x * 2;
```

**And that's a lambda expression!**

We can use this lambda expression and pass it into functions as arguments!

```java
myFunction(myDoubleLambda);

myFunction((int x) -> x * 2);
```


## Simplifying further with type inference
Type inference refers to the automatic deduction of the data type of an expression in a programming language. If we use the concept of interfaces to help us declare our lambda types, we can simplify our code using type inference.

If we declared an interface as
```java
interface MyLambda {
  int foo(int a);
}
```

We can now assign our lambda expression from before a type.
```java
MyLambda myDoubleFunction = (int x) -> x * 2;
```

The interface declaration now infers that our parameter must be of type `int` so we can further simplify our expression by removing the `int`.
```java
MyLambda myDoubleFunction = (x) -> x * 2;
```

We can also remove the brackets because there is only one parameter as a final step
```java
MyLambda myDoubleFunction = x -> x * 2;
```


## Using Java interfaces
Java 8 does not have a `function` type to let the compiler know that we have created a lambda expression. We have to create our own functional interfaces, however, there are some common patterns available. So instead of creating new interfaces for the common patterns, there are out-of-the-box interfaces in Java 8. See [java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html).

See [example 3](#example-3) below for an example.

## Exception handling
A good way to handle exceptions when working with lambdas is to create a wrapper lambda which takes the lambda you create as a parameter and returns it.

For example, if we have a function that takes a lambda and does something
```java
private void upload(MyLambda myLambda) {
  myLambda.upload(data);
}

interface MyLambda {
    void upload(Data data);
}
```

And I call this function with the following lambda as the param
```java
public void main(String[] args) {
  upload( (data) -> System.out.println("foo") );
}
```

Instead of having a try/catch block here, I should create a lambda wrapper that takes the lambda as a param and returns it, like this:
```java
private MyLambda lambdaWrapper(MyLambda myLambda) {
  return (data) -> System.out.println("foo");
}
```

I can then update the call to this - which at this point is identical to before, there's just an extra layer that the code passes through.
```java
public void main(String[] args) {
  upload( lambdaWrapper((data) -> System.out.println("foo")) );
}
```

Now to include the exception handling inside the wrapper to make it useful.
```java
private MyLambda lambdaWrapper(MyLambda myLambda) {
  return (data) -> {
    try {
      myLambda.upload(data);
    }
    catch (RuntimeException e){
      System.out.println(e);
    }
  }
}
```

## Closures
A closure is a block of code that can be referenced (and passed around) with access to the variables of the enclosing scope.

For example, the following code has a `Process` interface that is being used when we call `doProcess`. We override the `process` method with an implementation that accesses a variable `b` that is scoped outside of the interface and looks like it shouldn't be able to access. The reason that it can access the variable `b` is because it effectively is a final value.

Java 8 let's us do this if the variable is declared `final` or effectively behave as a `final` (i.e. No assigning a new value to the variable).


```java
public class ClosureExample {

    public static void main(String[] args) {

        int a = 10;
        int b = 20;

        doProcess(a, new Process() {

          @Override
          public void process(int i) {
            System.out.println(i + b);
          }
        });

        // or
        // doProcess(a, (int i) -> System.out.println(i + b));

    }

    public static void doProcess(int y, Process p) {
        p.process(y);
    }

}


interface Process {
    void process(int x);
}
```

If we had `b = 30;` before our override method, this bit of code would not compile.


## Method References
Method reference is the shorthand syntax for a lambda expression that executes just **ONE** method.

Here's the general syntax of a method reference:

`Object :: methodName`

We know that we can use lambda expressions instead of using an anonymous class. But sometimes, the lambda expression is really just a call to some method, for example:
```java
Consumer<String> c = s -> System.out.println(s);
```

To make the code clearer, you can turn that lambda expression into a method reference:

```java
Consumer<String> c = System.out::println;
```
In a method reference, you place the object (or class) that contains the method before the :: operator and the name of the method after it without arguments.

So, in other words:


>Instead of using **AN ANONYMOUS CLASS**

>you can use **A LAMBDA EXPRESSION**

>And if this just calls one method, you can use **A METHOD REFERENCE**


**Here are the main 4 types of method references:**

#### Example 1
```java
(args) -> Class.staticMethod(args)
```
Can be turned into
```java
Class::staticMethod
```

#### Example 2
```java
(obj, args) -> obj.instanceMethod(args)
```
Can be turned into
```java
ObjectType::instanceMethod
```

#### Example 3
```java
(args) -> obj.instanceMethod(args)
```
Can be turned into
```java
obj::instanceMethod
```

#### Example 4
```java
(args) -> new ClassName(args)
```
Can be turned into
```java
ClassName::new
```

#### Another example
```java
public class MethodReferenceExample {

  public static void main(String[] args) {

    Thread t = new Thread(() -> printMessage());
  }

  public static void printMessage() {
    System.out.println("hello");
  }
}
```
We can turn
```java
Thread t = new Thread(() -> printMessage());
```
into this
```java
Thread t = new Thread(MethodReferenceExample::printMessage);
```



## Lambda Examples

### Example 1

A simple interface with its lambda expression following
```java
interface MyAddType {
  int blah(int x, int y);
}

addFunction = (int a, int b) -> a + b;

MyAddType addFunction = (a, b) -> a + b;
```

The method name `blah` in the interface definition actually doesn't matter, because it's main purpose is to supply a signature for lambads to match and compile against.

Java looks only at the return value and the parameters of an interface to match lambda signatures to compile!

**So the name of the interface and the name of the abstract method can be ANYTHING!**

*Note: This is why lambdas must only match to interfaces that have only 1 abstract method.*


### Example 2

Say I want to print people from a list of people those that satisfy a condition. Setting this up so that Condition can take in lambdas is useful, as we can pass in all sorts of implementations of Condition. Conditions where the last name starts with "C", or conditions where the firstname starts with "B", etc.

```java
private static void printWithConditions(List<Person> people, Condition condition) {
    for (Person p: people) {
        if(condition.validate(p)) {
            System.out.println(p.toString());
        }
    }
}
```

The interface for Condition would look like this. It takes a person and returns a boolean. The implementation of this method is left un-coded for the lambda expression later.
```java
interface Condition {
    boolean validate(Person p);
}
```

Now we might use the `printWithConditions()` method like this

```java
List<Person> people = Arrays.asList(
  new Person("Charles", "Dickens"),
  new Person("Lewis", "Carrols"),
  new Person("Thomas", "Chickens"),
  new Person("Charlotte", "Bronte"),
  new Person("Matthew", "Arnold")
);

printWithConditions(people, (Person someone) -> someone.getLastname().startsWith("C"));
printWithConditions(people, person -> person.getLastname().startsWith("B"));
printWithConditions(people, (p) -> p.getFirstname().startsWith("L");
printWithConditions(people, p -> true);
```

The lambdas all take 1 parameter: a `Person`. The above show 4 different ways of supplying this parameter. The `type` Person isn't needed as it's inferred via the interface, but we can still supply it to be explicit.

The lambdas all need an implementation that returns a `boolean` on the right side of the arrow.

### Example 3
Using java.util.function ready to go functional interfaces.

The `Predicate` interface accepts an object and returns a boolean. This matches the signature of our lambda that we want to use.

The `Predicate` interface has 1 abstract method that is given to us, which is called `test()`.

```java
private static void printWithConditions(List<Person> people, Predicate<Person> condition) {
    for (Person p: people) {
        if(condition.test(p)) {
            System.out.println(p.toString());
        }
    }
}
```

## Resources

[Youtube lambda basics](https://www.youtube.com/watch?v=gpIUfj3KaOc&list=PLqq-6Pq4lTTa9YGfyhyW2CqdtW9RtY-I3)
