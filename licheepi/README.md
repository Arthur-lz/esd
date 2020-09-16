# licheepi配置

* 系统环境配置


>这是我实验过的系统配置

| 标题        | 内容           | 说明  |
| ------------- |:-------------:| -----:|
|操作系统|manjaro|i3
| cpu      | 64位 |工具链要用64位的 |

* 移植内容
> 写到开发板上的需要的那些东西

| 标题        | 内容           | 说明  |
 | ------------- |:-------------:| -----:|
 |u-boot|用于引导linux内核|
 |linux内核      |  | |
 |buildroot|根文件系统|

---

### 详细移植过程
>一步不差的那种，实验过程的每一步，详细记录。

| 标题        | 内容           | 说明  |
 | ------------- |:-------------| -----|
 |安装make|sudo pacman -S make|
 |进入/opt/目录|cd /opt/||
 |下载工具链|sudo wget https://snapshots.linaro.org/components/toolchain/binaries/7.5-2019.12-rc1/arm-linux-gnueabihf/gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf.tar.xz|在arm-linux-guneabihf目录下有多个不同版本的可供选择，这里我使用的是64位的arm-linux-gnueabihf版本|
 |解压工具链|sudo tar xvf gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf.tar.xz |解压成功后，会在当前文件夹下出现gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf |
 |切换成root|接下来需要操作/etc/profile，所以切到root||
 |设置环境变量|echo "PATH=$PATH:gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf/bin" >> /etc/profile | |
 |设置环境变量|echo "LD_LIBRARY_PATH=/opt/develop/gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf/lib" >> /etc/profile|我这里LD_LIBRARY_PATH是首次定义，所以直接赋值，如果你不是首次使用，这里要把原来的值加上，像这样old=$old:new|
 |退出root|||
 |进入licheepi实验目录|我的是~/esd/licheepi/||
 |下载u-boot|git clone https://github.com/Lichee-Pi/u-boot.git -b v3s-current --depth 1|这里我只clone最新一版|
 |下载licheepi的linux内核|git clone https://github.com/Lichee-Pi/linux.git --depth 1|同样我只下载最后一版|
 |下载buildroot|wget https://buildroot.org/downloads/buildroot-2017.08.tar.gz||
 |解压buildroot|tar -xvf https://buildroot.org/downloads/buildroot-2017.08.tar.gz||
 |||上述完成u-boot, linux内核, buildroot|
|创建u-boot|tar -czf u-boot.tar.gz u-boot/||
|创建linux内核备份|tar -czf linux.tar.gz linux/||
|||创建备份是我的习惯，因为github下载速度太慢了|
|进入linux内核目录|cd linux|准备开始生成linux内核镜像|
|注：|我在首次make的时候出现了yylloc重复定义的错误，根据错误提示是文件scripts/dtc/dtc-lexer.lex.c 和scripts/dtc/dtc-parser.c两个文件中都定义了yylloc变量，我将前者中定义的yylloc注释后不再出现此问题||
||make ARCH=arm licheepi_zero_defconfig CROSS_COMPILE=arm-linux-gnueabihf-||
|make menuconfig|make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig|不做任何修改直接退出|
|make|make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4|如果make成功的话，可以得到镜像zImage和sun8i-v3s-licheepi-zero-dock.dts|
|进入u-boot目录|cd u-boot||
|添加启动命令参数，vim u-boot/include/configs/sun8i.h|#define CONFIG_BOOTCOMMAND   "setenv bootm_boot_mode sec; load mmc 0:1 0x41000000 zImage; load mmc 0:1 0x41800000 sun8i-v3s-licheepi-zero-dock.dtb; bootz 0x41000000 - 0x41800000; 
|添加启动命令参数，vim u-boot/include/configs/sun8i.h|#define CONFIG_BOOTARGS      "console=ttyS0,115200 panic=5 console=tty0 rootwait root=/dev/mmcblk0p2 earlyprintk rw  vt.global_cursor_default=0"||
||make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_defconfig||
||make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_480x272LCD_defconfig||
||make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4|make成功的话，会在当前目录下生成u-boot-sunxi-with-spl.bin|
|python版本问题|tools/binman/binman，这个文件里哈，如果你的电脑上安装的python版本是3.8.5需要修改一下前面的文件将#!/usr/bin/python改为#!/usr/bin/python2|因为python3不识别函数print|
|关于fdt的问题|如果你的电脑上/usr/include/下有fdt.h,libfdt_env.h,libfdt.h，那么需要将其暂时删除，不编译u-boot时再恢复哈，这么做是因为u-boot里也有这三个文件，他们如果同时存在，则会出现fdt64_t类型冲突、一些重复定义以及其他一些错误。||
|进入buildroot目录|cd buildroot||
|make menuconfig|在Tool chain配置以下几项：||
||**Toolchain Type为External toolchain**||
||**Toolchain Path为/opt/gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf**||
||**Toolchain prefix为arm-linux-gnueabihf**||
||**External toolchain gcc version为7.x**||
||**External toolchain kernel heads series为4.10.x**||
||**External toolchain c library 为glibc**||
||**在System Configuration下配置以下三项**||
||**System hostname-----》lizhi_host**||
||**System banner------> Welcome to lizhi host.**||
||**Root password------> your passwd**||
||**Run a getty (login prompt) after boot/TTY Port由原来的console调整为ttyS0, Baudrate由原来的keep kernel default调整为115200**||
||**在Target Options里配置一下**||
||**arget Architecture为Arm(little endian)**|必须配置|
|硬浮点需要配置的|**Enable VFP extension support**||
|硬浮点需要配置的|**Target ABI (EABIhf)**||
|安装依赖程序|flex, bison, patch, cpio|sudo pacman -S flex|
|make|make需要下载，所以打开网，make时间略长|make成功后会在当前目录找到output/images/rootfs.tar|
|安装qemu|我是下载源码安装|在manjaro上直接sudo pacman -S qemu安装后没有qemu可执行文件，只在usr/lib, share目录下有一个库文件，原因不明。源码安装时，需要安装pkg-config, 之后就可以正常执行./configure生成Makefile了，|
