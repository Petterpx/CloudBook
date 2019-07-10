# Retrofit

基于OkHttp实现，是Okhttp的二次封装，本质依然是Okhttp

## 使用方法：

导包：

```java
implementation 'com.squareup.retrofit2:retrofit:2.5.0'
implementation 'com.squareup.retrofit2:converter-gson:2.3.0'
implementation 'com.google.code.gson:gson:2.8.5'
```

其中 com.squareup.retrofit2:converter-gson:2.3.0' 是为了增加返回值为 Gson 类型数据所需要添加的依赖包。



### Retrofit 的注解分类

Retrodit 与其他请求框架不同的是，它使用了注解。Retrofit 的注解分为3大类，分别是 Http 请求方法注解，标记类注解和参数类注解。其中：

HTTP请求方法注解有8中：分别是 **GET,POST,PUT,DELETE,HEAD,PATCH,OPTIONS 和 HTTP**.其前7种分别对应 HTTP 的请求方法：HTTP 则可以替换以上 7 中，也可以扩展请求方法。

标记类注解有3 中，它们是 FromUrlEncoded,Multipart,Streatming.FormUrlEncoded 和 Multipart后面会讲到：Streaming 代表响应的数据以流的形式如果不适用它，则默认会把全部数据加载到内存，所以下载大文件是需要加上这个注解。参数类注解有 Header，Headers,Body,Path,Field.FieldMap,Part,PartMap,Query和 QueryMap等。

### Get请求访问网络

我们首先实现 Get 请求方法来访问网络。这里的测试我用我自己写的简单PHP后端。

先写相应的数据类：

```java
public class Info {
    private String code;
    private String message;
    private String data;

    public Info(String code, String message, String data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }
}
```

再编写网络请求接口：

```java
/**
 * @author Petterp on 2019/7/9
 * Summary:Get请求
 * 邮箱：1509492795@qq.com
 */
public interface IpService {
    @GET("API/get.php")
    Call<Info> getInfo(@Query("name") String name,@Query("pswd") String pswd);
}
```

Retrofit 提供的请求方式注解有 @Get和  @Post等，分别代表 GET请求 和 POST 请求，我们在这里用的是 GET 请求。访问的地址是 API/get.php ，另外定义了 getInfo 方法，这个方法返回 Call<Info> 类型的参数，接下来创建 Retrofit ,并创建接口文件。

代码如下：

```java
new Retrofit.Builder()
        .baseUrl("http://xxx.xxx.xx.xxx/")
            
        .build()
        .create(IpService.class)
        .getInfo("name","pswd")
        .enqueue(new Callback<Info>() {
            @Override
            public void onResponse(Call<Info> call, Response<Info> response) {
                Toast.makeText(MainActivity.this, response.body().getData(), Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onFailure(Call<Info> call, Throwable t) {

            }
        });
```

Retrofit 是通过建造者构建出来的。请求 url 是拼接而成的，它是由 baseUrl 传入的 Url 加上请求网络接口的 @Get("API/get.php") 中的URL 拼接而成的。接下来用 该接口定义的 getInfo 方法，传入相应的参数，并得到 Call对象，接下来用 Call请求网络并处理回调。因为是异步请求，回调的 Callback 是运行在 UI 线程的。得到返回的 Response 后将返回数据的用Toast弹出。 **注意：如果想同步请求，请调用 call.execute,链式调用即可。如果想中断网络请求，可以使用 call.cancel()**



## 一些技巧

#### 动态配置 URL 地址：@Path

@Path 用来动态的配置 URL 地址，请求网络接口代码如下所示：

```java
@GET("{path}/get.php?name=name&pswd=pswd")
Call<Info> getInfoPath(@Path("path")String path);
```

Retrofit的配置

