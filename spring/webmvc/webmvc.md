# spring-webmvc启动流程

`web.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         id="WebApp_ID" version="2.5">
    <display-name>springtest</display-name>

    <context-param>
        <!-- 初始化器 -->
        <param-name>globalInitializerClasses</param-name>
        <param-value>com.wt.test.webmvc.config.MyContextInitializer</param-value>
    </context-param>
    <!-- 配置spring容器监听器 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring/applicationContext-*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <listener>
        <listener-class>com.wt.test.webmvc.config.MyDataServletContextListener</listener-class>
    </listener>

    <!-- 前端控制器 -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>com.wt.test.webmvc.config.MyDispatcherServlet</servlet-class>
        <!-- 加载springmvc配置 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- 配置文件的地址 如果不配置contextConfigLocation， 默认查找的配置文件名称classpath下的：servlet名称+"-serlvet.xml"即：springmvc-serlvet.xml -->
            <param-value>classpath:spring/springmvc.xml</param-value>
        </init-param>
        <init-param>
            <!-- 初始化器 -->
            <param-name>contextInitializerClasses</param-name>
            <param-value>com.wt.test.webmvc.config.MyContextInitializer</param-value>
        </init-param>
        <init-param>
            <param-name>contextId</param-name>
            <param-value>abcdefg</param-value>
        </init-param>
        <!-- 配置一个已经存在的容器，可以是上面的父容器，value值是容器在servletContext的Attribute中的key，
我这里将contextLoaderListener初始化的父容器加到属性中，mvc初始化的时候会查找该值，存在容器就不会重新初始化mvc子容器，
        默认会存在父子容器，当然也可以不初始化父容器，只用servlet一个mvc子容器,
测试的时候请注释掉，不然视图解析路径会有问题，因为配置的解析器是在springmvc.xml中，子容器会读取这个文件 -->
        <init-param>
            <param-name>contextAttribute</param-name>
            <param-value>org.springframework.web.context.WebApplicationContext.ROOT</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!-- 可以配置/ ，此工程 所有请求全部由springmvc解析，此种方式可以实现 RESTful方式，需要特殊处理对静态文件的解析不能由springmvc解析
            可以配置*.do或*.action，所有请求的url扩展名为.do或.action由springmvc解析，此种方法常用 不可以/*，如果配置/*，返回jsp也由springmvc解析，这是不对的。 -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- post乱码处理 -->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.css</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        ...
        静态文件的解析用默认的default，都跟上面一样配置，为了简短这个web，下边就不ctrl+v了
    </servlet-mapping>
    
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

```
springmvc启动过程中，我们需要配置的web如上图所示，这个基本上算是我们平时常用的比较完善的配置了。当然我们在做项目的时候完全可以省略掉ContextLoaderListener这个父容器的配置。因为我们的dispatcherServlet的init方法中是会初始化mvc容器的，也就是我们常说的子容器，contextLoaderListener加载的是父容器。
```

## 父容器的初始化

contextLoaderListener是实现了ServletContextListener的。我们直接来看contextInitialized(ServletContextEvent)方法。

```java
//ContextLoaderListener.java,这货继承了ContextLoader
public void contextInitialized(ServletContextEvent event) {
		this.contextLoader = createContextLoader();
		if (this.contextLoader == null) {
			this.contextLoader = this;
		}
		this.contextLoader.initWebApplicationContext(event.getServletContext());
	}
```

```java
//ContextLoader.java
private static final Properties defaultStrategies;

	static {
		...
            //这个文件与当前class同目录
            //它的值是org.springframework.web.context.WebApplicationContext=org.springframework.web.context.support.XmlWebApplicationContext
			ClassPathResource resource = new ClassPathResource("ContextLoader.properties", ContextLoader.class);
			defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
		}
		...
	}
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
    .....
        //创建父容器context，默认是XmlWebApplicationContext
        if (this.context == null) {
				this.context = createWebApplicationContext(servletContext);
			}
    .....
       	//将初始化完的context放到当前servletContxt当中，
        //key是WebApplicationContext.class.getName() + ".ROOT"
      servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
    .....
}

protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
		Class<?> contextClass = determineContextClass(sc);
		.....
		return (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
}

protected Class<?> determineContextClass(ServletContext servletContext) {
    	// 初始化的父容器的class名字，通过web.xml的servlet的<init-param>属性来配置contextClass的的值
		String contextClassName = servletContext.getInitParameter(CONTEXT_CLASS_PARAM);
    if (contextClassName != null) {
        ......
    } else {
        //默认值，在这个类的静态初始化块中
			contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());
}
```

```
从上面可以看出我们其实默认初始化了一个XmlWebApplicationContext类型的父容器，并且将其放到了servletContext的attribute当中。
```

## 子容器的初始化

```
子容器的初始化是在DispatcherServlet中完成的，springmvc对所有请求的拦截也是基于servlet的，这个类就是spring队请求处理的核心，所有的请求都会到这里，然后由它去分发。既然这货是个servlet，那我们来看一看它的init方法,这个方法在他的父类HttpServletBean中。
```

```java
// HttpServletBean.java
public final void init() throws ServletException {
	...
	//子类实现FrameworkServlet.java
	initServletBean();
	...
}
```

```java
// FrameworkServlet.java
protected final void initServletBean() throws ServletException {
	...
    try {
			this.webApplicationContext = initWebApplicationContext();
			initFrameworkServlet();
		}
	...
}

