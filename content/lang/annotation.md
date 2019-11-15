---
title: "annotation"
date: 2019-08-15T13:19:18+08:00
draft: true
---

# 注解Annotation

> [注解解析](http://www.cnblogs.com/swiftma/p/6838654.html) 

注解是从`JDK5`开始引入的新特性,注解为我们带来了`声明式编程`的风格.注解可以应用到包,类型,构造方法,成员变量等语句中.
## 声明一个注解
以`override`为例:
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
`元注解`
* **Target**:表明对哪种类型起作用,`ElementType`是目标值的枚举,主要有TYPE,FIELD,METHOD,PARAMETER,CONSTRUCTOR,LOCAL_VARIABLE,ANNOTATION_TYPE,PACKAGE几种,在`override`注解中,其只对方法起作用
* **Retention** 表示注解保留到什么时候,也是通过`RetentionPolicy`枚举值来表明的.
    * SOURCE 只在源代码中保留,编译器将代码编译为字节码文件时就会丢掉
    * CLASS 保留到字节码文件中,但虚拟机将class文件加载到内存时不一定会在内存中保留
    * RUNTIME 一直保留到运行时
* **Inherited** 注解是不能继承的,但是可以通过该注解变相的实现它.
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Inherited
    @interface test {

    }

    @test
    static class A {

    }

    static class B extends A {

    }

    public static void main(String[] args) {
        System.out.println(B.class.isAnnotationPresent(test.class));//out: true
    }

    ```
* **@Documented** 生成javadoc时会包含注解


## 实现一个简单的注入
自定义了`newObj`注解,`Class A`中注入了`Class B`的,通过反射我们可以得到`ClassA`中的属性,并给属性初始化一个实例.
```java
public class Test {

    public static void main(String[] args) {
        A instance = NewObjContainer.getInstance(A.class);
        instance.printA();

    }



}
 class NewObjContainer{
    public static <T> T getInstance(Class<T> clz){
        try {
            T obj = clz.newInstance();
            Field[] declaredFields = clz.getDeclaredFields();
            for (Field field : declaredFields) {
                if (field.isAnnotationPresent(newObj.class)) {//有该注解
                    if (!field.isAccessible()){
                        field.setAccessible(true);
                    }
                    Class aClass = field.getType();
                    field.set(obj,getInstance(aClass));
                }
            }
            return obj;
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return null;
    }
}


class A {
    @newObj
    private B b;

    public void printA(){
        b.printB();
        System.out.println("this is A...");
    }

}

class B {
    public void printB(){
        System.out.println("this is b...");
    }
}


@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface newObj {

}
```
