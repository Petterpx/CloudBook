# Android7.0自带的DownloadManager下载及后续的文件打开相关。

xml添加相应，Android7.0提高了文件访问相关权限

```xml
<!--Android7.0 增加文件访问功能-->
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="com.petterp.record.FileProvider"
    android:exported="false"
    android:grantUriPermissions="true"
    tools:ignore="WrongManifestParent">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/rc_file_path" />
</provider>
```



下载流程：

```java
				//创建下载任务,downloadUrl就是下载链接
        DownloadManager.Request request = new DownloadManager.Request(Uri.parse("https://www.wenkuxiazai.com/word/455c1c35dd36a32d737581fd-1.doc"));
        //指定下载路径和下载文件名
        request.setDestinationInExternalPublicDir("/清锋网评/", "测.doc");
        //获取下载管理器
        DownloadManager downloadManager = (DownloadManager) Latte.getContext().getSystemService(Context.DOWNLOAD_SERVICE);
        request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_MOBILE | DownloadManager.Request.NETWORK_WIFI);
        request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);
        request.allowScanningByMediaScanner();
        // 设置为可见和可管理
        request.setVisibleInDownloadsUi(true);
        //将下载任务加入下载队列，否则不会进行下载
        long enqueue = downloadManager.enqueue(request);
        LoadingDialog dialog = new LoadingDialog();
        dialog.setLoadingInformation("正在下载");
        dialog.show(view.delegate().getFragmentManager(), "show");

			//注册广播接收器
        IntentFilter filter = new IntentFilter(DownloadManager.ACTION_DOWNLOAD_COMPLETE);
        BroadcastReceiver receiver = new BroadcastReceiver() {
            public void onReceive(Context context, Intent intent) {
                long myDwonloadID = intent.getLongExtra(DownloadManager.EXTRA_DOWNLOAD_ID, -1);
                //判断下载Id是否是刚下载的
                if (enqueue == myDwonloadID) {
                  
                  //具体的文件下载后的处理方式
                    //转为uri
                    Uri fileUri = downloadManager.getUriForDownloadedFile(myDwonloadID);
                    FileIntentUtils.openFileEx(FileUtil.getRealFilePath(view.delegate().getActivity(),fileUri), "doc", view.delegate().getActivity());
                    LatteLogger.e("demo","当前下载的目录"+fileUri);
                    LatteLogger.e("demo","真实目录"+FileUtil.getRealFilePath(view.delegate().getActivity(),fileUri));
                    dialog.dismiss();
                    //删除文件
//                    FileIntentUtils.deleteUri(view.delegate().getActivity(),fileUri);
                }


            }
        };
			//绑定广播接收器
        view.delegate().getActivity().registerReceiver(receiver, filter);


//注意最后的解绑
view.delegate().getActivity().unregisterReceiver(receiver);
```





参考链接https://blog.csdn.net/yan822/article/details/80508938

https://blog.csdn.net/u012527802/article/details/48464803/