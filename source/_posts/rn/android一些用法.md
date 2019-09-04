---
title: android一些方法
tags: [android]
---
## 神奇权限
```java
String[] str = {Manifest.permission.READ_EXTERNAL_STORAGE,
                Manifest.permission.WRITE_EXTERNAL_STORAGE};
        try{
            int permission = ContextCompat.checkSelfPermission(context ,
                    Manifest.permission.WRITE_EXTERNAL_STORAGE);
            if( permission != PackageManager.PERMISSION_GRANTED){
                ActivityCompat.requestPermissions(this, str, 10);
            }
        } catch (Exception e){
            e.printStackTrace();
        }
```