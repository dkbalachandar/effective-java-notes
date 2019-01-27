## Effective Java - 3rd edition Notes

#### Disclaimer

This is a summary of the book "Effective Java" wrote by Joshua Bloch. The below are my personal notes and I hope it's not a copyright infringement. If it is, please contact me in order to remove this from github.

I have not included all the items but will do it gradually. 

### Chapter 2: Creating and Destroying Objects

This chapter concerns creating and destroying objects. When and how to create them, when and how to avoid creating them, how to ensure they are destroyed in a timely manner, and how to manage any cleanup actions that must precede their destruction.

#### Item 1: Consider static factory methods instead of constructors
Instead of adding multiple constructors for a class, we can create static factory methods which has advantages and disadvantages.

For example, the constructor BigInteger(int, int, Random), which returns a BigInteger that is probably prime, would have been better expressed as a static factory method named BigInteger.probablePrime.

Advantages:

1.Unlike constructors, static factor methods they have names
2.They are not required to create a new object each time
3.We can return an object of any subtype of their return type
4.They reduce the verbosity of creating parameterized type instances

Disadvantages:

1.The classes without providing public or protected constructors but have only static factory methods can’t be sub classed
2.They are not readily distinguishable from other static methods

Here are some of the common static factory methods.
1.from – Type conversion which takes a simple parameter and return an instance of this type.
Date d = Date.from(instant)

2.Of – An aggregation method that takes multiple parameters and return returns an instance of this type that incorporates them.
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING)
	
3.valueOf- A more verbose alternative to from and of. Returns an instance that has the same value its parameters.
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);

4.instance or getInstance – Takes parameters  if any and return an instance of the class.
StackWalker luke = StackWalker.getInstance(options);

5.create or newInstance – Same as getInstance but each instance that is returned is distinct from all others.

6.getType – Like getInstance but if the factory method is in different class. 
FileStore fs = Files.getFileStore(path);

7.newType – Like newInstance but if the factory method is in different class.
BufferedReader br = Files.newBufferedReader(path);

8.type – A concise alternative to getType and newType,
List<Complaint> litany = Collections.list(legacyLitany);
	
Often static factories are preferable, so avoid the reflex to provide public constructors without first considering static factories.


