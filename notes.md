Quotations and code examples are from "Functional Programming in Java" by Venkat Subramaniam

## Stream.reduce()

Elements of the reduce function: 
	Identity
	Accumulator
	Combiner

https://www.baeldung.com/java-stream-reduce

Identity – an element that is the initial value of the reduction operation and the default result if the stream is empty

Accumulator – a function that takes two parameters: a partial result of the reduction operation and the next element of the stream

Combiner – a function used to combine the partial result of the reduction operation when the reduction is parallelized or when there's a mismatch between the types of the accumulator arguments and the types of the accumulator implementation



Example: Test if the sum of the first 6 positive integers is equal to 21. 
``` java
List<Integer> numbers = Arrays.asList(1,2,3,4,5,6); 
int results = numbers
	.stream()
	.reduce(0, (subtotal, element) -> subtotal + element); 

assertThat(result).isEqualTo(21); 
```
0 is the identity. The starting value, and the default if the stream is empty. 


(subtotal, element) -> subtotal + element 
^ This is the accumulator. subtotal is the running total, element is the next elemement in the series. 


The accumulator does not have to be a lambda. It can be another function. 
``` java
int results = numbers
        .stream()
        .reduce(0, Integer::sum);  // == 21, same as above. 
```

With Parallel Streams, we need another function. The job of that function is to combine the results of the parrallel streams into one end result. This was creatively named Combiner. 
``` java
List<Integer> ages = Arrays.asList(25, 30, 45, 28, 32);
int computedAges = ages.parallelStream().reduce(0, (a, b) -> a + b, Integer::sum);
``` 

``` Integer.sum() ``` is the combiner. In this case, when we are just adding subtotals and then combining those subtotats into one final result... We can just use a copy of the accumulator as the combiner. 

like so: 
``` java
List<Integer> ages = Arrays.asList(25, 30, 45, 28, 32);
    int computedAges = ages
	.parallelStream()
	.reduce(0, 
	(a, b) -> a + b, 
	(a, b) -> a + b);
```

External Iterators, like the usual for loop or the foreach loop. Let us control what we want to do, and how we do it. This is not always desirable. We can use internal iterators,which we tell what we want to do - and we don't care about how it is done (how it is iterated over). 

The external iterators fail the "Tell, don't ask" Design Principle. 
We want to "Tell" the underlying libraries to handle the low level details where possible. 


Let's consider enumerating a list of names, printing each name. 
``` java
final List<String> friends = 
    Arrays.asList("Brian", "Nate", "Neal", "Raju", "Sara", "Scott");

// Using the familiar for loop, we might write. 

for(int i = 0 ; i < friends.size() ; i++){
System.out.println(friends.get(i); 
}

// Or maybe we fancy the enhanced for loop

for(String name : friends){
System.out.println(name); 
}
```

Approaching with internal iterators and functional interfaces (an interface with one abstract method, such as Runnable, EventListener, Consumer, etc)
``` java
friends.forEach( new Consumer<String>() {
	@Override // Consumer is a functional interface
	public void accept(final String name) {
		System.out.println(name); 
	}
}
```
This uses an anonymous inner class of type Consumer.

With the internal iterator, we are defining an anonymous function that specified what we want to do. In this case, Consumer's method (it's only one), accept() is defined by us. Whatever we define will be executed for each element in the collection. 

Forfeiting control over iteration came at the cost of readability. 

This is where lambda expressions shine. 


friends.forEach((final String name) -> System.out.println(name) ); 

Replacing the anonymous inner class with a lambda expression reduced the noise and produced a concise piece of code. 

We can even omit the type information for the parameter, name. 

friends.forEach((name) -> System.out.println(name) ); 

The type will be inferred. 

And in cases where only one parameter is used, we can even remove the parentheses surrounding the parameter list. 

friends.forEach( name  -> System.out.println(name) ); 

However... With this stripped-down, inferred parameter, the parameter is non-final. The motivation for keeping the parameters final is to avoid modifications, which would taint the pure function. 

Pure functions are functions with no side effects, a goal of functional programming. 

But wait, there's more... 

