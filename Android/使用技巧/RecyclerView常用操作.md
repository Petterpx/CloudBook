# 关于RecyclerView常用操作，你肯定经常边搜边吐槽？

RecyclerView是原生开发使用最多的控件，其也算的上我们经常使用了，本篇就记录一下日常用到的一些常见操作，以免2022年了，浪费大家搜索时间。

### 刷新列表时，glide图片闪烁

看到这个问题，肯定有同学会说，简单啊，直接动画=null,够简单粗暴的，但是仔细想想，如果你现在是一个评论列表呢？对于一个没有动画效果的列表，体验无疑是差劲不少。而网上相关的人云亦云的复制粘贴法，说什么Glide缓存没刷新问题。

正确打开方式：

请使用 **notifyItemChanged(it.position, "payloads")**

然后在你的 Adapter里，重写 onBindViewHolder ，注意是三个参数的方法。

```kotlin
fun onBindViewHolder(holder: VH, position: Int, payloads: MutableList<Any>){
	 // payloads就是你上面notifyItemChanged时触发的刷新值，通过自定义的值去执行相关逻辑
}
```

### 滚动到指定位置

```kotlin
private fun toRvPostion(position: Int) {
        recyclerView.scrollToPosition(position)
        linearLayoutManager.scrollToPositionWithOffset(position, 0)
}
```



### 指定item是否占满一行

```kotlin
// spanCount 代表了列数，即一行摆放几个item
inline fun RecyclerView.gradLayoutManager(
    spanCount: Int,
    crossinline map: (ItemViewType) -> Int = {
      	//这里的意思是，对于这个type,其在一行中占用的条目数
        if (it.type == BaseQuickAdapter.LOAD_MORE_VIEW) spanCount else 1
    }
) {
    if (layoutManager == null) {
        val gradLayoutManager = GridLayoutManager(context, spanCount)
        layoutManager = gradLayoutManager
    }
    setGridLookup(map)
}

inline fun RecyclerView.setGridLookup(crossinline map: (ItemViewType) -> Int) {
    (layoutManager as? GridLayoutManager)
        ?: throw IllegalAccessException("layoutManager is not GridLayoutManager")
    (layoutManager as GridLayoutManager).spanSizeLookup =
        object : GridLayoutManager.SpanSizeLookup() {
            override fun getSpanSize(position: Int): Int {
                val viewType = adapter?.getItemViewType(position) ?: -1
                return map(ItemViewType(viewType))
            }
        }
}
```



### 为gradLayoutManager设置间距

```kotlin
fun RecyclerView.addGridItemPadding(
    hPadding: Int,
    vPadding: Int,
    leftPadding: Int,
    rightPadding: Int,
    topPadding: Int,
    bottomPadding: Int
) {
    val halfHPadding = hPadding / 2
    val halfVPadding = vPadding / 2
    setPadding(
        leftPadding - halfHPadding,
        topPadding - halfVPadding,
        rightPadding - halfHPadding,
        bottomPadding - halfVPadding
    )
    clipToPadding = false
    addItemDecoration(object : ItemDecoration() {
        override fun getItemOffsets(
            outRect: Rect,
            view: View,
            parent: RecyclerView,
            state: State
        ) {
            val manager = layoutManager as? GridLayoutManager ?: return
            val p = manager.getPosition(view)
            val itemType = manager.getItemViewType(view)
            if (itemType == BaseQuickAdapter.EMPTY_VIEW) return
            if (p == RecyclerView.NO_POSITION) return
            val spanCount: Int = manager.spanCount
            val itemSpanSize = manager.spanSizeLookup.getSpanSize(p)
            if (itemSpanSize != 1) {
                return
            }
            val spanIndex = manager.spanSizeLookup.getSpanIndex(p, spanCount)
            val spanGroupIndex = manager.spanSizeLookup.getSpanGroupIndex(p, spanCount)
            LCLogger.debug(
                "RecyclerView.addGridItemPadding",
                "position = $p,spanIndex = $spanIndex, spanGroupIndex = $spanGroupIndex,itemSpanSize = $itemSpanSize , spanCount=$spanCount, counts = ${manager.itemCount}"
            )
            when {
                // 第一列 || 最后一列
                spanIndex == 0 || spanIndex == spanCount - 1 -> {
                    when (spanGroupIndex) {
                        // 左上
                        0 -> {
                            outRect.set(halfHPadding, halfVPadding, halfHPadding, halfVPadding)
                        }
                        else -> {
                            outRect.set(halfHPadding, halfVPadding, halfHPadding, halfVPadding)
                        }
                    }
                }
                // 第一行（ 不包括第一个和最后一个
                spanGroupIndex == 0 && spanIndex != 0 && spanIndex != spanCount - 1 -> {
                    outRect.set(halfHPadding, halfVPadding, halfHPadding, halfVPadding)
                }
                // 中间的
                else -> {
                    outRect.set(halfHPadding, halfVPadding, halfHPadding, halfVPadding)
                }
            }
        }
    })
}
```



### 瀑布流Manager设置item占满一行

```kotlin
在onBindView中
val layoutParams=itemView.layoutParams
if (layoutParams is StaggeredGridLayoutManager.LayoutParams) {
       layoutParams.isFullSpan = true
}
```

