```java
// AbstractApplicationContext.java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      // 准备刷新的上下文环境
      prepareRefresh();
       
      // Tell the subclass to refresh the internal bean factory.
      // 初始化Beanfactory，并进行xml文件读取
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
         //初始话剩下的单实例（非惰性）
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         // 完成刷新过程，通知生命周期处理器lifecycleProcessor刷新过程，同时发出ContextRefreshEvent通知别人
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