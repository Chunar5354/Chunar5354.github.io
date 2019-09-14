---
layout: article
title: Play With Raspberry Pi
tags: PaspberryPi Linux
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

关于树莓派的基本使用，基于官方的Raspbian系统

<!--more-->

# 硬件准备篇

首先树莓派是一定的啦，然后还需要需要PC机、TF(SD)卡，和树莓派的电源（树莓派标准是2.5A，一般的手机充电器可能不好用）

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

> country=CN  
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev  
update_config=1  
network={  
  ssid="xxxxxx" // 这里把 xxx 替换为 WiFi 的名字  
  psk="xxxxxxx" // 这里是 WiFi 的密码  
  priority=x    // 当环境中存在多个无线网络时，在这里用阿拉伯数字来表明 WiFi 的优先级，数字越大，优先级越高  
  id_str="xxxxxx" // 当前网络的备注，passwd_home 之类的  
  }  

这样在树莓派插上电之后就可以自动连接wifi，之后可以登录路由器管理界面查看树莓派的ip地址,得到ip地址之后，打开Putty，输入地址，就可以连接树莓派
进入之后首先是输入用户名和密码的界面，用户名默认为'pi'，密码默认为'raspberry
输入之后就进入了树莓派命令行啦。

# 初始化操作

### 1.在终端中输入以下内容：
`sudo raspi-config`
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
在终端中输入以下内容进入 `/etc/apt/` 目录：  
`cd /etc/apt`

首先备份原有文件：
`sudo cp sources.list sources.list.bak`

之后打开文件进行修改：
`sudo nano sources.list`

将原有内容全部注释掉，添加以下内容：
`deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ stretch main contrib non-free rpi`  （也可以使用其他源）

保存退出，进入下级目录：
`cd sources.list.d`

编辑之前先备份 `raspi.list` 文件：  
`sudo cp raspi.list raspi.list.bak`  
然后编辑文件：  
`sudo nano raspi.list`

将原有内容全部注释掉，添加以下内容：
`deb https://mirrors.ustc.edu.cn/archive.raspberrypi.org/ stretch main ui`（要把这个和上一个修改的内容对应）

- 2019年6月之后，Raspbian官方发布了新的buster版本系统，更换源的时候要将上面的`stretch`替换成`buster`

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
`sudo apt-get update`  
`sudo apt-get upgrade`

### 4.可以设置一下静态ip，这样就不用每次都查看ip（不过更换路由器的时候最好改回来）：

这一步需要编辑 `/etc/dhcpcd.conf` 文件：
`sudo nano /etc/dhcpcd.conf`
（nano最基本操作：Ctrl+o 保存，Ctrl+x退出）

可以看到文件中也提供了相关说明，这里的 interface eth0 代表以太网连接，wlan0 代表 WiFi 连接
比如之前树莓派的随机IP地址为 192.168.1.103，根据自己网络状态按如下方式修改即可

>   interface eth0  
   static ip_address=192.168.1.222/24  
   static routers=192.168.1.1  
   static domain_name_servers=192.168.1.1 1.1.1.1  

>   interface wlan0  
   static ip_address=192.168.1.222/24  
   static routers=192.168.1.1  
   static domain_name_servers=192.168.1.1 1.1.1.1  )  

这样树莓派的基本设置就做好啦，之后想怎么折腾就随自己了。

# 远程桌面连接

想要在图形界面看看树莓派的话，用远程桌面就可以了，`vnc`当然也行，需要在树莓派上安装xrdp：

>sudo apt-get install xrdp  
sudo /etc/init.d/xrdp start // 启动xrdp服务  
sudo update-rc.d xrdp defaults // 将xrdp加入到系统默认启动服务  

然后（不做这两步可能会出错）：  

>sudo apt-get purge tightvnc xrdp  
sudo apt-get install tightvncserver xrdp  

最后重启xrdp：

>sudo /etc/init.d/xrdp restart

然后就可以使用电脑上的远程桌面，输入树莓派的IP来连接树莓派，查看树莓派的图形界面了。

# 常用指令整理

## 基本操作
>sudo shutdown -h now // 关机  
sudo shutdown -r now // 重启   
sudo apt-get update // 更新系统  
sudo qpt-get upgrade // 更新软件包  
sudo apt-get install // 可以安装一些东西  
mkdir // 创建文件夹  
rmdir // 删除文件夹  
rm // 删除文件  
rm -rf // 强制删除文件夹（针对非空文件夹）  
vi // 创建文件，并使用vi编辑，或者用vi打开已有的文件  
cp a b // 把a复制到b   
cp -r a b // 把a复制到b，如果b不存在就创建它（-r都有强制的意味）  
mv a b // 把a移动到b  
cat // 直接打开文件，不能编辑  
cd path // 进入目录  
cd .. // 返回上一级目录  
cd ~ // 进入/home/pi文件夹的快捷方式  
ls // 显示当前目录下的多有文件  
ls -l // 查看文件详细情况  
df -h // 查看挂载硬盘情况  
df -hl // 查看硬盘使用情况  
wget 网址 // 可以在网上下载  
tar // 解压  