We can shorten this code further by using "Method Reference" 

``` friends.forEach(System.out::println); ```

Beautiful. 

"Perfection is achieved not when there is nothing more to add, but when there is nothing left to take away" - Antoine de Saint Exupery



## Transforming a list with lambdas. 

Imperative approach. External iteration. 
``` java
 final List<String> uppercaseNames = new ArrayList<String>();
    
    for(String name : friends) {
      uppercaseNames.add(name.toUpperCase());
    }
```
Declarative approach - Internal iteration. 

``` 
 final List<String> uppercaseNames = new ArrayList<String>();

 friends.forEach(name -> uppercaseNames.add(name.toUpperCase())); 
```
Can we do away with the empty List?

We can use the ```.map() ``` function from java Streams. 

.map() is one of the fluent functions in the stream API

https://java-design-patterns.com/patterns/fluentinterface/

This will help us avoid mutability and keep the code consise. 

``` java
friends
.stream()
.map(name -> name.toUpperCase()) //infered type as String.
.forEach(System.out::println); 

```
## Method References 

In the above example, ``` System.out::println ```is used in place of
``` (name) -> System.out.println(name). ```


The java compiler will take either a lmbda expression or a reference to a method where an implementation of a functional interface is expected. 


When should method reference be used in place of a lambda function? 

Some smells that should stear you towards using method reference: 
* The lambda expression is just passing the paramater through. 
... Like  (name) -> System.out.println(name). 
* The lambda expression is very short. 
* Direct calls to an instance method or a static method. 


If you have to manipulate parameters before sending them through to the next function... You can't use method reference. 
"


Removing duplication in lambdas. 

In the list of friends we want the number of  names that start with 'B'

Easy enough. 
``` java
friends.stream().filter(name -> name.startsWith("B")).count(); 
```

what about the number of names that start with 'N'? 

Same thing.. 

``` java
friends.stream().filter(name -> name.startsWith("N")).count(); 
```
But here we have some duplication. Violates the DRY principle. 

We can assign lambda expressions to variables and reuse them. 

The filter method, the receiver of the lambda expression in the examples above, takes a reference to java.util.function.Predicate, which is a functional interface. 

Under the hood, the Java compiler synthesizes an implementation of Predicate's test() method. 


A Predicate is a functional interface. Its functional method is testhttps://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html

Let's define a Predicate. 
``` java
final Predicate<String> startsWithN = name -> name.startsWith("N"); 
```
We can rewrite our code to: 
``` java
friends.stream().filter(startsWithN).count(); 
```
But we will run into the same duplicate code when we need to search for other names. We'd need 
``` java
final Predicate<String> startsWithB = name -> name.startsWith('B') 
```
... and so on. We have noisier duplication. That's bad. 

## Lexical Scoping 

We could try parameterzing a function. But filter() will only accept a function that accepts one parameter representing the context eleming in the collection, and returning a boolean result... That is the definition of the Predicate interface. The context element being the element currently in the pipeline. 

``` java
public static Predicate<String> checkIfStartsWith(final String letter) {

return name -> name.startsWith(letter); 

}
```

Where is the lambda getting the 'letter' variable? There is nothing before it in the pipeline, no stream() passing the value through... 

This is called Lexical Scoping. Java 'reaches' outside its current scope and looks for the 'letter'. This lets us cache values from one context for later use in another context. 

A caveat: we can only access local variables that are final, in the enclosing scope. In this case, the parameter "final String letter"

The variables must be final because lambda expressions may be invoked immediately, or they may be invoked lazily, or from multiple threads. To avoid race conditions, we make local variables immutable using the final modifier. 


Okay... back to looking for names. 

final long countFriendsStartN = friends.stream().filter(checkIfStartsWith("N")).count(); 

So in ``` filter() ```  we pass a function, which returns a predicate. 


There are still smells in the code. 
We don't want to use the static method to cache each variable. We want to narrow the scope to where it is actually needed. 

We can do that using the Function Interface. This is another functional interface in Java 8. From the java.util.function.Function class docs: Functional interfaces provide target types for lambda expressions and method references.

