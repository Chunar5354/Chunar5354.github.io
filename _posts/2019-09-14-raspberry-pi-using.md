---
layout: article
title: Raspberry Pi Initial Configuration
tags: PaspberryPi Linux
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

树莓派的开机配置，基于官方的Raspbian系统

<!--more-->

# 硬件准备篇

首先树莓派是一定的啦，然后还需要需要PC机、TF(SD)卡，和树莓派的电源（树莓派4B已经达到了5V3A，一般的手机充电器可能不好用）

# 软件准备篇

在电脑上需要下载的软件：
## 1.Putty
下载链接：[Putty](https://www.putty.org)
用来连接ssh，可以用电脑远程操作树莓派

## 2.一个格式化SD卡的软件
这个我使用的是 SD Memory Card formatter
下载链接：[SD Memory Card formatter](https://www.sdcard.org)
用它来将SD卡格式化，为写入系统做准备

## 3.烧录软件
这个我用的是 Win32 Disk Imager
下载链接：[Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/)
用它来将树莓派的系统文件写入内存卡

## 4.树莓派系统
官方网站：https://www.raspberrypi.org/downloads/

# 第一次开机

如果有显示器的话直接插上怼就可以了，没有的话就需要ssh连接用电脑来操作，首先要知道树莓派的ip地址，需要有一个路由器。

在把内存卡插入树莓派之前，要先在内存卡的boot区中添加两个文件

- 1.出于安全的考量，树莓派的系统默认关闭了 SSH 选项，为了开启 SSH，需要新建一个
文本文件，命名为 ssh 后去掉扩展名变成空白文件

- 2.再新建一个文本文件重命名为 `wpa_supplicant.conf`，大多数文本编辑器可以打开，输入以下内容：

```
country=CN  
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev  
update_config=1  
network={  
  ssid="xxxxxx" // 这里把 xxx 替换为 WiFi 的名字  
  psk="xxxxxxx" // 这里是 WiFi 的密码  
  priority=x    // 当环境中存在多个无线网络时，在这里用阿拉伯数字来表明 WiFi 的优先级，数字越大，优先级越高  
  id_str="xxxxxx" // 当前网络的备注，passwd_home 之类的  
  }  
```

这样在树莓派插上电之后就可以自动连接wifi，之后可以登录路由器管理界面查看树莓派的ip地址,得到ip地址之后，打开Putty，输入地址，就可以连接树莓派

进入之后首先是输入用户名和密码的界面，用户名默认为`pi`，密码默认为`raspberry`

输入之后就进入了树莓派命令行啦。

# 初始化操作

### 1.在终端中输入以下内容：
```
# sudo raspi-config
```

在接下来出现的界面使用方向键与回车操作：

- 选择 `Change User Password` 后输入新设定的密码 // 重设密码，没啥好说的  
- (可选) 选择 `Network Options` 后选择 `Hostname`，给树莓派起一个中意的名字，注意不能包含空格及大写字母  
- 选择 `Localisation Options`:   
  - 选择 `Change Timezone`，选择其中的 `Asia Shanghai` // 选择树莓派的时间  
  - 选择 `Change WiFi Country`，向下找到 `CN China` 选定  
- 选择 `Advanced Options` 后选择 `Expand Filesystem` // 扩展内存空间  
完成以上工作后重启树莓派等候片刻，重新连接。  

### 2.然后要更换国内源 // 不换的话很多功能（比如vnc）用不了：

这一步需要修改 `sources.list` 文件  
进入 `/etc/apt/` 目录：  
```
# cd /etc/apt
```

首先备份原有文件：
```
# sudo cp sources.list sources.list.bak
```

之后打开文件进行修改：
```
# sudo vi sources.list
```

将原有内容全部注释掉，添加以下内容 （也可以使用其他源）：
```
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ stretch main contrib non-free rpi
```
- 2019年6月之后，Raspbian官方发布了新的buster版本系统，更换源的时候要将上面的`stretch`替换成`buster`

保存退出，进入下级目录：
```
# cd sources.list.d
```

编辑之前先备份 `raspi.list` 文件：  
```
# sudo cp raspi.list raspi.list.bak
```  
然后编辑文件（**如果是buster系统的话可以不修改这个文件**）：  
```
# sudo vi raspi.list
```

将原有内容全部注释掉，添加以下内容（要把这个和上一个修改的内容对应）：
```
deb https://mirrors.ustc.edu.cn/archive.raspberrypi.org/ stretch main ui
```


其他一些源:
> 中国科学技术大学
Raspbian http://mirrors.ustc.edu.cn/raspbian/raspbian/

> 阿里云
Raspbian http://mirrors.aliyun.com/raspbian/raspbian/

> 清华大学
Raspbian http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/

> 华中科技大学
Raspbian http://mirrors.hustunique.com/raspbian/raspbian/
Arch Linux ARM http://mirrors.hustunique.com/archlinuxarm/

> 华南农业大学（华南用户）
Raspbian http://mirrors.scau.edu.cn/raspbian/

> 大连东软信息学院源（北方用户）
Raspbian http://mirrors.neusoft.edu.cn/raspbian/raspbian/

> 重庆大学源（中西部用户）
Raspbian http://mirrors.cqu.edu.cn/Raspbian/raspbian/

> 中山大学 已跳转至中国科学技术大学源
Raspbian http://mirror.sysu.edu.cn/raspbian/raspbian/

> 新加坡国立大学
Raspbian http://mirror.nus.edu.sg/raspbian/raspbian

> 牛津大学
Raspbian http://mirror.ox.ac.uk/sites/archive.raspbian.org/archive/raspbian/

> 韩国KAIST大学
Raspbian http://ftp.kaist.ac.kr/raspbian/raspbian/

### 3.之后更新一下系统和软件包：  
```
# sudo apt-get update
# sudo apt-get upgrade
```

### 4.可以设置一下静态ip，这样就不用每次都查看ip（不过更换路由器的时候最好改回来）：

这一步需要编辑 `/etc/dhcpcd.conf` 文件：
```
# sudo vi /etc/dhcpcd.conf
```

可以看到文件中也提供了相关说明，这里的 interface eth0 代表以太网连接，wlan0 代表 WiFi 连接
比如之前树莓派的随机IP地址为 192.168.1.103，根据自己网络状态按如下方式修改即可

```
interface eth0  
static ip_address=192.168.1.222/24  
static routers=192.168.1.1  
static domain_name_servers=192.168.1.1 1.1.1.1  

interface wlan0  
static ip_address=192.168.1.222/24  
static routers=192.168.1.1  
static domain_name_servers=192.168.1.1 1.1.1.1  )
```

这样一块新的树莓派就配置好了

