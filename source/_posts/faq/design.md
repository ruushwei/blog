---
title: 设计模式
date: 2019-07-01T12:54:24+02:00
tags: 
- 面试
- 设计模式
- 算法
categories: 设计模式
---

#### 1. 写一个单例模式

##### 双检锁版本 (mi)

```java
public class Singleton{  
  private static volatile Singleton instance; 
  private Singleton(){}  
    
  public static Singleton getSingleton(){  
    if(instance == null){     
        synchronized (Singleton.class) {   
            if(instance == null){      
                instance = new Singleton();  
        }     
      }  
    }    
    return instance;   //返回创建好的对象   
  }  
}  
```
##### 枚举版本 (amazon)

```
public enum PpEnum {

    GROUP(1, "立减", new HashMap<String, String>() {
        {
            put("", "");
        }
    }),

    private int id;
    private String name;
    private Map<String, String> map;

    PpEnum(int id, String name, Map<String, String> map) {
        this.id = id;
        this.name = name;
        this.map = map;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public Map<String, String> getMap() {
        return map;
    }
}
```


#### 2. 都用过哪些设计模式 (amazon)

策略(user)，单例 (饿汉或枚举)，简单工厂（多参数获取对象），外观（老接口封新接口）