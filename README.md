# Effective Java - 3rd edition Notes

#### Disclaimer

This is a summary/notes of the book "Effective Java by Joshua Bloch".  The below notes are my personal notes and I hope it's not a copyright infringement. If it is, please contact me in order to remove this file from github.

### Chapter 2: Creating and Destroying Objects

This chapter concerns creating and destroying objects. When and how to
create them, when and how to avoid creating them, how to ensure they are
destroyed in a timely manner, and how to manage any cleanup actions that
must precede their destruction.

#### Item 1: Consider static factory methods instead of constructors
Instead of adding multiple constructors for a class, we can create static factory methods which has advantages and disadvantages.
For example, the constructor BigInteger(int, int, Random), which returns a BigInteger that is probably prime, would have been better expressed as a static factory method named BigInteger.probablePrime.
Advantages:
1.	Unlike constructors, static factor methods they have names
2.	They are not required to create a new object each time
3.	We can return an object of any subtype of their return type
4.	They reduce the verbosity of creating parameterized type instances.
Disadvantages:
1.	The classes without providing public or protected constructors but have only static factory methods can’t be sub classed
2.	They are not readily distinguishable from other static methods
 Here are some of the common static factory methods
1.	from – Type conversion which takes a simple parameter and return an instance of this type.
Date d = Date.from(instant)
2.	Of – An aggregation method that takes multiple parameters and return returns an instance of this type that incorporates them.
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING)
3.	valueOf- A more verbose alternative to from and of. Returns an instance that has the same value its parameters.
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
4.	instance or getInstance – Takes parameters  if any and return an instance of the class.
StackWalker luke = StackWalker.getInstance(options);
5.	create or newInstance – Same as getInstance but each instance that is returned is distinct from all others.
6.	getType – Like getInstance but if the factory method is in different class. 
FileStore fs = Files.getFileStore(path);
7.	newType – Like newInstance but if the factory method is in different class.
BufferedReader br = Files.newBufferedReader(path);
8.	type – A concise alternative to getType and newType,
List<Complaint> litany = Collections.list(legacyLitany);
Often static factories are preferable, so avoid the reflex to provide public constructors without first considering static factories.


```
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