Target Types and Context -  https://stackoverflow.com/questions/33196325/java-8-target-typing

``` java
final Function<String, Predicate<String>> startsWithLetter = 

(string letter) -> { Predicate<String> checkStarts = (String name) -> name.startsWith(letter); 
return checkStarts; 
};
```

instead of explicitly creating the Predicate, we can replace it with a lambda expression. 
``` java
final Function<String, Predicate<String>> startsWithLetter = 

(String letter) -> (String name) -> name.startsWith(letter); 
```
So now we can write: 
``` java
final long countFriendsStartN = friends.stream()
	.filter(startsWithLetter.apply("N")).count(); 
```
Note what we are passing to filter. 
We are passing an instance of Function<String, Predicate<String>>. 

That is, a function that takes a String, and returns a predicate. The functional method of the Function interface is apply(). 

Difference between Functional<T, R> and Predicate<T> 

Predicate<T>
   * takes one parameter of type T and returns a boolean for whatever 'check' is being performed. 	

Function<T, R> 
 * takes a parameter of type T and returns a result of type R. 

We can use Predicate anytime we have a boolean target type. (see SO article on target types and context) 

methods like filter() that evaluate elements can take Predicate as their parameters.

We can use Function anywhere we want to transorm an input to another value.    map() uses Function as its parameter. 




## Strings, Comparators and Filters. 

Iterating over a string
``` java
final String str = "W00t"; 
str.chars().forEach(ch -> sout(ch)); 
```
we can replace this with a method reference since we are just passing the char right through the pipeline. 
``` java
str.chars().forEach(System.out::println); 
```
Right now, this function will print the integer representing the char. We want to print the character. We can write a function to replace the method reference of println()
``` java
private static void printChar(int aChar){
system.out.println( (char) aChar); 
}

```



str.chars().forEach(class_path_to_printChar::printChar); 

We could also just convert the integers to char earlier in the pipline with a mapping function. 

``` java
	str.chars()
	.mapToObj(ch -> Character.valueOf( (char) ch)
	.forEach(System.out::println); 
```
Selecting only the digits in a String. 

``` java
str.chars()
	.filter(ch -> ch.isDigit(ch))
	.forEach(ch -> System.out.println( (char) ch)); 
```

## Comparisons

consider the Person class

``` java
public class Person {
  private final String name;
  private final int age;
  
  public Person(final String theName, final int theAge) {
    name = theName;
    age = theAge;
  } 
  
  public String getName() { return name; }
  public int getAge() { return age; }
  
  public int ageDifference(final Person other) {
    return age - other.age;
  }
  
  public String toString() {
    return String.format("%s - %d", name, age);
  }
}

public class Person{
  final List<Person> people = Arrays.asList(
      new Person("John", 20),
      new Person("Sara", 21),
      new Person("Jane", 21),
      new Person("Greg", 35));


public int ageDifference(final Person other){
return age - other.age;
}

}

```

// sort the list in ascending order. 

``` java
people.stream().sorted((p1,p2) -> p1.ageDifference(p2)).collect(toList())
```
> Sorted Definition from docs
.sorted()

sorted
Stream<T> sorted(Comparator<? super T> comparator)
Returns a stream consisting of the elements of this stream, sorted according to the provided Comparator.
For ordered streams, the sort is stable. For unordered streams, no stability guarantees are made.

This is a stateful intermediate operation.

Parameters:
comparator - a non-interfering, stateless Comparator to be used to compare stream elements
Returns:
the new stream

< End Definition

We can use method reference to really clean this up. 
``` java
people.sorted(Person::ageDifference); 
```
Just like in the first example, we are using Person p1 as the target of the ageDifference method, and Person p2 is passed as a parameter. 

It must be certain that the first parameter is the target, and the second parameter is the argument for the invoked method. 

Using Comparator to reduce duplicated code. 

If we want to sort ascending and descending, the two expressions would be identical besides the order in which they are passed to the ageDifference function. 

