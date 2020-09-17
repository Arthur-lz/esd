# 基于设备树的驱动实验
### 需要掌握的知识
#### 设备树与驱动程序是什么关系，各自负责什么工作？
##### 设备树是如何工作的
* 设备树如何写，需要程序员写一个DT Binding文档，这个文档存放于
linux内核源码/Documentation/devicetree/bindings目录下
* 这里截取部分sun8i-v3s.dtsi为例说明设备树是如何定义的
```dts
/ { // 根结点
         #address-cells = <1>; //表示子节点地址项只有一个
         #size-cells = <1>; // 表示子节点地址长度只有一个
         interrupt-parent = <&gic>;

         chosen {
                 #address-cells = <1>;
                 #size-cells = <1>;
                 ranges;

                 simplefb_lcd: framebuffer@0 { // simplefb_lcd是label，定义它的目的是可以在别的节点中通过&simplefb_lcd来引用该label
                         compatible = "allwinner,simple-framebuffer",
                                      "simple-framebuffer"; //这里兼容项有两个，前面的是具体的，后面的是系列的范围更大。
                         allwinner,pipeline = "de0-lcd0";
                         clocks = <&ccu CLK_BUS_TCON0>, <&ccu CLK_BUS_DE>,
                                  <&ccu CLK_DE>, <&ccu CLK_TCON0>;
                         status = "disabled";
                 };
         };

   cpus {
                #address-cells = <1>;
                #size-cells = <0>;

                cpu@0 { //单核心
                        compatible = "arm,cortex-a7";
                        device_type = "cpu";
                        reg = <0>;
                        clocks = <&ccu CLK_CPU>;
                };
        };
....
	soc {
                compatible = "simple-bus";
                #address-cells = <1>;
                #size-cells = <1>;
                ranges;

		....
                mmc0: mmc@01c0f000 {
                        compatible = "allwinner,sun7i-a20-mmc";
                        reg = <0x01c0f000 0x1000>;
                        clocks = <&ccu CLK_BUS_MMC0>,
                                 <&ccu CLK_MMC0>,
                                 <&ccu CLK_MMC0_OUTPUT>,
                                 <&ccu CLK_MMC0_SAMPLE>;
                        clock-names = "ahb",
                                      "mmc",
                                      "output",
                                      "sample";
                        resets = <&ccu RST_BUS_MMC0>;
                        reset-names = "ahb";
                        interrupts = <GIC_SPI 60 IRQ_TYPE_LEVEL_HIGH>;
                        status = "disabled";
                        #address-cells = <1>;
                        #size-cells = <0>;
                };
		...
		i2c0: i2c@01c2ac00 {
                         compatible = "allwinner,sun6i-a31-i2c";
                         reg = <0x01c2ac00 0x400>;
                         interrupts = <GIC_SPI 6 IRQ_TYPE_LEVEL_HIGH>;
                         clocks = <&ccu CLK_BUS_I2C0>;
                         resets = <&ccu RST_BUS_I2C0>;
                         pinctrl-names = "default";
                         pinctrl-0 = <&i2c0_pins>;
                         status = "disabled";
                         #address-cells = <1>;
                         #size-cells = <0>;
                 };
		...
}
```
再看另一个dts，它引用了上面的dtsi
```dts
/ {
        model = "Lichee Pi Zero";
        compatible = "licheepi,licheepi-zero", "allwinner,sun8i-v3s";/*这是根结点的兼容项，它有两个兼容项，
	 第一个是板级的，第二个是芯片级的，
         也就是说，第一个兼容项指明了它是为开发板licheepi-zero写的设备树，
	 第二个兼容项表示它是全志sun8i-v3s系列芯片的设备树*/

        aliases {
                serial0 = &uart0;
        };

        chosen {
                stdout-path = "serial0:115200n8";
        };
};// 注意到这里根结点定义结束了哟!!!这表示当前dts文件对根结点的补充就上面这些。

/* 下面这些全是通过label来引用dtsi中的节点，并补充定义
 */
&mmc0 {
        pinctrl-0 = <&mmc0_pins_a>;
        pinctrl-names = "default";
        broken-cd;
        bus-width = <4>;
        vmmc-supply = <&reg_vcc3v3>;
        status = "okay";
};

&i2c0 {// 引用了dtsi中定义的i2c0节点，并且在下面增加了子节点ns2009
        status = "okay";
	/*这里增加了子节点ns2009
	 */
        ns2009: ns2009@48 {
                compatible = "nsiway,ns2009";
                reg = <0x48>; //这里的这个地址0x48表示子节点ns2009绑定在i2c设备的0x48地址处
        };
	};
```

### 具体的触摸屏实验
>首先要明白一件事，我们买回来的lcd屏是由RGB屏与触摸屏两者组合而成的，触摸屏又分为电阻屏和电容屏，我用的是电阻屏ns2009。

那么问题来了，全志芯片是如何控制它们的呢？

触摸屏在开发板上由单独的ns2009触摸屏芯片控制，在Linux内核中make menuconfig时默认已经选上了Device Drivers → Input device support → Touchscreens→ [\*]Nsiway NS2009 touchscreen

##### 触摸屏的设备树和驱动
1. 设备树: sun8i-v3s-licheepi-zero.dts 
```dts
 &i2c0 {
         status = "okay";
 
	 // ns2009:冒号前面的ns2009是label，不是必须的，它的作用是在其他设备结点中可以通过&ns2009来访问这个label
         ns2009: ns2009@48 {
                 compatible = "nsiway,ns2009";//nsiway表示厂家,ns2009表示具体的芯片
                 reg = <0x48>; // 这里只有一个地址0x48，地址是必须的，长度不是，所以这里没有定义长度
         };
 };
```
2. 驱动程序在 linux/drivers/input/touchscreen/ns2009.c

##### lcd屏的设备树和驱动
1. 设备树源文件在linux/arch/arm/boot/dts/sun8i-v3s.dtsi
> lcd屏全志(allwinner)系列都支持，所以放在了设备树头文件(\*.dtsi)中
```dts
chosen {
                #address-cells = <1>; //表示子结点的地址只有一个
                #size-cells = <1>; //表示子结点的占用空间大小只有一项
                ranges;

                simplefb_lcd: framebuffer@0 {
                        compatible = "allwinner,simple-framebuffer",
                                     "simple-framebuffer";
                        allwinner,pipeline = "de0-lcd0";
                        clocks = <&ccu CLK_BUS_TCON0>, <&ccu CLK_BUS_DE>,
                                 <&ccu CLK_DE>, <&ccu CLK_TCON0>;
                        status = "disabled";
                };
        };
```
2. 驱动文件在linux/drivers/video/fbdev/simplefb.c
> 开发环境：manjaro, i3;

> 开发板：licheepi zero
 
>嵌入式系统:u-boot、linux内核、buildroot，在 https://github.com/Lichee-Pi上下载 
* 实验步骤

|题目|内容|说明|
|:--|:--|:--|
|触摸屏驱动|||
