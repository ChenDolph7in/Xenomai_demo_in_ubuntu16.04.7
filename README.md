# 基于Xenomai的IGH-EtherCAT开发



## 环境配置

* Oracle VM VirtualBox 6.1.18

* ubuntu-16.04.7-desktop-amd64

* Visual Studio Code 1.67.2

  

# 虚拟机中Ubuntu16.04.7下Xenomai3.2.1编译&安装&配置步骤（Linux-4.16.99）

***/xenomai/Xenomai编译安装.md***

### 环境配置

* Xenomai-v3.2.1
* linux-4.19.66

## 基本步骤

### 1.下载Ubuntu16.04.7镜像源并安装

* 到学校或其他网站下载Ubuntu镜像源ubuntu-16.04.7-desktop-amd64
  例如 

  * [中国科学技术大学](http://mirrors.ustc.edu.cn/ubuntu-releases/16.04/)
    http://mirrors.ustc.edu.cn/ubuntu-releases/16.04/
  * [阿里云](http://mirrors.aliyun.com/ubuntu-releases/16.04/)
    http://mirrors.aliyun.com/ubuntu-releases/16.04/
  * [兰州大学](http://mirror.lzu.edu.cn/ubuntu-releases/16.04/)
    http://mirror.lzu.edu.cn/ubuntu-releases/16.04/
  * [北理工](http://mirror.bit.edu.cn/ubuntu-releases/16.04/)
    http://mirror.bit.edu.cn/ubuntu-releases/16.04/
  * [浙大](http://mirrors.zju.edu.cn/ubuntu-releases/16.04/)
    http://mirrors.zju.edu.cn/ubuntu-releases/16.04/ 
* 在Virtualbox安装Ubuntu-16.04.7
* ***<font color=red>！！！注意：编译打包内核时虚拟机磁盘可用空间可能超过20G，如果新安装虚拟机，磁盘空间最好分配大一点！！！</font>***
* <font color=red>为了避免不可挽回的错误，在安装过程中注意在关键点备份虚拟机</font>



### 2.基本配置和网络配置

#### 2.1基本配置

虚拟机需要的基本的应用安装

* vim安装: `sudo apt-get install vim`
* ssh安装（参考https://www.cnblogs.com/lishubin/p/11672037.html，若不使用ssh连接可跳过)



#### 2.2网络配置

* 在***未进入虚拟机***前为虚拟机配置两个网卡：
  * 网络地址转换NAT：用于通过宿主主机接入互联网
  * 仅主机网络：用于宿主主机ssh远程连接（可跳过）
* 终端输入`ifconfig`，查看仅主机网络IP地址，并使用宿主主机测试是否可以ping通（可跳过）
* 使用Xhell等ssh连接(可跳过)



### 3.下载Xenomai相关安装包和补丁文件

#### 3.1下载Xenomai3.2.1源码

* 到https://source.denx.de/Xenomai/xenomai/-/tags下载Xenomai3.2.1



#### 3.2 下载内核

* 查看当前的内核版本
  * `at /proc/version_signature ` 或
  * `uname -a`
* Ubuntu16.04.7基本内核版本是：Linux4.15.0-112-generic，下载版本相近的Linux内核和ipipe补丁，这里选择***linux-4.19.66.tar.gz*** 
* 到https://mirrors.edge.kernel.org/pub/linux/kernel/在目录逐级进入/pub/linux/kernel/v4.x/
* 寻找***linux-4.19.66.tar.gz*** 并下载



 #### 3.3下载ipipe补丁

* 到https://xenomai.org/downloads/ipipe在目录逐级进入/downloads/ipipe/v4.x/x86/
* 寻找***ipipe-core-4.19.66-x86-6.patch***，复制其内容
* 创建新文件***ipipe-core-4.19.66-x86-6.patch***，粘贴网页内容到该文件下，作为补丁文件



#### 3.4文件结构和解压

* 创建/xenomai文件夹，并在/xenomai下创建***子文件夹***/4.19.66，并将***ipipe-core-4.19.66-x86-6.patch***和***linux-4.19.66.tar.gz***放入/4.19.66，将***xenomai-v3.2.1.tar.gz***放入/xenomai

* 解压压缩文件（<font color=red>注：下面的指令需要在压缩包所在目录中解压，否则指定解压后的目标文件路径，使解压后的文件目录结构如下面所示</font>）

  * `sudo tar zxvf xenomai-v3.2.1.tar.gz`
  * `sudo tar zxvf linux-4.19.66.tar.gz`

* 解压完成后文件目录结构

  ```shell
  ~/xenomai$ tree -L 2
  .
  ├── 4.19.66
  │   ├── ipipe-core-4.19.66-x86-6.patch
  │   ├── linux-4.19.66
  │   └── linux-4.19.66.tar.gz
  ├── xenomai-v3.2.1
  └── xenomai-v3.2.1.tar.gz
  ```

  若解压到错误路径，删除该目录`sudo rm -rf 目录`，然后重新解压到正确路径。



### 4.安装必要的组件

* `sudo apt-get install build-essential libncurses5-dev bison flex libssl-dev kernel-package` 
* 用时一小时左右



### 5.准备Linux内核

#### 5.1Linux内核打ipipe补丁

在/xenomai/4.19.66/linux-4.19.66目录下打开终端执行：

`sudo ../../xenomai-v3.2.1/scripts/prepare-kernel.sh --ipipe=../ipipe-core-4.19.66-x86-6.patch --arch=x86_64`

出现以下问题注意：

* patch unexpectedly ends in middle of line：patch文件以LF结束一行，以CRLF为行尾序列的需要去掉CR
* patch unexpectedly ends in middle of line：不知原因



#### 5.2配置内核

在/xenomai/4.19.66/linux-4.19.66目录下打开终端执行（由于需要足够的空间打开图形化界面，终端尺寸要够大）：

* `sudo make menuconfig`
* 若报错，注意libncurses5-dev是否安装成功；也可能为权限导致无法找到ncurses，使用sudo执行命令试一试。
* 图形化界面显示成功后，存在***Xenimai/cobalt(New)***，为刚刚在***5.1***打的补丁

在配置内核的图形界面中依次操作（Pressing <Y> includes, <N>  excludes）：

1. Processor type and features >

   * Linux guest support 关闭
   * Processor family > 根据需要选择
   * Multi-core scheduler support 关闭

2. Power management and ACPI options >

   * Suspend to RAM and standby 关闭
   * Hibernation (aka 'suspend to disk') 关闭
   * CPU Frequency scaling 关闭
   * ACPI (Advanced Configutation and Power Interface) Support > Processor 关闭
   * CPU Idle 关闭

3. Memory Management options

   * Transparent Hugepage Support 关闭
   * Contiguous Memory Allocator 关闭
   * Allow for memory compaction 关闭
   * Page migration 关闭

4. Xenomai/cobalt > 根据需要选择

   * Sizes and static limits 可以适当调大(例如3个heap大小都调至4096)
   * Drivers (根据需要打开CAN、RTnet等驱动)

5. 检查EFI设置，保证其被使能

   * Power management and ACPI options > 
         ACPI (Advanced Configutation and Power Interface) Support > 只关闭Processor，其余不关闭
   * Processor type and features >
         EFI runtime service support 开启
         EFI stub support 开启
   * Firmware Drivers >
         EFI (Extensible Firmware Interface) Support >
             EFI Variable Support via sysfs 开启

6. 关闭Module签名需求（<font color=red>Virtualbox特别需求，原因见IGH-EtherCAT环境搭建->运行->1启动主站</font>）

   * Enable loadable module support >

     	Module signature verification 关闭



#### 5.3编译和打包内核和头文件

编译和安装分离，在高性能机器编译并打包，转义并安装到其他的机器

1. 在/xenomai/4.19.66/linux-4.19.66目录下创建文件REPORTING-BUGS，否则头文件会编译错误，报错<font color=red>无法获取REPORT-BUGS的文件状态stat</font>

   `sudo touch REPORTING-BUGS`

2. 在/xenomai/4.19.66/linux-4.19.66目录下打开终端执行(<font color=red>耗时较长，根据主机配置耗时可能长达2~3h。注意修改自动息屏/睡眠时间；以及最好不要使用SSH终端运行，防止链接中断导致编译中断；再者注意磁盘剩余空间，根据内核配置需求中间文件可能会占用不下于20G的磁盘空间</font>)

   * `sudo make-kpkg --initrd --revision 03 --append-to-version -xeno20220611 kernel_image kernel_headers`

3. 编译完成后，/xenomai/4.19.66/目录下出现编译完的***linux-headers-4.19.66-xeno20220611_03_amd64.deb***和***linux-image-4.19.66-xeno20220611_03_amd64.deb***



### 6.编译和安装内核

1. 在/xenomai/4.19.66目录下打开终端执行以下指令安装内核和头

   `sudo dpkg -i linux-headers-4.19.66-xeno20220611_03_amd64.deb linux-image-4.19.66-xeno20220611_03_amd64.deb`

2. 查看已安装的内核镜像和头，验证是否已经安装成功（可跳过）

   ```shell
   dpkg -l | grep linux-image
   dpkg -l | grep linux-headers
   ```

3. 删除已安装的镜像和头（需要的话）

   ```shell
   sudo dpkg --purge linux-headers-4.19.66-xeno20220611 
   sudo dpkg --purge linux-image-4.19.66-xeno20220611
   ```

4. 编译和安装Xenomai

   1. 安装自动生成makefile的工具，因为Xenomai3.2版本源码里面不提供configure脚本和makefile。若使用Xenomai3.1版本可省略此步

      ```shell
      sudo apt-get install autoconf automake libtool
      sudo apt-get install fuse
      sudo apt-get install debhelper findutils autotools-dev autoconf automake libtool pkg-config libltdl-dev
      ```

   2. 前面下载好的xenomai源码包里面没有configure脚本和makefile，需要手动生成，在源码根目录下(/xenomai/xenomai-v3.2.1)使用`sudo ./scripts/bootstrap`命令。

   3. 在/xenomai/xenomai-v3.2.1配置并安装Xenomai

      ```shell
      sudo ./configure --with-core=cobalt --enable-smp --enable-pshared
      sudo make
      sudo make install
      ```



### 7.配置启动界面，在启动菜单选择内核

1. 修改GRUB配置信息`sudo vim /etc/default/grub`

   GRUB_HIDDEN_TIMEOUT=0 注释掉
   GRUB_TIMEOUT=5
   GRUB_CMDLINE_LINUX_DEFAULT="text"

   具体含义参考https://blog.csdn.net/piaopiaopiaopiaopiao/article/details/11772053

2. 更新GRUB配置信息`sudo update-grub`，然后重启(`reboot`)

3. 在启动界面选择：***ubuntu高级选项 > 新安装的内核***

4. `uname -a`查看是否选择成功



### 8.测试Xenomai

```shell
cd /usr/xenomai/bin
sudo ./latency
```

```shell
sudo apt-get install stress
stress -v -c 8 -i 10 -d 8
```



### 9.更新环境变量

1. 在~目录下向.bashrc文件(隐藏文件，可用`ls -a`查看到)末尾添加

   ```shell
   export LD_LIBRARY_PATH="/usr/xenomai/lib:$LD_LIBRARY_PATH"
   export PATH="/usr/xenomai/bin:$PATH"
   export XENOMAI_ROOT_DIR="/usr/xenomai"
   export XENOMAI_PATH="/usr/xenomai"
   export PATH="$PATH:$XENOMAI_PATH/bin:$XENOMAI_PATH/sbin"
   export PKG_CONFIG_PATH="$PKG_CONFIG_PATH:$XENOMAI_PATH/lib/pkgconfig"
   export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$XENOMAI_PATH/lib"
   export OROCOS_TARGET="xenomai"
   ```

2. 更新.bashrc

   `source ~/.bashrc`



### 10.添加xenomai的链接库路径

1. 在/etc/ld.so.conf.d目录下新建xenomai.conf

   ```shell
   cd /etc/ld.so.conf.d
   sudo touch xenomai.conf
   ```

2. 打开xenomai.conf并添加"/usr/xenomai/lib"

   `sudo vim xenomai.conf`

3. 使更改生效

   `sudo /sbin/ldconfig -v`



### <font color=red>建议此处备份系统快照</font>



### 仍未明确原因的问题

* 当成功编译并安装一次内核xeno20220529后，由于virtualbox不能使用未签名module、需要更改内核配置（详细原因见 IGH_EtherCAT环境搭建->运行->1.启动主站），在/xenomai/4.19.66/linux-4.19.66目录下第二次配置并编译内核xeno20220610后，虽然第二次编译的内核安装成功，但两个内核均不能正常使用xenomai api程序，报错：

  ```shell
  low_init(): [main] Cobalt core not enabled in kernel
  ```

  仍不明原因，可能是ethercat配置错误，或二次编译导致错误

* 在步骤5.1内核打补丁时，在/xenomai/4.19.66/linux-4.19.66目录下打开终端执行：

  `sudo ../../xenomai-v3.2.1/scripts/prepare-kernel.sh --ipipe=../ipipe-core-4.19.66-x86-6.patch --arch=x86_64`

  出现以下问题注意：

  * patch unexpectedly ends in middle of line：patch文件以LF结束一行，以CRLF为行尾序列的需要去掉CR
  * patch unexpectedly ends in middle of line：不知原因

  第二个问题不明原因，但打补丁仍运行正常



# IGH_EtherCAT主站环境搭建

详见***/xenomai/IGH_EtherCAT环境搭建.md***

### 环境配置

* IGH-EtherCAT 1.5-stable

## 1.下载IGH_EtherCAT源码

https://github.com/ribalda/ethercat.git

将其解压后放在/xenomai目录下。

或在/xenomai目录下

```shell
sudo apt-get install git
git clone https://github.com/ribalda/ethercat.git
```

假设源码目录名为ethercat。



## 2.下载所需依赖

```shell
sudo apt install autoconf automake libtool net-tools
sudo apt-get install linux-headers-$(uname -r)
```



## 3.进行编译安装

进入目录/xenomai/ethercat

### 3.1 生成configure脚本

执行命令

```shell
./bootstrap
```



### 3.2 查看网卡型号

1. 首先查看网卡信息`ifconfig`

```shell
cdx@cdx-VirtualBox:~/xenomai/ethercat$ ifconfig 
enp0s3    Link encap:以太网  硬件地址 08:00:27:df:7c:81  
          inet 地址:10.0.2.15  广播:10.0.2.255  掩码:255.255.255.0
          inet6 地址: fe80::ff36:9f1:ccf2:a98e/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  跃点数:1
          接收数据包:30317 错误:0 丢弃:0 过载:0 帧数:0
          发送数据包:7335 错误:0 丢弃:0 过载:0 载波:0
          碰撞:0 发送队列长度:1000 
          接收字节:42845040 (42.8 MB)  发送字节:447597 (447.5 KB)

enp0s8    Link encap:以太网  硬件地址 08:00:27:dc:8b:99  
          inet 地址:192.168.56.107  广播:192.168.56.255  掩码:255.255.255.0
          inet6 地址: fe80::82b5:4049:11d9:b8f7/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  跃点数:1
          接收数据包:1148 错误:0 丢弃:0 过载:0 帧数:0
          发送数据包:1423 错误:0 丢弃:0 过载:0 载波:0
          碰撞:0 发送队列长度:1000 
          接收字节:92554 (92.5 KB)  发送字节:252094 (252.0 KB)

lo        Link encap:本地环回  
          inet 地址:127.0.0.1  掩码:255.0.0.0
          inet6 地址: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  跃点数:1
          接收数据包:278 错误:0 丢弃:0 过载:0 帧数:0
          发送数据包:278 错误:0 丢弃:0 过载:0 载波:0
          碰撞:0 发送队列长度:1000 
          接收字节:21920 (21.9 KB)  发送字节:21920 (21.9 KB)
```

2. 从上面的结果找到桥接网卡，记下网卡名、MAC地址(硬件地址)，然后使用ethtool查看网卡类型`ethtool -i 网卡名`

```shell
cdx@cdx-VirtualBox:~/xenomai/ethercat$ ethtool -i enp0s8
driver: e1000
version: 7.3.21-k8-NAPI
firmware-version: 
expansion-rom-version: 
bus-info: 0000:00:08.0
supports-statistics: yes
supports-test: yes
supports-eeprom-access: yes
supports-register-dump: yes
supports-priv-flags: no
```

3. 记下driver类型为***e1000***



### 3.3 进行configure配置生成规则文件Makefile

进入目录/xenomai/ethercat

调用./configure生成规则文件，使用选项指定网卡类型、内核路径和安装路径等

```shell
./configure --with-linux-dir=/home/cdx/xenomai/4.19.66/linux-4.19.66 --enable-generic=yes --enable-8139too=no --enable-e1000=yes --enable-e1000e=no --enable-r8169=no --enable-rtdm=yes --enable-cycles=yes --enable-hrtimer=yes --with-xenomai-dir=/usr/xenomai --prefix="/opt/etherlab"
```

注：

* 注意enable网卡类型，否则容易出错：

  ```shell
  checking for kernel for r8169 driver... configure: error: kernel 4.19 not available for r8169 driver!
  ```

* 对于选项--with-linux-dir网络上常用/usr/src/下的kernel-headers，

  ```shell
  ./configure --with-linux-dir=/usr/src/linux-headers-4.19.66-xeno20220611/ --enable-generic=yes --enable-8139too=no --enable-e1000=yes --enable-e1000e=no --enable-r8169=no --enable-rtdm=yes --enable-cycles=yes --enable-hrtimer=yes --with-xenomai-dir=/usr/xenomai --prefix="/opt/etherlab"
  ```

  但在步骤3.4的make modules会出错：

  ```shell
  In file included from ./include/linux/compat.h:16:0,
                   from ./include/linux/ethtool.h:17,
                   from ./include/linux/netdevice.h:41,
                   from /home/cdx/xenomai/ethercat/devices/e1000/e1000-4.19-ethercat.h:18,
                   from /home/cdx/xenomai/ethercat/devices/e1000/e1000_main-4.19-ethercat.c:4:
  ./include/linux/if.h:28:54: fatal error: sys/socket.h: 没有那个文件或目录
  ```

  和

  ```shell
  make[1]: Entering directory '/usr/src/linux-headers-4.19.66-xeno20220610'
  make[4]: *** No rule to make target '/home/cdx/xenomai/ethercat/devices/e1000/e1000_main-4.19-ethercat.o', needed by '/home/cdx/xenomai/ethercat/devices/e1000/ec_e1000.o'。 停止。
  scripts/Makefile.build:544: recipe for target '/home/cdx/xenomai/ethercat/devices/e1000' failed
  ```

  改为安装xenomai内核时使用的/xenomai/4.19.66/linux-4.19.66不会报错，原因不明

* 如果不指定--prefix，1.5-stable版本的ethercat被安装在/usr/local

  若指定--prefix="/opt/etherlab"，ethercat被安装在/opt/etherlab

  网上教程多是安装在/opt下，故这里安装在/opt/etherlab



### 3.4 使用Makefile编译

```shell
make #编译用户态的库
make modules #编译ethercat驱动
sudo make install #安装库文件
sudo make modules_install #安装驱动
```



### 3.5 网络配置

1. 安装成功后，ethercat被安装在/opt/etherlab

   ```shell
   cdx@cdx-VirtualBox:~/xenomai/ethercat$ ls /opt/etherlab/
   bin  etc  include  lib  sbin
   ```

2. 做如下复制

   ```shell
   cd /etc
   sudo mkdir sysconfig
   sudo cp /opt/etherlab/etc/sysconfig/ethercat /etc/sysconfig
   sudo cp /opt/etherlab/etc/init.d/ethercat /etc/init.d
   sudo cp /opt/etherlab/etc/ethercat.conf /etc
   ```

3. 修改文件/etc/sysconfig/ethercat和/etc/init.d，在文件中加入`MASTER0_DEVICE=后添加你的网卡的mac地址`和`DEVICE_MODULES=默认的网卡`,如

   ```shell
   sudo vim /etc/sysconfig/ethercat
   sudo vim /etc/ethercat.conf
   ```

   例如按如下修改（文件中有MASTER0_DEVICE=""和DEVICE_MODULES="" ）

   ```shell
   MASTER0_DEVICE="08:00:27:df:7c:81"     
   DEVICE_MODULES="generic"
   ```

4. 执行

   ```shell
   sudo depmod
   ```

   

## 4.配置用户态库和依赖

### 4.1 修改ethercat设备权限

```
cd /etc/udev/rules.d #进入udev rules文件夹
sudo vim 99-ethercat.rules #新建一个ethercat的rule文件
```

在`99-ethercat.rules`文件中添加下面内容

```
KERNEL=="EtherCAT[0-9]", MODE="0777"
```

保存后退出，然后执行`sudo udevadm control --reload-rules` 然后重启电脑。



### 4.2 配置库

将`/opt/etherlab/include`下的2个头文件放入`/usr/local/include`

```shell
sudo cp /opt/etherlab/include/ecrt.h /usr/local/include
sudo cp /opt/etherlab/include/ectty.h /usr/local/include
```

将`/opt/etherlab/lib`下的`libethercat_rtdm.so.1.1.0`放入`/usr/local/lib`

```shell
sudo cp /opt/etherlab/lib/libethercat_rtdm.so.1.1.0 /usr/local/lib
```

将`/opt/etherlab/bin`下的`ethercat`改名为`ethercat-tool`并放入`/usr/local/bin`

```shell
sudo cp -R /opt/etherlab/bin/ethercat /usr/local/bin/ethercat-tool
```

然后执行`ldconfig` 确保`/usr/local/lib`在系统的动态链接库路径里面。

```shell
sudo ldconfig
```



### 4.3 配置cmake

使用`sudo apt-get install cmake`安装 cmake。



## 5.配置实时权限

```
sudo vim /etc/security/limits.conf
```

然后在该文件中添加

```
<username> hard rtprio 99
```

其中99为实时调度的优先级最大为139（139为调度程序本身的优先级）。保存后退出，然后重启电脑。 在terminal查看`ulimit -Hr`是不是为99。



## 6.配置环境变量

1. 在~目录下向.bashrc文件(隐藏文件，可用`ls -a`查看到)末尾添加

   ```shell
   export LD_LIBRARY_PATH="/opt/etherlab/lib:$LD_LIBRARY_PATH"
   export PATH="/opt/etherlab/bin:$PATH"
   export ETHERCAT_ROOT_DIR="/opt/etherlab"
   export ETHERCAT_PATH="/opt/etherlab"
   export PATH="$PATH:$ETHERCAT_PATH/bin:$ETHERCAT_PATH/sbin"
   export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$ETHERCAT_PATH/lib"
   ```

2. 更新.bashrc

   `source ~/.bashrc`

3. `ethercat -help`查看是否成功



## 参考

* https://github.com/ART-robot/ethercat_install
* 《基于Xenomai的EtherCAT主站开发与实时性优化》邱昌华



***



# 运行

## 1.启动主站

```shell
sudo /etc/init.d/ethercat start
```

出现错误：

```shell
 ERROR: could not insert 'ec_master': Operation not permitted
```

找到类似错误[modprobe: ERROR: could not insert 'rtw89pci': Operation not permitted · Issue #67 · lwfinger/rtw89 (github.com)](https://github.com/lwfinger/rtw89/issues/67)和[uefi - Why do I get "Required key not available" when install 3rd party kernel modules or after a kernel upgrade? - Ask Ubuntu](https://askubuntu.com/questions/762254/why-do-i-get-required-key-not-available-when-install-3rd-party-kernel-modules)

尝试使用其中方法解决问题记录如下（失败）：

* 第一次尝试进入BIOS取消Secure Boot

  * 尝试进入BIOS，但按照BIOS display界面右下角提示“Press F12 to select boot device”按F12只进入了***virtualbox temporary boot device***，只能选boot设备；

  * 找到类似“virtualbox ubuntu16.04进入BIOS”问题结果[virtualbox怎么开启进入EFI(bios)-百度经验 (baidu.com)](https://jingyan.baidu.com/article/ca00d56c3bcea7e99eebcfc5.html)，但按照给出的步骤更改设置后，EFI仍更改不了Secure Boot

  * 使用mokutil

    ```shell
    sudo apt install mokutil
    sudo mokutil --disable-validation
    ```

    出错：

    ```shell
    efi variables are not support on this system
    ```

    

* 第二次尝试注册ec_master.ko等kernel module

  * 建立公私钥

    * root身份创建目录存放密钥

      ```shell
      sudo -i
      mkdir /root/module-signing
      cd /root/module-signing
      ```

    * 生成公私钥对，并修改权限

      ```shell
      openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=ec_master"
      chmod 600 MOK.priv
      ```

      req是证书请求的子命令，-newkey rsa:2048 -keyout private_key.pem 表示生成私钥(PKCS8格式)，-outform 参数制定输出格式，-nodes 表示私钥不加密，若不带参数将提示输入密码；
      -x509表示输出证书，-days365 为有效期，此后根据提示输入证书拥有者信息；

      -subj表示自动输入信息

  * 注册相关modules

    * 查看module文件路径

      ```shell
      root@cdx-VirtualBox:~/module-signing# modinfo ec_master
      filename:       /lib/modules/4.19.66-xeno20220529/ethercat/master/ec_master.ko
      version:        1.5.2 unknown
      license:        GPL
      description:    EtherCAT master driver module
      author:         Florian Pose <fp@igh-essen.com>
      srcversion:     A79573A1B0657E01E477856
      depends:        
      retpoline:      Y
      name:           ec_master
      vermagic:       4.19.66-xeno20220529 SMP mod_unload 
      parm:           main_devices:MAC addresses of main devices (array of charp)
      parm:           backup_devices:MAC addresses of backup devices (array of charp)
      parm:           eoe_interfaces:EOE interfaces (array of charp)
      parm:           eoe_autocreate:EOE atuo create mode (bool)
      parm:           debug_level:Debug level (uint)
      parm:           pcap_size:Pcap buffer size (ulong)
      ```

      filename即为下一步`modinfo -n ec_master`结果

    * 在上一步的filename所在的module下的scripts中找到sign-file

      ```shell
      sudo /usr/src/linux-headers-4.19.66-xeno20220529/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n ec_master) 
      ```

      仍然报错：

      ```shell
      efi variables are not support on this system
      ```

* 最终解决办法：参考[command line - Ubuntu - EFI variables are not supported on this system - Ask Ubuntu](https://askubuntu.com/questions/901039/ubuntu-efi-variables-are-not-supported-on-this-system)，按照回复者说法几乎没有办法补救virtualbox的这个问题，只能重新编译安装内核，在配置内核时关闭Module签名需求（Enable loadable module support >Module signature verification 关闭）。

重新编译并启动，成功后：

```shell
cdx@cdx-VirtualBox:~$ sudo /etc/init.d/ethercat start 
Starting EtherCAT master 1.5.2  done
```



注：在***xenomai/Xenomai编译安装.md***->***5.2编译内核***中步骤6已经说明需要***关闭Module签名需求***，按照[command line - Ubuntu - EFI variables are not supported on this system - Ask Ubuntu](https://askubuntu.com/questions/901039/ubuntu-efi-variables-are-not-supported-on-this-system)说法，因为virtualbox的虚拟机特殊性不得不在编译内核的配置阶段处理这个问题；然而对于其他虚拟机或实体机，若出现以上问题，可以尝试使用上面尝试失败的方法或其他可行方法。



另外，关闭和重启：

```shell
sudo /etc/init.d/ethercat stop
sudo /etc/init.d/ethercat restart
```



# 其他

Virtualbox设置BIOS display展示时间等BIOS设置方法，参考[7.8. VBoxManage modifyvm (oracle.com)](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/vboxmanage-modifyvm.html)

```shell
vboxmanage.exe modifyvm ubuntu16.04.7 --bioslogodisplaytime 3000
vboxmanage.exe modifyvm ubuntu16.04.7 --biosbootmenu messageandmenu
```



