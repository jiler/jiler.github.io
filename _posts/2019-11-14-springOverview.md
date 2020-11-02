---
title: Spring简介
---
# Spring简介

## Spring IOC
IOC全称Inversion of Controll，控制反转。容器负责创建Bean，将功能类的bean注入到需要的Bean中。ApplicationContext就是一个容器接口，AnnotationConfigApplicationContext就是一个容器类，它接受一个配置类（被`@Configuration`注解的类）来实例化。
``` java
public class ContextExample {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        UserService UserService = context.getBean(UserService.class);

        System.out.println("Hello, world!");
        context.close();
    }
}
```
Spring提供xml、注解、java配置、groovy配置实现Bean的创建和注入。
注解配置指的是业务Bean配置时使用的注解（`@Service`、`@Controller`、`@Component`），java配置适用于全局配置（数据库相关，MVC相关配置，主要通过`@Configuration`实现）。

## Spring AOP
AOP全称Aspect Oriented Programming，面向切面编程。Spring支持AspectJ注解式切面编程，todo wujunpeng 使用书中1.3.3的切面实现一下需求。

## Spring Bean的Scope

1. Singleton：一个Spring容器只有一个Bean实例，默认配置，全容器共享一个实例。
2. Prototype：**每次调用** 新建一个Bean实例。每次调用的意思是：一个被`@Scope("prototype")`注解修饰的类，每次被用`@Autowired`注入到不同类，都是一个新的Bean。下有例子
3. Request：web服务中，每个http request新建一个Bean。
3. Session：web服务中，每个http session新建一个Bean。
5. GlobalSession：只在 **portal应用** 中有用，给每个global http session新建一个Bean。

Scope的使用方式举例
``` java
@Service
@Scope("prototype")
public class UserService {

}
```
对于Prototyp的解释：下面两个类中的两个userService属性，他们的hashCode值不同，是两个Bean，因为限定了prototype。而没有`@Scope("prototype")`注解修饰的类，则默认是singleton，用`@Autowired`注入到不同的类中，hashCode值相同，因为是一个Bean。
``` java
@Service
public class QueryService {
    @Autowired
    private UserService userService;
}

@Service
public class InsertService {
    @Autowired
    private UserService userService;
}
```
> `@Autowired`是Spring提供的注解，`@Resource`是JSR-250提供的注解。JSR是Java Specification Requests的缩写，意思是Java 规范提案
`
## 其他注解
`@PostConstruct`， 在构造函数执行完后执行。
`@PreDestroy`，在Bean销毁之前执行。

## Application Event
Bean之间传递消息可以用事件，通过实现ApplicationListener接口的方式，实现监听事件。通过继承ApplicationEvent来定义事件，通过ApplicationContext的publishEvent方法来发送事件。

## 其他功能点
### Spring Aware
Spring管理的Bean对Spring容器的存在是无感知的，如果Bean中要用到Spring容器本身的资源，就要对其有感知，Spring Aware就是用来做这个的。
### Spring的多线程
Spring的多线程可以通过任务执行器TaskExecutor来实现，配置时用`@EnableAsync`开启对异步任务的支持，Bean中用`@Async`来声明这是一个异步任务。todo wujuneng 可以进一步学习线程池的设置，研究如何设置最大线程数量。
### 条件注解`@Conditional`
todo wujunpeng 除了条件注解，还可以学习自定义注解的实现，组合注解的原理等。利用书中的3.6，总结`@Enable*`相关注解的的原理，尤其是`@Import`注解来导入配置的三种类型。

## Spring MVC
两个概念
1. MVC：Model（数据模型）、View（视图）、Controller（控制器）  
2. 三层架构：Presentation tier展示层、Application tier应用层、Data tier数据访问层
- MVC实际上对应的只是三层架构的展示层，也就是说：展示层的实现用的是MVC的思想。Model是用来和View之间做数据交互、传值的，Controller就是MVC的`@Controller`注解的类。目前前后端分离的纯后端接口开发，是没有view的。Model就是接口提供的json。
- 三层架构是整个应用的架构，是由Spring框架负责管理的。项目结构中的Service层和DAO层就是三层架构中的应用层和数据访问层。  

### 一些之前用的不太熟悉的Controller注解
``` java
@Controller // 1
@RequestMapping("/api/user")
public class UserController {
    @@ResponseBody // 2
    @RequestMapping(value = "/query/{userId}", produces = "application/xml;charset=UTF-8") // 3
    public UserVO getUserInfo(@PathVariable String userId, HttpServletRequest request) { // 4
        String str = "URL:" + request.getRequestURL() + " can access. userId:" + userId; // 5
        System.out.println(str);
        return new UserVO();
    }
}
```
上述代码，根据注释的位置有以下解释：
1. 声明是一个Controller，可以接受请求
2. 声明返回值是放在response体中，而不是返回一个页面。可以和注释1合用，直接把注释1写成`@RestController`
3. produces可以定制返回数据的response媒体类型和字符集。"application/json;charset=UTF-8"返回json格式，"application/xml;charset=UTF-8"返回xml格式。还可以通过继承AbstractHttpMessageConverter的方式，自定义媒体类型。json和xml依赖的包如下：

``` xml
<!-- json依赖的包 -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.3</version>
</dependency>
<!-- xml依赖的包 这个包含了json的依赖，因此如果不支持xml，可以只用上面那个；如果要支持两种，只需要这个就够了-->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.3</version>
</dependency>
```
4. `@PathVariable`这里的用法是用来接受路径参数，和注释3中的{userId}配合使用。
5. 这里有一点需要注意，如果请求是：http://localhost:8080/api/user/query/33.5，在Spring MVC的默认配置下，userId参数只能获取到33，无法获取到33.5；假如userId是Integer类型，则此请求错误；假如userId是Double类型，则可以正常获取33.5。要想让String类型的`@PathVariable`注解获取到正确的数据，则应该通过在配置中重写configurePathMatch的方式来实现；Spring boot则没有这个顾虑，不配置也能正常获取到带“英文句号”的字符串值。

### Controller的全局配置
通过`@ControllerAdvice`这个注解，我们可以定义控制器的全局配置，比如用`@ExceptionHandler`异常处理；通过`@InitBinder`定制WebDataBinder来过滤参数；通过`@ModelAttribute`配置全局的键值对。

### 服务器端推送技术
早起的解决方案是通过Ajax向服务器轮训，但轮训的频率不好控制，增加了服务器的压力。可行的方案是客户端向服务器端发送请求，服务器端抓住这个请求不放，等有数据更新的时候再返回给客户端。客户端收到消息，再向服务器端发请求，周而复始，这样就解决了上述两个问题。还有一种双向通信技术，用**WebSocket**实现。

#### 1. 基于SSE的服务器端推送
SSE（Server Send Event 服务端发送事件）技术需要浏览器的支持，目前只有新式浏览器支持，比如chrome和firefox。实现时，服务端在Controller层，对应的接口媒体类型是text/event-stream；客户端在前端界面，需要用到EventSource对象，在前端代码中实现监听来获取服务端推送的消息。

#### 2. Servlet+异步方法
1. 开启异步
2. 实现一个返回DeferredResult类型结果的接口
3. 上述接口从一个Service Bean中获取一个DeferredResult。
