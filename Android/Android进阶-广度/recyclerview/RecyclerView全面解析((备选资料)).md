# RecyclerView全面解析(备选资料)

> 为有限的屏幕提供大量数据的灵活View

ListView的局限

- 只有纵向列表的一种布局
- 没有支持动画的API
- 接口设计和系统不一致
  - setOnItemClickListener()
  - setOnItemLongClickListener()
  - setSelection()
- 没有强制实现ViewHolder
- 性能不如RecyclerView

RecyclerView的优势

- 支持多种布局样式
- 强制实现ViewHolder,性能更好
- 代码解耦



- LayoutManager 负责view的摆放
- Item Animator 动画
- Adapter 适配器模式





### ViewHolder是什么？

> 保存View引用的容器类

为什么要存在ViewHolder?

为了View的复用？ViewHolder的存在仅仅是为了缓存View中的id,避免重复的findViewById

 

- ViewHolder 和 itemView 是什么关系，一对一？一对多？多对一？ 
- ItemView与ViewHolder一一对应





### RecyclerView 缓存机制

![image-20210117114319853](https://tva1.sinaimg.cn/large/008eGmZEly1gmqjm6o3k0j31d90u04qp.jpg)

![image-20210117114625233](https://tva1.sinaimg.cn/large/008eGmZEly1gmqjpee6qqj31g60u0khl.jpg)

 scrap与cache通过 position来找缓存，即缓存一定存在；

RecycleViewPool 通过 viewType来找缓存，找到缓存后需要重新绑定；







### 一些有意思的方法

onViewAttchedToWindows

> 当View每次可见时调用



### RecyclerView性能优化策略

- 在onCreateViewHolder中设置点击监听

- LinearLayoutManager. setInitialPrefetchItemCount()

  > - 横向列表初次显示时可见的item个数
  > - 只有嵌套在内部的RecyclerView才会生效

- RecyclerView.setHasFixedSize()

  > <img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmqkmp1sb4j31770u0du0.jpg" alt="image-20210117121825148" style="zoom:33%;" />
  >
  > 如果adapter 的数据变化不会导致RecyclerView的大小变化 -> RecyclerView.setHasFixedSize(true)

- 多个RecyclerView共用 RecycledViewPool

  > ![image-20210117122548408](https://tva1.sinaimg.cn/large/008eGmZEly1gmqkudsa5cj32k20omhdt.jpg)

- DiffUtil

  > 异步差异计算

### 为什么 ItemDecoration 可以绘制分割线

