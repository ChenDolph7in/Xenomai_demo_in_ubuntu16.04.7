# 虚拟机中Ubuntu16.04.7下Xenomai3.2.1编译&安装&配置步骤（Linux-4.16.99）

## 环境配置

* Oracle VM VirtualBox 6.1.18
* ubuntu-16.04.7-desktop-amd64
* Xshell 7 7.0.0076
* Xftp 7 7.0.0074
* Visual Studio Code 1.67.2



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

  ***<font color=red>！！！注意：编译打包内核时虚拟机磁盘可用空间可能超过20G，如果新安装虚拟机，磁盘空间最好分配大一点！！！</font>***

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

### 5.准备Linux内核

#### 5.1Linux内核打ipipe补丁

在/xenomai/4.19.66/linux-4.19.66目录下打开终端执行：

`../../xenomai-v3.2.1/scripts/prepare-kernel.sh --ipipe=../ipipe-core-4.19.66-x86-6.patch --arch=x86_64`

出现以下问题注意：

* patch unexpectedly ends in middle of line：patch文件以LF结束一行，以CRLF为行尾序列的需要去掉CR
* patch unexpectedly ends in middle of line：不知原因

#### 5.2配置内核

在/xenomai/4.19.66/linux-4.19.66目录下打开终端执行（由于需要足够的空间打开图形化界面，终端尺寸要够大）：

* `make menuconfig`
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

#### 5.3编译和打包内核和头文件

编译和安装分离，在高性能机器编译并打包，转义并安装到其他的机器

1. 在/xenomai/4.19.66/linux-4.19.66目录下创建文件REPORTING-BUGS，否则头文件会编译错误，报错<font color=red>无法获取REPORT-BUGS的文件状态stat</font>

   `sudo touch REPORTING-BUGS`

2. 在/xenomai/4.19.66/linux-4.19.66目录下打开终端执行(<font color=red>耗时较长，根据主机配置耗时可能长达2~3h。注意修改自动息屏/睡眠时间；以及最好不要使用SSH终端运行，防止链接中断导致编译中断；再者注意磁盘剩余空间，根据内核配置需求中间文件可能会占用不下于20G的磁盘空间</font>)

   * `sudo make-kpkg --initrd --revision 01 --append-to-version -xeno20220529 kernel_image kernel_headers`

3. 编译完成后，/xenomai/4.19.66/目录下出现编译完的***linux-headers-4.19.66-xeno20220529_01_amd64.deb***和***linux-image-4.19.66-xeno20220529_01_amd64.deb***

#### 6.编译和安装内核

1. 在/xenomai/4.14.13目录下打开终端执行以下指令安装内核和头

   `sudo dpkg -i linux-headers-4.19.66-xeno20220529_01_amd64.deb linux-image-4.19.66-xeno20220529_01_amd64.deb`

2. 查看已安装的内核镜像和头，验证是否已经安装成功（可跳过）

   ```shell
   dpkg -l | grep linux-image
   dpkg -l | grep linux-headers
   ```

3. 删除已安装的镜像和头（需要的话）

   ```shell
   sudo dpkg --purge linux-headers-4.19.66-xeno20220529 
   sudo dpkg --purge linux-image-4.19.66-xeno20220529
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

#### 7.配置启动界面，在启动菜单选择内核

1. 修改GRUB配置信息`sudo vim /etc/default/grub`

   GRUB_HIDDEN_TIMEOUT=0 注释掉
   GRUB_TIMEOUT=3
   GRUB_CMDLINE_LINUX_DEFAULT="text"

   具体含义参考https://blog.csdn.net/piaopiaopiaopiaopiao/article/details/11772053

2. 更新GRUB配置信息`sudo update-grub`，然后重启(reboot)

3. 在启动界面选择：***ubuntu高级选项 > 新安装的内核***

4. `uname -a`查看是否选择成功

#### 8.测试Xenomai

```shell
cd /usr/xenomai/bin
sudo ./latency
```

```shell
sudo apt-get install stress
stress -v -c 8 -i 10 -d 8
```

#### 9.修复latency负延迟问题

如果在latency中观察到last best项为负值，说明xenomai需要校准。执行：

```shell
sudo -s
echo 0 > /proc/xenomai/latency
```

#### 10.更新环境变量

1. 在~目录下向.bashrc文件(隐藏文件，可用`ls -a`查看到)末尾添加

   `export LD_LIBRARY_PATH="/usr/xenomai/lib:$LD_LIBRARY_PATH"
   export PATH=/usr/xenomai/bin:$PATH
   export XENOMAI_ROOT_DIR=/usr/xenomai
   export XENOMAI_PATH=/usr/xenomai
   export PATH=$PATH:$XENOMAI_PATH/bin:$XENOMAI_PATH/sbin
   export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:$XENOMAI_PATH/lib/pkgconfig
   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$XENOMAI_PATH/lib
   export OROCOS_TARGET=xenomai`

2. 更新.bashrc

   `source ~/.bashrc`

#### 11.添加xenomai的链接库路径

1. 在/etc/ld.so.conf.d目录下新建xenomai.conf

   ```shell
   cd /etc/ld.so.conf.d
   sudo touch xenomai.conf
   ```

2. 打开xenomai.conf并添加"/usr/xenomai/lib"

   `sudo gedit xenomai.conf`

3. 使更改生效

   `sudo /sbin/ldconfig -v`