Lets use a Comparator. 
``` java
Comparator<Person> compareAscending = (p1, p1) -> p1.ageDifference(p2)
Comparator<Person> compareDescending = compareAscending.reversed(); 
```
Now we can use these in place of a lambda in the sorted() method, or any other method that accepts  Comparator<T>

``` java
people.stream().sorted(compareAscending).collect(toList()); 
```


Sorting by name 
``` java
people.stream().sorted( (p1,p2) -> p1.getname().compareto(person2.getname()); 
```

Lets define a Function that compares two names. 
``` java
final Function<Person, String> byName = person -> person.getName(); 


people.stream().sorted(comparing(byName); 



// Collecting. 

// people older than 20 years
List<Person> olderThan20 = people.stream()
	.filter(person -> person.getAge() > 20)
	.collect(ArrayList::new, ArrayList::add, ArrayList::addAll)
```
The collect() method gathers elements. To do that, it needs to know how to handle three things: 

	1. How to make the container 
	2. How to add one element
	3. How to merge elements. 


`.collect()` also takes a ```Collector``` as a parameter. 

https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html


Using Collectors.toList(), we can replace the above code with: 

``` java
List<Person> olderThan20 = people.stream()
	.filter(person -> person.getAge() > 20)
	.collect(Collectors.toList())
```

// grouping

``` java
Map<Integer, List<People>> peopleByAge = people.stream()
	.collect(Collectors.groupingBy(Person::getAge)); 
```


































## Consumers

A `Consumer`is functional interface with one method, accept(). This method takes one input and does not return anything. 





## Default methods
In the past, interfaces were only allowed to have abstract methods. 
Now, interfaces can have default methods. 

``` java
public class DefaultMethods {
  public interface Fly {
    default void takeOff() { System.out.println("Fly::takeOff"); }
    default void land() { System.out.println("Fly::land"); }
    default void turn() { System.out.println("Fly::turn"); }
    default void cruise() { System.out.println("Fly::cruise"); }
  }
  
  public interface FastFly extends Fly {
    default void takeOff() { System.out.println("FastFly::takeOff"); }    
  }

  public interface Sail {
    default void cruise() { System.out.println("Sail::cruise"); }
    default void turn() { System.out.println("Sail::turn"); }
  }

  public class Vehicle {
    public void turn() { System.out.println("Vehicle::turn"); }
  }
  
  public class SeaPlane extends Vehicle implements FastFly, Sail {
    private int altitude;
    //...
    public void cruise() { 
      System.out.print("SeaPlane::cruise currently cruise like: "); 
      if(altitude > 0)
        FastFly.super.cruise();
      else
        Sail.super.cruise();
    }
  }
  
  public void useClasses() {
    SeaPlane seaPlane = new SeaPlane();
    seaPlane.takeOff();
    seaPlane.turn();
    seaPlane.cruise();
    seaPlane.land();    
  }
  
  public static void main(final String[] args) {
    new DefaultMethods().useClasses();
  }
}

```


Running the main method above, generates: 

FastFly::takeOff
Vehicle::turn
SeaPlane::cruise currently cruise like: Sail::cruise
Fly::land

What's happening? 
	"FastFly::takeOff" is printed from the call to `seaPlane.takeOff()`
	SeaPlane implements FastFly and Sail. 
	So it 'sees' the default method `takeOff` in the `FastFly` interface
	`FastFly` carries forward the default methods from its supertype, `Fly`.
	Additionally, it overrides the default method `takeOff` in the supertype. 
	
	"Vehicle::turn" is printed from the call to `seaPlane.turn()` the supertype' implementation of the `turn` method is used

	The compiler forces us to implement the `cruise` function in `SeaPlane` becuase the defulat implementaions in the interfaces `FastFly` and `Sail` conflict. 
	


## Creating Fluent Interfaces Using Lambda Expressions

Domain

``` java

public class Mailer {
  public void from(final String address) { /*... */ }
  public void to(final String address)   { /*... */ }
  public void subject(final String line) { /*... */ }
  public void body(final String message) { /*... */ }
  public void send() { System.out.println("sending..."); }

  //...
  public static void main(final String[] args) {
    Mailer mailer = new Mailer();
    mailer.from("build@agiledeveloper.com");
    mailer.to("venkats@agiledeveloper.com");
    mailer.subject("build notification");
    mailer.body("...your code sucks...");
    mailer.send();
  }
}

```

