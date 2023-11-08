# apt包管理工具

[ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

```shell
# 软件源文件
/etc/apt/sources.list
```


| 操作                                                     | 指令                      |
| -------------------------------------------------------- | ------------------------- |
| 安装                                                     | apt-get install xxx       |
| 从软件源获取最新的软件列表保存在本地                     | apt-get update            |
| 从软件源检查当前安装的所有软件是否为最新的，可以选择更新 | apt-get upgrade           |
| 更新某个软件                                             | apt-get upgrade xxx       |
| 列出可更新的软件                                         | apt list --upgradable     |
| 列出可用指令                                             | apt-get help              |
| 删除软件                                                 | apt-get remove package    |
| 查看已安装版本                                           | dpkg -l 软件              |
|                                                          | sudo apt install -f       |
| 查看软件可安装版本                                       | apt-cache madison 软件    |
|                                                          | apt-cache depends .deb    |
| 安装指定版本软件                                         | apt-get install 软件=版本 |
|                                                          |                           |
| 查看依赖关系                                             | apt depends 软件          |
| 可以连同依赖一起下载的工具                               | apt install rdepends      |
| 只下载deb不安装                                          | apt download *.deb        |
| 使用deb安装包安装                                        | sudo dpkg -i deb安装包    |
| 卸载deb                                                  | dpkg -r *.deb             |
| 修复损坏的包                                             | apt --fix-broken install  |
| 从URL下载                                                | wget URL                  |

下载依赖包

|                                |                                                              |
| ------------------------------ | ------------------------------------------------------------ |
| 只下载安装包不安装             | sudo apt-get install --download-only vim                     |
| 安装工具                       | sudo apt install apt-rdepends                                |
| 下载所有依赖项                 | apt download $(apt-rdepends vim grep -v "^ ")                |
| 如果出现如下错误，使用下条指令 | E: Can't select candidate version from package debconf-2.0 as it has no candidate |

```shell
apt-get download $(apt-rdepends vim | grep -v "^ " | sed 's/debconf-2.0/debconf/g')
```

### 安装ssh

```shell
#ssh和openssh-server好像是同一个库，安装一个就行，下次装机可以试试，验证下
apt install ssh
apt install openssh-server
systemctl enable ssh
/etc/init.d/ssh restart
```

### 安装telnet

[在Ubuntu上安装Telnet Server服务 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/503294216)

```shell
apt install xinetd
apt install telnetd
nano /etc/xinetd.d/telnet
#写入如下配置
service telnet
{
    disable = no
    flags           = REUSE
    socket_type     = stream
    wait            = no
    user            = root
    server          = /usr/sbin/telnetd
    server_args     = -h
    log_on_failure  += USERID
}
/etc/init.d/xinetd restart
```

### 安装ftp

```shell
apt install vsftpd
nano /etc/vsftpd.conf
#打开读写权限
/etc/init.d/vsftpd restart
```

### 环境变量

/etc/ld.so.config文件中保存着环境变量

系统目录/etc/profile文件开启运行，加入export语句编辑后重启生效，所有用户均生效

系统目录/etc/environment文件保存着环境变量的信息，直接修改重启生效

创建进程后，进程可以访问环境变量

`/etc/rc.local`开机启动脚本，可以加入自己的服务脚本，实现开机自启

### 使用iso安装

[(77条消息) Ubunt 20.04 使用CDROM或ISO作为安装源-CSDN博客](https://blog.csdn.net/alfiy/article/details/123174391?ydreferer=aHR0cHM6Ly9jbi5iaW5nLmNvbS8%3D)

只挂载一个设备默认就会挂载iso了

[(77条消息) Ubuntu挂载iso文件和配置apt本地源_ubuntu挂载iso镜像_liuccn的博客-CSDN博客](https://blog.csdn.net/lc_2014c/article/details/84190765)

# 网络

### 设置静态ip

[(39条消息) 银河麒麟设置静态IP_银河麒麟配置ip地址_Mr_青青子衿的博客-CSDN博客](https://blog.csdn.net/qq_41587516/article/details/119677953)

```shell
#修改配置
nano /etc/network/interfaces
auto ens32
iface ens32 inet static
address 192.168.154.129
netmask 255.255.255.0
gateway 192.168.154.2

#重启网络
service networking restart
```