```
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

#### Item 2: Consider a builder when faced with many constructor parameters
Static factories and constructors have some limitation and they do not scale well to large number of optional parameters. So, we may use any one of the below options.

1.Telescoping constructor pattern – provide a constructor with only required parameters, another one with a single optional parameter, a third with two optional parameters and so on. It still works but its harder to read it.

2.JavaBean pattern – Create a parameter less constructor and then call the setter methods to set the required parameters and each optional parameter. A JavaBean may be in an inconsistent state partway through its construction and also it prevents the possibility of making a class immutable. 

3.Builder Pattern - Combines the safety of the telescoping pattern and the readability of JavaBean pattern.  In order to create an object, we have to create Builder class. So, the cost of creating builder class is the disadvantage here. It’s good when constructors or static factories would have more than a handful of parameters

#### Item 3: Enforce the singleton property with a private constructor or an enum type

There are different ways to create singletons.
We can create a private constructor in the class and static factory method to return an instance of that class.  We can throw an exception in the private constructor if client code access it through reflections.  

````
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE;}
    public void leaveTheBuilding() { ... }
}

````
A single-element enum type is often the best way to implement a singleton.

````
// Enum singleton - the preferred approach
public enum Elvis {
   INSTANCE;
   public void leaveTheBuilding() { ... }
}
````
#### Item 4: Enforce noninstantiability with a private constructor
Attempting to enforce noninstantiability by making a class abstract does not work as the subclasses can extend this and instantiate it. So create a private constructor in that class and optionally throw Assertion Error if someone tries to instantiate that class within that class itself

````
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ...  // Remainder omitted
}
````
#### Item 5: Prefer Dependency Injection to hardwiring resource
Do not use singleton or static utility classes to implement a class the depends on one or more underlying resource whose behavior affects that of the class, and do not have the class create that resource directly. Instead pass the resources, or factories to create them. This practice is known as dependency Injection, will greatly enhance the flexibility, reusability, and testability of a class.

````
// Dependency injection provides flexibility and testability
public class SpellChecker {
  private final Lexicon dictionary;
  public SpellChecker(Lexicon dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary);
  }
  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
````
### Item 6: Avoid creating unnecessary objects

Its often appropriate to reuse a single object instead of creating a new object every time.

As an extreme example of what not to do, consider this statement:
String s = new String("bikini"); // DON'T DO THIS!

The improved version is simply the following:
String s = "bikini";

The above version uses a single String instance rather than creating a new one.

You can often avoid creating objects by using static factory methods. 

Another way to create unnecessary objects is autoboxing.
````
private static long sum() {
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++){
      sum += i;
      return sum;
     }
   }
````

The variable Sum is declared as Long so every time when we perform the add operation it creates new Long instance. So, prefer primitives to boxed primitives, and watch out for unintentional autoboxing.

“Don’t create a new object when you should reuse an existing one,”

#### Item 7: Eliminate obsolete object references

Nulling out object references should be the exception rather than the norm. Do not overcompensate by nulling out every object. 
Whenever a class manages its own memory, the programmer should be alert for memory leaks. 

For example, Refer the below code from Stack implementation. The reference to an item becomes obsolete as soon as its popped out off the stack. So we have to nulllif the object reference so that it will be garbage collected soon. 
```
public Object pop() {
  if (size == 0)
     throw new EmptyStackException();
  Object result = elements[--size];
  elements[size] = null; // Eliminate obsolete reference
  return result;
}
```
Another common source of memory leaks is caches, Hence use WeakHashMap to represent the cache.
A third common source of memory leaks is lisenters and other callbacks. If clients register callbacks, but never deregister them explicitly. To solve it store only weak references to them, for instance storing them as keys in a WeakHashMap.

Use a Heap Profiler tool to find unseen memory leaks.

#### Item 8: Avoid finalizers and cleaners
Finalizers are unpredictable, often dangerous, and generally unnecessary. 

Cleaners are less dangerous than finalizers but still unpredictable slow and generally unnecessary.

In summary, don’t use cleaners, or in releases prior to Java 9, finalizers,
except as a safety net or to terminate noncritical native resources. Even then,
beware the indeterminacy and performance consequences.

#### Item 9: Prefer try-with-resources to try-finally
Always use try-with-resources in preference to try finally
when working with resources that must be closed. The resulting
code is shorter and clearer, and the exceptions that it generates are more
useful. 

The try-with-resources statement makes it easy to write correct code
using resources that must be closed, which was practically impossible using
try-finally.

### Chapter 3. Methods Common to All Objects
This chapter tells you when and how to override the nonfinal Object methods.

#### Item 10: Obey the general contract when overriding equals

When you override the equals method, you must adhere to its general contract. Here is the contract
1.	Reflexive, For any non-null reference value x, x.equals(x) must return true.
2.	Symmetric, For any non-null reference value x and y, x.equals(y) return true if only if y.equals(x) returns true.
3.	Transitive, For any non-null reference values x, y, z, if x.equals(y), returns true and y.equals(z) returns true, then x.equals(z)
4.	Consistent, For any non-null reference values x and y, multiple invocations of x.equals(y) must consistently return true or consistently return false, provided no information used in equals comparisons is modified.
5.	For any non-null reference value x, x.equals(null) must return false.

#### Item 12: Always override toString
The general contract for toString says that the returned string should be “a concise but informative representation that is easy for a person to read". Providing a good toString implementation makes your class much more pleasant to use and makes systems using the class easier to debug.

#### Item 13: Override clone judiciously
A better approach to object copying is to provide a copy constructor or copy factory. A notable exception to this rule is arrays, which are best copied with the clone method.

#### Item 14: Consider implementing Comparable
Whenever you implement a  value class that has a sensible ordering, you should have that class implement the comparable interface so that its instances can be easily sorted, searched and used in comparison based collections.

### Chapter 4. Classes and Interfaces

#### Item 15: Minimize the accessibility of classes and members
A well-designed component hides all its implementation details, cleanly separating its API from its implementation. Components then communicate only through their APIs and are oblivious to each other’s inner workings. This concept, known as information hiding or encapsulation, is a fundamental tenet of software design.

The rule of thumb is simple: make each class or member as inaccessible as possible. In other words, use the lowest possible access level consistent with the proper functioning of the software that you are writing.

#### Item 16: In public classes, use accessor methods, not public fields
Public classes should never expose mutable fields. Its less harmful though still questionable, for public classes to expose immutable fields. It is, however sometimes desirable for package-protected or private nested classes to expose fields, whether mutable or immutable.

#### Item 17: Minimize Mutability
An immutable class is simple a class whose instance can’t be modified. All of the information contained in each instance is fixed for the lifetime of the object. 
1.Don’t provide methods that modify the object’s state
2.Ensure that the class can’t be extended
3.Make all fields final
4.Make all fields private
5.Ensure excessive access to any mutable components

#### Item 18: Favor composition over inheritance
Inheritance violates the encapsulation. In other way, a subclass depends on the implementation details of its superclass for its proper function. 

With inheritance, you don't know how your class will react with a new version of its superclass. For example, you may have added a new method whose signature will happen to be the same than a method of its superclass in the next release. You will then override a method without even knowing it.

Also, if there is a flaw in the API of the superclass you will suffer from it too. With composition, you can define your own API for your class. As a rule of thumb, to know if you need to choose inheritance over composition, you need to ask yourself if B is really a subtype of A.

``` 
//Wrapper class – Uses composition in place of inheritance.
public class InstrumentedSet<E> extends ForwardingSet<E> {
	private int addCount = 0;
	public InstrumentedSet (Set<E> s){
		super(s)
	}

