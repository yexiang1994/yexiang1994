---
title: UDP协议实现rn app之间通信
tags: [react-native]
---
最近要求要用rn做一个udp通信的软件，首先先了解下这些udp这些概念。
然后在android里面开始写组件

## 简单介绍

这里需要用到DatagramSocket方法
```java
try {
    /******* 接收数据流程**/
    // 创建一个数据报套接字，并将其绑定到指定到当前要发送的端口，如8084
    datagramSocket = new DatagramSocket(8084);
    // 组装数据报,绑定要发送的ip地址，本地测试如127.0.0.1
    inetAddress =  new InetSocketAddress("127.0.0.1", 8085);
} catch (SocketException e) {
    e.printStackTrace();
}
```
创建好了连接就发送信息
```java
public void sendMsg(String data){
        // 要发送的数据data
        mes = data;
        byte[] buf = mes.getBytes();
        DatagramPacket sendPacket = new DatagramPacket(buf, buf.length, inetAddress);
        try{
            // 组装成功后发送
            datagramSocket.send(sendPacket);
        }catch (IOException e){
            e.printStackTrace();
        }
    }
```

接收信息，
```java
public void receive(){
    try {
        if (receiveSocket == null) {
            // 创建DatagramSocket，绑定8085端口,
            receiveSocket = new DatagramSocket(8085);
        }
        byte[] bytes = new byte[1024];
        datagramPacket= new DatagramPacket(bytes, 0, bytes.length);
        // 启用新线程监听receive方法，因为receiveSocket.receive() 方法会阻塞线程，
        // 所有必须要开新的线程
        newThreads = new Thread(new NewThread());
        newThreads.start();
    } catch (SocketException e) {
        e.printStackTrace();
    }
}
// 创建线程
class NewThread extends Thread{
        @Override
        public void run() {
            for(int i = 0 ; i < Integer.MAX_VALUE ; i++) {
                try {
                    receiveSocket.receive(datagramPacket);
                    // 获取接收到的数据（为string类型），然后去掉前后的空格
                    String msg = new String(datagramPacket.getData()).trim();
                    把数据组合成 jsonobject
                    JSONObject msgObj = new JSONObject();
                    try{
                        msgObj.put("msgs", msg);
                    }catch (JSONException e){
                        e.printStackTrace();
                    }
                    // 然后放到jsonarray里面,我这里最好是以json的格式传到rn的
                    message.put(msgObj);
                } catch (SocketException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
```

## 全部代码：

发送数据

```java
package com.socket;

import android.util.Log;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.SocketException;

public class DatagramServer {
    private int MAX_LENGTH = 1024; // 最大接收字节长度
    private int port = 8084;  
    private byte[] receMsgs = new byte[MAX_LENGTH];
    private DatagramSocket datagramSocket = null;
    private DatagramPacket datagramPacket = null;
    public String message = "";
    private String tag = "sockets";
    private String mes= "i had send";
    private InetSocketAddress inetAddress;
    public void openServer(){
        try {
            datagramSocket = new DatagramSocket(port);
            inetAddress =  new InetSocketAddress("127.0.0.1", 8085);
        } catch (SocketException e) {
            e.printStackTrace();
        }
    }

    public void sendMsg(String data){
        mes = data;
        byte[] buf = mes.getBytes();
        inetAddress =  new InetSocketAddress("127.0.0.1", 8085);
        DatagramPacket sendPacket = new DatagramPacket(buf, buf.length, inetAddress);
        try{
            datagramSocket.send(sendPacket);
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}

```

接收数据

```java
package com.socket;
import android.util.Log;
import com.facebook.react.bridge.Arguments;
import com.facebook.react.bridge.WritableMap;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Map;

public class DatagramClint {

    private DatagramSocket receiveSocket = null;
    private int receivePort = 8085;
    private DatagramPacket datagramPacket;
    public JSONArray message = new JSONArray();
    private String tag = "sockets";
    private Thread newThreads;
    public void receive(){
        try {
            if (receiveSocket == null) {
                receiveSocket = new DatagramSocket(receivePort);
            }
            byte[] bytes = new byte[1024];
            datagramPacket= new DatagramPacket(bytes, 0, bytes.length);
             newThreads = new Thread(new NewThread());
             newThreads.start();
        } catch (SocketException e) {
            e.printStackTrace();
        }
    }

    class NewThread extends Thread{
        @Override
        public void run() {
            for(int i = 0 ; i < Integer.MAX_VALUE ; i++) {
                try {
                    receiveSocket.receive(datagramPacket);
                    Long times = new Date().getTime();
                    String msg = new String(datagramPacket.getData()).trim();
                    JSONObject tmpObj = new JSONObject();
                    try{
                        tmpObj.put("msgs", msg);
                        tmpObj.put("time", times);
                    }catch (JSONException e){
                        e.printStackTrace();
                    }
                    message.put(tmpObj);
                } catch (SocketException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

}

```

封装成rn方法

```java
package com.socket;

import android.util.JsonToken;
import android.util.Log;

import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;

import org.json.JSONArray;

public class DatagramUtils extends ReactContextBaseJavaModule{
    ReactApplicationContext context;

    private DatagramClint datagramClint;
    private DatagramServer datagramServer;
    public DatagramUtils(ReactApplicationContext reactContext) {
        super(reactContext);
        context = reactContext;
    }

    @Override
    public String getName(){
        return "datagram";
    }

    @ReactMethod
    public void serverOpent(){
        datagramServer = new DatagramServer();
        datagramServer.openServer();
    }

    @ReactMethod
    public void sendMsg(String mes){
        datagramServer.sendMsg(mes);
    }

    @ReactMethod
    public void clintOpent(){
        datagramClint = new DatagramClint();
        datagramClint.receive();
    }
    @ReactMethod
    public void receiveMsg(Callback calback){
        String data = "";
        // 以string的方式传递数据
        calback.invoke(datagramClint.message.toString());
    }
}

```

package rn的方法

```java
package com.socket;
import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class DatagramPackage implements ReactPackage{
    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();
        modules.add(new DatagramUtils(reactContext));
        return modules;
    }
}
```
添加到主函数里面

```java
@Override
protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
        new MainReactPackage(),
        new DatagramPackage()
    );
}
```

前端用法：
```js
import {
  NativeModules,
} from 'react-native';

// 首先打开通信
NativeModules.datagram.clintOpent();
NativeModules.datagram.serverOpent();
//然后根据方法传递过去信息
press() {
    NativeModules.datagram.sendMsg(this.state.text);

    // 这里返回监听的数据
    NativeModules.datagram.receiveMsg(rs=>{
        this.setState({
            // 返回回来的string数据转换成数组，
          data: JSON.parse(rs)
        })
      });
}
```



