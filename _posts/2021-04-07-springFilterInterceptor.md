---
title: spring的拦截器和过滤器
---

## 概述
spring有拦截器有过滤器，对其过滤路径和场景做说明。

## 过滤器
`implements javax.servlet.Filter`，然后实现其方法就写好一个过滤器了。注入bean的方式，添加过滤器代码如下。过滤器属于servlet规范，基于函数回调实现，所有请求都可以拦截，比如静态资源图片、jsp、js等。其实现代码如下
```java
    @Bean
    public FilterRegistrationBean crossDomainFilter() {
        FilterRegistrationBean crossDomain = new FilterRegistrationBean();
        crossDomain.setFilter(new CrossDomainFilter());
        crossDomain.setName("cross-domain-filter");
        crossDomain.setOrder(0);
        crossDomain.setUrlPatterns(ImmutableList.of("/*"));
        return crossDomain;
    }
```
上述代码中，setUrlPatterns参数可以是个正则路径，这里使用`/*`拦截所有请求，使用`/api/*`拦截api开头的请求。

## 拦截器
`implements org.springframework.web.servlet.HandlerInterceptor`，然后重写各方法。拦截器属于spring规范，基于反射实现，只拦截请求，比如静态资源图片、js等不会拦截。拦截器定义的代码如下
```java
public class LogInterceptor implements HandlerInterceptor {

    private static final Logger LOGGER = LoggerFactory.getLogger(LogInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        LOGGER.info("LogInterceptor preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
        LOGGER.info("LogInterceptor postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
        LOGGER.info("LogInterceptor afterCompletion");
    }
}
```
拦截器使用的代码
```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class WebsiteApplication implements WebMvcConfigurer {

	public static void main(String[] args) {
		SpringApplication.run(WebsiteApplication.class, args);
	}

	@Override
	public void addInterceptors(InterceptorRegistry registry) {
    // 拦截器有序 ArrayList
		registry.addInterceptor(new LogInterceptor()).addPathPatterns("/**");
		registry.addInterceptor(new UploadSqlInterceptor()).addPathPatterns("/api/**");
	}
}
```
### 拦截路径
addPathPatterns和excludePathPatterns  
如果二者都存在，则优先判断exclude，再经过include。其拦截方式源码在拦截器的代码在`org.springframework.web.servlet.handler.MappedInterceptor`类的`matches`方法。

拦截器的拦截路径匹配关系与过滤器不同，
`/*`和`/abd/*`表示只拦截该匹配路径。比如`/ab`、`/abd/a`，不能拦截`/abd/a/b`
`/**`和`/abd/**`表示拦截匹配路径 及其子路径。比如`/abd/a/b`
### 拦截器使用方式
如果想让一个拦截器拦截两个路径，用过只调用一次addInterceptor，并一次性写完拦截路径。如果写两个addInterceptor语句，则容易达不到预期。例子如下。
```java
// 1. 意思是：不能拦截api/v3和api/v4开头的接口。
registry.addInterceptor(loggerInterceptor).excludePathPatterns("/api/v3/**", "/api/v4/**"); 

// 2. 前一句意思是不能拦截api/v3开头的接口，但是可以拦截api/v1、api/v2、api/v4等开头的接口。
//    后一句意思是不能拦截api/v4开头的接口，但是可以拦截api/v1、api/v2、api/v3开头的接口。
//    所以，以下两行代码加起来的意思是，所有接口都能被拦截，且/api/v2/开头的，要以语句定义顺序，被拦截两遍。
registry.addInterceptor(loggerInterceptor).excludePathPatterns("/api/v3/**");
registry.addInterceptor(loggerInterceptor).excludePathPatterns("/api/v4/**");
```
## 过滤器和拦截器调用顺序
虚拟机启动时，过滤器初始化方法`init`调用顺序未知；同理，虚拟机关闭是，过滤器销毁方法`destroy`调用顺序未知；

接受用户请求时，过滤器和拦截器的调用顺序
![sequence](/images/20210407sequence.png)
