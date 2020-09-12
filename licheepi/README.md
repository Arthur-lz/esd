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
 | ------------- |:-------------:| -----|
 |安装make|sudo pacman -S make|
 |进入/opt/目录|cd /opt/||
 |下载工具链|sudo wget https://snapshots.linaro.org/components/toolchain/binaries/7.5-2019.12-rc1/arm-linux-gnueabihf/gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf.tar.xz|在arm-linux-guneabihf目录下有多个不同版本的可供选择，这里我使用的是64位的arm-linux-gnueabihf版本|
 |解压工具链|sudo tar xvf gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf.tar.xz |解压成功后，会在当前文件夹下出现gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf |
 |切换成root|接下来需要操作/etc/profile，所以切到root||
 |设置环境变量|echo "PATH=$PATH:gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf/bin" >> /etc/profile | |
 |设置环境变量|echo "LD_LIBRARY_PATH=/opt/develop/gcc-linaro-7.5.0-2019.12-rc1-x86_64_arm-linux-gnueabihf/lib" >> /etc/profile|我这里LD_LIBRARY_PATH是首次定义，所以直接赋值，如果你不是首次使用，这里要把原来的值加上，像这样old=$old:new|
 |退出root|||
 |进入licheepi实验目录|我的是~/esd/licheepi/||
 |创建两个文件夹|mkdir buildroot, mkdir linux|u-boot不需要创建新的文件夹|
 |下载u-boot|git clone https://github.com/Lichee-Pi/u-boot.git -b v3s-current --depth 1|这里我只clone最新一版|
