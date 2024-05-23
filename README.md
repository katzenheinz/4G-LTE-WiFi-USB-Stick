# 4G-LTE-WiFi-USB-Stick
Archive/collection of information for Chinese (Aliexpress) Wifi/LTE/4G/USB sticks, which can be used as 8€ SBC.

Where to buy?
```
https://de.aliexpress.com/item/1005006394474912.html | white model | UZ801 V3.2
https://de.aliexpress.com/item/1005006394474912.html | black model | UZ801 V3.2
https://de.aliexpress.com/item/1005006189041594.html | black model | UZ801 V3.0
```
Resources
```
https://github.com/OpenStick
https://www.kancloud.cn/handsomehacker/openstick/2637565
https://wvthoog.nl/openstick/
https://wiki.postmarketos.org/wiki/Zhihe_series_LTE_dongles_(generic-zhihe)
```

How to compile kernel 5.15 - Guide/Summary
- Assumptions: Working Linux Kernel Build System
```
export CROSS_COMPILE=aarch64-linux-gnu-
export ARCH=arm64
make msm8916_defconfig
make menuconfig
make -j2
fakeroot make-kpkg --initrd --cross-compile aarch64-linux-gnu- --arch arm64 kernel_image kernel_headers
cd ..
cp linux/arch/arm64/boot/Image.gz
cp linux/arch/arm64/boot/dts/qcom/msm8916-handsome-openstick-uz801.dtb .
cat Image.gz msm8916-handsome-openstick-uz801.dtb > kernel-dtb
// Install .deb Packages on devices, copy created initrd.img to working dir
mkbootimg ---base 0x80000000 --kernel_offset 0x00080000 --ramdisk_offset 0x02000000 ---tags_offset 0x01e00000 --pagesize 2048 --second_offset 0x00f00000 --ramdisk initrd.img --cmdline “earlycon root=PARTUUID=a7ab80e8-e9d1-e8cd-f157-93f69b1d141e console=ttyMSM0,115200 no_framebuffer=true rw” --kernel kernel-dtb -o boot.img
```


From https://www.kancloud.cn/handsomehacker/openstick/2637565 (translated using deepl.com):

```
About OpenStick
Here is how to customize your own kernel using linux. The recommended distribution is Ubuntu 20.04.

Required packages
The naming of packages may vary from one distribution to another, here is an example of the naming for Ubuntu 20.04.

binfmt-support
qemu-user-static
gcc-10-aarch64-linux-gnu
kernel-package
fakeroot
simg2img
img2simg
mkbootimg
bison
Generate kernel debian package
Clone the linux kernel, here choose a depth of 1 to reduce the size.

$ git clone https://github.com/OpenStick/linux --depth=1
Start compiling

$ export CROSS_COMPILE=aarch64-linux-gnu-
$ export ARCH=arm64
$ make msm8916_defconfig
$ make menuconfig
$ make -j16
Generate the package in debian format

$ fakeroot make-kpkg --initrd --cross-compile aarch64-linux-gnu- --arch arm64 kernel_image kernel_headers
After that, take the generated deb package (which will be generated in the upper level of the code directory) and keep it with arch/arm64/boot/Image.gz as a backup
Convert rootfs.img to img format and mount it

$ simg2img rootfs.img root.img
$ sudo mount root.img /mnt
Under the chroot environment, install the deb installer such as linux-image that you generated earlier. Note that you need to remove the boot/initrd.img that was generated after the installation is complete (it is best to uninstall the original linux-image package before installing it).

$ sudo mount --bind /proc /mnt/proc 
$ sudo mount --bind /dev /mnt/dev
$ sudo mount --bind /dev/pts /mnt/dev/pts
$ sudo mount --bind /sys /mnt/sys
$ sudo chroot /mnt
Once the installation is complete, unmount everything and convert the img to simg again

$ img2simg root.img rootfs.img
Generate boot.img
Take the file /boot/initrd.img** after installing the new kernel on the previous filesystem and the Image.gz generated during the kernel compilation phase, and the *.dtb of the corresponding device under arch/arm64/boot/dts/qcom/.
Attach devicetree(dtb) to image.gz

 cat Image.gz dtb > kernel-dtb
Generate boot.img

$ mkbootimg \     
        ---base 0x80000000 \
        --kernel_offset 0x00080000 \
        --ramdisk_offset 0x02000000 \
        ---tags_offset 0x01e00000 \
        --pagesize 2048 \
        --second_offset 0x00f00000 \
        --ramdisk initrd.img \
        --cmdline “earlycon root=PARTUUID=a7ab80e8-e9d1-e8cd-f157-93f69b1d141e console=ttyMSM0,115200 no_framebuffer=true rw”\
        --kernel kernel-dtb -o boot.img

After that, you can flush the two img files into the corresponding partitions.
```

