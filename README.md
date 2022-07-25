# Realtek RTL8811C/RTL8821CU USB wifi adapter driver version 5.4.1 for Linux 4.4.x up to 5.x and Android

This is a Makefile configuration of the 8821CU drivers that you can find on other repos. I successfully built and installed this driver for my Atom N450 netbook running Android-x86 (Android 8.1) and an RTL8821CU wireless AC + BT Adapter. 

This repo contains a few updates for the Makefile needed per the RTL SDK Spec (they were all missing from the Makefile). 


### Pre-Requisites 

You will need a few things before you begin
1. Your Android Source and/or compiled Kernel. It took my machine (Atom N450 based) 8 hours to compile the Android-x86 kernel from sources. 

The compiled Kernel is necessary as it contains the headers needed for your version of Android to build the driver successfully.

2. The appropriate cross-compiler/toolchain for your version of Android, and Processor Architecture (i386, i686, x86_64, arm, etc...)

3. Your USB Device's vid:pid if attempting Option 2 to install the driver.

4. NO existing device on your Android system 'wlan0' (VIRT_WIFI, etc.). The Realtek driver will not initialize and bind correctly to the OS if you already have wlan0,

I was using a Linux Mint 20.3 machine to build this driver and had numerous tools gcc, etc. installed to build already. You may need to get these yourself (via apt or other sources).


### First, clone this repository
```
mkdir -p ~/build
cd ~/build
git clone https://github.com/herb2k/8821CU-android.git
```

### Define the Cross-Compiler
Under the section ifeq ($(CONFIG_PLATFORM_ANDROID_X86), y)

Update CROSS_COMPILe:= with the location (actually prefix) of the cross-compiler for your version of Android
```
CROSS_COMPILE := <location>/android-x86/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin/x86_64-linux-android-
```

### Specify the Kernel Location
Update KSRC with the location of the *compiled* kernel for your version of Android
```
KSRC := <location>/android-x86/out/target/product/x86/obj/kernel
```
### Manually Specify the Architecture
My device was 32bit (other possible values - x86_64, arm64, etc...)
```
ARCH := i386 
```
## Build the Driver
```
make build
```

## Test the driver on your device
Copy the driver (8821cu.ko) to your device - using adb or other file-transfer method.
Open a terminal or `adb shell` on your device
Nagivate to the directory where you put 8821cu.ko

```
insmod 8821cu.ko
```

### Check the driver loaded

Checking dmesg will show a number of lines with RTW. Some will be errors but that is normal until you associate to an AP.

```
dmesg | grep RTW
```

### Permanently install the driver
1. Create a folder called 8821cu and copy the 8821cu.ko file to that folder.
2. Copy the folder containing the 8821cu.ko file to a location on your device which is persistent and available at boot. You may need to rebuild your kernel or bootimg if using a real Android device (not required on Andoroid-x86)

I was able to use '/lib/modules/4.19.195-android-x86-22805-g0676905e8791/kernel/drivers/net/wireless/realtek/8821cu'

### Load Driver at Boot (OPTION 1)
Android does not load loadable kernel objects by default after running insmod.

On Android-x86 you can create `init.sh` in `/vendor/etc/` which is called during boot.

This method loads the driver at the END of the boot process, so there is a small delay after starting Android before wireless is available. 

Add these two lines
```
insmod /lib/modules/4.19.195-android-x86-22805-g0676905e8791/kernel/drivers/net/wireless/realtek/rtl8821c/8821cu.ko
# Force start the wifi service
svc wifi on
```

### Load Driver at Boot (OPTION 2)

This option loads the driver much earlier in the boot process, and there is no delay to activating wireless on start up.

NOTE: You will need your USB device vid and pid - eg: '0BDA:C820'. You can get this by running `lsusb`

2. Update modules.order: Add `kernel/drivers/net/wireless/realtek/rtl8821c/8821cu.ko` to `/system/lib/modules/4.19.195-android-x86-22805-g0676905e8791/modules.order`

3. Update modules.alias: Add the following strings (replacing the characters following "v" and "p" with your device's vid and pid.
```
 alias usb:v0BDApC820d*dc*dsc*dp*ic*isc*ip*in* rtl8821c
 alias usb:v0BDApC820d*dc*dsc*dp*ic*isc*ip*in* rtl8821cu
```
To file `/system/lib/modules/4.19.195-android-x86-22805-g0676905e8791/modules.alias`

4. Update modules.dep: Add `kernel/drivers/net/wireless/realtek/rtl8821c/8821cu.ko: kernel/net/mac80211/mac80211.ko kernel/net/wireless/cfg80211.ko` to `/system/lib/modules/4.19.195-android-x86-22805-g0676905e8791/modules.dep`