Venkat points out two code smells. 
	1.repeated use of the `mailer` object. 
	2.unclear object lifetime

## Method Chaining

Instead of each method returning void, they can return an instance. We can use `this` as the instance, after all, `this` is the object that invokes the first method. 


Refactored: 

``` java

public class MailBuilder {
  public MailBuilder from(final String address) { /*... */; return this; }
  public MailBuilder to(final String address)   { /*... */; return this; }
  public MailBuilder subject(final String line) { /*... */; return this; }
  public MailBuilder body(final String message) { /*... */; return this; }
  public void send() { System.out.println("sending..."); }

  //...
  public static void main(final String[] args) {
    new MailBuilder()
      .from("build@agiledeveloper.com")
      .to("venkats@agiledeveloper.com")
      .subject("build notification")
      .body("...it sucks less...")
      .send();
  }
}
```

## Making the API More Intuitive and Fluent

``` java 
public class FluentMailer {
  private FluentMailer() {}
  
  public FluentMailer from(final String address) { /*... */; return this; }
  public FluentMailer to(final String address)   { /*... */; return this; }
  public FluentMailer subject(final String line) { /*... */; return this; }
  public FluentMailer body(final String message) { /*... */; return this; }
  
  public static void send(final Consumer<FluentMailer> block) { 
    final FluentMailer mailer = new FluentMailer();
    block.accept(mailer); 
    System.out.println("sending..."); 
  }

  //...
  public static void main(final String[] args) {
    FluentMailer.send(mailer ->
      mailer.from("build@agiledeveloper.com")
            .to("venkats@agiledeveloper.com")
            .subject("build notification")
            .body("...much better..."));
  }
}

```

Changes made: 

Constructor is private. Stopping us from direct object creation. 
`Send` is static and expects a `Consumer`. Instead of creating an instance, users will invoke send, and pass a block of code (most likley a lambda) 



## Dealing with Exceptions 

Types of Exceptions
** Checked **  - Errors outside the control of a program. FileNotFoundException, for example. Java verifies checked exceptions at compile-time. 

** Unchecked ** - Errors that reflect some error inside the program logic. i.e., ArithmeticException during divide by zero.  

``` java
// This will not compile 
public class HandleException {
  public static void main(String[] args) throws IOException {
    Stream.of("/usr", "/tmp")
          .map(path -> {
            try {
             return new File(path).getCanonicalPath();             
            } catch(IOException ex) {
             return ex.getMessage();
            }
           }) 
          .forEach(System.out::println);    
  }
}

```

The compile error is in the `map` method. `map` expects `Function<T,R>` as a parameter
`Function.apply()` does not throw any checked exceptions. So, the lambda is ont permitted to throw any checked exceptions. 

We have two options: handle the exception within the lambda, or catch it and rethrow it as an unchecked exception. 

The first option looks like: 

``` java 
Stream.of("/usr", "/tmp").map(path -> {
	try{
		return new File(path).getCanonicalPath(); 
	} catch (IOException ex){
	return ex.getMessage(); 
	}
} // end lambda


The second approach is to `throw new RuntimeException(ex)` within the catch block. 


```

## Working with Resources

Sample Class

``` java 

public class FileWriterExample {
  private final FileWriter writer;
  
  public FileWriterExample(final String fileName) throws IOException {
    writer = new FileWriter(fileName);
  }
  public void writeStuff(final String message) throws IOException {
    writer.write(message);
  }
  public void finalize() throws IOException {
    writer.close();
  }
  //...

  public void close() throws IOException {
    writer.close();
  }

/*
  public static void main(final String[] args) throws IOException {
    final FileWriterExample writerExample = 
      new FileWriterExample("peekaboo.txt");
    writerExample.writeStuff("peek-a-boo");      
  }
*/

public static void callClose(final String[] args) throws IOException {
    final FileWriterExample writerExample = 
      new FileWriterExample("peekaboo.txt");
      
    writerExample.writeStuff("peek-a-boo");      
    writerExample.close();
}

  public static void main(final String[] args) throws IOException {
    final FileWriterExample writerExample = 
      new FileWriterExample("peekaboo.txt");
    try {
      writerExample.writeStuff("peek-a-boo");            
    } finally {
      writerExample.close();      
    }
  }

}


```

