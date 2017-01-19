#<img src="./figures/logo.png" style="width: 300px;" onclick="window.location='https://haiyang-sun.github.io/tool/intro.html'"/>  
--
#Installation Guide
##Try the VMWare Image
Download the VM image [here](http://195.176.181.79/ADRENALIN-RV/resources/vm.ova), which is Ubuntu (16.04.1 LTS). It contains the script to run the emulator and scripts needed to run the [information leak analysis](https://haiyang-sun.github.io/tool/dataleak-uc.html).
###Login
To login to the VM, use the login user name "user" and the password "password".
###Run the example
All the related stuff is in $HOME/tool. 

``` bash
##start the emulator and instrumentation server here
./startTool.sh
#wait until the emulator is ready (The main UI menu shows up)
##this process takes 1 min+ for the emulator to load and weave the bytecodes

#the malware sample application is already installed
##in case the emulator reset the installed app
./installSampleApk.sh to install the apk

#use monkey to test it
./runMonkey.sh

#show result from the log
./show_result.sh

```

##Installation from scratch
###Prerequisites
For your convenience, we provide these pacakges [here](http://195.176.181.79/ADRENALIN-RV/resources/tool.tgz), including

- instrumentation server
- arm emulator images
- a sample test case with script to run
- necessary android tools (binaries for Ubuntu 14 or above, for other OSes, you have to download Android SDK on your own) with
	- adb 
	- emulator 
	- fastboot (for building Nexus 5 image)

Or you have to prepare the following on your own:

1. Download latest Android SDK, and add the following tools into your _PATH_ environment variable:
	- adb
	- emulator (for emulator)
	- fastboot (for building Nexus 5 image)

2. Java version 8 or above (to run the instrumentation server)
3. The instrumentation server [here](http://195.176.181.79/ADRENALIN-RV/resources/disl.tgz)


###For Nexus 5
1. Download the image file [here](http://195.176.181.79/ADRENALIN-RV/resources/nexus.zip)

2. Flush the Android image with dynamic weaving with the following bash command:

	~~~bash
	adb reboot-bootloader;
	fastboot -w update PATH_TO_IMAGE_FILE
	~~~

Wait until the installation finish and the system reboots (You will see Android Logo after reboot).
 
###For Android arm-emulator
1. The images file are included in the provided all-in-one package mentioned [above](http://195.176.181.79/ADRENALIN-RV/resources/tool.tgz). Or you can also download the emualtor images alone [here](http://195.176.181.79/ADRENALIN-RV/resources/arm-emu.tgz). It includes files as below:
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
1. get our latest instrumentation server and workspace from [ADRENALIN-RV](https://github.com/Haiyang-Sun/ADRENALIN-RV.git)
2. compile using java 7

	~~~bash
	export JAVA_HOME="PATH_TO_JAVA_7"
	ant
	~~~
3. start instrumentation server using 

	~~~bash
	./start-instrumentation.sh [config_file]
	./start-instrumentation.sh disl.config.sample #using the sample config
	~~~

4. install the target app "com.fdhgkjhrtjkjbx.model.apk"

	~~~bash
	adb install com.fdhgkjhrtjkjbx.model.apk
	~~~

	
7. Use [monkey](http://antoine-merle.com/using-monkey-tool/) to dynamically test your application. For example, below bash script will use monkey to launch an app and inject some random test events.

	~~~bash
	adb shell monkey -p com.myapp -c android.intent.category.LAUNCHER 1
	adb shell monkey -p com.myapp --throttle 500 -v 1000
	~~~
8. Dumpping log via Android default [Logging](https://developer.android.com/reference/android/util/Log.html). You can fetch the log and filter your result by

	~~~bash
	adb logcat | grep YOUR_TAG
	~~~
	e.g., use the following scripts you will get the output of our [information leak analysis](https://haiyang-sun.github.io/tool/dataleak-uc.html).
	
	~~~bash
	./get-violations.sh
	./get-violation-detail.sh [idx]
	~~~
