# 4G-LTE-WiFi-USB-Stick
Archive/collection of information for Chinese (Aliexpress) Wifi/LTE/4G/USB sticks, which can be used as 8€ SBC.

Resources
https://github.com/OpenStick
https://www.kancloud.cn/handsomehacker/openstick/2637565


From https://www.kancloud.cn/handsomehacker/openstick/2637565 (translated using deepl.com):

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
