---
title: java基础知识
tags: [java]
---

1. JSON格式数组包含obj类型转成 list, 
如 [{ContentId: 1, IsForever: false}] 格式，

定义实体类 
```java
import com.fasterxml.jackson.annotation.JsonProperty;

public class Content {
    @JsonProperty("ContentId")
    private String contentId;

    @JsonProperty("IsForever")
    private boolean isForever;

    public String getContentId() {
        return contentId;
    }

    public void setContentId(String contentId) {
        this.contentId = contentId;
    }

    public boolean isForever() {
        return isForever;
    }

    public void setForever(boolean forever) {
        isForever = forever;
    }
}
```
当遇到有字段开头大写的时候需要用 JsonProperty 注解，不然java不能正确映射 大写的开头的字段，他会转成小写的，从而导致转换失败， 
这里使用ObjectMapper来转换
```java

public class JacksonUtil {
    private static ObjectMapper objectMapper = null;
    static{
        objectMapper = new ObjectMapper();
        // 如果json中有新增的字段并且是实体类类中不存在的，不报错
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    }

    // 类转化成JSON
    public static String GetJSON(Object obj){
        try {
            return objectMapper.writeValueAsString(obj);
        } catch(Exception e){
            e.printStackTrace();
        }
        return null;
    }
    // 数组里面包含obj格式的类的json转换成类
    public static <T> T getAnyBean(String json, TypeReference<?> valueTypeRef){
        T dataBean=null;
        try {
            dataBean= objectMapper.readValue(json, valueTypeRef);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return dataBean;
    }
    // 普通obj类型的json类转换成类
    public static <T> T getAnyBean(String json, Class c){
        T dataBean=null;
        try {
            dataBean= (T)objectMapper.readValue(json, c);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return dataBean;
    }
}
// cpData的数据格式为[{ContentId: "1",IsForever: false}]
List<Content> fileTextList = JacksonUtil.getAnyBean(cpData, new TypeReference<List<Content>>(){});
// 转换成了Content的list

// star数据格式是{ContentId: "1",IsForever: false}
Content content = JacksonUtil.getAnyBean(star, Content.class);

```


