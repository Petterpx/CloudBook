  

# 源码分析 | xml解析过程

<img src="https://tva1.sinaimg.cn/large/0081Kckwly1glkxlmft7pj30v60sowgr.jpg" alt="image-20201212115325384" style="zoom:33%;" />

1. 制作皮肤包

2. 收集 xml 数据

   - 利用View生产对象的过程中的 Factory2 接口 , 从而得到View中的信息

   

   

   

   记录需要换肤的属性   ( **Factory2** 中生产View的时候)

   - SkinAttribute -> List<SkinView>(存放所有View) -> List<SkinPair>（存放View属性合集）

<img src="/Users/petterp/Library/Application Support/typora-user-images/image-20201212121335686.png" alt="image-20201212121335686" style="zoom:33%;" />



3. 读取皮肤包内容  并加载进来  asset,addAssetPath(xxx)
4. 执行换肤   对我所有记录的属性进行重新设置