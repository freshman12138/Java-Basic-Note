# JAVA 异常 #
2019-03-15 16:07:59 
## 1.异常处理机制 ##
### 1.1Error和Exception的区别: ###
1. Error:程序无法处理的系统错误,编译器不做检查(遇到此类异常,建议让程序终止)
2. Exception:程序可以处理的异常,捕获后可能恢复(遇到此类异常,建议主动处理,不能随意终止异常)

---
**总结::Error是程序无法处理的错误,Exception是可以被处理的异常**
![](https://i.imgur.com/62eTE5k.png)

### 1.2常见的Error以及Exception ###
**RuntimeException:**
1. NullPointerException - 空指针引用异常
2. ClassCastException - 类型强制转换异常
3. IllegalArgumentException - 传递非法参数异常
4. IndexOutOfBoundsException - 下标越界异常
5. NumberFormatException - 数字格式异常

----
**非RuntimeException:**
1. ClassNotFoundException - 找不到指定的class的异常
2. IOException - IO操作异常

----
**Error:**
1. NoClassDefFoundError - 找不到class定义的异常
2. StackOverFlowError - 深度递归导致栈被耗尽而抛出的异常
3. OutOfMemoryError - 内存溢出的异常

## 2.java的异常处理机制 ##
- 抛出异常:创建异常对象,交由运行时系统处理
- 捕获异常:寻找合适的异常处理器处理异常,否则终止运行



