# JetPack Paging3上手指南

官方代码地址

```
https://codelabs.developers.google.com/codelabs/android-paging
```



首先导入依赖：

```groovy
//paging
implementation "androidx.paging:paging-runtime:3.0.0-alpha03"

implementation "androidx.recyclerview:recyclerview:1.2.0-alpha05"

//kt全家桶
implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.72"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.7"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.8"
//ktx扩展
implementation "androidx.core:core-ktx:1.3.1"
implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0"
implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"



implementation 'com.github.people-rmxc:LiveHttp:1.1.5'
implementation 'com.squareup.retrofit2:retrofit:2.7.2'
implementation 'com.squareup.retrofit2:converter-gson:2.7.2'
implementation "com.squareup.okhttp3:logging-interceptor:4.3.0"
```





使用建议

Kotlin推荐使用协程，Java推荐使用LiveData

- **`PagingData`** - 用于存储分页数据的容器。每次数据刷新都会有一个相应的单独 `PagingData`。
- **`PagingSource`** - `PagingSource` 是用于将数据快照加载到 `PagingData` 流的基类。
- **`Pager.flow`** - 根据 `PagingConfig` 和一个定义如何构造实现的 `PagingSource` 的构造函数，构建一个 `Flow<PagingData>`。
- **`PagingDataAdapter`** - 一个用于在 `RecyclerView` 中呈现 `PagingData` 的 `RecyclerView.Adapter`。`PagingDataAdapter` 可以连接到 Kotlin `Flow`、`LiveData`、RxJava `Flowable` 或 RxJava `Observable`。`PagingDataAdapter` 会在页面加载时监听内部 `PagingData` 加载事件，并于以新对象 `PagingData` 的形式收到更新后的内容时，在后台线程中使用 `DiffUtil` 计算细粒度更新。
- `RemoteMediator` - 帮助接收来自网络和数据库的数据，实现分页。

## 开始使用

### 创建PagingSource

```kotlin
private const val GITHUB_STARTING_PAGE_INDEX = 1

class GitHubPagingSource(private val service: GithubService) : PagingSource<Int, Repo>() {
		var query: String = ""
    //调用此函数触发异步加载，LoadParams对象保留与装入操作有关的信息
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Repo> {
        //如果是第一次调用 params.key==null
        val key = params.key

        // LoadResult.Page 成功
        // LoadResult.Error 错误

        val position = params.key ?: GITHUB_STARTING_PAGE_INDEX
        val apiQuery = query + IN_QUALIFIER

        return try {
            val response = service.searchRepos(apiQuery, position, params.loadSize)
            val repos = response.items
            LoadResult.Page(
                    data = repos,
              			//理解为上一页
                    prevKey = if (position == GITHUB_STARTING_PAGE_INDEX) null else position - 1,
              			//理解为下一页
                    nextKey = if (repos.isEmpty()) null else position + 1
            )
        } catch (e: Exception) {
            return LoadResult.Error(e)
        }
    }
}
```

load函数用于 触发异步加载，该 LoadParams对象保留与写入有关的信息。

**LoadResult**

- LoadResult.Page   数据加载成功
- LoadResult.Error   数据加载失败

当构建`LoadResult.Page`，通过`null`对`nextKey`或者`prevKey`如果列表不能在相应的方向被加载。例如，在我们的例子中，我们可以考虑，如果网络响应成功但列表是空的，我们没有任何剩下的数据要加载；所以`nextKey`可以`null`。



### 配置PagingData

要构建PagingData，首先要确定使用 什么方式来传递 PagingData 给其他层。

- Kotlin `Flow`使用`Pager.flow`。
- `LiveData`-使用`Pager.liveData`。
- RxJava- `Flowable`使用`Pager.flowable`。
- RxJava- `Observable`使用`Pager.observable`。

**注意：**

无论PagingData 选择哪个生成器，都必须传递以下参数。

- **`PagingConfig`**。此类设置有关如何从中加载内容的选项，`PagingSource`例如，要加载多远，初始加载的大小请求等。您必须定义的唯一必需参数是页面大小，即每个页面中应加载多少个项目。默认情况下，“分页”会将所有加载的页面保留在内存中。为确保您不会在用户滚动时浪费内存，请在中设置`maxSize`参数`PagingConfig`。默认情况下，如果Paging可以计算已卸载的项目并且`enablePlaceholders`config标志为true ，则Paging将返回空项目作为占位符，以用于尚未加载的内容。这样，您将能够在适配器中显示占位符视图。为了简化此代码实验室中的工作，让我们通过传递禁用占位符`enablePlaceholders = false`。

