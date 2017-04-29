---
title: Spring MVC 配置 CORS 标准支持
date: 2016-08-22 23:22:45
tags: java
---

## CORS 简介

CORS (Cross-origin resource sharing), 是 W3C 关于跨域资源共享的一种标准。它的具体介绍可以参考[阮一峰的这篇文章](http://www.ruanyifeng.com/blog/2016/04/cors.html)，本文不再详述。

从该文章中可知，CORS 请求可以分为两种：简单请求和非简单请求。
对于简单请求，浏览器直接发出请求，并在头信息中添加一个 `Origin` 字段，用于说明本次请求来自哪个源（包括协议/域名/端口）。
对于非简单请求，浏览器会在正式通信前，发送一个 HTTP OPTIONS 请求，用于服务器确认，确认通过后浏览器发出原本的通信请求。

## Spring MVC 实现基本支持

Spring MVC 自 4.2 版本内建了对 CORS 标准的支持，开发者可以通过简单的配置，将原有接口转变为支持跨域请求的接口。而在此版本之前，开发者需要自己实现该功能。
下面将分别以 Spring MVC 3.1.4 和 4.2.0 两个版本为例，添加对 CORS 的支持。

(1) Spring MVC 3.1.4
对于简单请求，浏览器在收到 `Response` 后，会检查 `Header` 里面的 `Access-Control-Allow-Origin` 字段。如果该字段不存在，或者字段对应的值和当前请求的源不一致，则抛出错误。

因此，服务器在收到浏览器发出的请求后，需要在 `Response Header` 中写入 `Access-Control-Allow-Origin` 字段，且字段对应的值必须为 * 或者允许请求的源。这里还可以添加其他可选字段，参考 CORS 的介绍文章，后面不再复述。
下面为一个简单示例：

```java
@RequestMapping(value = "/cors", method = RequestMethod.POST)
@ResponseBody
public String cors(@RequestParam String orderId, HttpServletResponse response) {
    response.setHeader("Access-Control-Allow-Origin", ALLOWED_ORIGIN);
    // other logic
}
```

需要注意的一点是，当 `Content-Type` 为 `application/x-www-form-urlencoded` 时，`@ResponseBody` 注解默认的 `FormHttpMessageConverter` 会把请求中的数据绑定为 `MultiValueMap`。如果你需要绑定为自定义的类型，可以使用 `@ModelAttribute`，或者实现一个自定义的 `HttpMessageConverter`。

```java
@RequestMapping(value = "/cors", method = RequestMethod.POST)
@ResponseBody
public String cors(@ModelAttribute SimpleOrderInfo orderInfo, HttpServletResponse response) {
    response.setHeader("Access-Control-Allow-Origin", ALLOWED_ORIGIN);
    // other logic
}
```

对于非简单请求，在正式请求前会有一个请求类型为 `OPTIONS` 的预检请求。而在低版本的 Spring MVC 中，`DispatcherServlet` 默认是不支持 `OPTIONS` 类型的请求的，具体可参考其父类 `FrameworkServlet` 的源码。

```java
private boolean dispatchOptionsRequest = false;

@Override
protected void doOptions(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

    if (this.dispatchOptionsRequest) {
        processRequest(request, response);
        ...
    }
    super.doOptions(request, response);
}
```

因此，你可以选择在 `Filter` 中对 `OPTIONS` 请求进行处理，也可以在 Spring 启动参数中进行设置，然后通过 Spring 进行处理。下面是通过 Spring 处理的过程。

设置 Spring 启动参数

```java
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>dispatchOptionsRequest</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        ...
    </init-param>
</servlet> 
```

回应预检请求。如果预检请求中包含 `Access-Control-Request-Headers` 字段，则服务器端的 `Response` 中除 `Access-Control-Allow-Origin` 外，还需要添加 `Access-Control-Allow-Headers` 字段，其值包括且不限于 `Access-Control-Request-Headers` 中传入的值。

```java
    @RequestMapping(value = "/cors", method = {RequestMethod.POST, RequestMethod.OPTIONS})
    @ResponseBody
    public String cors(@ModelAttribute SimpleOrderInfo orderInfo, HttpServletRequest request, HttpServletResponse 
        response) {
        response.setHeader("Access-Control-Allow-Origin", ALLOWED_ORIGIN);
        String accessControlReqHeaders;
        if ((accessControlReqHeaders = request.getHeader("Access-Control-Request-Headers")) != null) {
            response.setHeader("Access-Control-Allow-Headers", accessControlReqHeaders);
        }
        // other logic
    }
```

回应正式请求。这里和简单请求类似，需要在 `Response Header` 中添加 `Access-Control-Allow-Origin` 字段。上面的代码中已经实现了这点。

(2) Spring MVC 4.2.0

4.2.0 之后，只需要在方法或者类上添加 `@CrossOrigin` 注解，简单配置，即可实现对简单请求和非简单请求的支持。

```java
@CrossOrigin(ALLOWED_ORIGIN)
@RequestMapping(value = "/cors", method = RequestMethod.POST)
@ResponseBody
public String cors(@ModelAttribute SimpleOrderInfo orderInfo) {
    // other logic
}
```

## 简单封装后的实现

比较上述两种实现，可以明显地发现 Spring MVC 4.2.0 之后基于注解的实现具有配置简单，侵入性低的优点。在学习了 CORS 请求的基本规则之后，我们也可以实现一个简单的通过注解配置的方案，使基于较老的 Spring 版本的项目也能享受到这种便利。

一个简单又具备可扩展性的方案包含以下部分：
 - `CrossOrigin` 注解
 - `CorsSetting` 存储注解配置的信息
 - `CorsSettingParser` 解析注解配置
 - `CorsInterceptor` 拦截和解析 CORS 请求的 `Interceptor` 实现类
 - `CorsUtil` 一些辅助方法

截取了部分代码，展示如下。

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CrossOrigin {

    CorsMode value() default CorsMode.CUSTOM;

    String[] origins() default {};

    String[] allowedHeaders() default {};
}
```

```java
public class CorsInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) 
    throws Exception {
        if (!CorsUtil.isCorsRequest(request)) {
            return true;
        }

        CorsSetting classCorsSetting = CorsSettingParser.CLASS.parseCorsSetting(handler);
        CorsSetting methodCorsSetting = CorsSettingParser.METHOD.parseCorsSetting(handler);
        CorsSetting corsSetting = classCorsSetting == null ? methodCorsSetting :
                classCorsSetting.merge(methodCorsSetting);

        if (CorsUtil.isPreFlightRequest(request)) {
            processPreFlightRequest(request, response, corsSetting);
        } else {
            processCorsRequest(request, response, corsSetting);
        }
        return true;
    }

    ...
}
```

```java
public enum CorsSettingParser {

