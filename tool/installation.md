#<img src="./figures/logo.png" style="width: 300px;" onclick="window.location='https://haiyang-sun.github.io/tool/intro.html'"/>  
--
#Installation Guide
##Prerequisites
1. Download latest Android SDK, and add the following tools into your _PATH_ environment variable:
	- adb
	- emulator (for emulator)
	- fastboot (for building Nexus 5 image)

2. Java version 7

3. Clone and build the repo at [ADRENALIN-RV](https://github.com/Haiyang-Sun/ADRENALIN-RV)

	~~~bash
	git clone https://github.com/Haiyang-Sun/ADRENALIN-RV.git
	cd ADRENALIN-RV
	export JAVA_HOME="PATH_TO_JAVA_7"
	ant
	~~~

4. Download the Nexus 5 image file [here](http://195.176.181.79/ADRENALIN-RV/demo/nexus.zip) and flush the Android image with dynamic weaving with the following bash command:

	~~~bash
	adb reboot-bootloader;
	fastboot -w update nexus.zip
	~~~

	Wait until the installation finish and the system reboots (You will see Android Logo after reboot) and then run in root mode:
	~~~bash
	adb root
	~~~

 
4. Download the intel emualtor image [here](http://195.176.181.79/ADRENALIN-RV/demo/artifacts-intel.tar.gz). It includes files as below:
	- kernel-qemu
	- cache.img         
	- ramdisk.img       
	- system.img        
	- userdata.img
	- sdcard.img        
2. You can start the emulator with the following bash command:

	~~~bash
	emulator64-x86 -sdcard ../sdcard.img -sysdir ./ -system ./system.img -ramdisk ./ramdisk.img -data ./userdata.img -kernel ./kernel-qemu -memory 1024
	~~~

##Run the tool
1. Go to the root directory of [ADRENALIN-RV](https://github.com/Haiyang-Sun/ADRENALIN-RV.git)
2. Compile using java 7

	~~~bash
	export JAVA_HOME="PATH_TO_JAVA_7"
	ant
	~~~
	
3. Start instrumentation server using 

	~~~bash
	./start-instrumentation.sh [config_file] #or
	./start-instrumentation.sh disl.config.sample #using the sample config
	~~~

4. Install the target app "com.fdhgkjhrtjkjbx.model.apk"

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
