#Use Case - Information Leak Detection
##Description
Information leaks represent one of the major security concerns of Android applications.

This case we show how we can detectinformation leaks, i.e., unauthorized transmissions of private data (such as device ID, contacts, messages, etc.) to an untrusted server. 

The InformationLeak property can be defined as follows: no private data from a pre-defined set of Android APIs must be sent to a third party. 

We monitor two events of interest: DataSource (i.e., information about private data is obtained) and DataSink (i.e., data is sent over a transmission channel). 

For this use case, we focus in particular on the transmission of the device ID over the network.

##Instrumentation Scope Specification
###Network Monitoring - DataSink
With the ability to instrument Android library as easy as any piece of application code, we achieve network monitoring by just instrumenting the [__libcore.io.IoBridge__](https://android.googlesource.com/platform/libcore/+/jb-mr2-release/luni/src/main/java/libcore/io/IoBridge.java) in the __core.jar__.

More specifically, we monitor IoBridge.bind, IoBridge.conect, IoBridge.send, and IoBridge.recv to get all network access from the application.

Static weaving instrumenting only the appication is not efficient to monitor network related APIs in Android. 

* Finding all callsites of network related work is not trival. There are many classes related to network operations. Java allow application to send/recv packets using Input/OutputStream or Reader/Writer which can be extended by application-defined classes. Not to mention Android also provide many libraries to access network. 
* With reflection, it's even more difficult
* Dynamically loaded classes would never get checked

###TelephonyManager.getDeviceId Monitoring - DataSource
The key is to monitor the invocation of _TelephonyManager.getDeviceId()_. 
We can achieve it either by instrumenting [__android.telephony.TelephonyManager__](https://developer.android.com/reference/android/telephony/TelephonyManager.html) in the __framework.jar__ or instrumenting all the callsites of _TelephonyManager.getDeviceId()_ in the application. To show the convinient and rich API of our specification language, we take the second choice.

###Method Trace
We produce the method trace of the application by monitoring all method entry/exit events. Together with the Data Source/Sink information, we are able to provide a more detailed report.


##Property Specification
###Network
Take the IoBridge.sendto which deals with all network sending as an example:

~~~java
@AfterReturning(marker=BodyMarker.class, 
	scope="IoBridge.sendto(java.io.FileDescriptor,byte[],int,int,int,java.net.InetAddress,int)")
public static void sendto(ArgumentProcessorContext apc, DynamicContext dc){
    Object [] args = apc.getArgs (ArgumentProcessorMode.METHOD_ARGS);
    FileDescriptor fd = (FileDescriptor)args[0];
    byte[] buffer = (byte[])args[1];
    int byteOffset = (int)args[2];
    //int byteCount = (int)args[3];
    int flags  = (int)args[4];
    InetAddress address = (InetAddress)args[5];
    int port = (int)args[6];
    //We can get the return value of the method call as below
    int sentSize = dc.getStackValue (0, int.class); 
    Monitor.onDataSource(new NetworkSendEvent(fd, buffer, byteOffset, sentSize, flags, address, port));
}
~~~

###GetDeviceId
Catching all invocation to _TelephonyManager.getDeviceId_

~~~java
@AfterReturning(marker=BytecodeMarker.class,guard=DeviceIdGuard.class, args = "invokevirtual")
public static void getDeviceId (DynamicContext dc) {
    String deviceId = dc.getStackValue (0, String.class); 
    Monitor.onDataSink(new DeviceIdEvent(deviceId));
}
~~~

guard class __DeviceIdGuard__ used to filter the instrumentation above to take effect only for _TelephonyManager.getDeviceId_

~~~java
public static class DeviceIdGuard{
        @GuardMethod
        public static boolean guard (CallContext callContext) {
            return callContext.getCalleeMethod().contains("android/telephony/TelephonyManager.getDeviceId");
        }
    }
~~~

##Result

We conduct the experiment on a [__malware sample__](https://github.com/ashishb/android-malware/tree/master/Android.Malware.at_plapk.a).

Below is the log we dumped when we detect such violation in information leak. The indent respects the method entry/exit.

We can notice that the code is obfuscated. Even though they separate the use of __DataSource__ and __DataSink__, we still manage to detect it.

```
com/jfdlplapk/g.run    com/jfdlplapk/f.b        com/jfdlplapk/by.e            com/jfdlplapk/by.b            * DataSource *                    <=======            com/jfdlplapk/by.c            com/jfdlplapk/bn.c                com/jfdlplapk/by.b                    com/jfdlplapk/by.a            com/jfdlplapk/by.a            com/jfdlplapk/by.a            com/jfdlplapk/bo.b                com/jfdlplapk/bo.a            com/jfdlplapk/bn.b                com/jfdlplapk/by.b                    com/jfdlplapk/by.a    com/jfdlplapk/au.a        com/jfdlplapk/au.a        * DataSink *					       <=======
```
We detect that the packet sent to 180.97.161.164:80 contains the deviceId we record at DataSource.

##Go Further
The code above is an illustration of a simple case. We are still working and enriching an API list of DataSource and DataSink, so that it can be applied to further cases.
In addition, we are also going to release a property dedicated for fully inspecting __Java Reflection__.


