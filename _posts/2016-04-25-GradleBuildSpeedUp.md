---
layout: post
title:  "加速Gradle build running"
date:   2016-04-25 21:00:00 +0800
categories: Gradle
permalink: /archivers/GradleBuildRunningSpeedUp
---

最近在使用Android Studio调试Android程序的时候发现在Gradle build running的阶段总是很慢，一个小的测试Demo程序也需要大约30秒左右的时间，虽然看起来不多，但是等待的时间总是很长，所以我通过搜索找到了几个方法来加速这个过程，为了避免以后忘记就在这里记录下来。

# 设置gradle.properties

在gradle.properties文件中设置下面几个项目：

> org.gradle.daemon=true  
> org.gradle.parallel=true  
> org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

# 设置gradle为离线编译

通常来说在经过了这些设置后gradle的build时间能够明显缩短。但是如果还是不行的话还有一个办法！
在如图划横线的地方打上勾，设置gradle离线编译。通常在经过了这几步之后就可以解决问题了。

![OfflineWord](/img/GradleBuildRunning.PNG)
