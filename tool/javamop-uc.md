#<img src="./figures/logo.png" style="width: 300px;" onclick="window.location='https://haiyang-sun.github.io/tool/intro.html'"/>  
--
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

##Property Specification
Below we take the _HasNext_ Property as example to show how we define the property to detect violation:

~~~java
@Before(marker = NextInvocationMarker.class)
public static void HasNext_next(ArgumentProcessorContext pc){
		Iterator receiver = (Iterator) pc.getReceiver(ArgumentProcessorMode.CALLSITE_ARGS);
		HasNextRuntimeMonitor.nextEvent(receiver);
}
@After(marker = HasNextInvocationMarker.class)
public static void HasNext_hasnext(ArgumentProcessorContext pc) {
		Iterator receiver = (Iterator) pc.getReceiver(ArgumentProcessorMode.CALLSITE_ARGS);
		HasNextRuntimeMonitor.hasnextEvent(receiver);
}

~~~
The implementation of _HasNextRuntimeMonitor_ is adapted from JavaMOP and can be found [here]().

##Result
<center>
<img src="https://haiyang-sun.github.io/tool/figures/gms1.jpg" style="width: 400px;"/>
<img src="https://haiyang-sun.github.io/tool/figures/gms2.jpg" style="width: 400px;"/>

<img src="https://haiyang-sun.github.io/tool/figures/malware1.jpg" style="width: 400px;"/>
<img src="https://haiyang-sun.github.io/tool/figures/malware2.jpg" style="width: 400px;"/>

<img src="https://haiyang-sun.github.io/tool/figures/coveragetable.png" style="width: 600px;"/>
</center>