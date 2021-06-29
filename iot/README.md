# 嵌入式开发经验汇总
## ARM
### 1.用busybox生成最小文件系统
* 版本1.32.1
* 设置环境变量
> export PATH=$PATH:/home/xxx/gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabi/bin
> export LD_LIBRARY_PATH=/home/xxx/gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabi/lib
> export ARCH=arm
> export CROSS_COMPILE=arm-linux-gnuebai-
* make install后生成的_install文件夹需要初始化，初始化的脚本为init_busybox_install.sh

### 2.内核编译
* 内核版本4.0.5
* 将1中busybox生成的_install文件拷贝到内核根目录下

#### 准备内核交叉编译器
* arm-linux-gnueabi下载（到Linaro.org下载，我选择的是7.5.0版本，最新的）
> wget https://snapshots.linaro.org/components/toolchain/binaries/7.5-2019.12-rc1/arm-linux-gnueabi/gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabi.tar.xz

#### 编译内核
* make vexpress_defconfig
> 生成.config
* make menuconfig
```
 General setup-->
  [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support
      (_install) Initramfs source file(s)

 Boot options -->
   ()Default kernel command string 

 Kernel Features -->
    Memory split (3G/1G user/kernel split) -->
    [*]High Memory Support
```

* make zImage ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j4
> 出现找不到compiler-gcc7.h头文件问题？因为交叉编译器是7.5版本的gcc，所以会到内核的include/linux/中找compiler-gcc7.h，看一下目录下有没有其他版本的，如compiler-gcc5.h，如果有复制一个到compiler-gcc7.h即可解决这个问题

* make dtbs
> 设备树dtb

#### qemu运行内核
* arm内核
> qemu-system-arm -M vexpress-a9 -smp 2 -kernel arch/arm/boot/zImage -append "console=ttyAMA0 rdinit=linuxrc" -nographic -dtb arch/arm/boot/dts/vexpress-v2p-ca9.dtb -m 1024M 

## 龙芯

