# 第一步 安装ubuntu16.04LTS

安装步骤略

# 第二步 在VirtualBox全局工具中设置虚拟机网络

1. 首先点击**创建**一个新的虚拟网卡，勾选启动DHCP服务器。

![](https://upload-images.jianshu.io/upload_images/11146099-9b397c3c88215305.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 宿主机中在powershell或者cmd中使用ipconfig命令，可以查询宿主机的ip地址。如图中即为**192.168.152.1**

![](https://upload-images.jianshu.io/upload_images/11146099-988230d3ff153eb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 点击虚拟机，右键，设置，网络。网卡1选择连接方式为**仅主机(Host-Only)网络**，界面名称为刚才创建的虚拟网卡名称。该步骤实现了主机和虚拟机网络之间的互相访问。

![](https://upload-images.jianshu.io/upload_images/11146099-d106bd79276f595b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时我们打开虚拟机，输入**ifconfig**命令，可以看到虚拟机ip地址被动态分配为**192.168.152.4**。并且可以ping通宿主机。

![](https://upload-images.jianshu.io/upload_images/11146099-ee6923ebb7e045ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 由于我们的虚拟机需要从网络上下载资源，仅根据第三步的配置是无法联网的，所以我们需要在网络中将网卡2设置为**网络地址转换(NAT)**，此时网卡1为步骤3配置的**仅主机(Host-Only)网络**。

![](https://upload-images.jianshu.io/upload_images/11146099-749d581d3adcad1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置好后我们就的虚拟机网络就配置成功了。

5. 若要通过XShell访问虚拟机，还需要安装openssh-server。安装指令为

```
sudo apt-get install openssh-server
```

之后XShell就可以成功的访问虚拟机了。如下图所示

![](https://upload-images.jianshu.io/upload_images/11146099-92c6003a85f10aba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 第三步 配置新的ubuntu系统

1. 修改apt源，我们使用清华镜像，地址https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/ ，选择ubuntu版本为**16.04 LTS**。

Ubuntu 的软件源配置文件是 /etc/apt/sources.list，首先进行备份

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
```

将原来的文件删除
```
sudo rm -rf /etc/apt/sources.list
```

创建新的文件
```
sudo vi /etc/apt/sources.list
```

将软件源的地址复制进文件并保存

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
```

更新软件包列表
```
sudo apt-get update
```

2. 安装vim, git
```
sudo apt-get install vim git
```


# 第四步 下载PIVX并编译

1. 首先在 ~目录下创建一个projects文件夹，将PIVX clone下来。

```
sudo git clone https://github.com/piaoliangkb/PIVX.git
```

切换到2.3版本

```
sudo git checkout 2.3
```

2. 安装依赖，文档https://github.com/piaoliangkb/PIVX/blob/2.3/doc/build-unix.md

### Debian, Ubuntu构建依赖
```
sudo apt-get install build-essential libtool autotools-dev autoconf pkg-config libssl-dev libboost-all-dev
```

>对于最新版本的PIVX，还需要安装gmp；PIVX2.3则不需要
>```
>sudo apt-get install libgmp-dev
>```

### db4.8


添加db4.8的repo
```
sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update
```
安装db4.8
```
sudo apt-get install libdb4.8-dev libdb4.8++-dev
```

### miniupnp


安装miniupnpc-1.6
```
sudo wget http://miniupnp.tuxfamily.org/files/download.php?file=miniupnpc-1.6.tar.gz
```
下载好后的文件为

![](https://upload-images.jianshu.io/upload_images/11146099-430d797b0276bd1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

改个名字。。
```
sudo mv download.php\?file\=miniupnpc-1.6.tar.gz miniupnpc-1.6.tar.gz
```

build miniupnpc
```
tar -xzvf miniupnpc-1.6.tar.gz
cd miniupnpc-1.6
make
sudo make install
```

### 安装BerkeleyDB

```
PIVX_ROOT=$(pwd)

BDB_PREFIX="${PIVX_ROOT}/db4"
mkdir -p $BDB_PREFIX

sudo wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz'
echo '12edc0df75bf9abd7f82f821795bcee50f42cb2e5f76a6a281b85732798364ef  db-4.8.30.NC.tar.gz' | sha256sum -c
tar -xzvf db-4.8.30.NC.tar.gz

cd db-4.8.30.NC/build_unix/
../dist/configure --enable-cxx --disable-shared --with-pic --prefix=$BDB_PREFIX
```

### to build PIVX

```
cd $PIVX_ROOT

./autogen.sh
./configure LDFLAGS="-L${BDB_PREFIX}/lib/" CPPFLAGS="-I${BDB_PREFIX}/include/" --without-gui
# 无gui方式进行配置，上方也没有安装gui的依赖库
make
make install

```