Copied from https://zebra.ddscentral.org/pub/downloads/openstick/firmware/uz801_v30/README

```



OpenStick Build for board UZ801 V3.0 (may also work on earlier revisions, try at your own risk)

Homepage: https://github.com/OpenStick/
Initial flashing Instructions: https://extrowerk.com/2022/07/31/OpenStick/

Directories:
* boot     - contains the replacement for aboot.img from base.zip with reset key fixed. Reset key will correctly boot to fastboot.
* kernel   - contains an up-to-date kernel build with DTB for UZ801.
* firmware - contains modem and WiFi firmwares

Disclaimer:
Be aware that by following these instructions, you will overwrite your stick's partition table. There's no easy way to revert back to factory firmware after this !
The author is not responsible for any breakage of devices caused by following these instructions.
These instructions are meant for a Linux-based OS. The user is asumed to have at least some basic Linux knowledge.
All commands should be run as the "root" user or prefixed with "sudo".

Instructions:
1. Download debian.zip and base.zip from https://github.com/OpenStick/OpenStick/releases
2. Check if ADB is enabled. If "adb devices" does not show any devices, open http://192.168.100.1/usbdebug.html to enable ADB.
3. Do initial flash as instructed in the URL above. In case the URL above is down, here's a short version. Basically, it boils down to four basic steps:
   * Unzip both debian.zip and base.zip
   * Donwload aboot.img from boot/ folder and copy it to base/
   * Reboot the stick to fastboot mode: adb reboot-bootloader
   * cd to base/ and run ./flash.sh
   * cd to debian/ and run ./flash.sh
   If all goes well, your device should reboot to Debian. SSH is on 192.168.68.1. Username is "user", password is "1". ADB should work as well.
4. Using SSH or ADB, copy the *.deb files from kernel/ folder. Also copy one of the boot.img files. Currently, two files are available:
   boot.img-5.15.0-orbital+        - standard version
   boot.img-5.15.0-orbital+-hddled - version with green LED set to disk activity.
   Upload whichever you choose and rename it to boot.img
5. SSH or "adb shell" to your device.
6. Install the two *.deb files using dpkg:
   dpkg -i linux-image-5.15.0-orbital+_5.15.0-orbital+-10.00.Custom_arm64.deb 
   dpkg -i linux-headers-5.15.0-orbital+_5.15.0-orbital+-10.00.Custom_arm64.deb
7. Flash the boot img of your choice
   dd if=boot.img of=/dev/mmcblk0p12
8. Reboot
   If everything went well, you should see the red LED blinking. Depending on your version of boot.img, the green LED will may blink as the device boots.
9. Install firmware for Modem and WiFi
   * Download firmware files from the modem_wifi/ folder.
   * Using SSH or ADB, upload WCNSS_qcom_wlan_nv.bin to /lib/firmware/wlan/prima/
   * Upload uz801_v30_modem.bin to your stick (location does not matter).
   * SSH or "adb shell" to your stick.
   * Mount uz801_modem.bin: mount -o loop -t vfat uz801_v30_modem.bin /mnt
   * cd /mnt/image/
   * cp modem.*, wcnss.* mba.mbn /lib/firmware/  
   * In modem_pr/mcfg/configs/mcfg_sw/generic/, look for the appropriate mcfg_sw.mbn for your region (and carrier if applicable). Copy it to /lib/firmware.
   * Reboot your device.
10. All basic functions should now work. Configure the device for your chosen use case.

Additional notes:
By default, USB is configured as Ethernet and WiFi is turned off. You can configure WiFi as client using "nmtui" command.
For AP mode, I recommend using hostapd as NetworkManager'ss AP support is really ancient. You will be stuck in "g" mode.
For modem, you can either try the included "modem" configuration or create your own using ModemManager's "mmcli" tool if the included configuration doesn't work for you. 
If you want router functionality, you will also need to configure your firewall, DHCP, DNS, etc.
To configure USB mode, check "mobian" files in /usr/bin/. But make sure to have a backup plan in case you break ADB ! If you lock yourself out, you'll need to re-flash rootfs and boot.img from fastboot mode. 

How to do the above mentioned configuration is outside of scope of these instructions. Google is your friend.

If you need help, I'm available by email: dds[alpha]ddscentral[dot]org or ddscentral[alpha]raspis[dot]eu

DDS Central
```





