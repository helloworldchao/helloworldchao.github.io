---
layout: post
title:  "Broadcast使用解析"
date:   2016-05-14 22:40:00 +0800
categories: Android
permalink: /archivers/Broadcast
---

Android四大组件中`Broadcast Receiver`是我们经常需要使用的，而在使用之前，我们需要首先了解`Broadcast`如何使用。

`Broadcast`如果需要接收其他程序发出的广播，需要实现一个继承了Broadcast类的新类并进行注册来接受信息，而注册方式又分为**静态注册**和**动态注册**。

如果我们需要发出广播，则同样有两种方式，一种是**普通广播**，另外一种是**有序广播**，其中**普通广播**全局播送**无法被程序截断**，而**有序广播可以被截断**。

还有一种广播方式只能在本应用程序中使用，那就是**本地广播**。

# 静态注册

首先需要发送广播。

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent("com.chao.broadcasttest.BROADCAST");
        sendBroadcast(intent);
    }     
```

接着实现一个继承了Broadcast类的新类来接收广播信息。

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "received", Toast.LENGTH_SHORT).show();
    }
}
```

然后在AndroidManifest.xml文件中注册。

```xml
<receiver android:name=".MyBroadcastReceiver">
              <intent-filter>
                  	<action android:name="com.chao.broadcasttest.BROADCAST" />
              </intent-filter>
</receiver>
```

然后直接运行程序就可以看到Toast显示，说明广播接收成功。

# 动态注册

动态注册与静态注册的区别就是动态注册需要在代码中进行注册来接收广播。

首先是发送广播代码：

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent("com.chao.broadcasttest.BROADCAST");
        sendBroadcast(intent);
    }     
```

然后实现一个继承了Broadcast类的新类来接收广播信息。

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "received", Toast.LENGTH_SHORT).show();
    }
}
```

接着进行注册接收信息，同时在接收完广播后取消注册。

```java
    //实例化接收广播的类
    private MyBroadcastReceiver broadcastReceiver;

    //进行注册
    public void registerReceiver() {
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction("com.chao.broadcasttest.BROADCAST");
        broadcastReceiver = new MyBroadcastReceiver();
        registerReceiver(broadcastReceiver, intentFilter);
    }

    //在接收完毕后取消注册
    public void unRegisterReceiver() {
        unregisterReceiver(broadcastReceiver);
    }
```

# 普通广播和有序广播

其实在之前的程序中已经展现了标准广播。前面的`sendBroadcast()`方法所发送的就是普通广播，那么接下来就直接展现有序广播。有序广播的发送方法只是使用了另外一个方法  `sendOrderedBroadcast(Intent intent, String receiverPermission);`  
接下来看看如何具体使用。

首先使用运用之前的静态注册代码并将`sendBroadcast()`改为`sendOrderedBroadcast();`。然后再新建一个程序，也实现一遍静态注册，但是删除`MainActivity`类中发送广播的方法，同时将`MyBroadcast`类中的Toast内容改为`Toast.makeText(context, "received in two", Toast.LENGTH_SHORT).show();`。接下来先运行第二个程序再运行第一个程序，就会发现同时接受到了信息。

然后我们开始截断信息。

首先修改第一个程序的`MyBroadcast`类。

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "received", Toast.LENGTH_SHORT).show();
        //加入这个方法进行截断
        abortBroadcast();
    }
}
```

然后将第一个程序的AndroidManifest.xml文件中的广播注册优先级修改为高
` android:priority="100"`，保证它首先接收广播。

```xml
          <receiver android:name=".MyBroadcastReceiver">
            <intent-filter android:priority="100">
                <action android:name="com.chao.broadcasttest.BROADCAST" />
            </intent-filter>
          </receiver>
```

接下来分别运行第二个程序和第一个程序，就会发现程序二接收不到广播了。

# 本地广播

本地广播可以发送和接收一些不希望系统中其他程序接收到的广播。

所有实现代码如下：

```java
public class MainActivity extends AppCompatActivity {

    private LocalBroadcastManager manager;
    private LocalReceiver localReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //发送广播
        localReceiver = new LocalReceiver();
        LocalBroadcastManager manager = LocalBroadcastManager.getInstance(this);
        Intent intent = new Intent("com.chao.broadcasttest3.BROADCAST");
        manager.sendBroadcast(intent);


        //注册并接受广播
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction("com.chao.broadcasttest3.BROADCAST");
        localReceiver= new LocalReceiver();
        manager.registerReceiver(localReceiver, intentFilter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        //取消注册
        manager.unregisterReceiver(localReceiver);
    }

    class LocalReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context, "received in Local", Toast.LENGTH_SHORT).show();
        }
    }
}
```

所有的广播内容至此都已经归纳完毕了。