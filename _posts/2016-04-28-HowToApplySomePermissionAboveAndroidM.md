---
layout: post
title:  "关于某些权限在Android6.0及以上的申请方式"
date:   2016-04-28 21:15:00 +0800
categories: Android
permalink: /archivers/HowToApplySomePermissionAboveAndroidM
---

# 问题介绍

今天在看《第一行代码》的broadcast部分时有一个小练习需要请求android.permission.SYSTEM_ALERT_WINDOW权限来弹窗提示用户。原代码如下：

```java
public class MainActivity extends AppCompatActivity {

    private Button btn_force_offline;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        btn_force_offline = (Button) findViewById(R.id.btn_force_offline);
        btn_force_offline.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent("com.chao.broadcastpractice.FORCE_OFFLINE");
                sendBroadcast(intent);
            }
        });
    }
}
```

当我在完成代码后在我的6.0的机子上运行时直接报错，错误信息如下：

> java.lang.RuntimeException: Unable to start receiver com.chao.broadcastpractice.ForceOfflineReceiver: java.lang.SecurityException: com.chao.broadcastpractice from uid 10125 not allowed to perform SYSTEM_ALERT_WINDOW

我在Google上搜索信息发现这是Android在6.0以后的版本对权限管控的改变，我就着手找寻解决办法，后来在StackOverFlow上找到了解决办法。

# 解决办法

这是原网址：[Permission denied for this window type](http://stackoverflow.com/questions/7569937/unable-to-add-window-android-view-viewrootw44da9bc0-permission-denied-for-t#answer-34061521)

我总结了一下就是主动向用户申请权限，在用户同意后再进行操作。然后我修改了一下代码，终于成功了。代码如下：

```java
public class MainActivity extends AppCompatActivity {

    private Button btn_force_offline;
    public boolean PERMISSION = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        btn_force_offline = (Button) findViewById(R.id.btn_force_offline);
        btn_force_offline.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (PERMISSION) {
                    Intent intent = new Intent("com.chao.broadcastpractice.FORCE_OFFLINE");
                    sendBroadcast(intent);
                } else {
                    Toast.makeText(getApplicationContext(), "permisson denied, please click allow!", Toast.LENGTH_SHORT).show();
                }
            }
        });
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (!Settings.canDrawOverlays(this)) {
                Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:" + getPackageName()));
                startActivity(intent);
            } else {
                PERMISSION = true;
            }
        }
    }
}
```

我设置了一个`PERMISSION`用`Settings.canDrawOverlays()`来判断能不能执行程序的界面叠加权限，如果没有权限则执行  

> Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:" + getPackageName()));  
> startActivity(intent);

来申请权限，如果有则将`PERMISSION`赋值为`true`执行相应动作。如果用户不同意那么直接弹Toast方法提示用户没有权限，只有用户同意了之后才能够执行接下来的动作。
