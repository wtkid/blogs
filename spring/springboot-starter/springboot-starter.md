> # 自定义springboot starter

上一篇[springboot-自动配置原理](https://my.oschina.net/wtkid/blog/3079375 "springboot-autoConfiguration")在最后提了一下，了解了autoConfiguration，离springboot-stater就只差一个demo，好了，最近比较闲，来搞一个demo看看，demo实现了两个功能：

1.  使用Aspectj方式的aop，来实现对某类函数参数监控;
2.  定义一个自己的配置类，读取我们在yml或properties里面配置的参数，同时提供一个service，用来获取我们读取的结果。

 [这里是示例工程starter](https://gitee.com/wt123/learn/tree/master/spring-boot-myaop-starter "示例工程starter"). 

# starter

看了自动配置原理之后，应该知道META-INF/spring.facotires这个家伙是配置的重点了吧，我们很多个的东西都是在这个里头。因此，我们自己的stater当然也少不了这个配置文件，先贴个工程结构。

```
spring-boot-myaop-starter
├─ pom.xml
├─ spring-boot-myaop-starter.iml
└─ src/main
        ├─ java
        └─ resources
            └─ META-INF
                └─ spring.factories
```

## AOP部分

先按照正常工程把一些配置都配好，先来写一个AspectJ的。

```java
package com.wt.myaop.aspect;
@Aspect
@Component
public class MyAspect {

    @Pointcut("@annotation(myAnno)")
    public void myAspectPointCut(MyAopAnnotation myAnno) {
    }

    @Before("myAspectPointCut(myAnno)")
    public void performanceTrance2(JoinPoint joinPoint, MyAopAnnotation myAnno) throws Throwable {
        System.out.println("--------------myaop-starter args monitor start-----------------");
        Object[] args = joinPoint.getArgs();
        Class<?>[] types = myAnno.argTypes();
        for (int i = 0; i < Math.min(args.length, types.length); i++) {
            System.out.println("type:" + types[i] + "<--->arg:" + args[i]);
        }
        System.out.println("--------------myaop-starter args monitor end-----------------");
    }
}
```

我们这里使用的是@Before，在这个方法里面获取到所有参数，然后将所有参数的类型和值都打印出来，方法前后输出日志，比较简单。

然后来看pointCut，**这是一个基于注解的，这样其他项目引入这个starter的时候就可以直接以注解来使用，对其他项目不具有侵入性**。所以，我们还需要定义一个注解。

```java
package com.wt.myaop.anno;

@Target(ElementType.METHOD)
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAopAnnotation {
    Class<?>[] argTypes() default {};
}
```

注解里面有属性，用来定义方法参数的类型顺序，当然也可以不需要，放这里只是为了方便。

## 属性读取部分

属性的读取我们需要用到一个springboot的注解@ConfigurationProperties.来看看这个配置类，为了缩短代码长度，都没有贴getter/setter方法哦。

```java
package com.wt.myaop.config;

@ConfigurationProperties(prefix = MyConfig.MY_PREFIX)
public class MyConfig {
    public static final String MY_PREFIX = "myconfig";
    /**
     * 不加该注解也能正常得到值
     */
    @NestedConfigurationProperty
    private UserConfig user;

    private String job;
}
```

**代码中我们定义了一个常量MY_PREFIX，这个是配置文件中的前缀，springboot会读取这个前缀的配置来注入到这个实体当中，当然，配置肯定不会是在我们这个starter项目中来配置，是我们实际使用的project中配置的哦！要不然我还不如直接写死，对吧，嘻嘻**。

除了基本属性之外，还提供了一个实体属性，UserConfig。来看看这个东西吧。

```java
package com.wt.myaop.config;

public class UserConfig {
    private String username;
    private Integer age;
    private String sex;
}
```

配置信息的接收实体有了，我们现在来定义一个service，这个service的作用就是注入这个实体，然后输出配置信息，以检查我们的配置生效了。service的接口就不贴了哈，只贴实现类了。

```java
package com.wt.myaop.service;

public class MyConfigServiceImpl implements MyConfigService {
    private final MyConfig myconfig;
    public MyConfigServiceImpl(MyConfig myConfig) {
        this.myconfig = myConfig;
    }
    @Override
    public void printMyConfig() {
        System.out.println(myConfig);
    }
}
```

看构造方法和属性，构造方法注入了我们的配置，当然，我们如果需要使用这个service，我们就需要手动将这个配置注入进来，以确保service实例化时，myconfig不会为null，里面的方法很简单，仅仅打印了我们的配置类MyConfig。

starter像这样就算完了吗？当然没有，可以感受到以上这些其实都跟普通的project的配置没有太多差别，很简单。

回忆一下我们自动配置的核心是不是叫xxxAutoConfiguration的东西呢？spring.vactories中的哦！

好了，来吧，开始定义我们自己的AutoConfiguration。

```java
package com.wt.myaop.autoconfig;

@Configuration
@ComponentScan({"com.wt.myaop.aspect"})
@AutoConfigureAfter(AopAutoConfiguration.class)
@EnableConfigurationProperties(MyConfig.class)
public class MyArgsMonitorAopAutoConfig {

    private MyConfig myConfig;

    public MyArgsMonitorAopAutoConfig(MyConfig myConfig) {
        this.myConfig = myConfig;
    }

    @Bean
    public MyConfigService getMyConfigService(){
        return new MyConfigServiceImpl(myConfig);
    }

}
```

我们现在来研究一下这个类都有什么。

```
@Configuration: 表明这个类是一个配置类;
@ComponentScan: 指出需要扫描的包，可以看到上面扫描的包中包含了Aspect注解的配置，当然使用aop这部分是必不可少的;
@AutoConfigureAfter: 指出当前类需要再某个自动配置完成之后才开始配置;
@EnableConfigurationProperties: 为@ConfigurationProperties注解提供支持，什么意思呢；
	解释一下这个，把@ConfigurationProperties标注的类(MyConfig)注册成bean，以支持依赖注入，本身这些类
	是不会被注册成bean的，当然我们可以在配置类上加@Component注解。使用这个注解可以在我们需要某个配置类
	注册成bean的时候，就使用，侵入性小，避免了配置类上加@Component注解。
```

从上面代码中可以看到我们的配置类MyConfig通过构造方法被注入到了MyArgsMonitorAopAutoConfig当中，而MyArgsMonitorAopAutoConfig是一个配置类，里面通过@Bean的方式配置了MyConfigService。

**看起来很简答吧，实际上也很简单，autoConfiguration的配置类就好了，引入这个类，就相当于引入了一个Myconfig的配置类，和一个我们能够使用的MyConfigService**。

自动配置的类有了，需要放到spring.factories中才能生效。

```properties
# META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.wt.myaop.autoconfig.MyArgsMonitorAopAutoConfig
```

这样starter部分就算是完成了。

> Tips: 自动配置其实很简单，把需要的配置都放到一个叫做starter的工程里面，然后创建一个XxxAutoConfiguration的自动配置类，把需要的配置都放到这个类里面配置好。最后把这个自动配置类加到META-INF/spring.factories中就行了，是不是很容易，嘻嘻。

# demo

demo其实也是使用的之前分析源码那个工程，下面有地址。重点来配置和启动一下我们自己的starter。

 [这里是示例工程demo](https://gitee.com/wt123/learn/tree/master/springboot_source_learn "示例工程demo"). 

有了starter之后，使用mvn clean install命令将其jar安装到本地。

在demo工程中引入这个依赖。

```xml
<dependency>
    <groupId>com.wt.starter</groupId>
    <artifactId>spring-boot-myaop-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

## 使用MyConfig配置

我们starter里面有个MyConfig的配置对吧，好了，现在我们可以在yml或者properties文件中来定义了。我这里用的是yml哈。

```yaml
myconfig:
  job: programer
  user:
    username: wt
    age: 25
    sex: real_man
```

**注意我们的前缀是myconfig，下面的job和user都是实体类MyConfig的属性，因为user是个实体了类，因此下面也列出了它的属性username，age，sex.(可以去前面的MyConfig和UserConfig两个实体的属性进行对比)。**

## 使用参数监控AOP

starter中除了上面的MyConfig之外，还有个aop监控参数的功能，我们马上把它用起来。

先定义一个service，接口就不贴了

```java
@Service
public class MyStarterServiceImpl implements MyStarterService {

    @Override
    @MyAopAnnotation(argTypes = {String.class})
    public void helloStarter(String msg, Long currentTime) {
        System.out.println("----hello starter,msg = " + msg + ",currentTime = " + currentTime);
    }
}
```

方法加上了注解MyAopAnnotation，为了被starter当中的aop拦截到。

再来定义一个controller用来测试。

```java
@RestController
@RequestMapping("/starter")
public class MyStarterController {

    @Autowired
    private MyConfigService myConfigService;

    @Autowired
    private MyStarterService myStarterService;

    @RequestMapping("/test")
    public Object starter() {
        myConfigService.printMyConfig();
        myStarterService.helloStarter("wt", System.currentTimeMillis());
        return "success";
    }
}
```

注意区分一下，MyConfigService是我们starter里面的，MyStarterService是我们demo工程里面的。

现在启动工程，访问http://127.0.0.1:8080/starter/test试试！

来看看输出(复制了输出，没有使用截图): 

```
MyConfig{user=UserConfig{username='wt', age=25, sex='real_man'}, job='programer'}
------------------myaop-starter args monitor start--------------------
type:class java.lang.String<--->arg:wt
------------------myaop-starter args monitor end--------------------
----hello starter,msg = wt,currentTime = 1564472340205
```

第一行是 myConfigService.printMyConfig()的输出，他输出了我们的在yml中的配置，这个输出说明了我们的配置被成功赋值给MyConfig了。说明我们的config生效了。

第二行和第四行是starter当中aop部分@Before方法的前后输出，中间第三行输出了参数类型String和参数的值wt，因为我们注解上只指定了一个String.class，因此在这里没有把第二个参数也输出来，具体逻辑就去看starter当中aop的那个@Before的方法了。

最后一行是被加强的方法的输出，打印了msg和currentTime。

现在可以来想象一下springboot在yml或properties文件中的Datasource等配置咯！

看到这里starter部分就算结束了，总的来讲这个家伙也不难，对吧，嘻嘻。



胜利的旗帜在向各位招手，Good Luck！



over~~~