## Using Lambda Expressions to Clean up Resources. 


Automatic Resource Management (ARM) - introduced in Java 7. Is a good way to ensoure that external resources are cleaned up. They use a special form of the `try` block. AutoCloseable is the interface that enforces this. 

We can also use lambda expressions 

In this example, FileWriter represents a resource that needs timely cleanup. 

``` java 

public class FileWriterEAM  {
  private final FileWriter writer;
  
  private FileWriterEAM(final String fileName) throws IOException {
    writer = new FileWriter(fileName);
  }
  private void close() throws IOException {
    System.out.println("close called automatically...");
    writer.close();
  }
  public void writeStuff(final String message) throws IOException {
    writer.write(message);
  }
  //...

  public static void use(final String fileName, 
    final UseInstance<FileWriterEAM, IOException> block) throws IOException {
    
    final FileWriterEAM writerEAM = new FileWriterEAM(fileName);    
    try {
      block.accept(writerEAM);
    } finally {
      writerEAM.close();
    }
  }

  public static void main(final String[] args) throws IOException {
  
    System.out.println("//" + "START:EAM_USE_OUTPUT");
    FileWriterEAM.use("eam.txt", writerEAM -> writerEAM.writeStuff("sweet"));
    System.out.println("//" + "END:EAM_USE_OUTPUT");

    FileWriterEAM.use("eam2.txt", writerEAM -> {
        writerEAM.writeStuff("how");
        writerEAM.writeStuff("sweet");      
      });

  }

}

```

Note the code in `use` ... this is the *execute around method* pattern. 
this pattern is useful when we want to ensure prompt cleanup of resources. 
the `synchronized` keyword is an example of the *execute around method*

`UseInstance` is a functional interface that we define. 

``` java 

@FunctionalInterface
public interface UseInstance<T,X extends Throwable> {

void accept( T instance) throws X; 

}

```

We also could have used a `java.function.Consumer` interface. However, Lambda expressions can only throw checked exceptions defined as part of the signature of the abstract method being synthesized. 

When we implemented this in `use`, we defined type `T` to be `FileWriterEAM`, and the generic exception `X` to be `IOException`


## Managing Locks

Some code that uses a `Lock`

``` java 

public class Locking {
  Lock lock = new ReentrantLock(); //or mock
  
  protected void setLock(final Lock mock) {
    lock = mock;
  } 

  public void doOp1() {
    lock.lock();
    try {
      //...critical code...
    } finally {
      lock.unlock();
    }
  }
  //...

  public void doOp2() {
    runLocked(lock, () -> {/*...critical code ... */});
  }
  
  public void doOp3() {
    runLocked(lock, () -> {/*...critical code ... */});
  }
  
  public void doOp4() {
    runLocked(lock, () -> {/*...critical code ... */});
  }

}

```





## Creating Concise Exception Tests

Suppose we have the following class: 

```java 

public class RodCutter {
  private boolean mustFail;
  
  public RodCutter(final boolean fail) { mustFail = fail; }

  public void setPrices(final List<Integer> prices) {
    //...
    if(mustFail) 
      throw new RodCutterException();
  }

  public int maxProfit(final int length) {
    if (length == 0) throw new RodCutterException();
    
    return 0;
  }
}

```

We also write a unit test to verify that `maxProfit` throws an exceptoin if the argument is zero. 

``` java 
 @Test public void VerboseExceptionTest() {
    rodCutter.setPrices(prices);
    try {
      rodCutter.maxProfit(0);
      fail("Expected exception for zero length");
    } catch(RodCutterException ex) {
      assertTrue("expected", true);
    }
  }

```

