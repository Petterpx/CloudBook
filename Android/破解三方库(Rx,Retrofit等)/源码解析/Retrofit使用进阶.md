# **Retrofit**使用进阶

#### 不得不说的代理模式？

### 静态代理：

**为其他对象提供一种代理，用以控制对这个对象的访问，比如代购。**



### 动态代理：

**程序运行时创建的代理方式**；





## Retrofit源码解析

Retrofit网络通信八步：

1. 创建 **Retrofit** 实例
2. 定义一个网络请求接口并为接口中的方法添加注解
3. 通过 **动态代理** 生成 网络请求对象 (通过接口解析网络请求中的注解)
4. 通过 **网络请求适配器** 将网络请求对象进行平台适配
5. 通过 **网络请求执行器** 发送网络请求 (Call）
6. 通过  **数据转换器** 解析数据
7. 通过 **回调执行器** 切换线程
8. 用户在主线程返回结果 



#### Retrfit中的变量

```java
//缓存 ,包含了我们网络请求的信息对象
private final Map<Method, ServiceMethod<?>> serviceMethodCache = new ConcurrentHashMap<>();
	
// Okhttp 工厂
  final okhttp3.Call.Factory callFactory;
  final HttpUrl baseUrl;
// 数据转换器工厂集合
  final List<Converter.Factory> converterFactories;
//适配器工厂集合
  final List<CallAdapter.Factory> callAdapterFactories;
//回调
  final @Nullable Executor callbackExecutor;
//是否立即解析Http中的接口
  final boolean validateEagerly;
```



```java
public static final class Builder {
  //适配的平台
    private final Platform platform;
  //请求网络的	Http工厂，默认OkHttpClient
    private @Nullable okhttp3.Call.Factory callFactory;
  //Http Url
    private @Nullable HttpUrl baseUrl;
  //数据转换器工厂集合，将网络上获取的 response对象转换成集合。
    private final List<Converter.Factory> converterFactories = new ArrayList<>();
  //网络转换适配器
    private final List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>();
 // 回调
    private @Nullable Executor callbackExecutor;
    private boolean validateEagerly;
```





## 为什么Retrofit可以在主线程回调数据？

我们看一下 Builder() 空参方法,里面调用了 Platform的get方法

```java
 public Builder() {
      this(Platform.get());
}
```



可以看到这是一个单例类，这里主要用于决定我们的使用平台，默认 Android

```java
class Platform {
  private static final Platform PLATFORM = findPlatform();

  static Platform get() {
    return PLATFORM;
  }

  private static Platform findPlatform() {
    try {
      Class.forName("android.os.Build");
      if (Build.VERSION.SDK_INT != 0) {
        return new Android();
      }
    } catch (ClassNotFoundException ignored) {
    }
    try {
      Class.forName("java.util.Optional");
      return new Java8();
    } catch (ClassNotFoundException ignored) {
    }
    return new Platform();
  }
```

我们可以看到 Platform 通过 findPlatform() 方法初始化。然后通过反射初始化了Android (Android平台) 。

我们接着看看 Android 这个类

```java
static class Android extends Platform {
    

    @Override public Executor defaultCallbackExecutor() {
      return new MainThreadExecutor();
    }

   。。。
    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());

      @Override public void execute(Runnable r) {
        handler.post(r);
      }
    }
  }
```

在build时，默认会初始化Android类，同时和主线程Handler绑定。这也就是它为什么结果最终会回调到主线程。





## BaseUrl 主要做什么？

我们从 baseUrl方法进入看看具体：

```java
    public Builder baseUrl(String baseUrl) {
      //判null，如果为null抛出异常
      checkNotNull(baseUrl, "baseUrl == null");
     //	HttpUrl.get(baseUrl) -> 将我们String url转换为HttpUrl
      return baseUrl(HttpUrl.get(baseUrl));
    }
```

接着我们继续去看看 baseUrl

```java
    public Builder baseUrl(HttpUrl baseUrl) {
      //判null
      checkNotNull(baseUrl, "baseUrl == null");
      //拆分我们的url，用于进行判断
      List<String> pathSegments = baseUrl.pathSegments();
      //监测我们最后一个集合是否以 / 结尾，也就是我们的baseUrl 必须以 / 结尾
      if (!"".equals(pathSegments.get(pathSegments.size() - 1))) {
        throw new IllegalArgumentException("baseUrl must end in /: " + baseUrl);
      }
      this.baseUrl = baseUrl;
      return this;
    }
```

经过上述查看，baseUrl做的主要就是转换工作，将String转换为 适合OkHttp的HttpUrl类型，并进行一些健壮性的判断。



## addConverterFactory 方法

数据转换器。

```java
    public Builder addConverterFactory(Converter.Factory factory) {
      converterFactories.add(checkNotNull(factory, "factory == null"));
      return this;
    }
```

主要是将我们 new 的数据转换器添加到相应的数据转换器集合中去。

比如 .addConverterFactory(GsonConverterFactory.create())

像上面这句代码，主要就是在里面初始化了 Gson 对象



## 从Retrofit.create开始

```java
TestService testService = retrofit.create(TestService.class);
```

```java
  public <T> T create(final Class<T> service) {
    //判断是否是接口，并且接口不可扩展
    Utils.validateServiceInterface(service);
    //是否提前初始化
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    //动态代理
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();
          private final Object[] emptyArgs = new Object[0];

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
          
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
  
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            //核心点在这 ->
            return loadServiceMethod(method).invoke(args != null ? args : emptyArgs);
          }
        });
  }
```



### loadServiceMethod

```java
  ServiceMethod<?> loadServiceMethod(Method method) {
    //从map(缓存)中去取Http方法，这里的http方法是指 get,post等
    ServiceMethod<?> result = serviceMethodCache.get(method);
    //如果已缓存，直接返回
    if (result != null) return result;

    //安全锁
    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        //从方法注解中去取
        result = ServiceMethod.parseAnnotations(this, method);
        //存到缓存中，存储方式 接口方法名-http方法
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```



上面的代码我们主要关注点在 **ServiceMethod.parseAnnotations(this, method)**:

```java
abstract class ServiceMethod<T> {
  static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
    //获取RequestFactory
    RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);
		
    //获取参数类型
    Type returnType = method.getGenericReturnType();
    if (Utils.hasUnresolvableType(returnType)) {
      throw methodError(method,
          "Method return type must not include a type variable or wildcard: %s", returnType);
    }
    if (returnType == void.class) {
      throw methodError(method, "Service methods cannot return void.");
    }

    return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
  }

  abstract T invoke(Object[] args);
}

```



#### RequestFactory

```java
static RequestFactory parseAnnotations(Retrofit retrofit, Method method) {
  //初始化RequestFactory，传入retrofit，网络请求方法
  return new Builder(retrofit, method).build();
}

	//api接口方法
  private final Method method;
	//http url
  private final HttpUrl baseUrl;
	//http方法 比如get,post
  final String httpMethod;
	//网络请求相对地址
  private final @Nullable String relativeUrl;
	//http请求头
  private final @Nullable Headers headers;
	//http请求的body类型
  private final @Nullable MediaType contentType;
  private final boolean hasBody;
  private final boolean isFormEncoded;
  private final boolean isMultipart;
	//方法参数处理器
  private final ParameterHandler<?>[] parameterHandlers;

。。。
		
		
    Builder(Retrofit retrofit, Method method) {
      this.retrofit = retrofit;
      this.method = method;
      //获取网络请求-方法上的注解
      this.methodAnnotations = method.getAnnotations();
      //获取网络请求方法中-参数的类型
      this.parameterTypes = method.getGenericParameterTypes();
      //获取网络请求方法中-注解的内容
      this.parameterAnnotationsArray = method.getParameterAnnotations();
    }

 RequestFactory build() {
 			//遍历注解，只允许使用一个 http方法，并将注解中的url取出
      for (Annotation annotation : methodAnnotations) {
        parseMethodAnnotation(annotation);
      }
			
			//判null处理
      if (httpMethod == null) {
        throw methodError(method, "HTTP method annotation is required (e.g., @GET, @POST, etc.).");
      }

      if (!hasBody) {
        if (isMultipart) {
          throw methodError(method,
              "Multipart can only be specified on HTTP methods with request body (e.g., @POST).");
        }
        if (isFormEncoded) {
          throw methodError(method, "FormUrlEncoded can only be specified on HTTP methods with "
              + "request body (e.g., @POST).");
        }
      }
			
			//获取方法参数的长度
      int parameterCount = parameterAnnotationsArray.length;
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      //遍历参数
      for (int p = 0; p < parameterCount; p++) {
      	//解析参数
        parameterHandlers[p] = parseParameter(p, parameterTypes[p], parameterAnnotationsArray[p]);
      }

      if (relativeUrl == null && !gotUrl) {
        throw methodError(method, "Missing either @%s URL or @Url parameter.", httpMethod);
      }
      if (!isFormEncoded && !isMultipart && !hasBody && gotBody) {
        throw methodError(method, "Non-body HTTP method cannot contain @Body.");
      }
      if (isFormEncoded && !gotField) {
        throw methodError(method, "Form-encoded method must contain at least one @Field.");
      }
      if (isMultipart && !gotPart) {
        throw methodError(method, "Multipart method must contain at least one @Part.");
      }

      return new RequestFactory(this);
    }

```



接着我们再去看最上面的 **HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);**

