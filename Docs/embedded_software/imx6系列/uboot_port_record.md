# uboot移植过程记录

## 一、硬件平台

使用embest出品的RIoTboard评估版

主控芯片为i.MX 6Solo（ARM Cortex A9）

## 二、开发平台

硬件：云主机

系统：ubuntu 18.04 server

代码源：本次使用的代码来自denx提供的代码（https://source.denx.de/u-boot/u-boot/-/tree/u-boot-2023.07.y?ref_type=heads）版本为u-boot-2023.07.y

## 三、移植过程

### 1.准备工作

准备工作同内核移植，这里不重复。

### 2. 编译uboot

#### 2.1 下载源码

``` bash
cd ~
git clone https://source.denx.de/u-boot/u-boot.git	# 克隆代码

cd u-boot	# 进入代码所在路径
```

#### 2.2 生成预配置文件

从当前u-boot的预配置文件中找到一个和目标板配置类似的预配置文件生成一个初始配置文件，之后在此基础上进行修改。

``` bash
make riotboard_defconfig	# 生成.config文件

make savedefconfig			# 将当前.config文件保存为defconfig文件，路径为u-boot根路径
mv   defconfig  configs/myboard_defconfig
```

#### 2.3 修改并保存配置文件

##### 2.3.1 打开配置界面

``` bash
make menuconfig
```

##### 2.3.2 主界面

在主界面，取消SPL使能，修改后如下图

<img src="images\uboot_main.png" alt="主界面" style="zoom:50%;" />

##### 2.3.3 ARM architecture界面

在此界面，需要确保如下几个关键设置的值：

- Target select (i.MX 6solo Soc support)
- MX6 board select (embestmx6boards)
- DCD script to use (board/boundary/nitrogen6x/nitrogen6s1g.cfg)

修改后的界面如下图

<img src="images\uboot_arm_arch.png" alt="arm_arch界面" style="zoom:50%;" />

##### 2.3.4 Boot options界面

在此界面确保Boot media选中**Support for booting from SD/EMMC**

##### 2.3.5 General setup界面

在此界面，确保Text Base项的值为**0x17800000**

修改Build target special image 为 u-boot.imx

##### 2.3.6 Device Tree Control 界面

在此界面，确保Default Device Tree for DT control项的值为**imx6dl-riotboard**

##### 2.3.7 Environment界面

需要修改的项：

- Environment in an MMC device：必须选中
- Environment offset：值修改为**0xC0000**
- mmc device number：2
- mmc partition number：0，这两项设置的是env所在设备及分区的编号

##### 2.3.8 Networking support界面

确保Random ethaddr if unset项被选中

##### 2.3.9 Ethernet PHY设置

在Device Drivers界面的Ethernet PHY support设置子界面，确保Atheros Ethernet PHYs support项被选中

在此界面，可以设置一个初始的IP地址，使用Set a default ‘ipaddr’项

##### 2.3.10 Network device设置

在Device Drivers界面的Network device support子界面，确保Enable MII项及FEC Ethernet controller项被选中

##### 2.3.11 Serial设置

在Device Drivers界面的Serial子界面：

- Specify the port number used for console：选中
- UART used for console：2
- IMX serial port support：选中

##### 2.3.12 保存预配置文件

在menuconfig界面保存当前设备到.config文件，然后退出menuconfig

``` bash
make savedefconfig
mv   defconfig  configs/myboard_defconfig
```

#### 2.4 修改embestmxboards.h文件

该文件的路径：include/configs/embestmx6boards.h

需要修改的内容

- \#define CFG_SYS_FSL_ESDHC_ADDR    0

  - 修改为\#define CFG_SYS_FSL_ESDHC_ADDR    3

- \#define PHYS_SDRAM_SIZE   (1u * 1024 * 1024 * 1024)

  - 根据实际情况修改

- \#define CFG_SYS_FSL_USDHC_NUM  3

  - 修改为：\#define CFG_SYS_FSL_USDHC_NUM  4

