# Java的Interface到底是什么？

曾经在学Java的时候，一直搞不懂Interface到底是什么？定义了空函数，起不到任何作用。甚至实现接口的类，也必须要实现Interface的全部函数，给使用的时候反而带来了麻烦。虽然Java8带来了default 实现，但是我认为 default 其实一定程度上破坏了Interface的特性，这个后面说。那么问题来了，Java的interface到底做了什么？有什么用？

在这里，不去用一些猫猫狗狗的例子，我将会用一些更贴近实际项目的例子。

## Interface 的定义是什么？

用人话来说其实无非就是，规定了一组函数的 方法名，参数，返回值。实现该Interface的类，必须实现这一组函数。暂时抛开default实现。

### 举个栗子🌰

#### 接口定义

这里定义了一个SmsService

```java
public interface SmsService {

  /**
   * 发送短信
   *
   * @param phone 手机号
   * @param msg   信息
   * @return 成功返回true/否则返回false
   */
  boolean sendSms(String phone, String msg);

}
```

规范其行为：接收 手机号 和 要发送的信息，返回布尔。

这里，接口坐了两件事：

1. 规定发送短信需要什么？必须要有手机号（发给谁），发送的内容（发什么）。
2. 发送成功后返回什么？成功返回ture，否则返回false。

所有的发送短信的服务，都需要遵循这两点。

#### 接口实现

```java
public class AliSmsService implements SmsService {

  @Override
  public boolean sendSms(String phone, String msg) {
    System.out.println("用户的手机号：" + phone);
    System.out.println("发送的信息是：" + msg);
    System.out.println("AliService 发送短信!");
    return true;
  }
}

public class TencentSmsService implements SmsService {

  @Override
  public boolean sendSms(String phone, String msg) {
    System.out.println("用户的手机号：" + phone);
    System.out.println("发送的信息是：" + msg);
    System.out.println("TencentSmsService 发送短信!");
    return true;
  }
}
```

这里分别是两个类，都实现了SmsService接口。并且遵循SmsService接口定义的函数，存在自己的实现。

可能现在你还不懂，接口的意义是什么？因为使用的时候，还不是直接new TencentSmsService（）,然后调用其sendSms函数么？那是不考虑扩展性。如果使用的地方是下面这种情况呢？

## interface的实际运用

#### 定义接口的多实现解决的问题

```Java
public class SmsDemo {

  private static List<SmsService> smsServiceList = new ArrayList<>();

  public static void main(String[] args) {
    addAli();
    addTencent();
    //使用已注册到smsService为用户发送短信
    for (SmsService smsService : smsServiceList) {
      smsService.sendSms("13333333333", "您的快递到了请注意查收！");
    }
  }

  /**
   * 注册AliSmsService到smsServiceList
   */
  private static void addAli() {
    SmsService smsService = new AliSmsService();
    smsServiceList.add(smsService);
  }

  /**
   * 注册TencentSmsService到smsServiceList
   */
  private static void addTencent() {
    SmsService smsService = new TencentSmsService();
    smsServiceList.add(smsService);
  }

}
```

这里模拟一个场景，smsServiceList，是一个短信服务中心，这里面装载了支持的短信服务。下面分别注册了Ali和Tencent的短信服务。发送短信的时候，使用所有的短信提供商为用户发送短信。

这时候，这个短信服务中心smsServiceList支持的是SmsService的实现，只要是SmsService的实现，都可以注册进去。这样，就算你需要支持再多的短信服务提供商，都不需要修改发送的地方。你只需注册更多的服务提供商的SmsService的实现即可。

当然，这时候也许会有人问“这么多短信提供商，都注册到服务中心，根本不符合业务啊？”好的，那下面这种情况呢？

#### 模拟真实业务场景的实现

首先，业务场景是： 公司业务量较大，单个短信服务提供商可用度不够。并且时常偶尔会出现服务商欠费的问题。所以，需要使用和对接多个短信服务商完成发送短信的业务。默认注册了阿里，后续又注册了腾讯的短信服务。

```java
SmsService interface不变
//两个实现，分别模拟了发送失败的情况


/**
 * 阿里短信服务
 */
public class AliSmsService implements SmsService {

  @Override
  public boolean sendSms(String phone, String msg) {
    System.out.println("用户的手机号：" + phone);
    System.out.println("发送的信息是：" + msg);
    System.out.println("AliService 发送短信!");
    //当前时间戳与2取余，模拟发送失败的情况
    return (System.currentTimeMillis() % 2 == 0);
  }
}

/**
 * 腾讯短信服务
 */
public class TencentSmsService implements SmsService {

  @Override
  public boolean sendSms(String phone, String msg) {
    System.out.println("用户的手机号：" + phone);
    System.out.println("发送的信息是：" + msg);
    System.out.println("TencentSmsService 发送短信!");
    //当前时间戳与2取余，模拟发送失败的情况
    return (System.currentTimeMillis() % 2 == 0);
  }
}


```

```Java
/**
 * 短信中心
 */
public class SmsCenter {

  private static final Set<SmsService> serviceSet = new HashSet<>();

  /**
   * 初始化自动添加阿里短信服务
   */
  static {
    serviceSet.add(new AliSmsService());
  }

  /**
   * 外部调用，可注册其他短信服务
   *
   * @param smsService SmsService的实现
   */
  public static void addService(SmsService smsService) {
    serviceSet.add(smsService);
  }


  /**
   * 发送短信，调用已注册的短信服务，直到发送成功或者全部尝试过为止
   * @param phone 手机号
   * @param msg 信息内容
   */
  public static void sendSms(String phone, String msg) {
    for (SmsService smsService : serviceSet) {
      boolean result = smsService.sendSms(phone, msg);
      if (result) {
        System.out.println(smsService.getClass() + "发送成功");
        return;
      } else {
        System.out.println(smsService.getClass() + "发送失败");
      }
    }
    System.out.println("所有的短信提供商均发送失败！");

  }

}


//测试用的主函数
public class SmsDemo {


  public static void main(String[] args) {
    SmsCenter.addService(new TencentSmsService());
    SmsCenter.sendSms("13333333333", "开门！查水表！");
  }

}
```





