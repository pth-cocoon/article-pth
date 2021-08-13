# Java的引用类型有几种？区别是什么？

在Java语言中，除了基本类型以外，其余对象均以“引用”的形式传递。在调用函数传参时，实际上传递的是指向对象的“引用”，并非对象的实际值。

## 什么是引用的传递？

举例来说，也就是：函数A，创建了对象objectA，作为参数传递给函数B，函数B对传进来的参数进行了修改，回到函数A时，对象ojbectA依然被修改。

如下面的例子：

```java
public class Demo {

  public static void main(String[] args) {
    funcA();
  }

  public static void funcA() {
    User objectA = new User();
    objectA.username = "admin";
    objectA.password = "admin";
    System.out.println("调用funcB之前,Password：" + objectA.password);
    funcB(objectA);
    System.out.println("调用funcB之后,Password：" + objectA.password);
  }

  public static void funcB(User user) {
    user.password = "funcBPassword";

  }


  public static class User {
    String username;
    String password;
  }

}
//打印结果：
//调用funcB之前,Password：admin
//调用funcB之后,Password：funcBPassword
```



## 四种引用以及其案例

### 强引用

强引用，是Java最常被使用的引用。当你给对象用=赋值一个新的对象，此时对象的引用就是强引用。

##### 应用场景

所有场景

#### 特点和生命周期

强引用的生命周期是最长的引用，直到被显式地清楚引用（赋值为null）前，该引用会一直存在。宁可抛出**OOM**(*OutOfMemoryError*)，也不会清理其内存的一种引用类型。

生命周期： 最长

#### 案例

##### 测试环境

java8，idea，内存指定为5m(实际上，可用空间不足5M)

```Java

public class ClassMemory {
  //模拟1M的内存
  private byte[] memory = new byte[1048576];

}


  private static void strongReference() {
    Object o1 = new ClassMemory();
    System.out.println(o1);
    Object o2 = new ClassMemory();
    System.out.println(o1);
    Object o3 = new ClassMemory();
    System.out.println(o1);
    Object o4 = new ClassMemory();
    System.out.println(o1);
  }
  
  //打印结果：
//vin.cocoon.refrence.ClassMemory@61bbe9ba
//vin.cocoon.refrence.ClassMemory@61bbe9ba
//vin.cocoon.refrence.ClassMemory@61bbe9ba
//Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```

可以看出：为o1,o2,o3分别创建对象，并传递引用时，打印o1的内存地址，均正常打印。直到为o4赋值新的对象。此时，内存不足。抛出了 **OOM**(*OutOfMemoryError*)。



### 软引用

软引用，只有在内存不足时，才会将其清理。

##### 应用场景

缓存

#### 特点和生命周期

软引用的特点是，gc并不会将其清理，只有在jvm认为内存不足时，在出发OOM之前，清理软引用；

生命周期： 次于强引用

#### 案例

```java
  private static void softReference() {
    SoftReference<ClassMemory> o1 = new SoftReference<>(new ClassMemory());
    System.out.println(o1.get());
    SoftReference<ClassMemory> o2 = new SoftReference<>(new ClassMemory());
    System.out.println(o1.get());
    SoftReference<ClassMemory> o3 = new SoftReference<>(new ClassMemory());
    System.out.println(o1.get());

    //这时候即将出发内存不足
    System.out.println("即将触发内存不足！");
    SoftReference<ClassMemory> o4 = new SoftReference<>(new ClassMemory());
    System.out.println("o1" + o1.get());
    System.out.println("o2" + o2.get());
    System.out.println("o3" + o3.get());
    System.out.println("o4" + o4.get());
    System.gc();
    System.out.println("o4" + o4.get());
  }

//打印结果
//vin.cocoon.refrence.ClassMemory@61bbe9ba
//vin.cocoon.refrence.ClassMemory@61bbe9ba
//vin.cocoon.refrence.ClassMemory@61bbe9ba
//即将触发内存不足！
//o1null
//o2null
//o3null
//o4vin.cocoon.refrence.ClassMemory@610455d6
//o4vin.cocoon.refrence.ClassMemory@610455d6
```

可以看出，在前三个对象的引用创建时，内存充足均正常打印o1引用的内存地址。直到o4的引用创建时，内存不足，将软引用内存释放。

o1,o2,o3均被清理，o4内存地址正常打印。于此同时，System.gc()后，o4的引用地址仍然正常打印。由此得出，System.gc() 并不会清理软引用。



### 弱引用

弱引用与软引用类似，同样在内存不足时被清理，但除此之外，弱引用无法豁免被gc。

##### 应用场景

缓存

#### 特点和生命周期

弱引用的特点是，在jvm认为内存不足时，在出发OOM之前，清理软引用；除此之外，在触发gc时，弱引用同样会被清理。

生命周期： 次于软引用

#### 案例

```java
  private static void wearReference() {
    WeakReference<ClassMemory> o1 = new WeakReference<>(new ClassMemory());
    System.out.println(o1.get());
    WeakReference<ClassMemory> o2 = new WeakReference<>(new ClassMemory());
    System.out.println(o1.get());
    System.out.println("即将触发内存不足！");
    WeakReference<ClassMemory> o3 = new WeakReference<>(new ClassMemory());
    System.out.println("o1" + o1.get());
    System.out.println("o2" + o2.get());
    System.out.println("o3" + o3.get());

    WeakReference<ClassMemory> o4 = new WeakReference<>(new ClassMemory());
    System.out.println("o4" + o4.get());
    System.out.println("即将触发GC");
    System.gc();
    System.out.println("o3" + o3.get());
    System.out.println("o4" + o4.get());
}
/**
打印结果：
vin.cocoon.refrence.ClassMemory@61bbe9ba
vin.cocoon.refrence.ClassMemory@61bbe9ba
即将触发内存不足！
o1null
o2null
o3vin.cocoon.refrence.ClassMemory@610455d6
o4vin.cocoon.refrence.ClassMemory@511d50c0
o3null
o4null
**/
```

可以看出，o1，o2引用创建时正常打印o1的内存地址。o3的引用创建时，内存不足，o1，o2被清理。o3正常打印；

创建o4的引用，仍然正常打印。随后执行 System.gc() 手动触发GC，o3，o4均被清理。





### 幻象引用

无法通过虚引用访问对象的任何属性或函数。幻象引用仅仅是提供了一种确保对象被 finalize 以后，做某些事情的机制。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

#### 特点和生命周期

幻象引用任何时候都可以被回收，仅使用于少量特定场景。

生命周期： 最短，任何时候均可能被回收。



## 总结

对比表格：

|             | 强引用 | 软引用     | 弱引用     |
| ----------- | ------ | ---------- | ---------- |
| 生命周期    | 最长   | 次于强引用 | 次于软引用 |
| OOM前被清理 | 否     | 是         | 是         |
| Gc前被清理  | 否     | 否         | 是         |
| 应用场景    | 全部   | 缓存等     | 缓存等     |



## 感谢

感谢能阅读到这里的你！

希望本文对你可以有帮助，存在的漏洞以及待完善的地方也欢迎指出！

期待你的点赞，收藏，以及关注！你的支持，是我继续努力的最大动力！

最后，也欢迎你关注我的公众号【猿树洞】

![yuanshudong_qr_code](https://pth-1252643971.file.myqcloud.com/typora/yuanshudong_qr_code.jpg)