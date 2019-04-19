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
        // 预留1024字节缓冲区
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
            // 因为 receiveSocket.receive是阻塞方法，只有接收到数据后，才会执行下一个for循环内容，
            // 这里可以随便给个最大值，比如100000
            for(int i = 0 ; i < 100000 ; i++) {
                try {
                    receiveSocket.receive(datagramPacket);
                    // 获取接收到的数据（为string类型），然后去掉前后的空格
                    String msg = new String(datagramPacket.getData()).trim();
                    // 把数据组合成 jsonobject
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

## 缓存问题
在实现发送信息的时候会出现，当下次接收到的数据字（a长度）节小于前一次的（b长度）的时候，接收到的信息只会更新前a个数据，b-a后面的数据依然存在，比如第一次发送abc，第二次发送12，这时接收到的数据就是12c而不是我们想要的12。 这时因为udp存在缓存问题，所以要清空缓存。
### 清空缓存方法
方法：在取出数据后，执行如下方法
```java
// 上面的这个创建缓存区的方法可以提取出来
byte[] bytes = new byte[1024];
// 然后在
Arrays.fill(bytes,(byte)0);
```
然后就清空了缓存区域

## 广播问题
需要先了解下ip和子网掩码二进制转化：
IP地址是一个32位的二进制数，通常被分割为4个“8位二进制数”（也就是4个字节）。IP地址通常用“点分十进制”表示成（a.b.c.d）的形式，其中，a,b,c,d都是0~255之间的十进制整数。例：点分十进IP地址（100.4.5.6），实际上是32位二进制数（01100100.00000100.00000101.00000110）

假如计算机的IP位址是192.15.156.205，子网掩码是255.255.255.224，
先把子网掩码255.255.255.224做 NOT 运算﹐可以得出﹕
00000000.00000000.00000000.00011111 
然后再和IP做一次 OR 运算﹐就可以得到 Broadcast Address: 
11000000.00001111.10011100.11001101 OR 00000000.00000000.00000000.00011111 
(192.15.156.205 OR 255.255.255.224) 
得出﹕ 11000000.00001111.10011100.11011111
(192.15.156.223) 
192.15.156.223就是那个子网的广播地址了. 知道广播地址后就可以以这个地址来发送消息了，局域网的都可以接收到了。

当网络间进行通信时，如需确定是否在同一网络，则用某台主机的网络号与另一台主机的子网掩码进行与运算，观察网络号与与运算的结果是否相同。
其中还有了解下子网掩码，不然还是不知所以,[子网掩码](https://baike.baidu.com/item/%E5%AD%90%E7%BD%91%E6%8E%A9%E7%A0%81/100207?fr=aladdin)

[参考地址](https://blog.csdn.net/Evan_love/article/details/79981574)
[参考地址](https://blog.csdn.net/fwt336/article/details/76538321)

## 为何不能跨域广播
因为中国网络十分复杂，一层套一层，端口，ip随时会变化。比如你所在的公司有个局域网，