##### 说明：

1. 两个实现均取时间戳，时间戳是奇数时失败，偶数时成功。用来模拟短信服务商服务不可用的情况。
2. SmsCenter做了几件事：
   1. serviceSet集合，模拟了已经注册了的短信提供商；
   2. serviceSet.add(new AliSmsService()); 静态函数默认注册 阿里短信服务；
   3. addService，提供对外注册短信服务的接口；
   4. sendSms，发送短信，具体实现是尝试所有已注册的短信服务发送短信；如果失败，则打印“所有的短信提供商均发送失败”
3. 测试主函数：因为默认只注册了阿里短信，测试这边模拟调用方，多注册一个腾讯的短信服务。然后就是执行发送指定内容了。

返回结果：

```
用户的手机号：13333333333
发送的信息是：开门！查水表！
AliService 发送短信!
class vin.cocoon.sms.impl.AliSmsService发送失败
用户的手机号：13333333333
发送的信息是：开门！查水表！
TencentSmsService 发送短信!
class vin.cocoon.sms.impl.TencentSmsService发送成功

Process finished with exit code 0
```

可以看到，SmsCenter在AliSmsService发送失败后，自动使用TencentSmsService尝试，并且发送成功！

## 小结一下

上面的例子，其实已经足矣完整展示接口的使用以及作用了。防止有同学没看懂，这里，再用文字给大家翻译一下。

没错，接口最主要的目的其实就是规定一个规范，以供调用方统一调配。这里的这个调用方，其实说的就是SmsCenter。作为一个多个服务的中心，自然不应该去关注其服务的具体实现。只要实现了SmsService接口，我就认为你有该功能。黑猫白猫，抓着耗子就是好猫。在这里，甚至不仅仅是黑白猫，哪怕你是霸王龙！只要你具备了指定接口就足矣。

## 填坑

### default 实现

java8开始，有了default关键字，表示接口定义时，可以定义默认实现，这样就可以无需接口的实现强制实现全部接口。

对于default关键字，如果滥用，我认为一定程度上是破坏了接口规范性的。因为如果一个接口，它不是所有的函数都是必须的话，那我们就有必要去思考是不是应该将接口进行拆分。而不是使用default函数去屏蔽这样的现象。

比如：验证码接口，定义了发送短信验证码和邮件验证码两个抽象函数。但是其实现，有的需要实现两个函数，有的只需要实现其中一个。为了方便，你可以将两个都定义default实现，然后默认报个错，或者打印一个“暂不支持”之类的提示。但是！会给其他调用方产生误解。你实现了这个接口，我就默认你两个验证码都能发。等到调用的时候才告诉我你不支持，你只是个默认实现。不就相当于“xx都脱了，你就给我看xx？”

所以，针对上面的情况。我认为的较好的解决方案应该是：分别定义 短信验证码接口 和 邮件验证码接口。然后分别实现。如果有些类两者都支持，那就**同时实现两个接口**。以保证，实现了的接口，能确保提供该接口应有的特性！

### 针对接口的多实现

其实，通过上面说的，拆分接口，有必要时再同时多实现已经足矣体现出接口的多继承意义何在了：一个接口只规定其实现必须实现该接口的功能，但如果需要实现多个接口的功能呢？那就多实现呗。摊手。

很多时候，接口总是和抽象类挂钩；实现接口总是和继承父类挂钩。其实，他们虽然有类似的地方，但是我建议把他们完全独立来看。

* 接口的作用： 就是提供规范！
* 继承或者抽象类的作用： 抽取公共代码，或者扩展父类使用。虽然，抽象类因为其抽象方法等特性，可以说也可以实现接口的作用。但是其核心的目的，我认为跟接口是存在区别的。从不支持多继承可见一斑，抽象类并不希望你把他当接口来使用。

## 最后

写在最后，接口的理解，其实脱离了Java技术本身。接口的理解程度如何，都不影响你编码。但是，却会影响你服务设计的思想。尤其是Java技术栈下。特别说一下，Spring系列可以说就是对Java接口的最佳应用，没有之一！

* 通过早期的通过接口注入Bean；
* 通过接口实现注入集合;
* @ConditionalOnMissingBean 注解来设定默认注入；
* 再到一系列的注入方式，来决定和控制注入接口的某一个实现；

等等特性，无疑体现了Spring对于Java接口的完美运用！

### 个人其他对于接口运用的例子：

[cocoon-base-security](https://github.com/pth-cocoon/cocoon-base-security)： 对于SpringSecurity进行了一定程度的默认封装，使其更加方便，易用。其中Spring对接口应用的体现

1. @ConditionalOnMissingBean ： SecurityBeanConfig下，对多bean进行了默认注入。在没有其他的实现的时候，注入默认的Bean

2. ```Java
   @Autowired
   private List<AuthenticationProvider> authenticationProviderList;
   ```

   这里对于集合的注入。注入全部实现AuthenticationProvider对组件。

## 感谢

十分感谢能阅读到这里的你！

希望本文对你可以有帮助，存在的漏洞以及希望不完善的地方也欢迎指出！十分期待你的点赞，收藏，以及关注！你的支持，是我继续努力的最大动力！

最后，也欢迎你关注我的公众号【猿树洞】

![yuanshudong_qr_code](https://pth-1252643971.file.myqcloud.com/typora/yuanshudong_qr_code.jpg)