```java
new Retrofit.Builder()
        .baseUrl("http://xxx.xxx.xx.xxx/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()
        .create(IpService.class)
        .getInfoPath("API") //1
        .enqueue(new Callback<Info>() {
            @Override
            public void onResponse(Call<Info> call, Response<Info> response) {
                Toast.makeText(MainActivity.this, response.body().getData(), Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onFailure(Call<Info> call, Throwable t) {

            }
        });
```

注释1的地方就是更改的地方。用 http://xxx.xxx.xx.xxx/API/get.php  来举例，

baseUrl为 http://xxx.xxx.xx.xxx,

Path为 API，也就是代码里{path}那块



#### 动态指定查询条件：@query

```java
public interface IpService {
    @GET("API/get.php")
    Call<Info> getInfo(@Query("name") String name,@Query("pswd") String pswd);
}
```

即就是我们最开始使用的，请求网络时传入相应的值即可。

```java
。。。
        .getInfo("name","pswd")
。。。
```



#### 动态指定查询条件组：@QueryMap

在网络请求中一半为了更精确的查找我们所需要的数据，需要传入很多查询数据。这时如果还使用 @Quey 会比较麻烦，这时可以采用个@QueyMap,将所有的参数继承在一个 Map 中统一传递。

更改接口：

```java
@GET("API/get.php")
Call<Info> getInfoQueryMap(@QueryMap Map<String,String> options);
```

接着更改 Retrofit 请求中的调用方法。

```java
//-->
HashMap<String,String> map=new HashMap<>();
map.put("name","name");
map.put("pswd","name");
//-->
new Retrofit.Builder()
        .baseUrl("http://xxx.xxx.xx.xxx/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()
        .create(IpService.class)
        .getInfoQueryMap(map) // ->
        .enqueue(new Callback<Info>() {
            @Override
            public void onResponse(Call<Info> call, Response<Info> response) {
                Toast.makeText(MainActivity.this, response.body().getData(), Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onFailure(Call<Info> call, Throwable t) {

            }
        });
```



## Post请求

#### 传输数据类型为 键值对： **@Field**

传输类型为键值对，这时我们最常用的 POST 请求数据类型，这里继续用我自己的简易 php 后端。

请求网络接口代码：

```java
/**
 * @author Petterp on 2019/7/9
 * Summary:Post请求
 * 邮箱：1509492795@qq.com
 */
public interface IpServerPost {
    @FormUrlEncoded
    @POST("API/post2.php")
    Call<Info> getInfo(@Field("data") String data);
}
```

首先用@FormUrlEncoded 注解标明这是一个表单请求，然后再 getInfo() 方法中使用 @Field 注解来标识所对应的 String 类型数据的键，从而组成一组键值对进行传递。

请求网络代码如下：

```java
new Retrofit.Builder()
        .baseUrl("http://xxx.xxx.xx.xxx/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()
        .create(IpServerPost.class)
        .getInfo("123") //1
        .enqueue(new Callback<Info>() {
            @Override
            public void onResponse(Call<Info> call, Response<Info> response) {
                Toast.makeText(MainActivity.this, response.body().getData(), Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onFailure(Call<Info> call, Throwable t) {

            }
        });
```



#### 传输数据类型Json 字符串：@Body

我们也可以用个Post 方式将 JSON 字符串作为请求体发送到服务器，请求网络接口的代码如下所示：

```java
@POST("API/post1.php")
Call<Info> getInfoBody(@Body Data data);
```

Data对象：

```java
public class Data {
    private String data;
    public Data(String data) {
        this.data = data;
    }
}
```

请求网络代码如下：

```java
new Retrofit.Builder()
        .baseUrl("http://101.132.64.249/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()
        .create(IpServerPost.class)
        .getInfoBody(new Data("Petterp_Post")) //1
        .enqueue(new Callback<Info>() {
            @Override
            public void onResponse(Call<Info> call, Response<Info> response) {
                Toast.makeText(MainActivity.this, response.body().getData(), Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onFailure(Call<Info> call, Throwable t) {

            }
        });
```



