# 融云SDK集成过程中的一些问题  

```
.appendPath("conversation").appendPath(ConversationType.)
```

首先：融云SDK官方知识库，https://www.rongcloud.cn/docs/android.html#base_functions



### 单聊功能：

```java
/**
 * 启动单聊界面。
 *
 * @param context      应用上下文。
 * @param targetUserId 要与之聊天的用户 Id。
 * @param title        聊天的标题，开发者需要在聊天界面通过 intent.getData().getQueryParameter("title")
 *                     获取该值, 再手动设置为聊天界面的标题。
 */
RongIM.getInstance().startPrivateChat(getActivity(), "9527", "标题");
```

需要注意的是，在Activity或者您需要嵌套在Fragment中时。以下的设置，决定了对话列表的title显示，虽然看起来这里tagetId,但其实这里面传递的是 title 的值，所以需要注意。

```java
ConversationFragment fragement = (ConversationFragment) fragmentManage.findFragmentById(R.id.conversation);
Uri uri = Uri.parse("rong://" + getApplicationInfo().packageName).buildUpon()
        .appendPath("conversation").appendPath(Conversation.ConversationType.PRIVATE.getName().toLowerCase())
        .appendQueryParameter("targetId", title).build();
fragement.setUri(uri);
```





```
app:RCShape="circle"
android:scaleType="centerCrop"
```