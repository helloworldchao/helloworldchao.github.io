---
layout: post
title:  "Android6.0权限请求--第一行代码笔记"
date:   2016-05-08 14:25:00 +0800
categories: Android Note
permalink: /archivers/RequestingPermissionsAtRunTime
---



最近在学习ContentProvider的时候需要获取系统联系人的信息，但是程序在运行的时候出现了问题，在Android6.0以下可以顺利读取，但是在Android6.0上无法读取信息。我怀疑是权限的问题，然后就去官网上找资料，找到了原因：[Requesting Permissions at Run Time](http://developer.android.com/training/permissions/requesting.html)(需自备梯子访问)。我在这里总结并记录下来方便以后查阅。

Android在6.0版本开始需要用户在使用程序的时候去获取程序需要的权限，而不是像以前版本一样直接在程序安装的时候直接赋予程序所需要的权限。因此在6.0上，如果需要使用某项权限需要主动去向用户申请。

而权限分为两种：

1.一种是普通权限，如网络访问，发送广播等等，这些权限在程序安装时会自动授予不需要再次申请。

2.另外一种则是危险权限，如通讯录访问，存储空间访问等等，需要主动向用户请求。

# 请求权限

首先检查是否拥有权限，这里以联系人权限为例

```java
int permissionCheck = ContextCompat.checkSelfPermission(thisActivity,
        Manifest.permission.READ_CONTACTS);
```

其中thisActivity代表这个Activity。如果拥有则会返回`PackageManager.PERMISSION_GRANTED` ，否则则会返回``PackageManager.ERMISSION_DENIED` 。

如果没有权限那我们就要主动去申请了

```java
if (ActivityCompat.shouldShowRequestPermissionRationale(this,
               Manifest.permission.READ_CONTACTS)) {
               /*这里是检查用户是否已经拒绝过授予我们权限，如果拒绝了则需要我们主
               动去向用户说明为什么需要这个权限，然后再次申请权限*/
            } else {
               /*这里是直接向用户申请权限，这会弹出一个窗口让用户选择是否授予权限，
               最后一个参数是requestCode，需要我们在复写
               onRequestPermissionsResult()函数的时候
               进行判断*/
                ActivityCompat.requestPermissions(this, 
               new String[]{Manifest.permission.READ_CONTACTS}, 1);
            }
```

在用户对窗口进行了操作之后系统会调用函数`onRequestPermissionsResult(int requestCode,        String permissions[], int[] grantResults)` ，我们需要复写这个函数来对用户的反应做一个回馈。

```java
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull
               String[] permissions, @NonNull int[] grantResults) {
        //匹配自己传入的requestCode
        switch (requestCode) {
            case 1:
                //如果请求取消则grantResults数组为空
                if (grantResults.length > 0) {
                    //如果请求通过则执行相应操作展示数据
                    if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                        readContacts();
                    } else {
                        //如果请求不通过则需要取消一些依赖这个权限的动作
                    }
                }
        }
    }
```

到了这里请求权限的完整过程就已经结束了。