---
title: Http接口返回数据的一些问题
---

## 概述
端午节跟游顺一起做了个小web系统，有账户模块和查询模块两部分。本来说三天做完这个初版的功能，但是仅账户登录就已经把我的时间耗尽了，也就没时间做查询了。登录用的Spring Security实现jwt认证，期间翻看过Spring文档和一篇[博客][article]，基本上还原出来了，也知道大概的原理。想说的不是这个登录，登录还没时间吃透。想说的是关于http接口请求的返回数据。

## 问题
SpringBoot接口返回数据极其简单，定义好一个对象，就可以返回对象格式的json了，甚至可以单纯返回字符串。

但是如果用`HttpServletResponse`对象如何操作呢？问题场景是，有时候在`filter`里就想返回请求结果。

如果想改变接口返回的HttpCode，又怎么做呢？场景是，有时候前端RD想用HttpCode做判断，而非一个正常的Http返回结果，需要判断返回数据中的某个字段。详见[这里][HttpCode]

## 解决方法
不多废话了，直接贴代码吧
``` java
import com.jiler.website.responseData.ResponseData;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {
    /**
     * 返回对象
     *
     * @return
     */
    @GetMapping("/info")
    public ResponseData<String> userInfo(){
        ResponseData<String> responseData = new ResponseData<>();
        responseData.setData("wujunpeng");
        return responseData;
    }

    /**
     * 返回字符串
     *
     * @return
     */
    @GetMapping("/name")
    public String userName(){
        return "wujunpeng";
    }

    /**
     * 用response对象返回
     */
    @RequestMapping(value = "/alive", method = RequestMethod.GET)
    public void monitorAlive(HttpServletResponse response) throws IOException {
        Map<String, Object> result = new HashMap<>();
        result.put("status", 0);
        result.put("message", "执行完毕");
        response.setContentType("text/html;charset=utf-8");
        response.getWriter().print(MyJsonUtils.toJson(result));
    }

    /**
     * 用response对象修改HttpCode
     */
    @GetMapping("/password")
    public void password(HttpServletResponse response) throws IOException {
        response.sendError(HttpServletResponse.SC_FORBIDDEN, "禁止请求密码接口");
         // response.setStatus(HttpServletResponse.SC_FORBIDDEN); // 用这个也行。
    }
}
```



[article]: https://javazhiyin.blog.csdn.net/article/details/112210811
[HttpCode]: https://jaskey.github.io/blog/2014/09/27/how-to-return-404-in-spring-controller/
