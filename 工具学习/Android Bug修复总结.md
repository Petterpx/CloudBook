# Android Bug修复总结



#### 避免使用 Random random=new Random

```java
//推荐使用
//原因 暂时todo
Random rand = SecureRandom.getInstanceStrong()
```



#### InterruptedException的异常不该忽略

错误示例

```java
  try {
                        sendMessage();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
```

正确示例

```java
 try {
                        sendMessage();
                    } catch (InterruptedException e) {
                        Log.e("FILE", e.getMessage());
                        Thread.currentThread().interrupt();
                    }
```



http://http://47.93.207.151//Shiyihui/comments-teach-cass.git