	@Override
	public boolean add(E e){
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll (Collection< ? extends E> c){
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}

// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {
	private final Set<E> s;
	public ForwardingSet(Set<E> s) { this.s = s ; }

	public void clear() {s.clear();}
	public boolean contains(Object o) { return s.contains(o);}
	public boolean isEmpty() {return s.isEmpty();}
…
}
``` 

#### Item 19: Design and document for inheritance or else prohibit it

The class must document its self-use of overridable methods. 

To document a class so that it can be safely sub classed, you must describe implementations details. To allow programmers to write efficient subclasses without undue pain, a class may have to provide hooks into its internal working in the form of judiciously chosen protected methods. The only way to test a class designed for inheritance is to write subclasses. You must test your class by writing subclasses before you release it.

Constructor must not invoke overridable methods.

If a class is not designed and documented for inheritance it should be me made forbidden to inherit it, either by making it final, or making its constructors private (or package private) and use static factories.

#### Item 20: Prefer Interfaces to Abstract Classes

Intefaces is generally the best way to define a type that permits multiple implementations.

Interfaces are ideal for defining mixins. Mixin is a type that a class can implement in addition to its primary type. For example, 
Comparable interface is a mixin interface that allows a class declare that its instance are ordered with respect to other mutable comparable objects.

Intefaces allow for the construction of nonhirearchical type frameworks. 

### Item 21: Design interfaces for posterity
With Java 8, it's now possible to add new methods in interfaces without breaking old implementations thanks to default methods. But It needs to be done carefully since it can still break old implementations that will fail at runtime.

For example, removeIf() method was added to the Collection interface in Java 8. The SynchronizedCollection class of Apache commons library implemented this interface but does not implement removeIf() method. So when we call removeIf on a SynchronizedCollection invokes the removeIf method of Collection interface and it throws a ConcurrentModificationException or other unspecified behavior may happen.

#### Item 22: Use interfaces only to define types
Interfaces should be used only to define types. They should not be used merely to export constants.
The constant interface pattern is a poor use of interfaces.
``` //Constant interface antipattern. Do not use!
public interface PhysicalConstants {
	static final double AVOGADROS_NUMBER = 6.022_140_857e23;
	static final double BOLTZMAN_CONSTANT = 1.380_648_52e-23;
	...
}
//Constant utility class
public class PhysicalConstants {
	private PhysicalConstants() {} //prevents instantiation
	
	public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
	public static final double BOLTZMAN_CONSTANT = 1.380_648_52e-23;
	...
}
```

#### Item 23: Prefer class hierarchies to tagged classes
Tagged classes are verbose, error-prone and inefficient.
They have lot of boilerplate code, bad readability, they increase the memory footprint and more shortcomings.
For example,

```
       // Tagged Class
	class Figure{
		enum Shape {RECTANGLE, CIRCLE};

                //Tag field 
		final Shape shape;

                //rectangle fields    
		double length;
		double width;

		//circle field
		double radius;

		// Constructor for circle
		Figure (double radius) {
			shape = Shape.CIRCLE;
			this.radius=radius;
		}

		// Constructor for rectangle 
		Figure (double length, double width) {
			shape = Shape.RECTANGLE;
			this.length=length;
			this.width=width;
		}

		double area(){
			switch(shape){
				case RECTANGLE:
					return length*width;
				case CIRCLE
					return Math.PI * (radius * radius);
				default:
					throw new AssertionError(shape);
			}
		}
	}
```
Class hierarchy is a better replacement for a tagged class. It makes the code is simple and clear. 

``` 
  abstract class Figure{
		abstract double area();
	}
	class Circle extends Figure{
		final double radius;

	  Circle(double radius) { this.radius=radius;}
	  double area(){return Math.PI * (radius * radius);}
	}
	class Rectangle extends Figure{
		final double length;
		final double width;