    CLASS {
        @Override
        public CrossOrigin getCrossOriginAnn(HandlerMethod handlerMethod) {
            return handlerMethod.getBeanType().getAnnotation(CrossOrigin.class);
        }
    }, METHOD {
        @Override
        public CrossOrigin getCrossOriginAnn(HandlerMethod handlerMethod) {
            return handlerMethod.getMethodAnnotation(CrossOrigin.class);
        }
    };

    public CorsSetting parseCorsSetting(Object handler) {
        if (!(handler instanceof HandlerMethod)) {
            return null;
        }

        CrossOrigin crossOrigin = getCrossOriginAnn((HandlerMethod) handler);
        if (crossOrigin == null) {
            return null;
        }

        CorsSetting setting = new CorsSetting();
        setting.setValue(crossOrigin.value());
        setting.setOrigins(crossOrigin.origins());
        setting.setAllowedHeaders(crossOrigin.allowedHeaders());
        return setting;
    }

    protected abstract CrossOrigin getCrossOriginAnn(HandlerMethod handlerMethod);

}
```

如果使用上述方案，还要注意由于低版本 Spring 本身的限制，需要在启动参数中配置 `dispatchOptionsRequest` 为 `true` ，如果是非简单请求，还要在接口声明的请求方法中添加对 `OPTIONS` 请求的支持。

```java
@RequestMapping(value = "/cors", method = {RequestMethod.POST, RequestMethod.OPTIONS})
@CrossOrigin(origins = "http://targetsite.com", allowedHeaders = "*")
@ResponseBody
public String cors(@ModelAttribute SimpleOrderInfo orderInfo) {
    // other logic
}
```