- #define  BOOT_TARGET_DEVICES(func)

  ``` c
  #define BOOT_TARGET_DEVICES(func) \
  	func(MMC, mmc, 0) \
  	func(MMC, mmc, 1) \
  	func(MMC, mmc, 2) \
  	func(MMC, mmc, 3) \		/* 增加此行 */
  	func(USB, usb, 0) \
  	func(PXE, pxe, na) \
  	func(DHCP, dhcp, na)
  ```

#### 2.5 修改mx6boards.c文件

``` c
static void setup_iomux_enet(void)
{
	imx_iomux_v3_setup_multiple_pads(enet_pads, ARRAY_SIZE(enet_pads));

	/* Reset AR8035 PHY */
	gpio_request(IMX_GPIO_NR(3, 31), "PHY reset");		// 添加该行
	gpio_direction_output(IMX_GPIO_NR(3, 31) , 0);
	mdelay(2);
	gpio_set_value(IMX_GPIO_NR(3, 31), 1);
}
```



#### 2.6 编译uboot

``` bash
make -j4			# 以4线程编译
make u-boot.imx		# 生成.imx文件，路径为uboot根路径
```

#### 2.7 烧写uboot

将编译得到的u-boot.imx文件替换mfg工具中的相应文件，进行烧写

#### 2.8 修改env

烧写后，板子并能不能启动系统，需要在uboot界面进行相应的修改

``` bash
bootargs=console=ttymxc1,115200 root=/dev/mmcblk0p2
loadimg=mmc dev 3; fatload mmc 3:1 0x12000000 uImage; fatload mmc 3:1 0x18000000 imx6solo_RIoTboard.dtb; bootm 0x12000000 - 0x18000000
bootcmd=run loadimg
```

或者参考mx6sabre_common.h中的方法，修改embestmxboards.h，在其中加入

