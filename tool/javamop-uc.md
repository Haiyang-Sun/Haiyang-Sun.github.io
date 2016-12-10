#Use Case - JavaMOP on Android
##Description
We choose five well-known JavaMOP properties:

Preperty | Description
------------ | -------------
HasNext | Program should always call hasNext() before next() on an iterator.
SafeEnum | Collection (with an associated enumeration) should not be modified while the enumeration is in use.
SafeSyncMap | Synchronized collection should always be accessed by a synchronized iterator, and the iterator should always be accessed in a synchronized manner.
UnsafeIterator | When the iterator associated with a collection is accessed, the collection should not be updated.
UnsafeMapIterator | WLike UnsafeIterator, with differences     related to the creation of iterators.

##Instrumentation Scope Specification
In this use case, our instrumentation cover the framework.jar which includes major Android framework implementation, the bytecode inside the target application and all dynamically loaded bytecode

##RV Property Specification
We use DiSL as the specification language in our tool, and reuse JavaMOP's monitoring logics.

TODO: put our DiSLCode here

##Result
Property | Joint Point Shadows || Joint Point Executions
