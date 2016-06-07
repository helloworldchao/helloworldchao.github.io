---
layout: post
title:  "ubuntu下Android开发环境的安装(JDK+Android Studio+Eclipse+SDK+Intel硬件模拟器加速配置)"
date:   2016-06-07 18:55:00 +0800
categories: Android
permalink: /archivers/ubuntuAndroidStudioAndEclipseSdkInstall
---

最近把windows的Android IDE从eclipse换成了Android Studio，结果在用的时候发现卡成狗，每次编译的时候那个gradle的时间都要好久，然后同学用ubuntu试验了一下跑Android Studio，发现快的飞起，而且用原生的模拟器也非常快，完全可以抛弃genymotion了，于是我果断的把自己的开发环境从windows换成了ubuntu。因为过程有些繁琐，就在这里记录下来，以备以后参考。（以下所有内容基于ubuntu16.04 LTS）

# JDK的安装

第一步首先必须安装JDK，这里直接从Oracle的官网下载JDK的包然后解压至自己的目录，这里我是`/usr/lib/jvm/jdk1.8.0_91`，然后就是在`.bashrc`中配置环境变量，这里我是在自己的home目录下的这个文件中配置的，因为这台电脑只有我自己一个用户，所以无所谓是不是在全局中配置。配置文件内容可以加在文件的最后面，信息如下：

```
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_91
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=$PATH:${JAVA_HOME}/bin

```
然后在命令行输入`JAVA`如果出现JAVA提示信息则说明命令行已经配置成功了。


# Android Studio的安装

软件可以直接去官网下载，当然了，需要自备爬墙方式，这里我选择的是只有Android Studio的压缩包。然后首次打开的时候它会自动检测SDK的位置，因为我这里没有安装SDK所以我选择让它自己下载，它会默认安装在`/home/用户目录/Android/Sdk`文件夹中。

如果机器是64位的在安装的过程中可能就会报错，因为缺少32位的运行库，这时我们可以运行`sudo apt-get install lib32z1 lib32stdc++6`来安装32位库然后选择`Retry`继续安装即可。

安装成功之后如果需要在terminal中使用的话同样需要在`.bashrc`文件中配置环境变量，在文件中添加下列内容即可，第一行是SDK文件夹的实际位置，需要根据自己的实际情况更改：

```
	export ANDROID_HOME=/home/chao/Android/Sdk
	export PATH=${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools:$PATH

```

之后可以在terminal中输入`adb`测试，如果有adb程序的相关提示信息就说明成功了。

如果一切顺利的话接下来就可以直接写程序编译运行了。

# Eclipse的安装

Eclipse可以去官网下载，这里我不知道电脑出了什么问题，只要用最新的MARS版本就会很卡，就是新建一个项目，甚至新建一个类可以一直卡着不动，不在terminal中KILL掉的话就一直停在那里。不得以我选择了LUNA版本。

Eclipse的安装其实和Android Studio差不多，都是直接解压即可，所不同的是在打开Eclipse的时候可能会报找不到JAVA运行环境的错误，这个时候只要配置一下eclipse根目录的eclipse.ini文件就可以了。在这个文件的`-vmargs`选项之前添加下列内容，其中第二行是JDK所在的目录，需要自己根据自己的环境更改：

```

-vm
/usr/lib/jvm/jdk1.8.0_91/bin

```

然后就可以直接打开了。如果需要配置快捷图标以便于在dashboard中搜索的话可以进入`/usr/share/applications`文件夹新建一个`eclipse.desktop`文件，然后在其中输入下列内容，其中`Icon`和`Exec`分别是程序所在的目录的图标位置和执行程序位置，需要根据自己的环境配置:

```
	[Desktop Entry]
	Type=Application
	Name=Eclipse
	Comment=Eclipse Integrated Development Environment
	Icon=/home/chao/eclipse/icon.xpm
	Exec=/home/chao/eclipse/eclipse
	Terminal=false
	Categories=Development;IDE;Java;
	StartupWMClass=Eclipse

```

# Eclipse中的ADT配置

ADT的安装方式有两种，其中一种是将ADT文件下载过来本地安装，另外一种就是直接在网络上安装。

## 网络安装

通过网络安装的话只要选择Eclipse菜单中的`Help->Install New SoftWare->add`，然后在弹出的对话框的Name中填写自己喜欢的名字，Location中填写下载地址`https://dl-ssl.google.com/android/eclipse/`，它会自动下载最近的ADT组件，接下来ADT的配置部分就和windows下没什么区别了。

## 本地安装

通过本地安装的话可以通过`http://dl.google.com/android/ADT-23.0.7.zip`，下载离线ADT文件，其中后面的文件名可以自己更改，我这个版本已经是目前（2016年6月7日）最新版的了（2015年8月更新的版本）。接下来跟网络安装差不多就是选择Eclipse菜单中的`Help->Install New SoftWare->add`，然后在弹出的对话框的Name中填写自己喜欢的名字，然后选择`Archive`，选择下载的ADT文件进行安装即可。


如果网络安装和本地安装的时间非常久可以把`contact all update sites during install to find required software`这个选项的勾给去掉，避免联网获取更新。


# Android模拟器的Intel硬件模拟器加速配置

在ubuntu中安装Intel的这个硬件加速器跟windows下有些不同。这里我直接拷贝了Intel官网的内容[https://software.intel.com/sites/landingpage/tw/speeding-up-the-android-emulator-on-intel-architecture.php](https://software.intel.com/sites/landingpage/tw/speeding-up-the-android-emulator-on-intel-architecture.php)，将官网的步骤列了出来。


下面是执行步骤：

1、执行`sudo apt-get update`检查软件更新  

2、输入`egrep –c ‘(vmx|svm)’ /proc/cpuinfo`检查CPU是否支持硬件虚拟化。如果结果为0，则代表CPU不支持。如果为1及以上，则代表CPU支持。  

3、接下來，除非 KVM 已经安装，否则必须安装KVM。如果检查处理器是否支持 KVM，输入`kvm-ok`。  
如果已安装会有以下信息（信息内容可能不同）：

```
INFO: /dev/kvm exists
KVM acceleration can be used
```

否则会出现（信息内容可能不同）：

```
INFO: KVM is disabled by your BIOS
HINT: Enter your BIOS setup and enable Virtualization Technology (VT),
and then hard poweroff/poweron your system
KVM acceleration can NOT be used
```
需要去BIOS启用Intel VT。  

4、接下来执行`sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils`安装KVM及其组件。  

5、然后执行下面的命令，将用户分别添加到两个群组中：

```
sudo adduser your_user_name kvm
sudo adduser your_user_name libvirtd
```

安装完成后需要重新登录后所有设置才会生效。  

6、如果要测试安装是否成功可以输入`sudo virsh -c qemu:///system list`

7、如果安装成功的话接下来就可以在Android虚拟机中启用Intel的加速了。


