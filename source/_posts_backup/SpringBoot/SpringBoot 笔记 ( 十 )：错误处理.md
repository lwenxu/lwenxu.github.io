---
title: SpringBoot 笔记(十)：错误处理
date: 2018-05-24 10:01:28
tags: SpringBoot
categories:
 - SpringBoot
---
# SpringBoot 笔记 ( 十 )：错误处理

## 1）、SpringBoot默认的错误处理机制

默认效果：

​		1）、浏览器，返回一个默认的错误页面
		2）、如果是其他客户端，默认响应一个json数据

<!--more-->

## 2）、自动配置原理

​	具体就是在    `ErrorMvcAutoConfiguration`，错误处理的自动配置。

  	给容器中添加了以下组件

### 	1、ErrorPageCustomizer：规定错误页面  /error 

```java
	@Value("${error.path:/error}")
	private String path = "/error";  //系统出现错误以后来到error请求进行处理,（类似与我们在web.xml注册的错误页面规则）
```

### 	2、BasicErrorController：处理默认 /error 请求

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    //处理产生html类型的数据；浏览器发送的请求来到这个方法处理
    @RequestMapping(produces = "text/html")
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
        //去哪个页面作为错误页面；包含页面地址和页面内容
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
	}

    //处理非Html的类型的数据。产生json数据，其他客户端来到这个方法处理；
	@RequestMapping
	@ResponseBody    
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<Map<String, Object>>(body, status);
	}
```

其实这里就解释了为什么我们不同的客户端会返回的数据不一样，我们采用浏览器和程序请求会不同，就是因为他们的header不一样导致的。

但是注意上面的`ModelAndView`对象我们是通过 `resolveErrorView` 来获取的，也就是下面的逻辑：

```java
protected ModelAndView resolveErrorView(HttpServletRequest request,
      HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
    // 所有的 ErrorViewResolver 得到 ModelAndView
   for (ErrorViewResolver resolver : this.errorViewResolvers) {
      ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
      if (modelAndView != null) {
         return modelAndView;
      }
   }
   return null;
}
```

然后上面在解析错误视图，调用了 resolver 的方法，这个方法其实是 `DefaultErrorViewResolver` 的方法。也就是下面的这个类的方法。

### 	3、DefaultErrorViewResolver：

```java
@Override
	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status,
			Map<String, Object> model) {
		ModelAndView modelAndView = resolve(String.valueOf(status), model);
		if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
			modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
		}
		return modelAndView;
	}

	private ModelAndView resolve(String viewName, Map<String, Object> model) {
        //默认SpringBoot可以去找到一个页面？  error/404
		String errorViewName = "error/" + viewName;
        
        //模板引擎可以解析这个页面地址就用模板引擎解析
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders
				.getProvider(errorViewName, this.applicationContext);
		if (provider != null) {
            //模板引擎可用的情况下返回到errorViewName指定的视图地址
			return new ModelAndView(errorViewName, model);
		}
        //模板引擎不可用，就在静态资源文件夹下找errorViewName对应的页面   error/404.html
		return resolveResource(errorViewName, model);
	}
