---
title: SpringBoot中Web应用统一异常处理
date: 2018-08-02 18:09:23
tags:
    - java
---

在Web应用开发中，异常一般是不可避免的，但是每个异常可能都不尽相同，并且异常的内容也是五花八门。针对这种情况，就最好需要一个统一的异常处理，那么`Spring Boot`提供了一个默认的映射：`/error`，当抛出异常之后，会转到该请求中处理，并且该请求有一个全局的错误页面用来展示异常内容。

例如以下代码
```java
    @RequestMapping("/index")
    public String index() throws Exception {
        throw new Exception("请求index出现了异常");
    }
```
<!--more-->
访问`http://localhost:8080/index`就会出现以下错误：
!['ex'](/images/exception0.jpg)

### 统一异常处理
虽然在`Spring Boot`中已经有默认的`error`映射，但是在实际效果中，这种异常抛出，一点用户体验都没有。<br/>
所以，我们需要统一格式的异常处理。

#### 创建全局异常类
通过注解`@ControllerAdvice`来定义统一异常处理类，并不需要在每个Controller中定义。<br/>
注解`@ExceptionHandler`用来定义方法具体处理什么异常，最终应该将异常映射到什么地方。
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(value = Exception.class)
    public ModelAndView defaultErrorHandler(HttpServletRequest req, Exception e) throws Exception {
        ModelAndView mav = new ModelAndView();
        mav.addObject("exception", e);
        mav.addObject("url", req.getRequestURL());
        mav.setViewName("error");
        return mav;
    }
}
```

#### 创建error.html
在`template`目录下，创建`error.html`文件，将异常信息输出（使用了`thymeleaf`模板）
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8" />
    <title>统一异常处理</title>
</head>
<body>
<h1>统一异常处理展示</h1>
<div th:text="${url}"></div>
<div th:text="${exception.message}"></div>
</body>
</html>
```
然后访问浏览器，返回的页面是
!['ex1'](/images/exception1.jpg)

#### 也可以自定义JSON格式的数据返回
一般提供一个`Restful Api`的接口，可能会返回统一的异常数据，一般格式是`JSON`。<br/>
只需要在全局异常处理类中添加一个方法，并添加一个异常类的映射，并在方法上加上注解` @ResponseBody`就行了。
代码如下：
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public ErrorInfo<String> jsonErrorHandler(HttpServletRequest req, Exception e) throws Exception {
        ErrorInfo<String> r = new ErrorInfo<>();
        r.setMessage(e.getMessage());
        r.setCode(ErrorInfo.ERROR);
        r.setData("JSON格式的数据");
        r.setUrl(req.getRequestURL().toString());
        return r;
    }
}
```
也可以自己写一个异常类，这里就不写了。<br/>
访问浏览器，返回的页面结果为：
!['ex2'](/images/exception2.jpg)

以上，基本已经完成了统一异常处理类，关于注解这一块，以后再说吧。