		Rectangle (double length, double width) {
			this.length=length;
			this.width=width;
		}

		double area(){return length*width;}
	}

	class Square extends Rectangle {
		Square(double side){
			super(side,side);
		}
	}
```

#### Item 24: Favor static member classes over nonstatic
If a member class does not need access to its enclosing instance, then declare it static. If the class is non static, each instance will have a reference to its enclosing instance. That can result in the enclosing instance not being garbage collected and memory leaks.

### Item 25: Limit sources files to a single top-level class
Never put multiple top-level classes or interfaces in a single source file. Following rule guarantees that you can’t have multiple definitions for a single class at compile time.
If you are tempted to put multiple top-level classes into single source file, consider using static member classes. Because it enhances readability.

### Chapter: 5 Generics

#### Item: 26 Don’t use raw types
A raw type is  a generic type without its type parameter. For example List is a raw type of List<E>.  It should not be used. It exists primarily for compatibility with pre-generics code.   
There are two exceptions to this rule.
1.	You must use raw types in class literals
List.class or String[].class is legal but not List<String>.class and List<?>.class
2.	Instanceof operator. 
Because generics information erased at runtime, its illegal to use the instance of operator on parameterized types. 
``` 
//Legitimate usa of instance of raw type
if (o instanceof Set){
	Set<?> = (Set<?>) o;
}
```

#### Item: 27 Eliminate unchecked warnings
Eliminate every unchecked warning that you can. 
If you can’t eliminate it, but you can prove that the code that provoked the warning is type safe, then suppress the warning with an @SuppressWarnings(“unchecked”) annotation and also add a comment why its safe to do.
Always use the SuppressWarnings on the smallest scope as possible.

```
        Set<Lark> exaltation = new HashSet(); //Warning, unchecked conversion.

 	Set<Lark> exaltation = new HashSet<>(); //Good
```

#### Item 28: Prefer lists to arrays
Arrays are covariant which means if Sub is a subtype of Super, then the array type Sub[] is a subtype of  the array type Super[]
Generics are invariant for any distinct types Type1 and Type2, List<Type1> is neither a subtype nor a supertype of List<Type2>

```
// Fails at runtime
	Object[] objectArray = new Long[1];
	objectArray[0] ="I don't fit in" // Throws ArrayStoreException

	// Won't compile
	List<Object> ol = new ArrayList<Long>();//Incompatible types
	ol.add("I don't fit in")
```

Arrays are reified: Arrays know and enforce their element types at runtime. 
Generics are erasure: Enforce their type constrains only at compile time and discard (or erase) their element type information at runtime.
Its always prefer to catch the error sooner than later so prefer Lists.

#### Item 29: Favor Generic Types
Generic types are safer and easier to use than types that require casts in client code. When you design new types, make sure that they can be used without such casts. 

#### Item 30: Generic Methods
Generic methods, like generic types, are safer and easier to use than methods requiring their clients to put explicit casts on input parameters and return values. Like types, you should make sure that your methods can be used without casts, which often means making them generic. And like types, you should generify existing methods whose use requires casts. This makes life easier for new users without breaking existing clients.

#### Item 31: Use bounded wildcards to increase API flexibility

Parameterized types are invariant. List<String> is not a subtype of List<Object>. 

If we have a stack implementation and want to add two methods pushAll, popAll, we can do it in the below way woth bounded wildcards

```
// Wildcard type for a parameter that serves as an E producer
public void pushAll(Iterable<? Extends E> src){
	for (E e : src) {
		push(e);
	}
}

// Wildcard type for parameter that serves as an E consumee
public void popAll(Collection<? super E> dst){
	while(!isEmpty()) {
		dst.add(pop());
	}
}
```

In the Stack example, pushAll’s src parameter produces E instances for use by the Stack, so the appropriate type for src is Iterable<? extends E>; popAll’s dst parameter consumes E instances from the Stack, so the appropriate type for dst is Collection<? super E>. Hence remember this mnemonic "PECS stands for producer-extends, consumer-super" to determine when to use extends and super.


#### Item 32: Combine generics and varargs judiciously

#### Item 33: Consider typesafe hetrogenous containers

### Chapter: 6 Enums and annotations

#### Item 34: Use Enum instead of int constants
Enums are more readbale, safer and more powerful. 

```
//The int enum pattern - serverly deficient
public static final int APPLE_FUJI=0;
public static final int APPLE_PIPPIN=1;
public static final int APPLE_GRANNY_SMITH=3;
....

//Using enum 
public enum Apple{FUJI, PIPPIN, GRANNY_SMITH}

```
#### Item 35: Use instance fields instead of ordinals
All enums have an ordinal method, which returns the numerical position of each enum constant in its type. You may be tempted to derive an associated int value from the ordinal as it’s a maintenance nightmare. If the constants are ordered the numberOfMusicians will break.
```
//Abuse of ordinal to derive an associated value. Don’t do this
	public enum Ensemble{
		SOLO, DUET, TRIO...;
		public int numberOfMusicians() {return ordinal() + 1}
	}

