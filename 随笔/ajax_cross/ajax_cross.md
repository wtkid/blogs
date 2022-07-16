> # ajax跨域请求的处理

## 1.代理：

可以通过代理的方式，借助代理去请求服务器,如nginx

## 2.XHR2

HTML5中提供的XMLHTTPREQUEST Level2（及XHR2）已经实现了跨域访问。但ie10以下不支持

只需要在服务端填上响应头：

```
header("Access-Control-Allow-Origin:*");
/*星号表示所有的域都可以接受，*/
header("Access-Control-Allow-Methods:GET,POST");
```

测试：

添加过滤器(也可以单独对某个接口直接设置，这里使用过滤器是因为做项目时所有接口都要开放跨域请求以满足app端的需要)

```java
public class CrossFilter implements Filter {

    @Override
    public void init(FilterConfig config) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) res;
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Headers", "x-requested-with");
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {
    }
}
```

注册该过滤器（也可以使用@Compnent的方式）

```java

@Configuration
public class Config {
    @Bean
    public FilterRegistrationBean xssFilterRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setDispatcherTypes(DispatcherType.REQUEST);
        registration.setFilter(new CrossFilter());
        registration.addUrlPatterns("/*");
        registration.setName("crossFilter");
        registration.setOrder(100);
        return registration;
    }
}
```

Controller:

```java

@RestController
public class TestController {
    @RequestMapping(value = "/anon/info/{id}", method = {RequestMethod.GET})
    public CustomResponseBody info(@PathVariable("id") String id) {
        ArticleEntity article = articleService.queryObjectWeixin(id);
        return CustomResponseBody.ok().put("article", article);
    }
}
```

Ajax:

```js
<script type="text/javascript">
    $(function () {
    $.ajax({
        url: "http://localhost:8080/CH_PC/article/anon/info/1",
        type: "GET",
        success: function (data) {
            alert(data);
            $(".aaa").html(data.article.content);
        }
    });
})
</script>
```

测试结果就不贴了，通过的。

## 3.Jsonp

服务器端用MappingJacksonValue包装返回结果，这种方式请求参数中要带参数callback。

如果不想用MappingJacksonValue这种方式的话也可以自己包装返回结果：像我注释掉的那样（最后好像要加一个分号）

Controller:

```java

@RestController
public class TestController {
    @RequestMapping(value = "/anon/info/{id}", method = {RequestMethod.GET})
    public Object info(@PathVariable("id") String id, @RequestParam("callback") String callback) {
        ArticleEntity article = articleService.queryObjectWeixin(id);
        MappingJacksonValue jsonp = new MappingJacksonValue(article);
        jsonp.setJsonpFunction(callback);
        //        return callback+"("+ JSON.toJSONString(article)+")";
        return jsonp;
    }
}
```

Ajax:(注意路径中callback的值是问号而不是其他的，其实请求的时候传动过去的是jsonpCallback，也就是脚本中jsonp得值)

```js
<script type="text/javascript">
    $(function () {
    $.ajax({
        url: "http://localhost:8080/CH_PC/article/anon/info/1?callback=?",
        type: "GET",
        dataType: "jsonp",
        jsonp: "jsonpCallback",
        success: function (data) {
            alert(data);
            $(".aaa").html(data.content);
        }
    })
})
</script>
```

> 博客参考
>
> 最简单易一种XHR2: http://blog.csdn.net/qq_32719003/article/details/78654061
>
> 跨域三种方式: http://www.jb51.net/article/77470.htm
