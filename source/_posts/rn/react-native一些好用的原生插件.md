---
title: react-native一些好用的插件
tags: [react-native]
---

## 一 调用地图功能
### 主方法
```java

package com.apps.maputil;

import android.content.Intent;

import android.net.Uri;
import android.content.Context;
import android.content.pm.PackageInfo;

import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.WritableNativeArray;
import com.facebook.react.bridge.WritableNativeMap;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.WritableArray;


public class UtilMap extends ReactContextBaseJavaModule{

    public UtilMap(ReactApplicationContext reactContext) {
        super(reactContext);
    }

    @Override
    public String getName() {
        return "UtilMap";
    }

    /*
    检查手机是否安装了相应的地图app。返回的数据格式为:[{title:'xxx'url:app地图的URL},{title:'xxx'url:app地图的URL},{title:'xxx'url:app地图的URL}]
     */
    @ReactMethod
    public void findEvents(
            String lon,
            String lat,
            String address,
            Callback successCallback) throws Exception {

        WritableArray array = new WritableNativeArray();

        //百度地图app检测
        if (isInstalled(getReactApplicationContext(), "com.baidu.BaiduMap")) {
            WritableNativeMap writableNativeMap = new WritableNativeMap();
            writableNativeMap.putString("title", "百度地图");
            writableNativeMap.putString("url", "baidumap://map/direction?origin=我的位置&destination=name:" +address+"|latlng:"+ lat+","+lon + "&mode=driving&coord_type=bd09ll");
            array.pushMap(writableNativeMap);
        }

        //高德地图app检测
        if (isInstalled(getReactApplicationContext(), "com.autonavi.minimap")) {
            WritableNativeMap writableNativeMap = new WritableNativeMap();
            writableNativeMap.putString("title", "高德地图");
            writableNativeMap.putString("url", "androidamap://navi?sourceApplication=导航功能&backScheme=nav123456&lat=" + lat + "&lon=" + lon + "&dev=0&style=2");
            array.pushMap(writableNativeMap);
        }

        //腾讯地图app检测
        if (isInstalled(getReactApplicationContext(), "com.tencent.map")) {
            WritableNativeMap writableNativeMap = new WritableNativeMap();
            writableNativeMap.putString("title", "腾讯地图");
            writableNativeMap.putString("url", "qqmap://map/routeplan?from=我的位置&type=drive&tocoord=" + lat + "," + lon + "&to=" + address + "&coord_type=1&policy=0");
            array.pushMap(writableNativeMap);
        }

        WritableNativeMap writableNativeMap = new WritableNativeMap();
        writableNativeMap.putString("title", "取消");
        writableNativeMap.putString("url", "");
        array.pushMap(writableNativeMap);

        successCallback.invoke(array);

    }

    /**
     * 查看是否安装了地图软件
     */
    public boolean isInstalled(Context context, String name) {
        PackageInfo packageInfo;
        try {
            packageInfo = context.getPackageManager().getPackageInfo(name, 0);
        } catch (Exception e) {
            packageInfo = null;
            e.printStackTrace();
        }

        if (packageInfo == null) {
            return false;
        } else {
            return true;
        }
    }
    /*
    外部调用，打开对应app，title：地址的名称
    url：`baidumap://map/direction?origin=我的位置&destination=name:${name}|latlng:${lat},${lon}&mode=driving&coord_type=bd09ll`
        `androidamap://navi?sourceApplication=导航功能&backScheme=nav123456&lat=${lat}&lon=${lon}&dev=0&style=2`
        `qqmap://map/routeplan?from=我的位置&type=drive&tocoord=${lat},${lon}&to=${name}&coord_type=1&policy=0`

    */
    @ReactMethod
    public void openApp(String url) {
        //打开对应的app
        if(!url.equals("")){
            Intent i1 = new Intent();
            i1.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            i1.setData(Uri.parse(url));
            getReactApplicationContext().startActivity(i1);
        }
    }
}
```
### 添加主方法

```java
package com.apps.maputil;
import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import com.apps.maputil.UtilMap;
public class UtilMapPackage implements ReactPackage {

