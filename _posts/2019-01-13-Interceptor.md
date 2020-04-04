---
layout:     post
title:      "SpringMVC拦截器"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-01-12 22:00:00
author:     "Panying"
header-img: "img/post-bg-2015.jpg"
tags:
    - SpringMVC
---

* content
{:toc}

## 简介
SpringMVC的处理器拦截器，类似于Servlet开发中的过滤器Filter，用于对请求进行拦截和处理。

## 常见应用场景
1、权限检查：如检测请求是否具有登录权限，如果没有直接返回到登陆页面。 
2、性能监控：用请求处理前和请求处理后的时间差计算整个请求响应完成所消耗的时间。 
3、日志记录：可以记录请求信息的日志，以便进行信息监控、信息统计等。

## 如何使用
1.创建一个MyInterceptor类，并实现HandlerInterceptor接口

```
public class MyInterceptor implements HandlerInterceptor {

    @Override
    public void afterCompletion(HttpServletRequest arg0,
            HttpServletResponse arg1, Object arg2, Exception arg3)
            throws Exception {
        System.out.println("afterCompletion");
    }

    @Override
    public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1,
            Object arg2, ModelAndView arg3) throws Exception {
        System.out.println("postHandle");
    }

    @Override
    public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1,
            Object arg2) throws Exception {
        System.out.println("preHandle");
        return true;
    }

}
```
2、配置
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
        registry.addInterceptor(new ThemeChangeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```
或者
```
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/admin/**"/>
        <bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/secure/*"/>
        <bean class="org.example.SecurityInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```