#### **HttpServiceMethod.parseAnnotations**

```java
//将接口方法转为 http 调用 
static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
      Retrofit retrofit, Method method, RequestFactory requestFactory) {
  	//赋值操作，返回相应的网络适配器
  	//核心关注点 1->
    CallAdapter<ResponseT, ReturnT> callAdapter = createCallAdapter(retrofit, method);
  	//获取网络适配器的返回类型
    Type responseType = callAdapter.responseType();
  	//如果响应类型是http
    if (responseType == Response.class || responseType == okhttp3.Response.class) {
      throw methodError(method, "'"
          + Utils.getRawType(responseType).getName()
          + "' is not a valid response body type. Did you mean ResponseBody?");
    }
  
    if (requestFactory.httpMethod.equals("HEAD") && !Void.class.equals(responseType)) {
      throw methodError(method, "HEAD method must use Void as response type.");
    }
		
  	//创建数据转换器
  //核心关注点 2->
    Converter<ResponseBody, ResponseT> responseConverter =
        createResponseConverter(retrofit, method, responseType);

    okhttp3.Call.Factory callFactory = retrofit.callFactory;
    return new HttpServiceMethod<>(requestFactory, callFactory, callAdapter, responseConverter);
  }
```



##### 1. createCallAdapter

```java
  private static <ResponseT, ReturnT> CallAdapter<ResponseT, ReturnT> createCallAdapter(
      Retrofit retrofit, Method method) {
    //获取网络请求方法中接口的返回类型
    Type returnType = method.getGenericReturnType();
    //获取网络请求接口中的注解
    Annotation[] annotations = method.getAnnotations();
    try {
      //noinspection unchecked
      //返回对应的网络请求适配器   -> 核心关注点
      return (CallAdapter<ResponseT, ReturnT>) retrofit.callAdapter(returnType, annotations);
    } catch (RuntimeException e) { // Wide exception range because factories are user code.
      throw methodError(method, e, "Unable to create call adapter for %s", returnType);
    }
  }
```