  @Override
  public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
    return Collections.emptyList();
  }

  @Override
  public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {

    List<NativeModule> modules = new ArrayList<>();

    modules.add(new UtilMap(reactContext));

    return modules;
  }
}
```

### 然后在MainApplication.java中添加 UtilMapPackage

```java
@Override
protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
        new MainReactPackage(),
        new UtilMapPackage(),
    );
}
```

### 方法的调用

```javascript
import {NativeModules} from 'react-native';

let UtilMapManager = NativeModules.UtilMap;

UtilMapManager.openApp(url);

UtilMapManager.findEvents(lon, lat, title, (events) => {
    if (events.length < 2) {
        return Alert.alert('请下载百度地图app','', [
            { text: '确定', onPress: () => {} },
        ]);
    }
})

```

------

## 二 获取精确的地理位置

### 获取地理位置主代码

获取地理位置这个必须在主函数里面写，因为ActivityCompat.requestPermissions函数调用参数包含 Activity活动对象，这个对象必须实例化，被调用了才行，不然运行不了。

```java
package com.apps;

import android.app.Activity;
import android.content.Context;
import android.content.pm.PackageManager;
import android.location.GpsStatus;
import android.location.Location;
import android.location.LocationListener;
import android.location.LocationManager;
import android.location.LocationProvider;
import android.os.Bundle;
import android.support.v4.app.ActivityCompat;
import android.support.v4.content.PermissionChecker;
import android.util.Log;
import android.view.View;
import android.widget.Toast;

import com.facebook.react.ReactActivity;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.WritableNativeMap;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends ReactActivity implements LocationListener,GpsStatus.NmeaListener {

    private LocationManager locationManager;
    private static final String TAG = "GpsActivity";
    private String mProvider = "";
    public static final int MY_PERMISSION_ACCESS_COURSE_LOCATION = 11;
    public String pointList;

    public static MainActivity mainActivity;

    public void getlocationGps(){
        //此处的判定是主要问题，API23之后需要先判断之后才能调用locationManager中的方法
        //包括这里的getLastKnewnLocation方法和requestLocationUpdates方法
        if(PermissionChecker.checkSelfPermission(this,android.Manifest.permission.ACCESS_FINE_LOCATION)== PackageManager.PERMISSION_GRANTED) {
            //获取定位服务
            locationManager = (LocationManager) getSystemService(Context.LOCATION_SERVICE);
            //获取当前可用的位置控制器
            List<String> list = locationManager.getProviders(true);

            if (list.contains(LocationManager.GPS_PROVIDER)) {
                //是否为GPS位置控制器
                Log.d(TAG,"1");
                mProvider = LocationManager.GPS_PROVIDER;
            } else if (list.contains(LocationManager.NETWORK_PROVIDER)) {
                Log.d(TAG,"2");
                //是否为网络位置控制器
                mProvider = LocationManager.NETWORK_PROVIDER;
            } else {
                Log.d(TAG,"3");
                Toast.makeText(this, "请检查网络或GPS是否打开",
                        Toast.LENGTH_LONG).show();
                return;
            }
            Location location = locationManager.getLastKnownLocation(mProvider);
            if (location != null) {
                //获取当前位置，这里只用到了经纬度
//                String string = "纬度为：" + location.getLatitude() + ",经度为："
//                        + location.getLongitude();
//                updateView(location);
            }else{
                Log.d(TAG,"location is null");
            }
            //绑定定位事件，监听位置是否改变
            //第一个参数为控制器类型第二个参数为监听位置变化的时间间隔（单位：毫秒）
            //第三个参数为位置变化的间隔（单位：米）第四个参数为位置监听器
            locationManager.requestLocationUpdates(mProvider, 10*1000, 2, this);
            locationManager.addNmeaListener(this);
        }else{
            Log.d(TAG,"not good");
            ActivityCompat.requestPermissions( this, new String[] {  android.Manifest.permission.ACCESS_FINE_LOCATION },
                    MY_PERMISSION_ACCESS_COURSE_LOCATION );
        }
    }

    // gps状态改变时
    @Override
    public void onStatusChanged(String provider, int status, Bundle extras) {
        switch (status) {
            //GPS状态为可见时
            case LocationProvider.AVAILABLE:
                Log.i(TAG, "当前GPS状态为可见状态");
                break;
            //GPS状态为服务区外时
            case LocationProvider.OUT_OF_SERVICE:
                Log.i(TAG, "当前GPS状态为服务区外状态");
                break;
            //GPS状态为暂停服务时
            case LocationProvider.TEMPORARILY_UNAVAILABLE:
                Log.i(TAG, "当前GPS状态为暂停服务状态");
                break;
        }
    }

    @Override
    public void onProviderEnabled(String provider) {
        Log.d(TAG,"in providerEnabled");
        getlocationGps();
    }

    @Override
    public void onProviderDisabled(String provider) {
        Log.d(TAG,"in providerDisabled");
//        updateView(null);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
//        stopGPS();

    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        switch (requestCode) {
            case MY_PERMISSION_ACCESS_COURSE_LOCATION: {
                if (grantResults.length > 0
                        && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                    Log.d(TAG,"permission good");
                } else {
                    Log.d(TAG,"permission denied");
                }
                return;
            }

        }
    }

    @Override
    public void onNmeaReceived(long timestamp, String nmea) {
//        Log.d(TAG, nmea);
        if((nmea.contains("GNRMC")|| nmea.contains("GPRMC"))&& nmea.length()>40){
            String [] array = nmea.split(",");
            if(array.length > 6){
                String a = array[3];
                String b = array[5];
                int aInt = Integer.parseInt(a.substring(0, a.indexOf(".") - 2));
                int bInt = Integer.parseInt(b.substring(0, b.indexOf(".") - 2));
                double aRes = Double.parseDouble(a.substring(a.indexOf(".") - 2));
                double bRes = Double.parseDouble(b.substring(b.indexOf(".") - 2));
                double aFinal = (double)aInt + aRes / 60;
                double bFinal = (double)bInt + bRes / 60;
                pointList = aFinal + "," + bFinal;
                Log.i(TAG, aFinal + "  " + bFinal);
//                gpsService.create(deviceId, bFinal, aFinal, str);
            }
        }
    }

    @Override
    public void onLocationChanged(Location location) {
//        updateView(location);
        Log.i(TAG, "时间："+location.getTime());
        Log.i(TAG, "经度："+location.getLongitude());
        Log.i(TAG, "纬度："+location.getLatitude());
        Log.i(TAG, "海拔："+location.getAltitude());

    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mainActivity = this;
//        getlocationGps();
    }

    @Override
    protected String getMainComponentName() {
        return "apps";
    }
}
```

###  添加地理位置函数

```java
package com.apps.gpsutil;

