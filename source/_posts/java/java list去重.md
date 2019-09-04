---
title: java list去重
tags: [java]
---

## 1. 简单的数据结构去重
比如String，int，long的list集合可以直接用Set转换下，

```java
public class duplicate() {
    public static void main(String[] arg){
        List list = new ArrayList();
        list.add(1);
        list.add(2);
        list.add(1);
        list.add(3);
        list = removeDupicate(list);
    }
    public static List removeDuplicate(List list) {   
		Set set = new HashSet();
		List newlist = new ArrayList<>();
		for (Iterator iter = list.iterator(); iter.hasNext();) {
			Object el = iter.next();
            //set能添加数据，说明不重复，则提取出不重复数据
			if (set.add(el)) {
				newlist.add(el);
			}
		}
	    return newlist;
	}
```
## 2. 对象去重
对象去重也能永Set，但是需要重写对象里面的hashCode的方法，和equals方法，

```java
public class FileTxt {
    private int ver;
    private String senderType;
    private String senderId;
    private String uuid;

    public FileTxt( int ver,
            String senderType,
            String senderId,
            String uuid){
        this.ver = ver;
        this.senderType = senderType;
        this.senderId = senderId;
        this.uuid = uuid;
    }
    public FileTxt() {

    }
    public int getVer() {
        return ver;
    }

    public void setVer(int ver) {
        this.ver = ver;
    }

    public String getSenderType() {
        return senderType;
    }

    public void setSenderType(String senderType) {
        this.senderType = senderType;
    }

    public String getUuid() {
        return uuid;
    }

    public void setUuid(String uuid) {
        this.uuid = uuid;
    }

    @Override
    public boolean equals(Object o){
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        FileTxt content = (FileTxt) o;
        if (!senderType.equals(content.senderType)) return false;
        if (!senderId.equals(content.senderId)) return false;
        if (ver != content.ver) return false;

        return uuid.equals(content.uuid);
    }
    @Override
    public int hashCode(){
        return senderType.hashCode() + senderId.hashCode()
                + ver + uuid.hashCode();
    }
}

public static List removeDuplicateList(List list){
        Set set = new HashSet();
        List newList = new ArrayList();
        for(Iterator iter = list.iterator(); iter.hasNext();){
            Object el = iter.next();
            if(set.add(el)){
                newList.add(el);
            }
        }
        return newList;
    }
```

