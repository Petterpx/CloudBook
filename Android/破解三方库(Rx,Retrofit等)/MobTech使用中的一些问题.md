# MobTech使用中的一些问题

#### App secret should not be null

> 如果检查确实添加了 secret ,还是出现了这个问题，请按照以下步骤解决：
>
>  **在您的manifest加下这个试试**
>
> ```xml
>   <meta-data
>             android:name="Mob-AppKey"
>             android:value="您的appkey" />
> 
>         <meta-data
>             android:name="Mob-AppSecret"
>             android:value="您的appsecret" />
> ```