import android.content.Context;
import android.content.pm.PackageManager;
import android.location.Criteria;
import android.location.Location;
import android.location.LocationListener;
import android.location.LocationManager;
import android.os.Bundle;
import android.support.v4.app.ActivityCompat;
import android.support.v4.content.PermissionChecker;
import android.telecom.Call;
import android.util.Log;
import android.widget.Toast;

import com.apps.MainActivity;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Callback;

import java.util.List;

public class GetLocation extends ReactContextBaseJavaModule{
    ReactApplicationContext context;

    private static final String TAG = "GpsActivity";
    private LocationManager locationManager;
    private String mProvider = "";
    public Location locations;
    public static final int MY_PERMISSION_ACCESS_COURSE_LOCATION = 11;
    public GetLocation(ReactApplicationContext reactContext) {
        super(reactContext);
        context = reactContext;
    }

    @Override
    public String getName() {
        return "Gps";
    }

    @ReactMethod
    public void startLocation() {
        // rn里调用这个函数 ，然后在调用主函数里面的方法打开gps
        MainActivity.mainActivity.getlocationGps();
    }
    @ReactMethod
    public void getLocation(Callback cal) {
        cal.invoke(MainActivity.mainActivity.pointList);
    }
}

```

### 添加到主函数

```java
@Override
protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
        new MainReactPackage(),
        new GpsPackage()
    );
}
```
### 函数调用
```js
NativeModules.Gps.startLocation()

NativeModules.Gps.getLocation(rs=>{})
```
