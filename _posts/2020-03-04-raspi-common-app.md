---
layout: article
title: Raspberry Pi Common Software
tags: PaspberryPi Linux
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

树莓派的一些常用软件，帮助用户更好的使用它

<!--more-->

# 远程桌面连接

没有外接屏幕又想使用树莓派的图形界面，用远程桌面就可以了，需要在树莓派上安装xrdp：

```
# sudo apt-get install xrdp  
# sudo /etc/init.d/xrdp start       // 启动xrdp服务  
# sudo update-rc.d xrdp defaults    // 将xrdp加入到系统默认启动服务
```

然后（不做这两步可能会出错）：  

```
# sudo apt-get purge tightvnc xrdp  
# sudo apt-get install tightvncserver xrdp 
```

最后重启xrdp：

```
# sudo /etc/init.d/xrdp restart
```

然后就可以使用电脑上的远程桌面，输入树莓派的IP来连接树莓派，查看树莓派的图形界面了。


# samba共享文件按

首先安装samba  

```
# sudo apt-get update  
# sudo apt-get install samba  
# sudo apt-get install samba-common-bin  
```

然后修改配置文件

```
# sudo vi /etc/samba/smb.conf
```

在末尾添加以下内容：  

```
[name] // 你自己起的共享文件夹名称  
    comment = name explain // 文件夹说明  
    path = /home/pi/name // 共享文件夹的路径，最好建在/home/pi/目录下  
    read only = no // 不是只读  
    create mask = 0777 // 创建文件权限  
    directory mask = 0777 // 创建文件夹权限  
    guest ok = yes // 无需密码访问，想要设置密码的改成no  
    browseable = yes // 可见  
```

之后**很重要**的一步：在/home/pi/里面创建一个上面path里面你自己设置的文件夹：

```  
# mkdir /home/pi/name
```

然后重启samba：

```
# sudo samba restart
```

设置密码（如果需要的话）：

```
# sudo smbpasswd -a pi  
```

之后出入两次密码设置成功  

还可以设置samba开机启动：  

```
# sudo update-rc.d samba defaults  
```

之后就可以在电脑的文件夹地址框中输入'//IP'(树莓派的ip)进入共享文件夹，在树莓派和电脑之间互传文件就方便多啦。


# 设置中文显示

树莓派默认是不支持中文字体的，需要手动配置

首先要安装中文字体库：`sudo apt-get install ttf-wqy-zenhei`

然后可以选择安装中文输入法（在桌面时可以使用）：`sudo apt-get install scim-pinyin`

然后进入配置：`sudo raspi-config`
- 进入`localization options`，再进入`change_locale`，在里面找到`en_GB.UTF-8 UTF-8`，按空格将其前面的`*`去掉
- 找到`en_US.UTF-8 UTF-8`，`zh_CN.UTF-8 UTF-8`和`zh_CN.GBK GBK`，按空格给它们打上`*`号
- 按`OK`，之后找到`zh_CN UTF-8`按回车，完成设置
- 重启`sudo reboot`

在树莓派的桌面可以使用`ctrl+space`切换输入法


# 修改键盘布局

由于树莓派默认使用英国的键盘布局，而中国键盘使用的是美国布局，所以有时会出现显示的字符与输入不匹配

修改方法：
- 在终端输如`sudo raspi-config`进入配置界面
- 进入`Localisation Options`
- 进入`Change Keyboard Layout`
- 进入`Generic 104-key PC`
- 现在应该都是UK的选项，选择`Other`进入
- 选择里面的`English(US)`
- 回到上一个界面现在应该已经都是US的选项了，选择`English(US) - English(US, alernative international)` 
- 之后一路确认，完成配置，重启

如果上面的方法还是不好用的话，就要在图形界面中修改：
- 右上角在输入法的地方右键，选择`Config Current input Method`
- 然后点击左下角的`+`，添加`English(US)`
- 然后右键输入法图标，在`Virtual Keyboard`中更改成`English(US)`
