---
title: "enum"
date: 2019-08-15T13:19:18+08:00
draft: true
---

# Enum

Enum的特点在于`易读性`,`线程安全`.`EffectiveJava`作者强烈推荐用枚举来实现`单例模式`

## Enum为什么是线程安全的?
> [Hollis-深度分析Java的枚举类型](http://www.hollischuang.com/archives/197)

使用`enum`来创建一个枚举后,编译器编译时会将该枚举继承`Enum`类,并且是一个`final`修饰的class文件.
常量用final修饰,所以不可变,并且都是静态加载的.**静态资源被初始化,Java类的加载和初始化过程都是线程安全的,所有创建一个enum也是线程安全的**.
```java
public final class T extends Enum
{
    private T(String s, int i)
    {
        super(s, i);
    }
    public static T[] values()
    {
        T at[];
        int i;
        T at1[];
        System.arraycopy(at = ENUM$VALUES, 0, at1 = new T[i = at.length], 0, i);
        return at1;
    }

    public static T valueOf(String s)
    {
        return (T)Enum.valueOf(demo/T, s);
    }

    public static final T SPRING;
    public static final T SUMMER;
    public static final T AUTUMN;
    public static final T WINTER;
    private static final T ENUM$VALUES[];
    static
    {
        SPRING = new T("SPRING", 0);
        SUMMER = new T("SUMMER", 1);
        AUTUMN = new T("AUTUMN", 2);
        WINTER = new T("WINTER", 3);
        ENUM$VALUES = (new T[] {
            SPRING, SUMMER, AUTUMN, WINTER
        });
    }
}
```

## 为什么推荐使用枚举来作为单例模式的实现?
1. 写法简单
[单例使用](https://www.jianshu.com/p/4e8ca4e2af6c)
```java
public enum simpleSingleton{
    SIMPLE;
}
```
2. 枚举自己来处理序列化
所有的单例如果实现了`Serializable`的接口,就不再单例了,因为每次调用`readObject()`返回的都是一个新创建出来的对象.但是枚举在序列化是只是仅仅把枚举对象的name属性输出到结果中,反序列化的时候是通过`java.lang.Enum`的`valueOf`来根据名字查找对象.并且编译器是不允许任何对这种序列化机制的定制的,因此禁用了`writeObject,readObject,readObjectNoData,writeReplace,readResolve`.
而在`valueOf`中:
```java
public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                            String name) {
    T result = enumType.enumConstantDirectory().get(name);//通过反射获得了对应的name
    if (result != null)
        return result;
    if (name == null)
        throw new NullPointerException("Name is null");
    throw new IllegalArgumentException(
        "No enum constant " + enumType.getCanonicalName() + "." + name);
}
```
3. 枚举的创建是`线程安全`的.


## EnumSet/EnumMap
> [EnumSet解析](https://www.cnblogs.com/swiftma/p/6044718.html)
> [EnumMap解析](http://www.cnblogs.com/swiftma/p/6044672.html)
官方推荐:若是有一组集合作为枚举值的话,推荐用EnumSet或者EnumMap,可以看看`EffectiveJava`的**32,33**条.介绍了EnumSet和EnumMap的应用场景
> Note that when using an enumeration type as the type of a set
  or as the type of the keys in a map, specialized and efficient
  {@linkplain java.util.EnumSet set} and {@linkplain
  java.util.EnumMap map} implementations are available.

## 枚举用法之一:接口组织枚举
还不太理解这种组织的方式,`EffectiveJava`第**34**条
```java
interface Food {
      enum Coffee implements Food {

          BLACK_COFFEE, DECAF_COFFEE, LATTE, CAPPUCCINO

      }

      enum Dessert implements Food {

          FRUIT, CAKE, GELATO

      }
}
```
