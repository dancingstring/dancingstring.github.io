---
title: volatile的使用及DCL模式
date: 2017-09-30
---

## volatile要点

- volatile 只保证变量的线程可见性，不保证变量的原子性（只对赋值起作用），另外一个作用是防止重排序。
- volatile 典型的使用场景，作为boolean，采用while来做信号通知
- 不采用volatile的dcl容易出错（DCL即Double Check Locking模式，就是双加锁检查模式。）

## 代码实例

单例的延迟加载实现

```
package cn.javass.dp.singleton.example2;
 
/**
* 懒汉式单例示例
*/
public class Singleton {
    /**
     * 定义一个变量来存储创建好的类实例
     */
    private static Singleton uniqueInstance = null;
    /**
     * 私有化构造方法，好在内部控制创建实例的数目
     */
    private Singleton(){
        //
    }
    /**
     * 定义一个方法来为客户端提供类实例
     * @return 一个Singleton的实例
     */
    public static synchronized Singleton getInstance(){
        //判断存储实例的变量是否有值
        if(uniqueInstance == null){
            //如果没有，就创建一个类实例，并把值赋值给存储类实例的变量
            uniqueInstance = new Singleton();
        }
        //如果有值，那就直接使用
        return uniqueInstance;
    }
}
```

上面的代码虽然在多线程的情况是线程安全的，也确保了只有一个实例，但是在高并发的情况下，synchronized 关键字会使得性能下降。

因此，不用synchronized修饰此方法，改在保证实例化实例的时候只有一个线程执行就可以了。因而有了双重检测模式的应用。

假设线程A,B同时进入该方法，都检测到instance为null，然后假设A先占用了同步锁，然后实例化对象，之后B也会占用同步锁去实例化，因而这里需要再一次检测实例时是否已经被创建了，此为双重检测的来源。

```
package cn.javass.dp.singleton.example10;
 
public class Singleton {
    /**
     * 对保存实例的变量添加volatile的修饰
     */
    private volatile static Singleton instance = null;
    private Singleton(){
 
    }
    public static  Singleton getInstance(){
        //先检查实例是否存在，如果不存在才进入下面的同步块
        if(instance == null){
            //同步块，线程安全的创建实例
            synchronized(Singleton.class){
                //再次检查实例是否存在，如果不存在才真的创建实例
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

## 为什么使用volatile关键字

### 1、volatile能够保证变量在多线程之间的可见性

即 JVM的volatile语义保证了一个线程更新了这个变量，其他线程再下次读取的时候，回去刷新本地缓存

### 2、为什么在JDK5之前是不安全的

在JDK5之前，java的内存模型允许out-of-writer，具体如下：

在JDK5之前，java的内存模型允许out-of-writer，具体如下：
（1）java的new不是原子的，具体在更细的层面还是有诸多步骤：

- A、 分配新对象的内存
- B、调用类的构造器，初始化成员字段
- C、 instance被赋为指向新的对象的引用。

说白了，JDK5之前不能保证有volatile修饰的对象构造内部是有序的。

（2）线程A发现instance没有被实例化，它获得锁，然后去实例化该对象，JVM容许在没有完全实例化完成的时候，将实例的指针赋给这个instance变量，而此时instance==null就为false了，在初始化完成之前，线程B进入此方法，发现instance不为空，认为已经初始化完成了，于是便使用了这个尚未完全初始化的实例对象，可能引起其他的异常。

## 64位long和double

在JVM规范中Java内存模型要求lock、unlock、read、load、assign、use、store、write这8个操作必须是原子的，但是对于64位的long和double来说，如果没有被volatile修饰符修饰，那么可以不是原子的，注意是可以，即虚拟机在实现的时候可以选择是否是原子操作。目前几乎所有的商用虚拟机都将此实现为原子操作，因此不必每次用到它们都去加volatile修饰。

## 参考

- [JVM并发机制的探讨——内存模型、内存可见性和指令重排序](http://www.importnew.com/5869.html)
- [关于Java中的volatile型变量](http://blog.csdn.net/pnet2008/article/details/16812115)