#### 创建你的数据源

```kotlin
@ExperimentalCoroutinesApi
class GithubRepository(private val service: GithubService) {
    //搜索数据
    fun getSearchResultStream(query: String): Flow<PagingData<Repo>> {
        pagingSource.query = query
        return pager.flow
    }

    companion object {
        private const val NETWORK_PAGE_SIZE = 50
    }

   private val pager by lazy {
        Pager(
                config = PagingConfig(
                        pageSize = "数据长度",
                        enablePlaceholders = false
                ),
                pagingSourceFactory = {
                    pagingSource
                }
        )
    }

    private  val pagingSource by lazy {
        GitHubPagingSource(service)
    }
}
```





### 在ViewModel中请求并缓存PagingData

我们现在会保存一个私有 PagingData数据，作为配置更改后的数据重建，相应的viewModel如下

`Flow`有一个方便的`cachedIn()`方法，使我们可以将内容缓存`Flow`在`CoroutineScope`。由于我们位于中`ViewModel`，我们将使用[`androidx.lifecycle.viewModelScope`](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary#(androidx.lifecycle.ViewModel).viewModelScope:kotlinx.coroutines.CoroutineScope)。

```kotlin
class SearchRepositoriesViewModel(private val repository: GithubRepository) : ViewModel() {

    private var currentQueryValue: String? = null

    private var currentSearchResult: Flow<PagingData<Repo>>? = null

    fun searchRepo(queryString: String): Flow<PagingData<Repo>> {
        val lastResult = currentSearchResult
      	//这里做了数据的回复
        if (queryString == currentQueryValue && lastResult != null) {
            return lastResult
        }
        currentQueryValue = queryString
        val newResult: Flow<PagingData<Repo>> = repository.getSearchResultStream(queryString)
                .cachedIn(viewModelScope)
        currentSearchResult = newResult
        return newResult
    }
}
```

**注意：**如果您要对`Flow`，`map`或进行任何操作`filter`，请确保`cachedIn`在执行这些操作后调用，以确保无需再次触发它们。



### 创建Adapter

要将 paginAdapter 绑定到RecyclerView，我们需要使用 PagingDataAdapter.该PagingDataAdapter 会通知PagingData的内容已改变。

```kotlin
class ReposAdapter : PagingDataAdapter<UiModel, RecyclerView.ViewHolder>(REPO_COMPARATOR) {
		
  	//创建viewHolder
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return if (viewType == R.layout.repo_view_item) {
            RepoViewHolder.create(parent)
        } else {
            SeparatorViewHolder.create(parent)
        }
    }

    //绑定不同的Bing
    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        val uiModel = getItem(position)
        uiModel?.let {
            when (uiModel) {
                is UiModel.RepoItem -> (holder as RepoViewHolder).bind(uiModel.repo)
                is UiModel.SeparatorItem -> (holder as SeparatorViewHolder).bind(uiModel.description)
            }
        }
    }

    override fun getItemViewType(position: Int): Int {
        return when (getItem(position)) {
            is UiModel.RepoItem -> R.layout.repo_view_item
            is UiModel.SeparatorItem -> R.layout.separtor_viewe_item
            null -> throw UnsupportedOperationException("Unknown View")
        }
    }

    companion object {
        private val REPO_COMPARATOR = object : DiffUtil.ItemCallback<UiModel>() {
            override fun areItemsTheSame(oldItem: UiModel, newItem: UiModel): Boolean =
                    (oldItem is UiModel.RepoItem && newItem is UiModel.RepoItem &&
                            oldItem.repo.fullName == newItem.repo.fullName) ||
                            (oldItem is UiModel.SeparatorItem && newItem is UiModel.SeparatorItem &&
                                    oldItem.description == newItem.description)

            override fun areContentsTheSame(oldItem: UiModel, newItem: UiModel): Boolean =
                    oldItem == newItem
        }
    }
}

```



### 更改Activity触发网络更新

```kotlin
 private var searchJob: Job? = null
    private fun search(query: String) {
        searchJob?.cancel()
        searchJob = lifecycleScope.launch {
          	//借用协程实现
            viewModel.searchRepo(query).collectLatest {
                adapter.submitData(it)
            }
        }
    }
```

#### 获取加载状态CombinedLoadStates

我们可以使用PagingDataAdapter.loadStateFlow，Flow每次监测到改变时， **CombindedLoadStates** 对象就会发出加载通知。

CombindLoadStates 允许我们获取三种不同类型的加载状态

- `CombinedLoadStates.refresh`-表示`PagingData`第一次加载的加载状态。
- `CombinedLoadStates.prepend` -表示在列表开头加载数据的加载状态。
- `CombinedLoadStates.append` -表示在列表末尾加载数据的加载状态。

在我们这个例子中，如果我们要初始化搜索，我们需要在每个流的发射处，执行以下操作。

```kotlin
private fun initSearch(query: String) {
    ...
        lifecycleScope.launch {
            adapter.loadStateFlow
                   //只有在刷新时处理
                   .distinctUntilChangedBy { it.refresh }
                   // Only react to cases where REFRESH completes i.e., NotLoading.
                   .filter { it.refresh is LoadState.NotLoading }
                   .collect { binding.list.scrollToPosition(0) }
        }
}
```



### 为Paging加入header和footer

借助于 LoadStateAdapte，我们可以轻松实现上述需求。需要注意的是，对于重试机制，我们使用 **adapter.retry()**。 在后台，此方法最终调用 PagingSource 以实现数据的正确加载。

与任何列表一样，我们需要创建如下几个文件才可以使用：

- 相应的header||footer布局
- 相应的ViewHolder ，继承自RecyclerView.ViewHolder
- LoadStateAdapter,继承自 LoadStateAdapter <ViewHolder>



#### 创建ViewHolder

```kotlin
class ReposLoadStateViewHolder(
        private val binding: ReposLoadStateFooterViewItemBinding,
        retry: () -> Unit
) : RecyclerView.ViewHolder(binding.root) {

    init {
        binding.retryButton.setOnClickListener { retry.invoke() }
    }

    fun bind(loadState: LoadState) {
        if (loadState is LoadState.Error) {
            binding.errorMsg.text = loadState.error.localizedMessage
        }
        binding.progressBar.isVisible = loadState is LoadState.Loading
        binding.retryButton.isVisible = loadState !is LoadState.Loading
        binding.errorMsg.isVisible = loadState !is LoadState.Loading
    }

    companion object {
        fun create(parent: ViewGroup, retry: () -> Unit): ReposLoadStateViewHolder {
            val view = LayoutInflater.from(parent.context)
                    .inflate(R.layout.repos_load_state_footer_view_item, parent, false)
            val binding = ReposLoadStateFooterViewItemBinding.bind(view)
            return ReposLoadStateViewHolder(binding, retry)
        }
    }
}
```



#### 创建LoadStateAdapter/即创建footer适配器

与其他方法Adapter一样，我们同样也需要实现 **onBind()** 和 **onCreate()** 

- 在onBindViewHoldeer() 中绑定ViewHolder
- 在onCreateViewHolder() 中定义如何基于父项的ViewGroup和重试函数。

```
class ReposLoadStateAdapter(private val retry: () -> Unit) : LoadStateAdapter<ReposLoadStateViewHolder>() {
    override fun onBindViewHolder(holder: ReposLoadStateViewHolder, loadState: LoadState) {
        holder.bind(loadState)
    }

    override fun onCreateViewHolder(parent: ViewGroup, loadState: LoadState): ReposLoadStateViewHolder {
        return ReposLoadStateViewHolder.create(parent, retry)
    }
}

```



#### 将load-Adapter与列表进行绑定

PagingDataAdapter支持三种方式进行绑定

- **withLoadStateHeader** 绑定在header
- **withLoadStateFooteer**  绑定在footer时
- **withLoadStateHeadrAndFooteere**  同时绑定heade和footer

```kotlin
private fun initAdapter() {
    binding.list.adapter = adapter.withLoadStateHeaderAndFooter(
            header = ReposLoadStateAdapter { adapter.retry() },
            footer = ReposLoadStateAdapter { adapter.retry() }
    )
}
```



### 监听Paging的加载进度

#### CombinedLoadStates

通过 **addLoadStateListener**

```kotlin
 adapter.addLoadStateListener { loadState ->
                               //是否刷新成功，但是无数据
        binding.list.isVisible = loadState.source.refresh is LoadState.NotLoading
                               //是否是否正在加载
        binding.progressBar.isVisible = loadState.source.refresh is LoadState.Loading
                              //加载是否报错
        binding.retryButton.isVisible = loadState.source.refresh is LoadState.Error
        val errorState = loadState.source.append as? LoadState.Error
                ?: loadState.source.prepend as? LoadState.Error
                ?: loadState.append as? LoadState.Error
                ?: loadState.prepend as? LoadState.Error
        errorState?.let {
            //处理报错
        }
    }
```
