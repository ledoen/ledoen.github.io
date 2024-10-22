# 内核移植过程记录

## 一、硬件平台

使用embest出品的RIoTboard评估版

主控芯片为i.MX 6Solo（ARM Cortex A9）

## 二、开发平台

硬件：云主机

系统：ubuntu 18.04 server

代码源：本次使用的代码来自freescale提供在github上的代码库（https://github.com/Freescale）

## 三、移植过程

### 1. 准备工作

#### 1. 1 安装依赖及编译工具

```bash
sudo apt install make gcc-arm-linux-gnueabihf gcc bison flex libssl-dev dpkg-dev lzop bc git unzip screen
```

#### 1.2 打开screen

由于编译内核时耗时较长，如果ssh终端失去连接，容易造成编译中断

除此之外，每次登录ssh时，都需要export环境变量

因此使用screen，打开一个常驻的section，用于编译工作

screen的使用方法

``` bash
screen -S section_name	# 新建一个名为 section_name 的section
screen -ls				# 列出当前所有的 section
screen -r section_name	# 恢复到指定的 section
```

在开始工作之前，打开一个新的section

``` bash
screen -S build
```

### 2. 编译uboot

在此处编译uboot的作用是编译生成mkimage，后续编译内核后生成uImage时需要该工具

####  2.1 下载uboot源码

``` bash
cd ~
git clone https://github.com/Freescale/u-boot-fslc.git
```

由于云主机配置太低，512M的ram，导致git clone报错，无法正常可控代码，因此使用wget获取代码

``` bash
wget https://github.com/Freescale/u-boot-fslc/archive/refs/heads/2023.04+fslc.zip
unzip 2023.04+fslc.zip
cd u-boot-fslc-2023.04-fslc
```

#### 2.2 export环境变量

``` bash
export ARCH=arm
export CROSS_COMPILE=arm-linux-eabihf-
```

#### 2.3 产生.config文件

找到一个imx6solo的预配置文件，进行预配置

``` bash
ls configs/mx6				# 双击 Table 键的补全功能，看当前所有的以mx6开头的预配置文件

make mx6slevk_defconfig		# 生成 .config 文件
```

#### 2.4 编译

``` bash
make -j5					# 以多线程的方式进行编译
```

#### 2.5 复制mkimage到可执行路径

``` bash
cp tools/mkimage  /usr/bin	# 复制到 /usr/bin
```

### 3. 编译内核

#### 3.1 下载代码

``` bash
cd ~
git clone https://github.com/Freescale/linux-fslc.git

# 或者使用 wget
wget https://github.com/Freescale/linux-fslc/archive/refs/heads/6.1.x+fslc.zip
unzip 6.1.x+fslc.zip

cd linux-fslc-6.1.x-fslc # 进入内核源码路径
```

#### 3.2 新建预配置文件

由于当前的预配置文件不满足当前的使用需求，需要从embest的仓库拷贝一份过来

embest预编译文件的地址为https://github.com/embest-tech/linux-imx/blob/imx_3.10.17_1.0.0_ga/arch/arm/configs/imx_v7_defconfig（本地文件地址）

复制后新建预编译文件，并拷贝进来

``` bash
nano arch/arm/configs/imx_v7_defconfig	# 新建
# 将拷贝的内容粘贴进来即可
```

#### 3.3 生成.config文件

``` bash
make imx_v7_defconfig
```

#### 3.4 编译内核

```bash
make -j5
```

编译后将产生Image、zImage文件，路径为arch/arm/boot

#### 3.5 生成uImage文件

如果使用官方的mfgtool工具烧写内核到板载emmc，需要使用uImage文件

``` bash
make uImage LOADADDR=0x10008000
```

uImage文件的路径同zImage

### 4. 烧写内核

将uImage文件替换mfgtool相关路径下的uImage，然后正常烧写即可

### 5. 其它

#### 5.1 mftools修改分区信息

修改`mksdcard-yocto.sh`文件

``` bash
#!/bin/sh

# partition size in MB
BOOT_ROM_SIZE=4


# call sfdisk to create partition table
# destroy the partition table
node=$1
dd if=/dev/zero of=${node} bs=1024 count=1

sfdisk --in-order --force -uM ${node} << EOF
${BOOT_ROM_SIZE},12,c
,,83
EOF
```

以上代码中`BOOT_ROM_SIZE`指的是存放uboot空间的大小，`${BOOT_ROM_SIZE},12,c`中12指的是存放内核分区的大小，`,,83`指的是存放rootfs文件的空间，该分区将分配所有剩余空间