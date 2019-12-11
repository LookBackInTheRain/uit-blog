上篇[Spring boot+Security OAuth2 爬坑日记（4）自定义异常处理 上](https://blog.csdn.net/qq_31063463/article/details/83752459) 我们处理了认证服务的自定义异常，接下来我们接着处理资源服务的自定义异常

1. 自定义`OAuth2AuthenticationEntryPoint` 该接口处理Token相关的异常，处理类如下

	```java
	public class HttpUtils {
	    public static void writerError(BaseResponse bs, HttpServletResponse response) throws IOException {
	        response.setContentType("application/json,charset=utf-8");
	        response.setStatus(bs.getStatus());
	        ObjectMapper objectMapper = new ObjectMapper();
	        objectMapper.writeValue(response.getOutputStream(),bs);
	    }
	
	}
	```
	
	```java
	
	@Component
	public class BootOAuth2AuthExceptionEntryPoint extends OAuth2AuthenticationEntryPoint {
	    @Override
	    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) throws IOException, ServletException {
	        HttpUtils.writerError(HttpResponse.baseResponse(HttpStatus.UNAUTHORIZED.value(),
	        e.getMessage()),response);
	    }
	}
	
	```

2.  自定义`AccessDeniedHandler` 处理权限相关异常
	```java
	@Component
	public class BootAccessDeniedHandler implements AccessDeniedHandler {
	    @Override
	    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException ex) throws IOException, ServletException {
	        HttpUtils.writerError(HttpResponse.baseResponse(HttpStatus.FORBIDDEN.value(),"没有权限"),response);
	    }
	}
	```
3. 将两个自定义的异常处理类配置到资源资源服务中
	```java
	@Configuration
	@EnableResourceServer
	public class OAuth2ResourceServerConfig  extends ResourceServerConfigurerAdapter{
	
	    @Autowired
	    private AuthenticationEntryPoint point;
	
	    @Autowired
	    private BootAccessDeniedHandler handler;
	
	    @Autowired
	    private TokenStore tokenStore;
	
	
	    @Override
	    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
	
	        resources.tokenStore(tokenStore)
	                .resourceId("boot-server");
	        // 异常处理
	        resources.authenticationEntryPoint(point).accessDeniedHandler(handler);

	    }
	
	    @Override
	    public void configure(HttpSecurity http) throws Exception {
	        http
	                .authorizeRequests()
	                    .anyRequest()
	                    .access("#oauth2.hasAnyScope('all','select')");
	    }
	}

	```

上诉的两个异常处理都没有进入Controller内部，当我们的Controller内部出现异常时，我们的异常信息还是默认的处理方式，这里也需要接管下，Spring MVC 为我们提供了方便的处理方式，使用一个`@RestControllerAdvice` 注解即可，看如下实现
```java
@RestControllerAdvice
public final class ExceptionAdviceHandler {

    private final static String SERVER_ERROR_TXT = "服务器内部错误";
    private final static String ARGUMENTS_ERROR_TXT = "参数错误";
    private final static String BAD_REQUEST_TXT = "错误的请求";

    @ExceptionHandler(value = Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public BaseResponse unKnowExceptionHandler() {
        return this.serverErrorHandler();
    }

    @ExceptionHandler(value = RuntimeException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public BaseResponse runtimeExceptionHandler(RuntimeException ex) {
        return this.serverErrorHandler();
    }

    /**
     * 空指针异常
     */
    @ExceptionHandler(NullPointerException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public BaseResponse nullPointerExceptionHandler(Exception e) {

        e.printStackTrace();

        return this.serverErrorHandler();
    }

    /**
     * 类型转换异常
     */
    @ExceptionHandler(ClassCastException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public BaseResponse classCastExceptionHandler() {
        return this.serverErrorHandler();
    }

    /**
     * IO异常
     */
    @ExceptionHandler(IOException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public BaseResponse iOExceptionHandler() {
        return this.serverErrorHandler();
    }

    /**
     *  未知方法异常
     */
    @ExceptionHandler(NoSuchMethodException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public BaseResponse noSuchMethodExceptionHandler() {
        return this.serverErrorHandler();
    }

    /**
     * 数组越界异常
     */
    @ExceptionHandler(IndexOutOfBoundsException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public BaseResponse indexOutOfBoundsExceptionHandler() {
        return this.serverErrorHandler();
    }

    /**
     * 400错误
     */
    @ExceptionHandler(value = HttpMessageNotReadableException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public BaseResponse requestNotReadable() {
        return baseResponse(400, BAD_REQUEST_TXT);
    }

    /**
     * 400错误 类型不匹配
     */
    @ExceptionHandler({TypeMismatchException.class})
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public BaseResponse requestTypeMismatch() {
        return this.argumentsError();
    }

    /**
     * 400错误 缺少参数
     */
    @ExceptionHandler({MissingServletRequestParameterException.class})
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public BaseResponse requestMissingServletRequest() {
        return this.argumentsError();
    }

    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public BaseResponse methodArgumentNotValidExceptionHandler() {
        return baseResponse(400, "参数错误");
    }

    @ExceptionHandler(value = NotAuthorityException.class)
    @ResponseStatus(HttpStatus.UNAUTHORIZED)
    public SimpleResponse notAuthority(NotAuthorityException ex) {
        return this.authErrorHandler(2, ex.getMessage());
    }

    @ExceptionHandler(UsernameNotFoundException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public BaseResponse usernameNotFound(UsernameNotFoundException ex){
        return baseResponse(400,ex.getMessage());
    }


    @ExceptionHandler(value = NotAuthException.class)
    @ResponseStatus(HttpStatus.UNAUTHORIZED)
    public SimpleResponse notAuth(NotAuthException ex) {
        return this.authErrorHandler(1, ex.getMessage());
    }

    @ExceptionHandler(value = AuthFailureException.class)
    @ResponseStatus(HttpStatus.UNAUTHORIZED)
    public SimpleResponse authFieald(AuthFailureException ex) {
        return this.authErrorHandler(1, ex.getMessage());
    }


    /**
     * 405错误
     */
    @ExceptionHandler({HttpRequestMethodNotSupportedException.class})
    @ResponseStatus(HttpStatus.METHOD_NOT_ALLOWED)
    public BaseResponse request405(HttpServletResponse resp) {
        return baseResponse(405, "请求方法不正确");
    }

    /**
     * 406错误
     */

    @ExceptionHandler({HttpMediaTypeNotAcceptableException.class})
    @ResponseStatus(HttpStatus.NOT_ACCEPTABLE)
    public BaseResponse request406(HttpServletResponse resp) {
        return baseResponse(405, "不接受该请求");

    }

    /**
     * 500错误
     */
    @ExceptionHandler({ConversionNotSupportedException.class, HttpMessageNotWritableException.class})
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public BaseResponse server500(HttpServletResponse resp,Exception e) {
        return this.serverErrorHandler();
    }

    @ExceptionHandler(value = HttpMediaTypeNotSupportedException.class)
    @ResponseStatus(HttpStatus.UNSUPPORTED_MEDIA_TYPE)
    public BaseResponse httpMediaTypeNotSupportedExceptionHandler(HttpServletResponse resp) {
        return baseResponse(415, "服务器无法处理请求附带的媒体格式");
    }


    @ExceptionHandler(value = ArgumentsFailureException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public BaseResponse argsErrorExceptionHandler(ArgumentsFailureException ex, HttpServletResponse response) {
        return baseResponse(400, ex.getMessage());
    }

    /**
     * 404
     */
    @ExceptionHandler(value = NoHandlerFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public BaseResponse notFoundException(HttpServletResponse response) {
        return baseResponse(404, "找不到服务");
    }

    @ExceptionHandler(value = ServerErrorException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public BaseResponse serverErrorExceptionHandler(HttpServletResponse response) {
        return this.serverErrorHandler();
    }


    private BaseResponse serverErrorHandler() {
        return baseResponse(500, SERVER_ERROR_TXT);
    }

    private BaseResponse argumentsError() {
        return baseResponse(400, ARGUMENTS_ERROR_TXT);
    }

    /**
     * @param code 1 认证错误（未认证）、2 未授权/没有权限
     * @param msg
     * @return
     */
    private SimpleResponse authErrorHandler(int code, String msg) {
        Map<String, Object> mp = new HashMap<>();
        mp.put("code", code);
        return simpleResponse(401, msg, mp);
    }

}
```
最后Spring MVC 的404错误想让上述的处理方式捕获到，必须要在配置文件`application.yml`中加如下配置
```yaml
spring:
	mvc:
	  throw-exception-if-no-handler-found: true
```
当spring MVC找不到对应的controller来处理请求时，就不会使用默认的方式处理了，直接抛出异常，就会被`ExceptionAdviceHandler` 捕获到。

## 自定义 `ErrorControoler` 处理进入Controller发生的异常，具体实现如下
如果使用这方方式处理异常需要将`@RestControllerAdvice` 注释掉，自定义Controller 的方式优先级要低一些
```java
/**
 * @author yuit
 * @date 2019/12/10 10:01
 * 优先读取 配置文件中server.error.path 的值，没有值就使用 /error 默认路径
 **/
@Controller
@RequestMapping("${server.error.path:/error}")
public final class BootErrorController implements ErrorController {

    private final ErrorProperties errorProperties;
    private final ErrorAttributes errorAttributes;

    public BootErrorController(ErrorAttributes errorAttributes, ServerProperties serverProperties) {
        this.errorProperties = serverProperties.getError();
        this.errorAttributes = errorAttributes;
    }

    @RequestMapping
    public void handleError(HttpServletRequest request, HttpServletResponse response) throws IOException {
        HttpUtils.writer(preHandle(request),response);
    }


    private BaseResponse preHandle(HttpServletRequest request){


        boolean isIncludeStackTrace = this.isIncludeStackTrace(request);
        // 获取异常信息
        Map<String,Object> errors = this.getErrorAttributes(request,isIncludeStackTrace);

        Throwable throwable = (Throwable) errors.get("throwable");
        int status = (int)errors.get("status");
        return  HttpResponse.baseResponse(status,errors.get("error").toString());

    }


    private Map<String,Object> getErrorAttributes(HttpServletRequest request,boolean isIncludeStackTrace){
        ServletWebRequest webRequest = new ServletWebRequest(request);
        Map<String,Object> errors = this.errorAttributes.getErrorAttributes(webRequest,isIncludeStackTrace);

        // 获取异常
        Throwable throwable = this.errorAttributes.getError(webRequest);

        errors.put("throwable",throwable);
        return errors;
    }


    /**
     * Determine if the stacktrace attribute should be included.
     * @param request the source request
     * @return if the stacktrace attribute should be included
     */
    private boolean isIncludeStackTrace(HttpServletRequest request) {
        ErrorProperties.IncludeStacktrace include = this.errorProperties.getIncludeStacktrace();
        if (include == ErrorProperties.IncludeStacktrace.ALWAYS) {
            return true;
        }
        if (include == ErrorProperties.IncludeStacktrace.ON_TRACE_PARAM) {
            return this.getTraceParameter(request);
        }
        return false;
    }

    private boolean getTraceParameter(HttpServletRequest request) {
        String parameter = request.getParameter("trace");
        if (parameter == null) {
            return false;
        }
        return !"false".equalsIgnoreCase(parameter);
    }



    /**
     * 请求路径
     * @return
     */
    @Override
    public String getErrorPath() {
        return this.errorProperties.getPath();
    }
}

```

####  源码地址 [ **Github** ](https://github.com/LookBackInTheRain/oauth-boot)