# JVM #
2019-03-07 10:49:59 
## 1.JVM内存模型 ##
![](https://i.imgur.com/MfkS59D.png)
--------
- Class Loader:加载特定格式的.class文件
- Execution Engine(解析器):对命令进行解析
- Native Interface:融合不同开发语言的原生库供java调用
- Runtime Data Area:JVM内存空间结构模型

## 2.java的反射机制 ##
**java的反射机制:在java运行的状态中,对于任何一个类都可以知道这个类的属性和方法,对于任何一个对象都可以调用该对象的属性和方法**
![](https://i.imgur.com/ETFIEMz.png)
    
package com.JavaBasicReflect;

public class Robort {

    private String Name;;
    public void SayHi(String name)
    {
        System.out.println(name+" say hi to "+Name);
    }
    private String SayHello(String name)
    {
        return "welcome,"+name;
    }
}  

---
package com.JavaBasicReflect;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Test {

    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {
        Class robort = Class.forName("com.JavaBasicReflect.Robort");
        Robort rb =(Robort) robort.newInstance();//类的初始化,T.newInstance()初始化得到的是Object类,要进行强制类型转换
        System.out.println("the class name :"+ robort.getName());
        Method method1 = robort.getDeclaredMethod("SayHello", String.class);
        method1.setAccessible(true);//必须设置为true,要不然无效
        Object ob= method1.invoke(rb,"Kobe");
        System.out.println(ob.toString());
        Method method2 = robort.getMethod("SayHi", String.class);
        method2.invoke(rb,"lu");
        Field field = robort.getDeclaredField("Name");
        field.setAccessible(true);
        field.set(rb,"Tom");
        method2.invoke(rb,"lu");
    }

}

![](https://i.imgur.com/AMjLQJb.png)

**上面可以发现,即使private只能在本类中使用,但是我们在别的类中调用private 属性和方法**

## 3.ClassLoader ##
类从编译到执行的过程?  
1. 编译器把Test.java源文件编译为Test.class字节码文件  
2. ClassLoader把Test.class字节码转换为Class<Test>对象  
3. JVM把Class<Test>对象实例化为Test对象

### a.谈谈ClassLoader ###
**ClassLoader在Java中有着非常重要的作用,它的主要作用是装载class的加载阶段,它的主要作用是从外部系统获得class的二进制数据流加载到系统内部,它是java的核心组件,所有的class都是由ClassLoader加载的,ClassLoader负责将二进制的数据流装载到系统内部,然后由JVM进行连接,初始化等操作**

### b.ClassLoader的种类 ###
- BootStrapClassLoader:c++编译的,加载类的核心库
- ExtClassLoader:java编译,加载扩展类
- AppClssLoader:java编译,加载应用程序下面的类
- 自定义的ClassLoader:java编写,定制化加载


### c.类加载的双亲委托加载机制 ###
从自定义ClassLoader的缓存中查找是否已经加载,如果有,直接加载,如果没有,请求AppClassLoader加载.从APPClassLoader的缓存中查找是否已经加载了类,如果没有,委托ExtClassLoader加载.从ExtClassLoader的缓存中查找是否已经加载了类,如果没有,委托BootStrapLoader加载.从BootStrapClassLoader的缓存中查找是否已经加载了类,如果没有就在指定的路径上尝试加载($JAVA_HOME/jre/lib),如加载失败.就委派ExtClassLoader在指定的路径上加载.如果失败会就会委托APPClassLoader去程序的路径下加载.如果还是没有就抛出类加载异常.

#### 类的双亲委托加载机制的优势:1.安全 2.确保每个类只加载一次,避免浪费资源 ####

### d.类的加载方式 ###
A-显示加载:new   
B-隐式加载:Class.forName()或者LoadClass

**Class.forName()加载的类是已经初始化的类,而LoadClass加载的类还没有链接**

---
### e.类的装载过程 ###
1.加载:通过ClassLoader加载.class字节码文件,生成Class对象

2.链接:  
a.校验:验证加载的class对象的正确性和安全性  
b.准备:为类变量分配存储空间和设置类变量的初始值  
c.解析:JVM将常量池中的符号引用转化为直接引用

3.初始化:执行类变量的赋值和执行静态代码块

---
## 4.java的内存结构模型 ##
### A.递归为什么会引起java.lang.StackOverflowError异常? ###
**在递归过程中,栈帧数超过虚拟机栈的深度(解决的办法:1.限制递归数 2.替换递归)**

栈帧的组成:局部变量表,操作数栈,动态链接(将常量池中的符号引用转换为直接引用),返回操作地址

虚拟机栈帧过多会造成java.lang.outofmemoryerror异常

### B.jvm的内存机构模型(根据线程划分的) ###
![](https://i.imgur.com/D10IrgO.png)
#### 线程私有:-1.程序计数器(不存在oom) -2.虚拟机栈(java方法,存在sof,oom) -3.本地方法栈(native方法,存在sof,oom) ####
#### 线程共享;-1.Metaspace (类加载信息 ,存在oom)-2.堆(数组和类对象,存在oom){jdk 1.8以后就把常量池(符号引用,存在oom)移到堆中了} ####
---
程序计数器(pc寄存器):

- 逻辑的计数器
- 线程独立(不存在线程安全的问题)
- 对java方法的计数器,对native方法计数是Undefined
- 不会发生内存泄露(oom)

---
java虚拟机栈(stack):

- java方法运行空间(方法调用进入栈帧,方法返回退出栈帧)
- 包含多个栈帧数
- 随着线程而创建或者消亡

---

本地方法栈(与虚拟机栈相似)--存储的是native方法的运行空间

---

MetaSpace(元空间):

- 使用的是本地内存,不是PermGen(永久代)使用的JVM内存
- 元空间把字符串常量池移到方法区了

#### MetaSpace和PermGen的区别: ####
1. 元空间使用的是本地内存,永久代使用的是虚拟机内存

#### MetaSpace比PermGen的优势: ####
1. 字符串常量池存在PermGen中,容易出现oom
2. PermGen会给GC带来复杂的操作
3. 类和方法的大小难以确定,PermGen对于他们指定的大小会带来困难
4. MetaSpace方便Hotspot与其他的JVM集成

---
堆(heap):**存放实例对象**

- 对象实例化分配的区域
- GC管理的主要区域

---
**总结:栈不需要GC(线程消亡后会被自动释放),堆需要GC**

---
### C.JVM内存模型中栈和堆的区别: ###
1. 垃圾回收管理方式:栈自动释放,堆需要GC
2. 空间大小:栈比堆小的多
3. 碎片相关:栈产生的碎片远远小于堆
4. 分配方式:栈->静态分配/动态分配,堆->动态分配
5. 效率:栈的效率大于堆

## 5.jVM的性能调优 ##
-Xss:规定了每个线程的所创建的虚拟机栈的大小

-Xms:堆的初始值

-Xmx:堆能达到的最大值