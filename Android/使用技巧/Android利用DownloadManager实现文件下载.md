# Android利用DownloadManager实现文件下载



Android中文件下载，app更新，我们一般利用的都是 Retrofit或者 Okhttp等实现，但其实Android 早在API 9 之后,就为我们提供了 DownLoadManager，这是Android提供的系统服务，通过这个服务下载文件，整个过程全部交给了系统负责，免去了我们别的操作。

下面我们就来实地演示一下操作。

测试api sdk28， Android Studio3.4  小米5s Plus

代码如下：

**//定义一个成功接口**

```java
public interface IDownloadlister {
    void success(Uri uri);
}
```

**工具类**，重要的代码我已经移动上来。

```java
/**
 * Created by Petterp
 * on 2019-10-26
 * Function: 文件下载工具类
 */
public class DownloadUtils {

  	  public void download() {
        IntentFilter filter = new IntentFilter(DownloadManager.ACTION_DOWNLOAD_COMPLETE);
        //创建下载任务，url即任务链接
        DownloadManager.Request request = new DownloadManager.Request(Uri.parse(url));
        //指定下载路径及文件名
        request.setDestinationInExternalPublicDir(FILE_URI, fileName);
        //获取下载管理器
        final DownloadManager downloadManager = (DownloadManager) context.getSystemService(Context.DOWNLOAD_SERVICE);
        //一些配置
        //允许移动网络与WIFI下载
        request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_MOBILE | DownloadManager.Request.NETWORK_WIFI);
        //是否在通知栏显示下载进度
        request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);

        //设置可见及可管理
        /*注意，Android Q之后不推荐使用*/
        request.setVisibleInDownloadsUi(true);

        //将任务加入下载队列
        assert downloadManager != null;
        final long id = downloadManager.enqueue(request);
        BroadcastReceiver receiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                //获取下载id
                long myDwonloadID = intent.getLongExtra(DownloadManager.EXTRA_DOWNLOAD_ID, -1);
                if (myDwonloadID == id) {
                    //获取下载uri
                    Uri uri = downloadManager.getUriForDownloadedFile(myDwonloadID);
                    lister.success(uri);
                }
            }
        };
        if (context instanceof Activity) {
            Activity activity = (Activity) context;
            activity.registerReceiver(receiver, filter);
        }
    }
  
  
    //测试url，下载链接
    private String url = "http://qn.yingyonghui.com/apk/653732" +
            "5/c1d876442e38f2555" +
            "d85c55a1d8e95b7?sign=a36530f5c08ffbb5d9e" +
            "53c2d50346eb7&t=5db45f8d&attname=c1d876442e" +
            "38f2555d85c55a1d8e95b7.apk";
    //加.好处是默认隐藏路径
    private final String FILE_URI = "/.测试路径/";
    private IDownloadlister lister = null;
  	//文件名
    private String fileName = "test";
  	//Context
    private Context context;

    public static DownloadUtils builder() {
        return new DownloadUtils();
    }

    public DownloadUtils setUrl(String url) {
        this.url = url;
        return this;
    }

    public DownloadUtils setLister(IDownloadlister lister) {
        this.lister = lister;
        return this;
    }


    public DownloadUtils setFileName(String fileName) {
        this.fileName = fileName;
        return this;
    }

    public DownloadUtils setContext(Context context) {
        this.context = context;
        return this;
    }
}
```

**使用时**，这里下载了个app

```java
DownloadUtils.builder()
        .setContext(this)
        .setLister(new IDownloadlister() {
            @Override
            public void success(Uri uri) {
               
                Intent install = new Intent(Intent.ACTION_VIEW);
                install.setDataAndType(uri, "application/vnd.android.package-archive");
                install.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                startActivity(install);
            }
        })
        .download();
```

最后记得给权限啊。

![image-20191026223824783](https://tva1.sinaimg.cn/large/006y8mN6ly1g8bzfzm0e3j30fc0sata7.jpg)

很简单吧，关于更多的操作，比如下载进度，DownloadManager并没有提供具体方法，不过我们可以通过定时获取已下载大小，然后计算相应的进度值。