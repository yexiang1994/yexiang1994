---
title: rn通知
tags: [react-native]
---

目前只实现了android的通知。通知是当app没有在前台运行的时候希望用户接收某些重要信息的时候在手机最上方状态栏中显示的一个通知的图标，下拉栏后可以看见相关内容。
## 基本用法
android使用NotificationManager来进行通知管理，使用Context的getSystemService（）方法里面获取到，此方法接收一个字符串用于确定获取系统的哪一个服务，这里使用Context.NOTIFICATION_SERVICE获取通知的方法，所有可以写成：
```java
NotificationManager manager = (NotificationManager) context.getSystemService(context.NOTIFICATION_SERVICE);
```
然后使用Builder来创建Notification对象，但是由于android版本问题，需要做兼容，这里使用support-v4库中的NotificationCompat类
```java
Notification notification = new NotificationCompat.Builder(context)
        .setContentTitle("this is e title") // 标题
        .setContentText("this is content text") //内容
        .setWhen(System.currentTimeMillis()) // 提示的时间
        .setSmallIcon(R.mipmap.ic_launcher) // 设置小图
        .setLargeIcon(BitmapFactory.decodeResource(context.getResources(), R.mipmap.ic_launcher))  // 设置大图
        .setContentIntent(pi)
        .setAutoCancel(true) // 打开提示后取消当前提示栏
        .setSound(Uri.fromFile(new File("/system/media/audio/ringtones/Luna.ogg"))) // 设置提示声音,这里是系统的声音
        .setDefaults(NotificationCompat.DEFAULT_ALL) // 设置LED灯闪烁
        .setPriority(NotificationCompat.PRIORITY_MAX) // 设置通知是否重要，max表示最高重要的，会弹出到界面来
        .build();
manager.notify(1, notification);  // 这里的1表示每个通知id，不同通知使用这个数字区分开
```
setLights()方法接收三个参数，第一个是LED灯的颜色，第二个LED灯亮起的时长，第三个是LED暗去的时长，这里时长都是毫秒为单位，
```java
Notification notification = new NotificationCompat.Builder(context)
...
.setLights(Color.GREEN,1000,1000)
.build();
```
setStyle()可以构建富文本通知，如文件太多一行会超出隐藏，想要展示多行，就需要这样做
```java
.setStyle(new NotificationCompat.BigTextStyle().bigText("new new newenewewnewenwenenwwenwnwenwnwnxxxxxxxxxxxxxxxxxxxxx"))
.build
```
添加一个图片
```java
.setStyle(new NotificationCompat.BigPictureStyle().bigPicture(BitmapFactory.decodeResource(getResources(),R.deawable.big_image)))
.build
```
setPriority()设置通知的重要性，PRIORITY_DEFAULT 默认的重要程度，PRIORITY_MIN最低的重要程度，比如用户下拉的时候才会展示一般不会展示，PRIORITY_LOW，表示较低的重要程度，系统会缩小这类通知的图标，或者改变他的顺序，PRIORITY_HIGH，系统会放大这类通知，或者排序提前，PRIORITY_MAX最高的通知，这类消息必须让用户离开看到，会弹出一个横幅。
```java
import android.content.Context;
import android.content.Intent;
import android.support.v4.app.NotificationCompat;
import android.app.Notification;
import android.app.NotificationManager;

Context context = this;
Intent intent = new Intent(context, NotificationActivity.class);
PendingIntent pi = PendingIntent.getActivity(context,0,intent,0);
NotificationManager manager = (NotificationManager) context.getSystemService(context.NOTIFICATION_SERVICE);
Notification notification = new NotificationCompat.Builder(context)
        .setContentTitle("this is e title") // 标题
        .setContentText("this is content text") //内容
        .setWhen(System.currentTimeMillis()) // 提示的时间
        .setSmallIcon(R.mipmap.ic_launcher) // 设置小图
        .setLargeIcon(BitmapFactory.decodeResource(context.getResources(), R.mipmap.ic_launcher))  // 设置大图
        .setContentIntent(pi)
        .setAutoCancel(true) // 打开提示后取消当前提示栏
        .setSound(Uri.fromFile(new File("/system/media/audio/ringtones/Luna.ogg"))) // 设置提示声音
        .setDefaults(NotificationCompat.DEFAULT_ALL) // 设置LED灯闪烁
        .setPriority(NotificationCompat.PRIORITY_MAX) // 设置通知是否重要，max表示最高重要的，会弹出到界面来
        .build();
manager.notify(1, notification);
```
一般的通知不会放到activity里面执行，活动被销毁你通知也发不了，所有我这里就把通知放在了一个server里面，
## 什么是服务（server）
服务（server）是android中实现程序后台运行的解决方案，适合做那些不需要和用户交互而且还要求长期运行的任务，服务不依赖于任何用户界面，即使程序被切换到后台或者用户打开了另外一个应用，服务仍然能够保持正常运行。但是服务不是运行在一个独立的进程中的，而是依赖于创建服务时所在的应用程序进程。当应用程序进程被杀掉时，所有依赖于该进程的服务也会停止运行。
服务不会开启一个线程，所有代码都是在主线程中运行的，也就是说服务是在主线程里面创建的子线程，
## 线程的基本用法
在了解下线程。在android中要执行一些比较耗时的操作时，通常不会再主线程里面做，而会单独开一个子线程去处理，防止主线程被堵塞了，线程用法：
```java
class TestThread extends Thread{
    @Override
    public void run()
    {
        //     
    }
}

// 调用
new TestThread().start()
```
```java
class TestThread implements Runnable{
    @Override
    public void run(){
        // 
    }
}
// 调用
TestThread testThread = new TestThread();
new Thread(testThread).start();
```
```java
new Thread(new Runnable(){
    @Override
    public void run(){
        //
    }
}).start();
```
如果要根据执行的任务来改变相应UI时需要用异步消息处理：
```java
public class MainActivity extends AppCompatActivity implements View.OnclickListener{
    public static final int TEXT =1;
    private TextView text;
    private Handler handler = new Handler(){
        public void handleMessage(Message msg){
            switch(msg.what){
                case TEXT:
                    // 进行UI操作
                    text.setText("welcome");
                    break;
                default:
                    break;
            }
        }
    }
    @Override
    public void onClick(View v){
        switch (v.getId()){
            case R.id.change_text: 
                new Thread(new Runnable(){
                    @Override
                    public void run (){
                        Message message = new Message();
                        message.what = TEXT;
                        handler.sendMessage(message); //将message发送出去,handler会接收这个消息
                    }
                }).start();
                break;
            default:
                break;
        }
    }
}
```
## 异步消息处理
android中异步消息处理由4部分组成：Message,Handler,MessageQueue,Looper, 
### 1. Message 
Message是在线程之前传递的消息，可以在内部携带少量的信息，用于在不同线程之间交换数据，
### 2. Handler
主要是用于发送和处理消息，发送消息一般是Handler和sendMessage方法，而发送的消息一般会回到Handler和handleMessage方法中
### 3. MessageQueue
MessageQueue是消息队列的意思，主要用于存放所有通过Handler发送的消息，这部分消息会一直存在消息队列里面，等待被处理，每个线程只有一个MessageQueue对象。
### 4. Looper
Looper是每个线程中的MessageQueue的管家，调用Looper的loop方法后，就会进入一个无线循环中，每当发现MessageQueue中存在一条消息，就会将他取出来，并传递到Handler的handlerMessage方法中，每个线程中也只有一个Looper对象

