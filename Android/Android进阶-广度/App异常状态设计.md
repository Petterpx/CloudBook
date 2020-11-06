# App异常状态设计



### 背景

一个用户体验良好的App一般都包含相应的状态提示页，为了适应更快的业务迭代，App端必须统一相应的错误提示页。



### 解决方案

Android 端技术层实现一键加载指定错误页，对于不同项目，可自由配置,具体为依赖库产出方式。

ios待考虑。todo

### 涉及到的错误提示

- 网络相关
  - 网络断开
  - 网络异常 (指网络无法正常使用或者网络很慢)
  - 涉及大量流量操作时(用户此时非wifi环境)
- 常规异常
  - 服务器异常时
  - 空数据状态
  - 页面加载失败时
  - 内容已被删除时，用户点击进入的提示。(常见资讯类，某资讯已被删除)
- 用户行为异常
  - 操作失败 (某个行为操作失败时)
  - 搜索无结果时
  - 无权限操作某行为时
- WebView相关
  - 加载中
  - 加载失败

更多待添加...

#### 网络相关

|                    弱网或者网络无法访问-1                    |                    弱网或者网络无法访问-2                    |                          网络断开时                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| <img src="https://tva1.sinaimg.cn/large/0081Kckwly1gkc2r7opxrj30u01rcn01.jpg" width="300" height="600" /> | <img src="https://tva1.sinaimg.cn/large/0081Kckwly1gkc2rg3es5j30u01rctba.jpg" width="300" height="600" /> | <img src="https://tva1.sinaimg.cn/large/0081Kckwly1gkc2qv0m6nj30u01rcwhj.jpg" width="300" height="600" /> |

| 大流量操作时                                                 |      |
| ------------------------------------------------------------ | ---- |
| <img src="https://tva1.sinaimg.cn/large/0081Kckwly1gkc49q42lrj30ty0q4150.jpg" width="533" height="533" /> |      |



#### 常规异常

| 服务器异常                                                   | 空数据状态处理                                               | 页面加载失败 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ |
| <img src="https://tva1.sinaimg.cn/large/0081Kckwly1gkc4mv0ngqj30oy174afd.jpg" width="300" height="600" /> | <img src="https://tva1.sinaimg.cn/large/0081Kckwly1gkc4rqztagj30e00o8ali.jpg" width="300" height="600" /> |              |
| **内容被删除**                                               |                                                              |              |
| <img src="https://tva1.sinaimg.cn/large/0081Kckwly1gkc4toslktj30qg19ygnm.jpg" width="300" height="600" /> |                                                              |              |

