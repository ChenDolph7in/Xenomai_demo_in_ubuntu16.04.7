# IGH_EtherCAT主站环境搭建

参考：

* https://github.com/ART-robot/ethercat_install
* 《基于Xenomai的EtherCAT主站开发与实时性优化》邱昌华



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

1. 首先查看网卡`ifconfig`

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

3. 记下driver类型为e1000



### 3.3 进行configure配置生成规则文件Makefile

进入目录/xenomai/ethercat

调用./configure生成规则文件，使用选项指定网卡类型、内核路径和安装路径等

```shell
./configure --with-linux-dir=/home/cdx/xenomai/4.19.66/linux-4.19.66 --enable-generic=yes --enable-8139too=no --enable-e1000=yes --enable-e1000e=no --enable-r8169=no --enable-rtdm=yes --enable-cycles=yes --enable-hrtimer=yes --with-xenomai-dir=/usr/xenomai --prefix="/opt/etherlab"
```

注：

* 注意网卡类型，否则容易出错：

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

* 最终解决办法：参考[command line - Ubuntu - EFI variables are not supported on this system - Ask Ubuntu](https://askubuntu.com/questions/901039/ubuntu-efi-variables-are-not-supported-on-this-system)，几乎没有办法补救virtualbox的这个问题，只能重新编译安装内核，配置内核时关闭Module签名需求（Enable loadable module support >Module signature verification 关闭）。

启动成功后：

```shell
cdx@cdx-VirtualBox:~$ sudo /etc/init.d/ethercat start 
Starting EtherCAT master 1.5.2  done
```



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



