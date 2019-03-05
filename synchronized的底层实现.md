# synchronized 底层实现方法 #
2019-03-02 12:12:14 

## 1、synchronized的底层语义原理 ##
java 虚拟机中的同步(Synchronization)是基于进入和退出管程(Monitor)对象实现的,无论是显示同步(同步代码块,既有明确的monitorenter和monitorexit指令)还是隐式同步(同步方法,既并不是由monitorenter和monitorexit指令来实现的,而是由方法调用指令读取运行时常量池中方法的ACC_SYNCHRONIZED标志来隐式实现的)都是如此.
### 理解JAVA对象头与Monitor ###
在JVM中,对象在内存中的布局分为三块区域:对象头,实例数据,对齐填充.
![](https://i.imgur.com/3B6wQKS.png)

- 实例数据:存放类的属性数据信息,包括父类的属性信息,如果是数组的实例部分还包括数组的长度,这部分内存按4字节对齐

- 对齐填充:由于虚拟机要求对象起始地址必须是8字节的整数倍,填充数据不是必须存在的,仅为了字节对齐

- 对象头:它是实现synchronized锁对象的基础,synchronized使用的锁对象是存储在java对象头里的,主要结构是由Mark Word 和 Class Metadata Address组成  

#### 说明:Mark Word存储对象的hashCode,锁信息或分代年龄或GC标志等信息...Class Metadata Address存储指针对象的类元数据,JVM通过这个指针确定该对象是哪个类的实例 ####
![](https://i.imgur.com/DoKbWkI.jpg)

## 2.重量级锁 ##
重量级锁也就是我们平时所说的synchronized的对象锁,锁的标识位为10,其中Mark Word指针指向堆中的monitor对象(管程或监视器)的起始位置,每一个对象都有着一个monitor对象与之关联.monitor对象可以与对象一起创建销毁,当前线程试图获取对象锁时自动生成,但当monitor被其他线程所持有时,该monitor就会被锁定,当前线程会进入EntryLIst等待线程释放掉对象锁的monitor然后获取对象锁的机会.  
monitor对象的关键字段:cxq,EntryList(锁池),WaitSet(等待池),owner...<cxq,EntryList,WaitSet>都是用来保存ObjectWaiter对象的链表结构,owner指向持有锁的线程

### 重量锁实现的过程 ###
当一个线程试图获得锁时,如果该锁已经被占用的话,就会把该线程封装成ObjectWaiter对象插入到cxq队列的尾部,然后暂停当前线程.当持有锁的线程释放掉锁之前,会把cxq里面的所有对象都移动到EntryList中,释放后会唤醒EntryList队首的线程,获取对象锁的monitor,并把owner置为当前线程.  
如果在该线程执行过程中执行了Object#wait方法,会把当前线程对应的ObjectWaiter移动到WaitSet中,然后释放掉锁,并把owner置为null.当wait的线程被notify后,其对应的ObjectWaiter就会从WaitSet转移到EntryList中.

### A.synchronized代码块的实现原理 ###
同步代码块的实现是使用了monitorenter和monitorexit字节码指令,其中monitorenter指向的是同步代码块的起始位置,monitorexit指向的是同步代码块的结束位置.当运行monitorenter时,当前线程试图获取对象锁的monitor的持有权,如果获得该对象锁的monitor对象,执行其中的同步代码.线程执行完毕后,monitorexit指令被执行,执行线程将释放掉monitor对象,并把owner置为null.

#### 总结:每段同步代码块一定具有一个monitorenter和两个monitorexit...其中一个monitorexit是与monitorenter对应的字节码指令,另一个monitorexit是当编译异常时,还是要进行monitorexit的操作即对monitor对象的持有权的释放,使当线程执行同步代码块的内容发生异常时,不会持续占有对象锁(cpu及共享数据). ####

### B.synchronized方法的底层原理 ###
同步方法并不是通过字节码的指令来实现同步的实现,而是通过方法的调用和返回操作.JVM可以从方法常量池中的方法表结构(method_info_Structure)中的ACC_SYNCHRONIZED访问标志来区分是否为同步方法.在方法调用时,调用指令会检查方法中的ACC_SYNCHRONIZED是否被设置,如被设置了,当前线程就会持有monitor对象(当然该monitor对象没有被其他线程所占用),然后执行方法体里面的代码,最后执行完毕后就释放掉monitor对象.**注:如果一个同步方法执行期间抛出异常,并且在方法内部无法处理次异常,那这个同步方法所持有的monitor对象还是会在异常抛出同步方法外自动释放.**

### 重量级锁总结:在jdk早期的版本,synchronized属于重量级锁,效率低下,因为monitor是依赖于底层的操作系统的Mutex Lock来实现的,而操作系统实现线程的切换,花销是巨大的.所以效率低下.但java 6以后就进行一定的优化,为了减少获得锁和释放锁所带来的消耗,引入了轻量级锁和偏向锁 ###

## 3.JVM对synchronized的优化 ##
锁的状态一共4种:1.无锁状态 2.偏向锁 3.轻量级锁 4.重量级锁---锁的升级:锁可以从偏向锁升级到轻量级锁,在升级到重量级锁.锁的升级是单向的,只能从低到高升级,不会出现锁的降级

### A.偏向锁 ###
偏向锁是java 6以后加入的新锁,它是一种针对加锁过程的优化手段.目的:是为了减少同一线程多次获得锁所带来的消耗.**偏向锁的核心思想:如果一个线程获得了锁,那么锁就进入了偏向模式,此时Mark Word的结构也变为偏向锁结构,当这个线程再次请求锁时,无需再做任何同步的操作,既在获取锁的过程,省去了大量有关锁申请的操作,从而也就提供程序的性能**
### 总结:偏向锁在同一线程连续多次申请同一个对象锁,具有很好的优化效果 ###

#### 偏向锁对象创建 ####
当新创建一个对象的时候,如果该对象的class属性没有关闭时(默认的所有的class的偏向模式都是开启的),那新创建对象的Mark Word就是可偏向状态,此时Mark Word中的Thread id为0,表示未偏向任何线程,也叫匿名偏向.

#### 偏向锁的加锁过程 ####
case1.当该对象锁第一次被线程获得的时候,如发现该对象锁是匿名偏向状态,则会用CAS指令,将Mark Word中的Thread id改为当前线程id.如果成功,则代表获得了偏向锁,继续执行同步代码块中的代码.否则,将偏向锁撤销,升级为轻量级锁.  

case2.当被偏向的线程再次进入同步块中,发现锁对象偏向的就是当前线程,经过一些额外的检查,就继续执行同步块的代码.

case3.当其他线程进入到同步块中,发现锁对象已经有其他的偏向线程了.则查看偏向的线程是否存活,如果存活且还在同步代码块中,则把锁升级为轻量级锁.原偏向的线程继续持有锁,当前线程则走向锁升级的逻辑里..如果偏向的线程不存活或者不在同步代码块中,就将锁对象的Mark Word改为无锁状态,或者升级为轻量级锁.

#### 总结:偏向锁升级的时机:当锁对象已经有了偏向线程,其他线程尝试获得该偏向锁,则该锁对象就会升级成轻量级锁(批量重偏向除外) ####

#### 偏向锁的解锁过程 ####
当有其他的线程尝试获得锁时,是根据遍历偏向线程的lock record来确定该线程是否还在执行同步代码块的代码.因此,偏向锁的解锁过程非常简单,仅仅将栈中最近的一条lock record的obj字段置为null.需要注意的是,偏向锁的解锁并不会修改锁对象的Thread id.

### B.轻量级锁 ###
倘若偏向锁失败,虚拟机并不会立即升级为重量级锁,它还会尝试使用一种叫做轻量级锁的优化手段,此时Mark Word的结构也变为轻量级锁的结构.在java运行的过程中,同步代码块的代码都是不存在竞争的.不同的线程交替的执行同步代码块的代码.这种情况下就引入了轻量级锁的概念.
### 轻量级锁适应的场景就是线程交替执行同步块的场合,如果存在多条线程同一时间访问同一个对象锁的时候,就会导致轻量级锁膨胀为重量级锁 ###

### C.自旋锁 ###
轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。这是基于在大多数情况下，由于线程持有锁的时间都不会太长，如果直接挂起操作系统层面的线程可能会得不偿失，毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，因此自旋锁会假设在不久将来，当前的线程可以获得锁，因此虚拟机会让当前想要获取锁的线程做几个空循环(这也是称为自旋的原因)，一般不会太久，可能是50个循环或100循环，在经过若干次循环后，如果得到锁，就顺利进入临界区。如果还不能获得锁，那就会将线程在操作系统层面挂起，这就是自旋锁的优化方式，这种方式确实也是可以提升效率的。最后没办法也就只能升级为重量级锁了。

### D.消除锁 ###
消除锁是虚拟机另外一种锁的优化，这种优化更彻底，Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间

## 关于synchronized可能需要了解的关键点 ##
### synchronized的可重入性 ###
从互斥锁的设计上来说，当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种情况属于重入锁，请求将会成功，在java中synchronized是基于原子性的内部锁机制，是可重入的，因此在一个线程调用synchronized方法的同时在其方法体内部调用该对象另一个synchronized方法，也就是说一个线程得到一个对象锁后再次请求该对象锁，是允许的，这就是synchronized的可重入性。
  
 package com.Thread;


public class SyncTest implements Runnable {

    static SyncTest instance = new SyncTest();
    static int i = 0;
    static int j = 0;

    @Override
    public void run() {
        for (int j = 0; j < 1000000; j++) {

            //this,当前实例对象锁
            synchronized (this) {
                i++;
                increase();//synchronized的可重入性
            }
        }
    }

    public synchronized void increase() {
        j++;
    }


    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i+".........."+j);
    }
}  
这种情况是可以的
属于synchronized的可重入性
### 线程中断与synchronized ###
中断的意思就是在Thread的运行(Run方法)打断它,java提供了三种有关线程中断的方法
`  

//中断线程（实例方法）
public void Thread.interrupt();

//判断线程是否被中断（实例方法）
public boolean Thread.isInterrupted();

//判断是否被中断并清除当前中断状态（静态方法）
public static boolean Thread.interrupted();
`  
#### 总结:1-当一个线程在阻塞状态时或者将要执行一个阻塞状态,使用Thread.Interrupt()方法中断线程,注意此时应抛出InterruptedExcution异常,并且将中断状态重置复位(中断状态改为非中断状态) 2-当线程处于执行的状态下,我们也可以调用Thread.Interrupt()进行线程中断,但是必须手动判断是否中断状态,并编写中断线程的代码(其实就是中断Run方法)------综上所述:Thread.Interrupt()方法只是给线程一个要中断线程的讯号,并不是马上就强制关闭线程####
`package com.Thread;

import java.util.concurrent.TimeUnit;

public class InterruptDemo {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(){
            @Override
            public void run() {
                //线程处于阻塞状态时的Demo
                /*try{
                    while(true)
                    {
                        TimeUnit.SECONDS.sleep(2);
                    }
                } catch (InterruptedException e) {
                    //e.printStackTrace();
                    System.out.println("interrupted when sleep");
                    boolean interrupt = this.isInterrupted();
                    System.out.println("interrupt"+"  =  "+interrupt);
                }*/
                /*
                * 输出的结果:
                * interrupted when sleep
                * interrupt  =  false
                * 在线程阻塞的状态下,调用.Interrupt()可以中断线程,抛出异常,并且将中断状态复位
                * */

                //在线程运行的情况Demo
               /* while(true)
                {
                    System.out.println("can't stop!");
                }*/
               /*
               * 输出结果:
               * can't stop
               * .
               * .
               * .
               * can't stop
               * 表明在线程执行的情况下,调用Interrupt()不会立即强制中断线程
               * */

               //在线程执行的情况下如何中断线程Demo
               while (true)
               {
                   if(this.isInterrupted())
                   {
                       System.out.println("线程中断");
                       break;
                   }

               }
               System.out.println("自己判断中断状态,然后手动中断run方法");
               /*
               * 输出结果:
               * 线程中断
               * 自己判断中断状态,然后手动中断run方法
               * 表明如果在线程运行的情况下,想要中断线程,必须自己手动判断线程的状态,然后终止Run方法
               * */
            }
        };
        thread.start();
        //TimeUnit.SECONDS.sleep(2);
        thread.interrupt();
    }

}
` 