```java
  public CallAdapter<?, ?> callAdapter(Type returnType, Annotation[] annotations) {
    //空壳方法，继续往下追
    return nextCallAdapter(null, returnType, annotations);
  }

  public CallAdapter<?, ?> nextCallAdapter(@Nullable CallAdapter.Factory skipPast, Type returnType,
      Annotation[] annotations) {
    //非null判断
    checkNotNull(returnType, "returnType == null");
    checkNotNull(annotations, "annotations == null");

    int start = callAdapterFactories.indexOf(skipPast) + 1;
    
    //用于创建CallAdpater，过程如下：
    //遍历callAdapter工厂集合
    for (int i = start, count = callAdapterFactories.size(); i < count; i++) {
      //寻找合适的工厂并返回
      CallAdapter<?, ?> adapter = callAdapterFactories.get(i).get(returnType, annotations, this);
      if (adapter != null) {
        return adapter;
      }
    }
```

> 结论：
>
>   **CallAdapter<ResponseT, ReturnT> callAdapter = createCallAdapter(retrofit, method);** 
>
> ***做的只是一个赋值操作，通过调用 createCallAdapter ，并传入retrofit对象和网络请求方法，通过我们请求方法中的注解最后寻找到我们网络请求中相应返回类型的网络适配器，然后进行赋值操作。***



##### 2. createResponseConverter

```java
  private static <ResponseT> Converter<ResponseBody, ResponseT> createResponseConverter(
      Retrofit retrofit, Method method, Type responseType) {
    //获取接口中网络请求方法的注解
    Annotation[] annotations = method.getAnnotations();
    try {
      //传入网络适配器返回类型，和我们网络请求方法中的注解
      //核心关注点->
      return retrofit.responseBodyConverter(responseType, annotations);
    } catch (RuntimeException e) { // Wide exception range because factories are user code.
      throw methodError(method, e, "Unable to create converter for %s", responseType);
    }
  }
```

```java
public <T> Converter<ResponseBody, T> responseBodyConverter(Type type, Annotation[] annotations) {
	//空壳方法，继续追
  return nextResponseBodyConverter(null, type, annotations);
}

/**
 * Returns a {@link Converter} for {@link ResponseBody} to {@code type} from the available
 * {@linkplain #converterFactories() factories} except {@code skipPast}.
 *
 * @throws IllegalArgumentException if no converter available for {@code type}.
 */
public <T> Converter<ResponseBody, T> nextResponseBodyConverter(
    @Nullable Converter.Factory skipPast, Type type, Annotation[] annotations) {
  checkNotNull(type, "type == null");
  checkNotNull(annotations, "annotations == null");

  int start = converterFactories.indexOf(skipPast) + 1;
  for (int i = start, count = converterFactories.size(); i < count; i++) {
    Converter<ResponseBody, ?> converter =
        converterFactories.get(i).responseBodyConverter(type, annotations, this);
    if (converter != null) {
      //noinspection unchecked
      return (Converter<ResponseBody, T>) converter;
    }
  }
```

> 和上面网络适配器的操作非常相似，所以逻辑也区别不大。都是从我们的工厂中寻找，并赋值



### 总结：loadServiceMethod

> 最开始通过缓存池获得了一个ServiceMethod对象，如果有缓存直接返回，否则开启一个线程锁保证我们的线程数据安全,然后通过 parseAnnotations 方法去创建ServiceMethod对象，这个preseAnnotatiions 方法内部去通过RequestFactory 这个请求工厂的 parseAnnotations 方法去解析并创建一个RequestFactory 对象，最后通过 HttpServiceMethod(主要是将接口方法的调用改为Http调用) 的 parseAnnotations方法解析创建了我们相应返回类型的 ServiceMethod对象。并将这个对象存进我们的缓存池当中。
>
> 所以当我们当最外面 调用接口方法时，传入了相应的参数，实则是由动态代理将我们接口中的信息转换成了ServiceMethod 对象。 





  