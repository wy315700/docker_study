Docker 的安装非常的简单，下面以Ubuntu，Centos和 Windows为例说明如何安装Docker

## Ubuntu

Docker支持以下的Ubuntu版本

 - *Ubuntu Trusty 14.04 (LTS) (64-bit)*
 - *Ubuntu Precise 12.04 (LTS) (64-bit)*
 - *Ubuntu Raring 13.04 and Saucy 13.10 (64
   bit)*

以最新版14.04为例，14.04使用了3.13.0内核，原生支持LXC，所以可以直接安装Docker

> **注意**:
> Ubuntu (以及 Debian) 包含了一个 KDE3/GNOME2 包 ``docker``, 所以
> 这里我们使用 ``docker.io`` 来代替docker

使用内置的apt-get命令来安装

    $ sudo apt-get update
    $ sudo apt-get install docker.io
    $ sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker
    $ sudo sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker.io
    $ source /etc/bash_completion.d/docker.io

当然，上面安装的Docker版本可能不是最新的，如果要获取最新版本，则必须使用以下步骤

首先，必须保证apt能处理 `https` 的 URLs: 也就是说 文件 `/usr/lib/apt/methods/https` 必须存在。
如果文件不存在，则必须安装 `apt-transport-https`.

    [ -e /usr/lib/apt/methods/https ] || {
      apt-get update
      apt-get install apt-transport-https
    }

然后，导入docker官方的库的密钥

    $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9

然后就可以使用apt-get安装 `lxc-docker` 软件

    $ sudo sh -c "echo deb https://get.docker.com/ubuntu docker main\
    > /etc/apt/sources.list.d/docker.list"
    $ sudo apt-get update
    $ sudo apt-get install lxc-docker

> **注意**:
>
> Docker官方也提供了一个脚本来获取最新版的docker
>
>     $ curl -sSL https://get.docker.com/ubuntu/ | sudo sh

## CentOS

Docker支持[CentOS6](http://www.centos.org)以及更高版本，内核版本必须高于2.6.32-431。

由于Docker的限制，Docker只能安装在 **64 位** 的系统上。

### CentOS-7

CentOS-7原生就包含了Docker，所以只需要使用yum进行安装即可

    $ sudo yum install docker

### CentOS-6

对于CentOS-6, Docker被包含在了[Extra Packages for Enterprise Linux (EPEL)](https://fedoraproject.org/wiki/EPEL)里，
所以只需要把EPEL装上即可。

紧接着就可以使用yum安装docker了

    $ sudo yum install docker-io

### 启动服务

CentOS下的docker是作为一项系统服务安装的，所以我们需要启动服务才可以使用。

    $ sudo service docker start

如果想要docker在开机的时候自动启动

    $ sudo chkconfig docker on

## Windows

以上都是在Linux下的安装，docker在Linux下都是使用的LXC容器的方法运行，但是在Windows下运行原理有点不太一样，由于Windows并不支持LXC容器，所以Docker在Windows下是采用VirtualBOX虚拟一个Linux的基础上运行的。

### 安装

1. 下载最新版的 [Docker for Windows Installer](https://github.com/boot2docker/windows-installer/releases)

2. 运行安装包，会自动安装 VirtualBox, MSYS-git, the boot2docker Linux ISO,
和 Boot2Docker management tool.

3. 运行桌面上的 `Boot2Docker Start` 会自动初始化docker,已经启动一个docker的shell。

### 升级


1. 下载最新版的 [Docker for Windows Installer](https://github.com/boot2docker/windows-installer/releases)

2. 运行安装包，会自动升级 Boot2Docker management tool.

3. 使用以下命令更新虚拟机

        boot2docker stop
        boot2docker download
        boot2docker start

### boot2docker命令

    $ ./boot2docker
    Usage: ./boot2docker [<options>] {help|init|up|ssh|save|down|poweroff|reset|restart|config|status|info|ip|delete|download|version} [<args>]