Helper class for exception testing
java ```
public class TestHelper {
  public static <X extends Throwable> Throwable assertThrows(
    final Class<X> exceptionClass, final Runnable block) {
      
    try {
      block.run();
    } catch(Throwable ex) {
      if(exceptionClass.isInstance(ex))
        return ex;
    }
    fail("Failed to throw expected exception ");
    return null;
  }
}

```

In `TestHelper`, the `assertThrows` static method expects an exception class and a blcok of code to run. Since this helper class is supposed to help with Exception testing, we expect the code block to produce an excpetion. In the catch block we compare the returned exception, to the `exceptionClass` that we provided. If no exception is thrown from `block.run()`, then our exception test failed altogether. 

Now we can write a concise test: 

``` java 
 @Test 
  public void ConciseExceptionTest() {
    rodCutter.setPrices(prices);
    assertThrows(RodCutterException.class, () -> rodCutter.maxProfit(0));
  }
```















# Chapter 6: Being Lazy 

## Delayed Initialization
Create things when you need them, no sooner. 

``` java 

public class Heavy {
  public Heavy() { System.out.println("Heavy created"); }

  public String toString() { return "quite heavy"; }
}

```

This class just represents a heavyweight resource that is expensive to create and manage. 

``` java 

public class HolderNaive {
  private Heavy heavy;
  
  public HolderNaive() {
    System.out.println("Holder created");
  }
  
  public Heavy getHeavy() {
    if(heavy == null) {
      heavy = new Heavy();
    }
    
    return heavy;
  }

//...

  public static void main(final String[] args) {
    final HolderNaive holder = new HolderNaive();
    System.out.println("deferring heavy creation...");
    System.out.println(holder.getHeavy());
    System.out.println(holder.getHeavy());
  }
}

```

`HolderNaive` is simple. We have an instance variable for `Heavy`. When we initialize a `NaiveHolder` will it eagerly create the burdensome `Heavy` object? 

Running the main method generates the following output: 
	Holder created
	deferring heavy creation...
	Heavy created
	quite heavy
	quite heavy

Easy enough. But this is not thread safe. For an instance of `HolderNaive` the dependent instance `heavy` is created on the first call to the `getHeavy` method. If two or more threads call the getHeavy method at the same time, then we might end up with multiple`Heavy` instances. 

I know what you're thinking... Race condition? Lets just make `getHeavy()` _synchronized_
``` java
public synchronized Heavy getHeavy(){

if(heavy == null){
heavy = new Heavy()
}

return heavy; 
}

```
Now when two threads try to access `heavy`, one will have to wait their turn. 

This solves one problem, but a performance issue has snuck in. Each call to `getHeavy` has to deal with the overhead of synchronization. 


We really only need to solve for the race condition when `heavy` is being initialized. Once the reference is created, we should be free to use the `getHeavy` method. 

Lets look at 
``` 
java 

 Supplier<T> 

```

Supplier is a functinoal interface with one method, `get`, which will return an instance. 

``` 
java

Supplier<Heavy> supp = () -> new Heavy();  // returns an instance
Supplier<Heavy> supp = Heavy::new;         // also return an instance
```


If we are to leverage this, we need to postpone, and cache the returned instance. 


Here's how the final `Holder` will look: 

``` 
java 

public class Holder {
  private Supplier<Heavy> heavy = () -> createAndCacheHeavy();
  
  public Holder() {
    System.out.println("Holder created");
  }

  public Heavy getHeavy() {
    return heavy.get();
  }
  //...

  private synchronized Heavy createAndCacheHeavy() {
    class HeavyFactory implements Supplier<Heavy> {
      private final Heavy heavyInstance = new Heavy(); // *Note* 

      public Heavy get() { return heavyInstance; }
    }

    if(!HeavyFactory.class.isInstance(heavy)) {
      heavy = new HeavyFactory();
    }
    
    return heavy.get();
  }

  public static void main(final String[] args) {
    final Holder holder = new Holder();
    System.out.println("deferring heavy creation...");
    System.out.println(holder.getHeavy());
    System.out.println(holder.getHeavy());
  }
} 


```

Now consider the case where two threads compete for a newly created instance of `Holder` by calling `getHeavy()`

