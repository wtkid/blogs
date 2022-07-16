> # spring扩展启动流程-refresh

# XmlWebApplicationContext

在[spring-webmvc启动流程](https://my.oschina.net/wtkid/blog/3056265)中，我们在对wac的刷新过程并没有详细的解释，只是一笔带过。

不管是从ContextLoaderListener进入的，还是Servlet的init方法进入的，都离不开spring整体的初始化。仅仅有bean的读取，加载的spring是不完整的，spring还需要这些扩展的部分，让spring更完善，先来看看入口吧。

```java
// ContextLoader.java
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
	//...
	wac.refresh();
}
```

```java
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
		//...
		wac.refresh();
}
```

# refresh

refresh函数中几乎包含了ApplicationContext提供的所有功能，而且此函数中的逻辑很清楚，瞅一下吧。

调试可以直接从ClassPathXmlApplicationContext这个类驶入，构造方法直接就能驶入这个方法，并初始化完成整个环境。

```java
// AbstractApplicationContext.java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      // 准备刷新的上下文环境
      prepareRefresh();
       
      // Tell the subclass to refresh the internal bean factory.
      // 初始化BeanFactory，并进行xml文件读取
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
       
      // Prepare the bean factory for use in this context.
      // 对BeanFactory进行各种功能填充
      prepareBeanFactory(beanFactory);
      try {
         // Allows post-processing of the bean factory in context subclasses.
         // 子类覆盖做额外的处理
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         // 激活各种BeanFactory处理器
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         // 注册拦截Bean创建的Bean处理器，这里只是注册，调用是在getBean的时候
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         // 为上下文初始化Message源，即不同语言的消息体，国际化处理
         initMessageSource();

         // Initialize event multicaster for this context.
         // 初始化应用消息广播器，并放入"ApplicationEventMulticaster" Bean中 
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         // 留给子类来初始化其他的Bean
         onRefresh();

         // Check for listener beans and register them.
         // 在所有注册的Bean中查找Listenter bean，注册到消息广播器中
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         //初始化剩下的单例（非惰性）
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         // 完成刷新过程，通知生命周期处理器lifecycleProcessor完成刷新过程，同时发出ContextRefreshEvent通知别人
         finishRefresh();
      }
      catch (BeansException ex) {
         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }
   }
}
```
## prepareRefresh

```java
//AbstractApplicationContext.java
protected void prepareRefresh() {
    //...
    //留给子类覆盖
    initPropertySources();
    //属性文件验证，确保需要的文件都已经放入环境中
    getEnvironment().validateRequiredProperties();
    //...
}
```

## obtainFreshBeanFactory

初始化BeanFactory，对xml文件的读取，就是从这个地方开始的，使用的是默认的DefaultListableBeanFactory。

```java
//AbstractApplicationContext.java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    //初始化BeanFactory
	refreshBeanFactory();
    //返回初始化之后的BeanFactory
	return getBeanFactory();
}
```

```java
//AbstractRefreshableApplicationContext.java
protected final void refreshBeanFactory() throws BeansException {
    //检查是否已经有了初始化的BeanFactory
    if (hasBeanFactory()) {
        //销毁所有的bean
        destroyBeans();
        //关闭并销毁BeanFactory
        closeBeanFactory();
    }
    try {
        //创建默认的DefaultListableBeanFactory，子类可以覆盖该方法
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        beanFactory.setSerializationId(getId());
        //定制BeanFactory，包括是否允许覆盖同名称的不同定义的对象以及循环依赖
        customizeBeanFactory(beanFactory);
        //初始化XmlBeanDefinitionReader，并进行xml文件的解析。
        loadBeanDefinitions(beanFactory);
        synchronized (this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
    }
    catch (IOException ex) {...}
}

protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
    //设置是否允许同名不同定义的bean覆盖
    if (this.allowBeanDefinitionOverriding != null) {
        beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
    }
    //设置是否允许循环依赖
    if (this.allowCircularReferences != null) {
        beanFactory.setAllowCircularReferences(this.allowCircularReferences);
    }
}
```

## prepareBeanFactory

至此，spring已经完成了对配置的解析，ApplicationContext在功能上的扩展也由此开始。

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    beanFactory.setBeanClassLoader(getClassLoader());
    //设置表达式语言处理器
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    //设置一个默认的属性解析器PropertyEditor
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    //添加BeanPostProcessor
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));

    //设置几个忽略自动装配的接口
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    //注册依赖解析，也是spring的bean，但是会特殊一些，下边会有解释
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    //添加BeanPostProcessor
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    //增加对AspectJ的支持
    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    //添加默认的系统环境bean
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```

```
主要扩展：
1. 增加对SPEL语言的支持;
2. 增加对属性编辑器的支持，这些PropertyEditors在这里只是注册，使用的时候是将bean包装成BeanWrapper，包装后的BeanWrapper的就包含了所有的这些PropertyEditors，以便后期给bean设置属性的时候使用;
3. 增加对一些内置类（实际上就是前置处理器），比如Aware接口的信息注入;
4. 设置依赖功能可忽略的接口;
5. 注册一些固定的bean，这些都是特殊的依赖解析，比如当注册了BeanFactory.class的依赖解析之后，当bean的属性注入的时候，一旦检测到属性为BeanFactory类型便会将beanFactory的实例注入进去;
6. 增加对AspectJ的支持;
7. 将相关环境变量及属性以单例模式注册。
```

## invokeBeanFactoryPostProcessors

激活注册的BeanFactoryPostProcessor。这个接口跟BeanPostProcessor类似，可以对bean的定义（配置元数据）进行处理，作用范围是容器级的，只对自己的容器的bean进行处理。典型应用PropertyPlaceholderConfigurer。

这部分代码有点长，不贴了，自己去看吧。

## registerBeanPostProcessors

注册所有的bean前后置处理器BeanPostProcessor，这里只是注册，调用是在bean创建的时候。

代码也有点长，不贴了，自己去看吧~~

## initMessageSource

国际化处理，如果想使用自定义的，bean name是指定了的MESSAGE_SOURCE_BEAN_NAME = "messageSource"。

```java
protected void initMessageSource() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    //根据指定的名称查找messageSource，如果容器中注册了指定名称的messageSource，就使用注册的
    if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
        this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);
        if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
            HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
            if (hms.getParentMessageSource() == null) {
                hms.setParentMessageSource(getInternalParentMessageSource());
            }
        }
        //...日志
    }
    else {
        //容器中没有注册指定名称的messageSource，使用默认的DelegatingMessageSource
        DelegatingMessageSource dms = new DelegatingMessageSource();
        dms.setParentMessageSource(getInternalParentMessageSource());
        this.messageSource = dms;
        beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
        //...日志
    }
}
```

## initApplicationEventMulticaster

初始化应用的事件广播器，用于事件监听，逻辑跟初始化国际化MessageSource是一样的，也是有一个特定的名称。如果beanFactory有就是用注册的，如果没有就是用默认的SimpleApplicationEventMulticaster。

应用中的listener都是注册到这个广播器，由广播器统一广播的。

## registerListeners

在所有注册的Bean中查找Listenter bean，注册到消息广播器中。

```java
protected void registerListeners() {
    // 首先注册静态注册的监听器（代码注册的）
    for (ApplicationListener<?> listener : getApplicationListeners()) {
        getApplicationEventMulticaster().addApplicationListener(listener);
    }

    // 获取容器中所有配置的listener并注册
    String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    for (String listenerBeanName : listenerBeanNames) {
        getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
    }

    // 对earlyApplicationEvents这些事件进行广播，实际上就是遍历所有的listener，找到可以处理event的listener处理
    Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
    this.earlyApplicationEvents = null;
    if (earlyEventsToProcess != null) {
        for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
            getApplicationEventMulticaster().multicastEvent(earlyEvent);
        }
    }
}
```

## finishBeanFactoryInitialization

初始化剩下的非惰性单例，如果某个单例依赖了惰性的单例，那么这个惰性的单例也会被初始化，这个很好理解吧。

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 初始化ConversionService，跟PropertyEditor类似
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
            beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        beanFactory.setConversionService(
                beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    //则注册默认的嵌入值解析器
    //例如PropertyPlaceholderConfigurer bean）之前注册过：
    //主要用于注解属性值的解析。
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    //单例bean初始化之前首先初始化LoadTimeWeaverAware，以支持aop，AspectJ
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }
    beanFactory.setTempClassLoader(null);

    //冻结bean定义(BeanDefinition)，表示所有的bean定义进不被修改或进行进一步处理
    beanFactory.freezeConfiguration();

    //初始化非惰性单例，实际上就是遍历所有的beanName，然后一一调用getBean()
    beanFactory.preInstantiateSingletons();
}
```

