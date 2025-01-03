# EAIDK 610 安装系统
搭载了rk3399的eaidk610，提供的mali-t860能够提供一定的GPU图形加速能力，但由于其官方网站已经无法访问，因此其系统的选择被限制在百度云镜像站与第三方系统的范围内。
##### - [百度云镜像](https://pan.baidu.com/s/1-upIbUf1v9CZNB-tTa07Lg?pwd=hptp "百度云镜像")
 1. Fedora 28 对GPU适配最好，但仅停留在kernel，貌似没有userspace的部分，可以驱动mipi，有fdt支持，镜像源需要[更换](https://github.com/Lasius-alienus/eaidk-610-device-tree/blob/main/tools/EAIDK_610.sh "更换")，只能使用fedora官方镜像，速度慢，rk维护的uboot貌似不倾向于在emmc有可启动系统的情况下卡启？
 1. ubuntu 16.04 没有对GPU的适配，可以驱动mipi但触摸需要校准，使用uboot传参未获取dtb，，若不扩容刷入后根目录大约只有300MiB剩余空间，不怎么能用，扩容的大致思路是用SD卡内的系统操作gparted修改emmc分区，但本人没有成功。
 1. Android 8.1 貌似没有适配GPU，mipi正常（毫无疑问），可以使用magisk获取root权限，修补boot就行了，在此不再赘述。
 1. armbian 两个镜像都没有适配GPU，无法驱动mipi，没有dtb支持，只有这个镜像支持烧录sd卡启动（毕竟别的都是分区镜像）。


 ##### - ophub（[armbian](https://github.com/ophub/amlogic-s9xxx-armbian/releases "armbian")/[op](https://github.com/ophub/amlogic-s9xxx-openwrt/releases "op")）
 1. 目前暂不支持驱动GPU和mipi，其他功能正常
 1. 卡启适配较好，如要直接烧录到emmc，使用[烧写工具](https://github.com/Lasius-alienus/eaidk-610-device-tree/tree/main/tools/RKDevTool "烧写工具")，修改第二项为你使用的镜像即可，从sd卡写入emmc在root下执行`$ armbian-config`，然后按提示操作（有时候会报`Error: no u-boot package found, exiting`但我这边多试了几次莫名其妙就成功了，不作方法参考） 。
 
### 但正如上文所述一个板卡有GPU支持（况且性能还不差），那就应该把GPU驱动起来。以下还有一些适配一般但驱动了GPU的镜像。
##### [orangepi 4 lts](http://www.orangepi.cn/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-4-LTS.html "orangepi 4 lts")
以下系统皆不能驱动USB3.0，mipi屏幕（至少不正常），type-c（未测试）
- debian
使用闭源驱动，支持硬解，对游戏优化较差（如MC），浏览器支持硬解视频，如果type-c在特定系统下能用，那应当是使用4.4内核的系统。
- ubuntu
使用5.18内核的系统使用了开源的panforst驱动，优化极佳，但貌似只支持kodi硬解，直接刷入emmc不正常（ssh无法连接，无法创建用户/修改密码）！从sd卡写入emmc报错。

# 不要按照orangepi官方文档的做法尝试驱动mipi屏幕，型号不同可能会导致屏幕烧毁！！！

# 反向适配的尝试

为没有dtb支持的系统适配GPU驱动很难，即使解决了kernel部分的驱动userspace也很麻烦，但是为适配了GPU的系统适配mipi屏幕和USB3.0可能难度会相对较低

所有目前已经获得的[dtb](https://github.com/Lasius-alienus/eaidk-610-device-tree/tree/main/dtb "dtb")/[dts](https://github.com/Lasius-alienus/eaidk-610-device-tree/tree/main/dts "dts")都分类放在了相应的文件夹中部分反编译的dts放在了[dts(DTC)](https://github.com/Lasius-alienus/eaidk-610-device-tree/tree/main/dts(DTC))文件夹中

目前使用的armbian官方提供的dts，编译后得到的[dtb](https://github.com/Lasius-alienus/eaidk-610-device-tree/tree/main/support%20orangepi%204%20lts "dtb")配合orangpi 4 lts已经可以驱动这块板卡的大多数功能。不能驱动的有：type-c（未测试）可能不能正常运行。Bluetooth可以连接设备，但连接蓝牙音响无输出。dts中没有mipi的描述，因此适配困难。目前主要的问题是，开机时出现`A start job is running for Load Kernel Modules`,超时时间为90s，需要超时后才能启动系统。

# 编译内核的说明
由于获取了一部分的dts（不是反编译的，那个没法大改），也许可以尝试修改一部分dts然后放入官方提供的kernel源码（EAIDK不行，太老了）或[获取](https://www.kernel.org/ "获取")linux kernel（[更快的下载源](https://mirror.bjtu.edu.cn/kernel/linux/kernel/v6.x/ "更快的下载源")），编译得到内核文件和dtb。最好使用6.x版本内核，因为新内核对mali和panforst适配应该较好，具体的适配在此不再赘述，一部分参考材料可在此处获取。
- [在 RK3399 上运行开源的 Mali GPU 驱动](https://aijishu.com/a/1060000000082887 "在 RK3399 上运行开源的 Mali GPU 驱动")
- [在 RK3399 上部署最新的 Linux 5.4 和 U-Boot v2020 .01](https://mp.weixin.qq.com/s/KZ1TflNpyWtZLMGcKtDyTQ "在 RK3399 上部署最新的 Linux 5.4 和 U-Boot v2020 .01")