getHeavy() will call heavy.get(), this will call `createAndCacheHeavy()`, which is synchronized. One of the threads will wait. Whichever thread enters will check if `heavy` is an instance of `HeavyFactory`, if not, a new instance of `HeavyFactory` is created. Note that a new instance of HeavyFactory creates `heavyInstance`, 




## Lazy streams. 


Streams have two types of methods, `intermediate` and `terminal`
`intermediate` functions, like `filter` and `map` return immediately. The lambda expressions provided to them are cached and are not evaluated until a`terminal` operation is called. Even then, not all cached code is executed.

Consider the following list of names 

``` java

    List<String> names = Arrays.asList("Brad", "Kate", "Kim", "Jack", "Joe",
        "Mike", "Susan", "George", "Robert", "Julia", "Parker", "Benson");
```

we want the first name that is only three letters long, we will print it in all caps. 


``` java
names.stream().filter(s -> s.length() == 3).map(s -> s.toUpperCase()).findFirst().get()

```

How lazy are streams? _very_ lazy. 

These  helper functions will give us a peak under the hood: 

```
java 

public static int getLength(final String name){

	System.out.println("get length for " + name); 
	return name.length(); 

}


public static String toUpper(final String name){

	System.out.println("changing to uppercase for " + name); 
	return name.toUpperCase(); 

}





``` 

replacing the intermediate functions, `filter` and `map` with our helpers: 


``` java
names.stream().filter(s -> s.getLength(s) == 3).map(s -> s.toUpper(s).findFirst().get()

```
Running this in the console shows that the order of execution is not _eager_, it is _lazy_. The intermidate functions delay their execution until the last possible moment. If a terminal method is satisfied, operations stop. If not, more elements from the collectoin are selected for processing. 


## Infinitely Lazy. 

How might we handle a growing set? A set that continues to grow while we process the data? 

Let's look at prime numbers, of which there are infinitely many. 

Lets write a helper function that determines if a number is prime. 

// A number is prime if it is not divisible by a number between 2 and the square root of the number. 

public static boolean isPrime(final int number){
return number > 1 && 
IntStream.rangeClosed(2, (int) Math.sqrt(number)).noneMatch(divisor -> number % divisor == 0)

}

`rangeClosed()` returns the range inclusive. i.e., rangeClosed(1,5) = 1,2,3,4,5


`noneMatch()` takes a `Predicate` as an argument and returns true if the lambda expression returns false for all values in the range. 

So our expression reads as, 
For all number between 2 and the square root of the number being evaluated, if no number in that range evenly divides the number, return true. 


Great. Now we can iterate over numbers, inrementing by 1 as we go, and build a list of all the primes, right? 

*wrong*. We would reach a recusion error, or we would run out of memory. We are dealing with an infinite set. 

Streams are a strong solution for this because they only return elements when they are asked. So we can tap into a collection with a `stream` and retrieve a finite number of elements. 

`Stream` has a static method, `iterate`, which can create an infinite `Stream`

```
java
static <T> Stream<T>	iterate(T seed, UnaryOperator<T> f)

```

`iterate` takes two parameters
	1. seed - a value to start the collection. 
	2. instance of UnaryOperator - supplier of data in the collection. 

```
java

@FunctionalInterface
public interface UnaryOperator<T>
extends Function<T,T>
```


```
java 

public class Primes {
  private static int primeAfter(final int number) {
    if(isPrime(number + 1))
      return number + 1;
    else
      return primeAfter(number + 1);
  }
  
  public static List<Integer> primes(final int fromNumber, final int count) {
    return Stream.iterate(primeAfter(fromNumber - 1), Primes::primeAfter)
                 .limit(count)
                 .collect(Collectors.<Integer>toList());
  }  


```

in `primes`, we iterate using `fromNumber - 1` as the `seed`, `primeAfter is the UnaryOperator'

`iterate` returns a stream that caches the UnaryOperator. When we ask for a number of elements, and only then, the `Stream` feeds the current element to the UnaryOperator to get the next element. This is repeated until we get the number of elements that we asked for. 