``` c
#ifdef CONFIG_SUPPORT_EMMC_BOOT
#define EMMC_ENV \
	"emmcdev=2\0" \
	"update_emmc_firmware=" \
		"if test ${ip_dyn} = yes; then " \
			"setenv get_cmd dhcp; " \
		"else " \
			"setenv get_cmd tftp; " \
		"fi; " \
		"if ${get_cmd} ${update_sd_firmware_filename}; then " \
			"if mmc dev ${emmcdev} 1; then "	\
				"setexpr fw_sz ${filesize} / 0x200; " \
				"setexpr fw_sz ${fw_sz} + 1; "	\
				"mmc write ${loadaddr} 0x2 ${fw_sz}; " \
			"fi; "	\
		"fi\0"
#else
#define EMMC_ENV ""
#endif

#define CFG_EXTRA_ENV_SETTINGS \
	"script=boot.scr\0" \
	"image=zImage\0" \
	"fdtfile=undefined\0" \
	"fdt_addr=0x18000000\0" \
	"boot_fdt=try\0" \
	"ip_dyn=yes\0" \
	"console=" CONSOLE_DEV "\0" \
	"dfuspi=dfu 0 sf 0:0:10000000:0\0" \
	"dfu_alt_info_spl=spl raw 0x400\0" \
	"dfu_alt_info_img=u-boot raw 0x10000\0" \
	"dfu_alt_info=spl raw 0x400\0" \
	"fdt_high=0xffffffff\0"	  \
	"initrd_high=0xffffffff\0" \
	"splashimage=" __stringify(CONFIG_SYS_LOAD_ADDR) "\0" \
	"mmcdev=" __stringify(CONFIG_SYS_MMC_ENV_DEV) "\0" \
	"mmcpart=1\0" \
	"finduuid=part uuid mmc ${mmcdev}:2 uuid\0" \
	"update_sd_firmware=" \
		"if test ${ip_dyn} = yes; then " \
			"setenv get_cmd dhcp; " \
		"else " \
			"setenv get_cmd tftp; " \
		"fi; " \
		"if mmc dev ${mmcdev}; then "	\
			"if ${get_cmd} ${update_sd_firmware_filename}; then " \
				"setexpr fw_sz ${filesize} / 0x200; " \
				"setexpr fw_sz ${fw_sz} + 1; "	\
				"mmc write ${loadaddr} 0x2 ${fw_sz}; " \
			"fi; "	\
		"fi\0" \
	EMMC_ENV	  \
	"mmcargs=setenv bootargs console=${console},${baudrate} " \
		"root=PARTUUID=${uuid} rootwait rw\0" \
	"loadbootscript=" \
		"load mmc ${mmcdev}:${mmcpart} ${loadaddr} ${script} || " \
		"load mmc ${mmcdev}:${mmcpart} ${loadaddr} boot/${script};\0" \
	"bootscript=echo Running bootscript from mmc ...; " \
		"source\0" \
	"loadimage=load mmc ${mmcdev}:${mmcpart} ${loadaddr} ${image} || " \
		"load mmc ${mmcdev}:${mmcpart} ${loadaddr} boot/${image}\0" \
	"loadfdt=load mmc ${mmcdev}:${mmcpart} ${fdt_addr} ${fdtfile} || " \
		"load mmc ${mmcdev}:${mmcpart} ${fdt_addr} boot/${fdtfile}\0" \
	"mmcboot=echo Booting from mmc ...; " \
		"run finduuid; " \
		"run mmcargs; " \
		"if test ${boot_fdt} = yes || test ${boot_fdt} = try; then " \
			"if run loadfdt; then " \
				"bootz ${loadaddr} - ${fdt_addr}; " \
			"else " \
				"if test ${boot_fdt} = try; then " \
					"bootz; " \
				"else " \
					"echo WARN: Cannot load the DT; " \
				"fi; " \
			"fi; " \
		"else " \
			"bootz; " \
		"fi;\0" \
	"netargs=setenv bootargs console=${console},${baudrate} " \
		"root=/dev/nfs " \
		"ip=dhcp nfsroot=${serverip}:${nfsroot},v3,tcp\0" \
	"netboot=echo Booting from net ...; " \
		"run netargs; " \
		"if test ${ip_dyn} = yes; then " \
			"setenv get_cmd dhcp; " \
		"else " \
			"setenv get_cmd tftp; " \
		"fi; " \
		"${get_cmd} ${image}; " \
		"if test ${boot_fdt} = yes || test ${boot_fdt} = try; then " \
			"if ${get_cmd} ${fdt_addr} ${fdtfile}; then " \
				"bootz ${loadaddr} - ${fdt_addr}; " \
			"else " \
				"if test ${boot_fdt} = try; then " \
					"bootz; " \
				"else " \
					"echo WARN: Cannot load the DT; " \
				"fi; " \
			"fi; " \
		"else " \
			"bootz; " \
		"fi;\0" \
		"findfdt="\
			"if test $fdtfile = undefined; then " \
				"if test $board_name = SABREAUTO && test $board_rev = MX6QP; then " \
					"setenv fdtfile imx6qp-sabreauto.dtb; fi; " \
				"if test $board_name = SABREAUTO && test $board_rev = MX6Q; then " \
					"setenv fdtfile imx6q-sabreauto.dtb; fi; " \
				"if test $board_name = SABREAUTO && test $board_rev = MX6DL; then " \
					"setenv fdtfile imx6dl-sabreauto.dtb; fi; " \
				"if test $board_name = SABRESD && test $board_rev = MX6QP; then " \
					"setenv fdtfile imx6qp-sabresd.dtb; fi; " \
				"if test $board_name = SABRESD && test $board_rev = MX6Q; then " \
					"setenv fdtfile imx6q-sabresd.dtb; fi; " \
				"if test $board_name = SABRESD && test $board_rev = MX6DL; then " \
					"setenv fdtfile imx6dl-sabresd.dtb; fi; " \
				"if test $fdtfile = undefined; then " \
					"echo WARNING: Could not determine dtb to use; fi; " \
			"fi;\0" \
```

## 四、遗留问题

- 如果使用编译得到的Device Tree文件（.dtb）进行烧写，则进入系统后无法发现eth0设备，无法使用网络功能