## 挂载外部设备时用到

 >sudo umount /media/piusb（自己设置的文件名） // 卸载外部存储设备  
sudo fdisk -l // 查看硬件信息  
sudo mount /dev/sda1 /media/piusb // 挂载（mount为挂载命令，中间是要挂载的文件，最后是挂载目标文件）  
sudo blkid // 查看储存设备的UUID（通用唯一识别码）  

# vim一些操作

基本上vi可以分为三种状态，分别是：  
1.命令模式：进行文档的操作，命令基本是针对这个模式  
2.插入模式：进行文档的编辑，就只要写写写就行啦  
3.底行模式（一般底行模式也算是命令模式）  

## 对文件进行操作

命令模式下：  
 
>:w // 保存文件  
:w a.b // 保存到a.b文件  
:q // 退出编辑器  
:q! // 强制退出，不保存  
:wq // 保存退出  

## 由命令模式进入插入模式

命令模式下：  

> a  // 在当前光标位置的右边添加文本  
i  // 在当前光标位置的左边添加文本  
A  // 在当前行的末尾位置添加文本  
I  // 在当前行的开始处添加文本(非空字符的行首)  
O  // 在当前行的上面新建一行  
o  // 在当前行的下面新建一行    
R  // 替换(覆盖)当前光标位置及后面的若干文本  
J  // 合并光标所在行及下一行为一行(依然在命令模式)  

## 移动光标类

命令模式下（方向键和h（左）j（下）k（上）l（右）：  

>Ctrl+b // 屏幕往后移动一页  
Ctrl+f // 屏幕往前移动一页  
Ctrl+u // 屏幕往后移动半页  
Ctrl+d // 屏幕往前移动半页  
数字 0 // 移到当前行的开头  
G // 移动到文章的最后  
$ // 移动到光标所在行的行尾  
^ // 移动到光标所在行的行首  
w // 光标跳到下个字的开头  
e // 光标跳到下个字的字尾  
b // 光标回到上个字的开头  
#l // 光标往后移的第#个位置，如：5l,56l  

## 删除类

命令模式下：  

>x // 每按一次，删除光标所在位置的后面一个字符  
nx // 删除光标所在位置的后面n个字符  
X // 每按一次，删除光标所在位置的前面一个字符  
nX // 删除光标所在位置的前面n个字符  
dd // 删除光标所在行  
ndd // 从光标所在行开始删除n行  

## 复制粘贴类

命令模式下：  

>yy // 将当前行复制，也可以用“ayy”复制，a为缓存区，可以有a到z多个缓存区来完成多个复制任务    
nyy // n是数字，将当前行向下n行复制到缓存区，也可以在前面加a到z，同理  
yw // 复制从光标开始到词尾的字符  
nyw // 复制从光标开始的n个单词  
y^ // 复制从光标到行首的内容  
y$ // 复制从光标到行尾的内容  
p // 把缓存区的内容粘贴在光标后，如果用a~z定义了多个缓存区，粘贴的时候也要在p前面加上相应的字母  
P（大写的哦） // 粘贴在光标前  

tips：
在vi里面不要用小键盘  
在输入的时候就只想着输入就得了，想要操作什么的先按esc  

# 使用samba实现局域网文件共享

首先安装samba  

>sudo apt-get update  
sudo apt-get install samba  
sudo apt-get install samba-common-bin  

然后配置文件

>sudo vi /etc/samba/smb.conf

在末尾添加以下内容：  

>[name] // 你自己起的共享文件夹名称  
comment = name explain // 文件夹说明  
path = /home/pi/name // 共享文件夹的路径，最好建在/home/pi/目录下  
read only = no // 不是只读  
create mask = 0777 // 创建文件权限  
directory mask = 0777 // 创建文件夹权限  
guest ok = yes // 无需密码访问，想要设置密码的改成no  
browseable = yes // 可见  

之后**很重要**的一步：在/home/pi/里面创建一个上面path里面你自己设置的文件夹：

>cd /home/pi  
mkdir name  

然后重启samba：

>sudo samba restart

设置密码：

>sudo smbpasswd -a pi  

之后出入两次密码设置成功  

还可以设置samba开机启动：  

>sudo update-rc.d samba defaults  

samba就设置好啦，可以在电脑的文件夹地址框中输入'//192.168.x.x'(树莓派的ip)进入共享文件夹，以后在树莓派和电脑之间互传文件就方便多啦。