  Do not do this instead store it instance field like below.
public enum Ensemble{
		SOLO(1), DUET(2), TRIO(3)...TRIPLE_QUARTET(12);
		private final int numberOfMusicians;
		Ensemble(int size) {this.numberOfMusicians = size;}
		public int numberOfMusicians() {return numberOfMusicians;}
	}
 
 ``` 

#### Item 36: Use EnumSet instead of bit fields
If the elements of an enumerated are used primarily in sets, use EnumSet.
```
public class Text{
		public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

		//Any Set could be passed. Best EnumSet
		public void applyStyles(Set<Style> styles){ ... }
	}

	//Use
	text.applyStyles(EnumSet.of(Style.BOLD, Style. ITALIC));
 ```
 
It is a good practice to accept the interface 'Set' instead of the implementation 'EnumSet'.

#### Item 37: Use EnumMap instead of ordinal indexing
Use EnumMap to store data by a certain enum. You can still use Enum’s ordinal as an index and store it but it’s a bad practice.  Prefer to use EnumMap to do that, So we can have a Enum as Key.

#### Item 38: Emulate Extensible enums with interface

Enums types can not extend another enum types.

``` 
public interface Operation{
		double apply(double x, double y);
	}
	public enum BasicOperation implements Operation{
		PLUS("+"){
			public double apply(double x, double y) {return x + y}
		},
		MINUS("-"){...},TIMES("*"){...},DIVIDE("/"){...};

		private final String symbol;
		BasicOperation(String symbol){
			this.symbol = symbol;
		}
		@Override
		public String toString(){ return symbol; }
	}
``` 

BasicOperation is not extensible, but the interface type Operation is, and it is the one used to represent operations in APIs.
``` 
public enum ExtendedOperation implements Operation{
		EXP("^"){
			public double apply(double x, double y) {return Math.pow(x,y)}
		}
		REMAINDER("%"){
			public double apply(double x, double y) {return x % y}
		}