### 文件上传

#### 多文件上传：@Part

接口写法

```java
@Multipart
@POST
Call<String> upload(@Url String url, @Part List<MultipartBody.Part> partList);
```

网络请求代码(简单工具)

**接口：**

```java
public interface Iupload {
    void onResponse(Response res);
    void onFailure(Throwable t);
}
```

**工具类：**

```java
/**
 * @author Petterp on 2019/7/10
 * Summary:文件上传工具类(支持多文件)
 * 邮箱：1509492795@qq.com
 */
public class RetrofitUtils {
    private static OkHttpClient client;
    private static IpServerPost ipServerPost;
    private static String API="http://101.132.64.249";
    private static final ArrayList<MultipartBody.Part> BODYS = new ArrayList<>();

    public static RetrofitUtils build() {
        BODYS.clear();
        return Client.RETROFIT_UTILS;
    }

    private RetrofitUtils() {

    }


    private static class Client {
        private static final RetrofitUtils RETROFIT_UTILS = new RetrofitUtils();
    }

    private OkHttpClient getClient() {
        if (client == null) {
            client = new OkHttpClient.Builder()
                    .connectTimeout(20, TimeUnit.SECONDS)
                    .readTimeout(20, TimeUnit.SECONDS)
                    .writeTimeout(20, TimeUnit.SECONDS)
                    //允许失败重试
                    .retryOnConnectionFailure(true)
                    .build();
        }
        return client;
    }

    private IpServerPost getRetrofit() {
        if (ipServerPost == null) {
            ipServerPost = new Retrofit.Builder()
                    .baseUrl(RetrofitUtils.API)
                    .addConverterFactory(ScalarsConverterFactory.create())
                    .client(getClient())
                    .build()
                    .create(IpServerPost.class);
        }
        return ipServerPost;
    }

    /**
     * 添加文件
     * @param file
     * @param name
     * @return
     */
    public RetrofitUtils add(File file, String name) {
        final RequestBody requestBody = RequestBody.create(MediaType.parse(MultipartBody.FORM.toString()), file);
        final MultipartBody.Part body = MultipartBody.Part.createFormData(name, file.getName(), requestBody);
        BODYS.add(body);
        return this;
    }

    public RetrofitUtils add(String url, String name) {
        File file=new File(url);
        final RequestBody requestBody = RequestBody.create(MediaType.parse(MultipartBody.FORM.toString()), file);
        final MultipartBody.Part body = MultipartBody.Part.createFormData(name, file.getName(), requestBody);
        BODYS.add(body);
        return this;
    }

    /**
     * 设置api,可以直接设为默认值
     * @param api
     * @return
     */
    public RetrofitUtils setHost(String api){
        RetrofitUtils.API=api;
        ipServerPost=null;
        return this;
    }

    /**
     * 获取返回值
     * @param url
     * @param iupload
     */
    public void success(String url, Iupload iupload) {
        getRetrofit().upload(url, BODYS)
                .enqueue(new Callback<String>() {
                    @Override
                    public void onResponse(Call<String> call, Response<String> response) {
                        iupload.onResponse(response);
                    }

                    @Override
                    public void onFailure(Call<String> call, Throwable t) {
                        iupload.onFailure(t);
                    }
                });
    }
}
```

**实际使用**

```java
File file = new File(Environment.getExternalStorageDirectory() + "/" + "test.text");
        RetrofitUtils
                .build()
//                .setHost("http://101.132.64.249") 如果不写改为默认即可
                .add(file,"name1")
                .add(file,"name1")
                .success("API/post1.php", new Iupload() {
                    @Override
                    public void onResponse(Response res) {
                        Log.e("demo", String.valueOf(res.body()));
                    }

                    @Override
                    public void onFailure(Throwable t) {

                    }
                });
```

记得导入

```java
api 'com.squareup.retrofit2:converter-scalars:2.5.0'
```



