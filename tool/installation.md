#<img src="./figures/logo.png" style="width: 300px;" onclick="window.location='https://haiyang-sun.github.io/tool/intro.html'"/>  
--
#Installation Guide
##Installation from scratch
###Prerequisites
1. Recommendation running OS: ubuntu 14 or above
2. Download latest Android SDK, and add the following tools into your _PATH_ environment variable:
	- adb
	- emulator (for emulator)
	- fastboot (for building Nexus 5 image)
3. Java version 8 or above (to run the instrumentation server)
4. Download the Android DiSL package [here](http://195.176.181.79/ADRENALIN-RV/resources/disl.tgz)


###For Nexus 5
1. Download the image file [here](http://195.176.181.79/ADRENALIN-RV/resources/nexus.zip)

2. Flush the Android image with dynamic weaving with the following bash command:

	~~~bash
	adb reboot-bootloader;
	fastboot -w update PATH_TO_IMAGE_FILE
	~~~

Wait until the installation finish and the system reboots (You will see Android Logo after reboot).
 
###For Android arm-emulator
1. Download the emualtor images [here](http://195.176.181.79/ADRENALIN-RV/resources/arm-emu.tgz). It includes files as below:
	- kernel-qemu-armv7
	- cache.img         
	- ramdisk.img       
	- system.img        
	- userdata.img
	- sdcard.img        
2. You can start the emulator with the following bash command:

	~~~bash
	emulator -sdcard ../sdcard.img -sysdir ./ -system ./system.img -ramdisk ./ramdisk.img -data ./userdata.img -kernel ./kernel-qemu-armv7 -memory 1024
	~~~

###Start Testing
1. Write your properties in DiSL code and compile them into DiSLClasses

	~~~
	javac -cp PATH/disl-server.jar:OTHER_LIB YOUR_MONITOR_CODE 
	~~~
2. Specify the _Scope Specs_ for the proxy service on Android by updating the bytecode list to be sent to the instrumentation server

	- A sample can be found [here](https://haiyang-sun.github.io/tool/resources/scope_specs.html) 
3. Upload the _Scope Specs_ to Android emulator/device and reboot to make it take effects

	~~~bash
	#In Nexus case, run `adb root` first to make sure the adb is ran is root mode
	adb root
	adb push SCOPE_SPECS_FILE /data/data/svm.prop
	adb reboot
	~~~
4. Prepare the RV specification at the instrumentation server
	- A sample can be found [here](https://haiyang-sun.github.io/tool/resources/rv_specs.xml)
5. Starting the instrumentation server with the following command:

	~~~bash
	#use the path of rv specs for RV_SPECS_FILE 
	#package your monitor's compiled Java classes into a jar file and provide it as YOUR_MONITOR_JAR
	# emulator
	./start-instrumentation.sh RV_SPECS_FILE YOUR_MONITOR_JAR
	# nexus
	./start-instrumentation-usb.sh RV_SPECS_FILE YOUR_MONITOR_JAR
	~~~
6. The emulator/device will be ready after several minutes depending on how much instrumentation the analysis requires. Run the target application to test
7. The most convenient way of dumping the result is via Android default [Logging](https://developer.android.com/reference/android/util/Log.html). You can fetch the log and filter your result by

	~~~bash
	adb logcat | grep YOUR_TAG
	~~~
	
##Use the Provided Virtualbox Image
The VirtualBox [image]() is Ubuntu (16.04.1 LTS). It contains the script to run the emulator and scripts needed to run the [information leak analysis](https://haiyang-sun.github.io/tool/dataleak-uc.html).

To login to the VM, use the login user name "user" and the password "password".