## finishRefresh

```java
protected void finishRefresh() {
    //清除上下文级别(context-level)的资源缓存，比如由scanning产生的ASM元数据
    clearResourceCaches();

    //初始化LifecycleProcessor
    initLifecycleProcessor();

    //使用LifecycleProcessor来启动Lifecycle子类
    getLifecycleProcessor().onRefresh();

    //上下文刷新完成，发布事件
    publishEvent(new ContextRefreshedEvent(this));

    // Participate in LiveBeansView MBean, if active.
    LiveBeansView.registerApplicationContext(this);
}
```

初始化LifecycleProcessor的时候，跟初始化MessageResource一样，没有自定义的就是用默认的DefaultLifecycleProcessor。

getLifecycleProcessor().onRefresh()会使用我们注册的LyfecycleProcessor来启动我们注册的SmartLifeCycle的子类。看一下代码吧。

```java
//默认的DefaultLifecycleProcessor.java
public void onRefresh() {
    startBeans(true);
    this.running = true;
}

private void startBeans(boolean autoStartupOnly) {
    //获取所有的LifeCycle类型的bean
    Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans();
    Map<Integer, LifecycleGroup> phases = new HashMap<>();
    lifecycleBeans.forEach((beanName, bean) -> {
        //默认的DefaultLifecycleProcessor只会启动SmartLifecycle的，或者非自动启动的类型
        //SmartLifecycle继承了Phases接口
        if (!autoStartupOnly || (bean instanceof SmartLifecycle && ((SmartLifecycle) bean).isAutoStartup())) {
            int phase = getPhase(bean);
            LifecycleGroup group = phases.get(phase);
            if (group == null) {
                group = new LifecycleGroup(phase, this.timeoutPerShutdownPhase, lifecycleBeans, autoStartupOnly);
                //添加到需要启动的集合中
                phases.put(phase, group);
            }
            group.add(beanName, bean);
        }
    });
    //启动需要启动的这些LifeCycle
    if (!phases.isEmpty()) {
        List<Integer> keys = new ArrayList<>(phases.keySet());
        Collections.sort(keys);
        for (Integer key : keys) {
            phases.get(key).start();
        }
    }
}
```



over ...