protected WebApplicationContext initWebApplicationContext() {
    //这个地方获取的就是我们之前初始化后放到servletContext的Attribute中的父容器XmlWebApplicationContext，有兴趣的可以点进去看看
	WebApplicationContext rootContext =
				WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    ...
    if (wac == null) {
		// 这里就是去找一个已经存在的容器，servlet的<init-param>属性中配置contextAttribute值，
         // 因为我们之前启动了一个父容器，所以如果我们把父容器的key[WebApplicationContext.class.getName() + ".ROOT"]配在这个地方，我们就可以拿到父容器了。
		wac = findWebApplicationContext();
	}
	if (wac == null) {
		// 正常情况下初始化子容器，传进去的rootContext就是之前初始化的父容器。
		wac = createWebApplicationContext(rootContext);
	} 
    if (!this.refreshEventReceived) {
		// 初始化requestMapping等，方法在DispacherServlet.java中
		onRefresh(wac);
	}
    if (this.publishContext) {
			// 生成子容器的名称，
			String attrName = getServletContextAttributeName();
        	// 将子容器也赋值给servletContext的Attribute
			getServletContext().setAttribute(attrName, wac);
			if (this.logger.isDebugEnabled()) {
				this.logger.debug("Published WebApplicationContext of servlet '" + getServletName() +
						"' as ServletContext attribute with name [" + attrName + "]");
			}
		}
    return wac;
}

protected WebApplicationContext findWebApplicationContext() {
		String attrName = getContextAttribute();
		...
		WebApplicationContext wac =
				WebApplicationContextUtils.getWebApplicationContext(getServletContext(), attrName);
		...
		return wac;
	}

protected WebApplicationContext createWebApplicationContext(ApplicationContext parent) {
		// 子容器的类型，可以通过当前servlet的getContextClass方法获得，我的xml文件就是配置过的	
    	Class<?> contextClass = getContextClass();
		...
		wac.setEnvironment(getEnvironment());
    	// 可以看到这里我们设置了子容器的父容器为之前初始化的XmlWebApplicationContext
		wac.setParent(parent);
		wac.setConfigLocation(getContextConfigLocation());
		...
         // 配置和刷新容获取并调用调用web.xml配置的ContextInitializers
		configureAndRefreshWebApplicationContext(wac);
		return wac;
}

protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
    ...
    //由子类实现，默认为空实现
    postProcessWebApplicationContext(wac);
    //获取并调用web.xml配置的ContextInitializers
	applyInitializers(wac);
    ...
}

protected void applyInitializers(ConfigurableApplicationContext wac) {
    //获取由web.xml <context-param>配置的initializer，key为globalInitializerClasses
		String globalClassNames = getServletContext().getInitParameter("globalInitializerClasses");
		if (globalClassNames != null) {
			for (String className : StringUtils.tokenizeToStringArray(globalClassNames, INIT_PARAM_DELIMITERS)) {
				this.contextInitializers.add(loadInitializer(className, wac));
			}
		}
//获取由web.xml 的servlet配置的<init-param>配置的initializer，key为contextInitializerClasses
		if (this.contextInitializerClasses != null) {
			for (String className : StringUtils.tokenizeToStringArray(this.contextInitializerClasses, INIT_PARAM_DELIMITERS)) {
				this.contextInitializers.add(loadInitializer(className, wac));
			}
		}
		//排序
		AnnotationAwareOrderComparator.sort(this.contextInitializers);
		for (ApplicationContextInitializer<ConfigurableApplicationContext> initializer : this.contextInitializers) {
			initializer.initialize(wac);
		}
}

public String getServletContextAttributeName() {
    // 生成子容器名称
	return SERVLET_CONTEXT_PREFIX + getServletName();
}
```

### onRefresh(wac)初始化web处理请求的组件

```java
//DispathcerServlet.java
protected void onRefresh(ApplicationContext context) {
	initStrategies(context);
}

protected void initStrategies(ApplicationContext context) {
    //初始化文件上传使用的MultiResolver
	initMultipartResolver(context);
    //初始化LocaleResolver,用于国际化
	initLocaleResolver(context);
    //初始化主题解析器ThemeResolver
	initThemeResolver(context);
    //初始化请求映射的HandlerMaping
	initHandlerMappings(context);
    //初始化请求处理的适配器HandlerAdapter
	initHandlerAdapters(context);
    //初始化异常处理解析器HandlerExceptionResolver
	initHandlerExceptionResolvers(context);
    //初始化用于从请求中获取viewname的RequtstToviewNameResolver
	initRequestToViewNameTranslator(context);
    //初始化视图解析器ViewResolver
	initViewResolvers(context);
    //初始化FlashMapperManager
	initFlashMapManager(context);
}
```

```
从上面可以看出我们其实创建了的子容器也同样放到了servletContext的attribute当中。
总的初始化流程可以总结如下：
1. 通过ContextLoaderListener创建父容器；
2. 将父容器放到servletContext的Attribute中；
3. 通过servlet的init()方法开始创建子容器；
3.1. 获取servletContext中的父容器；
3.2. 通过配置的contextClass的值查找是否已经存在一个容器；
3.3. 如果上一步没有找到，则开始重新创建一个子容器；
3.4. 将父容器设置为子容器的parent；
3.5. 刷新子容器
3.5.1 调用子类扩展方法postProcessWebApplicationContext(wac);
3.5.2 查找并调用web.xml配置的contextInitializers；
4. 通过onRefresh(wac)方法初始化requestMapping等；
5. 将初始化的子容器也丢给servletContext的Attribute;
6. 执行子类扩展的initFrameworkServlet()方法，如果没有实现就是空的；
7. springmvc初始化结束。
```



over ...