```

这里主要做的事情就是，根据状态码得到模板的页面地址，然后让我们的模板引擎解析一下，如果能解析到就是返回对应的ModelAndView 否则的话就直接去静态资源的static目录找对应的状态码对应的页面404.html 4xx.html ...。然后这里的model也就是我们的异常错误数据，是通过下面的组件完成的。

### 	4、DefaultErrorAttributes

```java
	@Override
	public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes,
			boolean includeStackTrace) {
		Map<String, Object> errorAttributes = new LinkedHashMap<String, Object>();
		errorAttributes.put("timestamp", new Date());
		addStatus(errorAttributes, requestAttributes);
		addErrorDetails(errorAttributes, requestAttributes, includeStackTrace);
		addPath(errorAttributes, requestAttributes);
		return errorAttributes;
}
```

这个方法就是在容器里注册了一个 Map 帮我们在页面共享信息。

​	步骤：

1. 一但系统出现4xx或者5xx之类的错误，ErrorPageCustomizer就会生效（定制错误的响应规则），来到/error请求

2. 就会被**BasicErrorController**处理
3. 响应页面：去哪个页面，错误信息是由**DefaultErrorViewResolver**解析得到的，错误信息由 `getErrorAttributes` 获取。

## 3）、定制错误响应

### 1）、定制错误的 Html 页面

#### 1）、有模板引擎

error/状态码【将错误页面命名为  错误状态码.html 放在模板引擎文件夹里面的 error文件夹下】，发生此状态码的错误就会来到  对应的页面；

我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态码.html）。

页面能获取的信息，是由Model来确定的，也就是我们的 `getErrorAttributes` 方法来确定的。具体的有：

- timestamp：时间戳
- status：状态码
- error：错误提示
- exception：异常对象
- message：异常消息
- errors：JSR303数据校验的错误都在这里

#### 2）、没有模板引擎

没有模板引擎或者说模板引擎找不到这个错误页面，那就去静态资源文件夹static下找，规则同模板引擎的规则。

#### 3）、没有任何错误页面

就是默认来到SpringBoot默认的错误提示页面

### 2）、如何定制错误的json数据；

#### 1）、自定义异常处理&返回定制json数据

这里其实就是用了 Spring 的统一异常处理，但是这样的话我们就是对所有的页面都会出现json了，没有针对不同的客户端有自适应效果。

```java
@ControllerAdvice
public class MyExceptionHandler {
    @ResponseBody
    @ExceptionHandler(UserNotExistException.class)
    public Map<String,Object> handleException(Exception e){
        Map<String,Object> map = new HashMap<>();
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        return map;
    }
}

```



#### 2）、转发到/error进行自适应响应效果处理

```java
 	@ExceptionHandler(UserNotExistException.class)
    public String handleException(Exception e, HttpServletRequest request){
        Map<String,Object> map = new HashMap<>();
        //传入我们自己的错误状态码  4xx 5xx，否则就不会进入定制错误页面的解析流程
        /**
         * Integer statusCode = (Integer) request
         .getAttribute("javax.servlet.error.status_code");
         */
        request.setAttribute("javax.servlet.error.status_code",500);
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        //转发到 /error 
        return "forward:/error";
    }
```

这里就是通过我们自动配置的默认错误页面的控制器来处理错误页面的请求，让我们吧一些特殊的错误数据发送过去，然后直接转发到我们的错误页面即可，接下来就是SpringBoot帮助我们处理自适应问题了。

又一点要注意，就是我们这里自己设置了状态码，为什么需要这样，这是由于我们这里拦截到了错误，然后我们并没有走默认的错误处理的逻辑也就是我们的默认的错误处理的Controller没有执行，导致一些错误的状态码没有设置，但是我们最终需要渲染视图，以及寻找错误页面都是通过我们的的错误状态码的，这里找不到状态码，我们必须手动的添加上才行。否则不是去往我们的错误页面。

还有一点需要注意的就是我们虽然已经设置了一个map，但是这个map并没有被我们的模板获取到，因为根本没用我们的数据，这里我们需要 定制一下 `getErrorAttributes（）` 方法才行。

#### 3）、将我们的定制数据携带出去；

出现错误以后，会来到/error请求，会被BasicErrorController处理，响应出去可以获取的数据是由getErrorAttributes得到的（是AbstractErrorController（ErrorController）规定的方法）；

我们有两种方式来自定义数据：

​	1、完全来编写一个ErrorController的实现类【或者是编写AbstractErrorController的子类】，放在容器中；

​	2、页面上能用的数据，或者是json返回能用的数据都是通过errorAttributes.getErrorAttributes得到，容器中DefaultErrorAttributes.getErrorAttributes()，默认进行数据处理的；

自定义ErrorAttributes

```java
//给容器中加入我们自己定义的ErrorAttributes
@Component
public class MyErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
        Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
        map.put("company","atguigu");
        return map;
    }
}
```

最终的效果：响应是自适应的，可以通过定制ErrorAttributes改变需要返回的内容，