		private final String symbol;
		ExtendedOperation(String symbol){
			this.symbol = symbol;
		}
		@Override
		public String toString(){ return symbol; }
	}
```

#### Item 39: Prefer annotations to naming pattern
All programmers should use the predefined annotation types that java provodes. 

#### Item 40: Consistently use the override annotation
Use the override annotation on every method declaration that you believe to override a superclass declaration, With one exception. In concreate classes, you need not annotate methods that you believe to override abstract method declarations (though its not harmful to do so).

#### Item 41: Use marker interfaces to define types
A marker interface is an interface that contains no method declaration. It only "marks" a class that implements this interface. One common example in the JDK is Serializable. Using marker interface results in compile type checking.

### Chapter:7 Lambdas and streams

#### Item 42 : Prefer Lambdas to anonymous classes
Lambdas are the best way to represent function objects. As a rule of thumb, lambdas need to be short to be readable. Three lines seems to be a reasonable limit. Also, the lambdas need to be self-explanatory since it lacks name or documentation. Always think in terms of readability.

#### Item 43: Prefer methods references to lambdas
Method references often provide a more succinct alternative to lambdas. Where method references are shorter and clearer, use them; where they are not, stick with lambdas.

| Method ref type     | Example                 | Lambda equivalent                               | 
| ------------------- | ------------------------| ------------------------------------------------| 
|Static	              | Integer::parseInt       | str -> Integer.parseInt(str) 			  |
|Bound	              | Instant.now()::isAfter  | Instant then = Instant.now(); t->then.isAfter(t)|
|Unbound              |	String::toLowerCase     | str -> str.toLowerCase()			  |
|Class Constructor    |	TreeMap<K,V>::new       | () -> new TreeMap<K,V>			  |
|Array Constructor    |	int[]::new              | len -> new int[len]      			  |

#### Item 44: Favor the use of standard functional interface

#### Item 45: Use Streams Judiciously
Some tasks are best accomplished with streams, and others with iteration. Many tasks are best accomplished by combining the two approaches. There are no hard and fast rules for choosing which approach to use for a task, but there are some useful heuristics. In many cases, it will be clear which approach to use; in some cases, it won’t. If you’re not sure whether a task is better served by streams or iteration, try both and see which works better.

#### Item 46: Prefer side-effect-free functions in streams
The essence of programming stream pipelines is side-effect free function objects. This applies to all of the many function objects passed to streams and related objects. The terminal operation forEach should only be used to report the result of a computation performed by a stream, not to perform the computation. 
In order to use streams properly, you have to know about collectors. The most important collector factories are toList, toSet, toMap, groupingBy, and joining

#### Item 47 Prefer Collection to Stream as a return type
When writing a method that returns a sequence of elements, remember that some of your users may want to process them as a stream while others may want to iterate over them. Try to accommodate both groups. If it’s feasible to return a collection, do so. 
If you already have the elements in a collection or the number of elements in the sequence is small enough to justify creating a new one, return a standard collection such as ArrayList. Otherwise, consider implementing a custom collection as we did for the power set. 
If it isn’t feasible to return a collection, return a stream or iterable, whichever seems more natural. If, in a future Java release, the Stream interface declaration is modified to extend Iterable, then you should feel free to return streams because they will allow for both stream processing and iteration.

#### Item 48: Use caution when making streams parallel
Do not even attempt to parallelize a stream pipeline unless you have good reason to believe that it will preserve the correctness of the computation and increase its speed. The cost of inappropriately parallelizing a stream can be a program failure or performance disaster.

If you believe that parallelism may be justified, ensure that your code remains correct when run in parallel, and do careful performance measurements under realistic 257 conditions. If your code remains correct and these experiments bear out your suspicion of increased performance, then and only then parallelize the stream in production code.


### Chapter: 8 Methods

#### Item 49 : Check for parameters for validity
When writing a public or protected method, you should begin by checking that the parameters are not enforcing the restrictions that you set. You should also document what kind of exception you will throw if a parameter enforce t
hose restrictions. The Objects.requireNonNull method should be used for nullability checks.

#### Item 50: Make defensive copies when needed
If a class has mutable components that it gets from or returns to any clients, the class must defensively copy these components. If the cost of the copy would be prohibitive and the class trusts its clients not to modify the components inappropriately, then the defensive copy may be replaced by documentation outlining the client’s responsibility not to modify the affected components.

For example,
```
public Period(Date start, Date end) {
		this.start = new Date(start.getTime());
		this.end = new Date(end.getTime());
		if(start.compare(end) > 0) {
			throw new IllegalArgumentException(start + " after " + end );
		}
	}
```

#### Item 51: Design method signatures carefully
1.	Choose method names carefully
2.	Don’t go overboard in providing convenience methods.
3.	Avoid long parameter lists. Use helper class and Builder pattern
4.	For parameter types, favor interfaces over classes.
5.	Prefer two-element enum types to Boolean parameter

#### Item 52: Use overloading judiciously
Avoid confusing use of overloading, Refer the below example,

```
// Broken! - What does this program print?
public class CollectionClassifier {
	public static String classify(Set<?> s) {
		return "Set";
	}
	public static String classify(List<?> lst) {
		return "List";
	}
	public static String classify(Collection<?> c) {
		return "Unknown Collection";
	}
	public static void main(String[] args) {
		Collection<?>[] collections = {
			new HashSet<String>(),
			new ArrayList<BigInteger>(),
			new HashMap<String, String>().values()
		};
	for (Collection<?> c : collections)
		System.out.println(classify(c)); // Returns "Unknown Collection" 3 times
	}
}
```
A safe, conservative policy is never to export two overloading with the same number of parameters. You can always give methods different names instead of overloading them.
For example, String exports two overloaded methods, contentEquals(StringBuffer) and contentEquals(CharSequence). StringBuffer is the sub class of CharSequence. So caller may not know which method will be invoked, but its of no consequences so long as they behave identically.  
String exports two overloaded factory methods, valueOf(Object) and valueOf(char[]), that do completely different things when passed the same object reference. There is no real justification for this and it violates this item.

#### Item 53: Use varargs judiciously
Varargs are invaluable when you need to define methods with a variable number of arguments. Precede the varargs with any required parameters and be aware of the performance consequences of using varargs(Array allocation and initialization).
```
// The right way to use varargs to pass one or more arguments
	static int min(int firstArg, int... remainingArgs) {
		int min = firstArg;
		for (int arg : remainingArgs)
			if (arg < min)
				min = arg;
			return min;
	}
```

#### Item 54: Return empty collections or arrays not nulls
Never return null in place of an empty array or collection. It makes your API more difficult to use more prone to error, it has no performance advantages.

#### Item 55 : Return Optionals judiciously
You should declare a method to return Optional if it might not be able to return a result and clients will have to perform special processing if no result is returned. You should never use an optional of a boxed primitive. For performance critical methods, it may be better to return a null or throw an exception.

#### Item 56: Write doc comments for all exposed API elements
Documentation summary are the best, more effective way to document your API. Their use should be considered mandatory for all the exported elements.  Adopt a consistent style that adheres to standard conventions. 

### Chapter 9: General programming

#### Item 57: Minimize the scope of the local variable
Declare local variable where it is first used.
Prefer for loops to while loops
Keep methods small and focused. Don’t combine two many activities in the same method.

#### Item 58: Prefer For Each loops to Traditional For Loops
For Loop provides compelling advantages over the traditional for loop in clarity, flexibility and bug prevention, with no performance penalty. Use for-each loops in preference to for loops wherever you can.
There are situations where you can’t use for each loop.
1.Destructive Filtering - If you need to traverse a collection and remove selected elements, then you need to use an explicit iterator so that you can call its remove method. You can avoid explicit traversal by using Collections’s removeIf method in java 8
2.Transforming - If you need to traverse a list or array and replace some or all of the values of its elements, you need the list iterator or array index in order to replace the value of the element.
3.Parallel Iteration – If you need to traverse multiple collection in parallel, then you need explicit control over the iterator or index variable. So, the all iterators or index variables can be advanced in lockstep.

#### Item 59: Know and use Libraries
By using a standard library,
1.You take advantage of the knowledge of the experts who wrote it and the experience of who used it before you.
2.Don’t have to waste your time writing ad hoc solutions to problems that are only marginally related to your work.
3.Their performance tends to improve over time
4.Your code will be easily readable, maintainable, and reusable.

Numerous features are added to the libraries in every major release, and it pays to keep abreast of these additions
Every programmer should be familiar with:
java.lang
java.util
java.io
java.util.concurrent

#### Item 60: Avoid float and double if exact answers are required
The float and double types are particularly ill-suited for monetary calculations, instead use BigDecimal, int and long. 
Don’t use float and double for any calculations that require an exact answer.  If the quantities don’t exceed 9 decimal digits, then you can use int, if they don’t exceed 18 digits, you can long. If it exceeds more than 18 digits, then you can use BigDecimal.

#### Item 61: Prefer primitive types to boxed primitives
Primitives: int, double, boolean
Boxed Primitives: Integer, Double, Boolean
Differences:
1.	Two boxed primitives could have the same value but different identity.
2.	Boxed primitives have one nonfunctional value: null.
3.	Primitives are more space and time efficient.

Applying the == operator to boxed primitives is almost always wrong
Performance can be perturbed when boxing primitives values due to the creation of unnecessary objects.
When you must use boxed primitives:
1.	As elements, keys and values in Collections
2.	As type parameters in parametrized types
3.	When making reflective invocations
In other cases prefer primitives.

#### Item 62: Avoid Strings where other types are more appropriate
Strings are more cumbersome than other types.
Strings are less flexible than other types.
String are slower than other types.
Strings are more error-prone than other types.
Strings are poor substitutes for other value types.
Strings are poor substitutes for enum types.
Strings are poor substitutes for aggregate types.
Strings are poor substitutes for capabilities.

Use String to represent text only. 
Avoid the natural tendency to represent objects as strings when better data types exist or can be written. Used inappropriately, strings are more cumbersome, less flexible, slower and more error prone than any other types.

#### Item 63: Beware the performance of string concatenation 
Using the string concatenation operator repeatedly to concatenate n strings requires time quadratic in n.
To achieve acceptable performance, use StringBuilder in place of String.

#### Item 64: Refer to objects by their interfaces.
If appropriate interface types exist, then parameters, return values, variables, and fields should all be declared using interface types.
If there is not an appropriate interface we can refer to the object by a class. Like:
1.Value classes: String, BigDecimal...
2.Framework classes
3.Classes that extend the interface functionality with extra methods.

#### Item 65: Prefer interfaces to reflection
Reflection is a powerful facility but has many disadvantages. When you need to work with classes unknown at compile time, try to only use it to instantiate object and then access them by using an interface of superclass known at compile time.

#### Item 66: Use native methods judiciously
It's rare that you will need to use native methods to improve performances. If it's needed to access native libraries use as little native code as possible. A single bug in the native code can corrupt your entire application.

#### Item 67: Optimize judiciously
Do not strive to write fast programs—strive to write good ones; speed will follow. But do think about performance while you’re designing systems, especially while you’re designing APIs, wire-level
protocols, and persistent data formats. When you’ve finished building the system, measure its performance. If it’s fast enough, you’re done. If not, locate the source of the problem with the aid of a profiler and go to work optimizing the relevant parts of the system. The first step is to examine your choice of algorithms: no amount of low-level optimization can make up for a poor choice of algorithm. Repeat this process as necessary, measuring the performance after every change, until you’re satisfied

#### Item 68: Adhere to generally accepted naming conventions

| Identifier Type     | Examples                                      |
| -------------       | -------------------------------------------   |
| Package             | org.junit.jupiter, com.google.common.collect  |
| Class or Interface  | Stream, FutureTask, LinkedHashMap, HttpServlet|
| Method or Field     | remove, groupBy                               |
| Constant Field      | MIN_VALUE, NEGATIVE_INFINITY                  |
| Local Variable      | i, denom, houseNum                            |
| Type Parameter      | T, E, K, V, X, R, U, V, T1, T2                |


### Chapter:10 Exceptions

#### Item 69: Use exceptions only for exceptional conditions
Exceptions are designed for exceptional conditions. Don’t use them for ordinary control flow and don’t write API’s that force others to do so. 

#### Item: 70: Use checked exception for recoverable conditions and runtime exceptions for programming errors.
Use checked exceptions for conditions from which the caller can reasonably be expected to recover. 
Use runtime exceptions to indicate programming errors. 
By convention, errors are only used by the JVM to indicate conditions that make execution impossible.Therefore, all the unchecked throwables you implement must be a subclass of RuntimeException.

#### Item : 71 Avoid unnecessary use of checked exceptions
When used sparingly, checked exceptions can increase the readability of programs.  When overused, they make API’s painful to use. If caller won’t be able to recover it from failures, then throw unchecked exceptions. If recovery may be possible and you want to force the caller to handle exceptional conditions, first consider returning an optional only if its provides sufficient information or else throw a checked exception. 

#### Item: 72 Favor the use of standard exceptions:
When appropriate, use the standard exception instead of creating a new one.  Here are the list of common exceptions.

Exception	                 Occasion for Use
IllegalArgumentException	 Non-null parameter value is inappropriate
IllegalStateException	         Object state is inappropriate for method invocation
NullPointerException	         Parameter value is null where prohibited
IndexOutOfBoundsException	 Index parameter value is out of range
ConcurrentModificationException	 Concurrent modification of an object has been detected where it is prohibited
UnsupportedOperationException	 Object does not support method
 
#### Item 73: Throw exceptions that are appropriate to the abstraction
Higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstraction. Make sure to use chaining in order to the underlying cause of failure.
For example,

```
  	 try {
               …..
         }  catch(LowerLevelException cause) {
         throw new HigherLevelException(cause) ;     
       }

```
       
Then create a chaining aware constructor in the HigherLevelException like below,

``` 
class HigherLevelException extends Exception {
    public HigherLevelException(Throwable cause){
         super(cause);
     }
}
```

#### Item: 74 Document all exceptions thrown by each method
Always declare checked exceptions individually, and document precisely the conditions under which one is thrown
Use the Javadoc @throws tag to document each exception that a method thrown and do not use it for unchecked exceptions.

#### Item: 75 Include failure-capture information in detail message
To capture a failure, the detail message of an exception should contain the information of all parameters and fields that contributed to the exception. Do not include passwords and encryption keys in the message.

```
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
//Generate a detail message that captures the failure.	
super(String.format("Lower bound : %d, Upper bound : %d, Index : %d", lowerBound, upperBound, index));
	
	//Save failure information for programmatic access.
	this.lowerBound = lowerBound;
	this.upperBound = upperBound;
	this.index = index;
}
```

#### Item: 76 Strive for failure atomicity
Any generated exceptions that is part of a methods specification should leave the object in the same it was in prior to the method invocation. 
For example, String has “substring” method and it does not change anything and return a new object. If this throws an exception, then we don’t get a new object but the original string was never changed. There is no code inside substring() that modifies the original string. So it failure atomicity.   

#### Item: 77 Don’t ignore the exceptions
An empty catch block defeats the purpose of exceptions, which is to force you to handle the exceptional conditions. If you choose to ignore an exception, the catch block should contain a comment explaining why it is appropriate to do so and the variable should be named ignored.

``` 
try{
...........
   }catch(TimeoutException |ExecutionException ignored){
   //use default: minimal coding is required and required
}

```