回到整体服务（service）
在android里面创建一个service类，new->Services->Service 他会自动添加到AndroidManifest.xml里面，
```xml
<application>
    <service
        android:name=".MyService"
        android:process=":push"></service>
</application>
```
```java
public class MyService extends Service {
    @Override
    public void onCreate(){  // service创建的时候调用
        super.onCreate();
    }
    // service每次启动的时候都执行一次
    @Override
    public int onStartCommand(final Intent intent, int flags, int startId){
        return super.onStartCommand(intent,flags,startId);
    }
    // 服务被停掉的时候调用
    @Override
    public void onDestroy(){
        super.onDestroy();
    }
}
```
## app保活一直能发通知
一般app进入后台后，service是正常运行的，不会被杀死，activity也是正常运行的，这时候手机是可以接受通知。但是当用户主动清空后台之后，这两个都被杀掉了，彻底发送不了通知了。google其实是有一个GCM通知的，这个是手机集成的，在主程序里面，发送者发送通知到google服务器，然后google服务器接受到发送的消息转发到手机上，手机就收到了通知。但是这就需要手机自带google的功能，在国内这种是使用不了的，被墙掉了。  
然后就出现了一堆第三方的推送，如极光推送，华为推送，小米推送，这些推送都是类似上面GCM通知一样，第三方转发通知。像极光推送是集成在app里面的，当app被杀掉本来这个也是没用了的，但是为啥还能推送，是因为很多app都集成了这个，如果其他app没有被杀掉那么其他app里面集成的极光就会代理发送当前通知，达到通知的目的。而华为小米，应该是手机内部之间有推送服务，这种应该更稳定些。
要说为啥微信，qq程序清空后台杀不掉，能一直推送，估计是因为和厂商有py交易，但是像我的华为手机关掉应用的 启动管理，允许后台活动，允许关联启动，允许自启动后，就不会推送了。但是用adb shell ps | findstr tencent
![](http://thdqn.cn:8081/imagePath/1556617737587.jpg)
可以看到就算关闭了后台运行，这些腾讯服务还是存在的，但是一般app就不行了。
所以想要实现保活就用第三方吧，具体稳定性也不好说，我这里使用的局域网，第三方没戏，所以也不必要求一定保活，只要确保后台运行时能推送就ok了。
## service代码
```java
package com.selectot;
import android.app.Notification;
import android.app.NotificationManager;
import android.app.PendingIntent;
import android.app.Service;
import android.content.BroadcastReceiver;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.ServiceConnection;
import android.graphics.BitmapFactory;
import android.net.Uri;
import android.os.IBinder;
import android.os.PowerManager;
import android.support.v4.app.NotificationCompat;
import android.util.Log;

import com.service.MyIntentService;
import com.service.WLWakefulReceiver;

import java.io.File;
import java.util.Timer;
import java.util.TimerTask;

public class MyService extends Service {
    private BroadcastReceiver mReceiver;
    private IntentFilter mIf;
    private Intent testIntent;
    private Context context;
    public MyService() {
    }
    private String TAG = "targets";
    @Override
    public IBinder onBind(Intent intent) {
        throw new UnsupportedOperationException("Not yet implemented");
    }
    @Override
    public void onCreate(){
        super.onCreate();
    }

    @Override
    public int onStartCommand(final Intent intent, int flags, int startId){
        Log.d(TAG, "onStartCommand: "+"fas");
        Timer timer = new Timer();
        TimerTask task = new TimerTask() {
            @Override
            public void run() {
                init();
            }
        };
        timer.schedule(task,0,5000);
        return super.onStartCommand(intent,flags,startId);
    }


    @Override
    public void onDestroy(){
        Log.d(TAG, "onDestroy: "+"服务被销毁");
        super.onDestroy();
    }

    public void sendNotice(){
        Context context = this;
        Intent intent = new Intent(context, NotificationActivity.class);
        PendingIntent pi = PendingIntent.getActivity(context,0,intent,0);
        NotificationManager manager = (NotificationManager) context.getSystemService(context.NOTIFICATION_SERVICE);
        Notification notification = new NotificationCompat.Builder(context)
                .setContentTitle("this is e title") // 标题
                .setContentText("this is content text") //内容
                .setWhen(System.currentTimeMillis()) // 提示的时间
                .setSmallIcon(R.mipmap.ic_launcher) // 设置小图
                .setLargeIcon(BitmapFactory.decodeResource(context.getResources(), R.mipmap.ic_launcher))  // 设置大图
                .setContentIntent(pi)
                .setAutoCancel(true) // 打开提示后取消当前提示栏
                .setSound(Uri.fromFile(new File("/system/media/audio/ringtones/Luna.ogg"))) // 设置提示声音
                .setDefaults(NotificationCompat.DEFAULT_ALL) // 设置LED灯闪烁
                .setPriority(NotificationCompat.PRIORITY_MAX) // 设置通知是否重要，max表示最高重要的，会弹出到界面来
                .build();
        manager.notify(1, notification);
    }

    private void init(){
        Log.d(TAG, "init: " + "发送");
        //（Application Processor），可以阻止屏幕变暗。所有的WakeLock被释放后，系统会挂起。
        PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);
        PowerManager.WakeLock wakeLock = pm.newWakeLock(PowerManager.ON_AFTER_RELEASE
                | PowerManager.PARTIAL_WAKE_LOCK, "MyWakelockTag");
        wakeLock.acquire();
        this.sendNotice();
        wakeLock.release();
    }
}
```
调用
```java
 // 启动服务
@ReactMethod
public void startService() {
    Context context = MainActivity.getContext();
    Intent in = new Intent(context, MyService.class);
    context.startService(in);
}
```














