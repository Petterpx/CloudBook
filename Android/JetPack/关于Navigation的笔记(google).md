# 关于Navigation的笔记

### popUpTo 和 popUpToInclusive

使用操作进行导航时，您可以选择从返回堆栈上弹出其他目的地。例如，如果您的应用具有初始登录流程，那么在用户登录后，您应将所有与登录相关的目的地从返回堆栈上弹出，这样返回按钮就不会将用户带回登录流程。

要在从一个目的地导航到另一个目的地时弹出目的地，请在关联的 `` 元素中添加 `app:popUpTo` 属性。`app:popUpTo` 会告知 Navigation 库在调用 `navigate()` 的过程中从返回堆栈上弹出一些目的地。属性值是应保留在堆栈中的最新目的地的 ID。

您还可以添加 `app:popUpToInclusive="true"`，以表明在 `app:popUpTo` 中指定的目的地也应从返回堆栈中移除。

### popUpTo 示例：循环逻辑

假设您的应用包含三个目的地：A、B 和 C，以及从 A 到 B、从 B 到 C 再从 C 返回到 A 的操作。对应的导航图如图 1 所示：

![img](https://tva1.sinaimg.cn/large/007S8ZIlly1gg9jvl2tt8j30j50e0q30.jpg)

**图 1.** 包含三个目的地的循环导航图：A、B 和 C。

每执行一次导航操作，都会将一个目的地添加到返回堆栈。如果您要通过此流程反复导航，则您的返回堆栈会包含多个集合，其中包含每个目的地（例如 A、B、C、A、B、C、A 等）。为了避免这种重复，您可以在从目的地 C 到目的地 A 的操作中指定 `app:popUpTo` 和 `app:popUpToInclusive`，如下例所示：

```xml
    <fragment
        android:id="@+id/c"
        android:name="com.example.myapplication.C"
        android:label="fragment_c"
        tools:layout="@layout/fragment_c">

        <action
            android:id="@+id/action_c_to_a"
            app:destination="@id/a"
            app:popUpTo="@+id/a"
            app:popUpToInclusive="true"/>
    </fragment>
    
```

在到达目的地 C 之后，返回堆栈包含每个目的地（A、B、C）的实例。当返回到目的地 A 时，我们也 `popUpTo` A，也就是说我们会在导航过程中从堆栈中移除 B 和 C。利用 `app:popUpToInclusive="true"`，我们还会将第一个 A 从堆栈上弹出，从而有效地清除它。请注意，如果您不使用 `app:popUpToInclusive`，则返回堆栈会包含两个目的地 A 的实例。





# 为目的地创建深层链接

在 Android 中，深层链接是指将用户直接转到应用内特定目的地的链接。

借助 Navigation 组件，您可以创建两种不同类型的深层链接：显式深层链接和隐式深层链接。

## 创建显式深层链接

[显式深层链接](https://developer.android.google.cn/training/app-links/deep-linking)是深层链接的一个实例，该实例使用 [`PendingIntent`](https://developer.android.google.cn/reference/android/app/PendingIntent) 将用户转到应用内的特定位置。例如，您可以在通知、应用快捷方式或应用微件中显示显式深层链接。

当用户通过显式深层链接打开您的应用时，任务返回堆栈会被清除，并被替换为相应的深层链接目的地。当[嵌套图表](https://developer.android.google.cn/guide/navigation/navigation-nested-graphs)时，每个嵌套级别的起始目的地（即层次结构中每个 `` 元素的起始目的地）也会添加到相应堆栈中。也就是说，当用户从深层链接目的地按下返回按钮时，他们会返回到相应的导航堆栈，就像从入口点进入您的应用一样。

您可以使用 [`NavDeepLinkBuilder`](https://developer.android.google.cn/reference/androidx/navigation/NavDeepLinkBuilder) 类构建 [`PendingIntent`](https://developer.android.google.cn/reference/android/app/PendingIntent)，如下例所示。请注意，如果提供的上下文不是 `Activity`，构造函数会使用 [`PackageManager.getLaunchIntentForPackage()`](https://developer.android.google.cn/reference/android/content/pm/PackageManager#getLaunchIntentForPackage(java.lang.String)) 作为默认 Activity 来启动（如果有）。

[KOTLIN](https://developer.android.google.cn/guide/navigation/navigation-deep-link#kotlin-kotlin)[JAVA](https://developer.android.google.cn/guide/navigation/navigation-deep-link#java-java)

```kotlin
    val pendingIntent = NavDeepLinkBuilder(context)
        .setGraph(R.navigation.nav_graph)
        .setDestination(R.id.android)
        .setArguments(args)
        .createPendingIntent()
    
```

如果您已有 [`NavController`](https://developer.android.google.cn/reference/androidx/navigation/NavController)，则还可以通过 [`NavController.createDeepLink()`](https://developer.android.google.cn/reference/androidx/navigation/NavController#createDeepLink()) 创建深层链接。

# 创建隐式深层链接

[隐式深层链接](https://developer.android.google.cn/training/app-links/deep-linking)是指应用中特定目的地的 URI。调用 URI（例如，当用户点击某个链接）时，Android 可以将应用打开到相应的目的地。

在触发隐式深层链接时，返回堆栈的状态取决于是否使用 [`Intent.FLAG_ACTIVITY_NEW_TASK`](https://developer.android.google.cn/reference/android/content/Intent#FLAG_ACTIVITY_NEW_TASK) 标记启动隐式 `Intent`：

- 如果该标记已设置，则任务返回堆栈会被清除，并被替换为相应的深层链接目的地。就像[显式深层链接](https://developer.android.google.cn/guide/navigation/navigation-deep-link#explicit)，当[嵌套图表](https://developer.android.google.cn/guide/navigation/navigation-nested-graphs)时，每个嵌套级别的起始目的地（即层次结构中每个 `` 元素的起始目的地）也会添加到相应堆栈中。也就是说，当用户从深层链接目的地按下返回按钮时，他们会返回到相应的导航堆栈，就像从入口点进入您的应用一样。
- 如果该标记未设置，则您仍然位于上一个应用的任务堆栈中，该应用中的隐式深层链接已触发。在这种情况下，如果按下返回按钮，则您会返回到上一个应用；如果按下向上按钮，则会在您的导航图中的层次父级目的地上启动应用的任务。

您可以使用 Navigation Editor 创建指向某个目的地的隐式深层链接，如下所示：

1. 在 Navigation Editor 的 **Design** 标签中，选择深层链接的目的地。

2. 点击 **Attributes** 面板 Deep Links 部分中的 + 。

3. 在随后显示的 **Add Deep Link** 对话框中，输入一个 URI。

   请注意以下几点：

   - 没有架构的 URI 会被假定为 http 或 https。例如，`www.google.com` 同时与 `http://www.google.com` 和 `https://www.google.com` 匹配。
   - 形式为 `{placeholder_name}` 的占位符与一个或多个字符相匹配。例如，`http://www.example.com/users/{id}` 与 `http://www.example.com/users/4` 匹配。Navigation 组件通过将占位符名称与已定义的[参数](https://developer.android.google.cn/guide/navigation/navigation-pass-data#define_destination_arguments)（为深层链接目的地所定义）相匹配，尝试将占位符值解析为相应的类型。如果没有定义具有相同名称的参数，则对参数值使用默认的 `String` 类型。
   - 您可以使用 .* 通配符匹配 0 个或多个字符。

4. （可选）选中 **Auto Verify** 可要求 Google 验证您是相应 URI 的所有者。如需了解详情，请参阅[验证 Android 应用链接](https://developer.android.google.cn/training/app-links/verify-site-associations)。

5. 点击 **Add**。所选目的地上方会显示链接图标 ![img](https://tva1.sinaimg.cn/large/007S8ZIlly1gg9k8gisu2j300w00wwen.jpg)，以表示该目的地具有深层链接。

6. 点击 **Text** 标签，以切换到 XML 视图。已向相应目的地添加嵌套的 `` 元素：

   ```xml
   <deepLink app:uri="https://www.google.com" />
       
   ```

要启用隐式深层链接，您还必须向应用的 `manifest.xml` 文件中添加内容。将一个 `` 元素添加到指向现有导航图的 Activity，如下例所示：

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.example.myapplication">

        <application ... >

            <activity name=".MainActivity" ...>
                ...

                <nav-graph android:value="@navigation/nav_graph" />

                ...

            </activity>
        </application>
    </manifest>
    
```

构建项目时，Navigation 组件会将 `` 元素替换为生成的 `` 元素，以匹配导航图中的所有深层链接。

**注意**：`` 元素在 Android Studio 3.0 或 3.1 中不受支持。使用这些版本时，您必须改为手动添加 `intent-filter` 元素。如需了解详情，请参阅[创建指向应用内容的深层链接](https://developer.android.google.cn/training/app-links/deep-linking)。





# 添加 Activity 目的地

当您应用中的每个屏幕都已连接起来以使用 Navigation 组件，并且您不再使用 `FragmentTransactions` 在基于 Fragment 的目的地之间转换后，下一步就是消除 `startActivity` 调用。

首先，确定您在应用中的哪些位置设置两个单独的导航图并使用 `startActivity` 在两者之间进行转换。

此示例包含两个图（A 和 B）以及从 A 转换到 B 的 `startActivity()` 调用。

![img](https://tva1.sinaimg.cn/large/007S8ZIlly1ggjks7tbapj31jj0jkq7m.jpg)

```kotlin
    fun navigateToProductDetails(productId: String) {
        val intent = Intent(this, ProductDetailsActivity::class.java)
        intent.putExtra(KEY_PRODUCT_ID, productId)
        startActivity(intent)
    }
    
```

接着，在图 A 中，使用表示导航到图 B 的托管 Activity 的 Activity 目的地替换上面的内容。如果您有要传递到图 B 的起始目的地的参数，您可以在 Activity 目的地定义中指定这些参数。

在以下示例中，图 A 定义 Activity 目的地，而后者具有 `product_id` 参数和操作。图 B 不包含任何更改。

![img](https://tva1.sinaimg.cn/large/007S8ZIlly1ggjks79iygj31jj0h5438.jpg)

图 A 和图 B 的 XML 表示形式可能如下所示：

```xml
<!-- Graph A -->
    <navigation xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/product_list_graph"
        app:startDestination="@id/product_list">

        <fragment
            android:id="@+id/product_list"
            android:name="com.example.android.persistence.ui.ProductListFragment"
            android:label="Product List"
            tools:layout="@layout/product_list_fragment">
            <action
                android:id="@+id/navigate_to_product_detail"
                app:destination="@id/product_details_activity" />
        </fragment>

        <activity
            android:id="@+id/product_details_activity"
            android:name="com.example.android.persistence.ui.ProductDetailsActivity"
            android:label="Product Details"
            tools:layout="@layout/product_details_host">

            <argument
                android:name="product_id"
                app:argType="integer" />

        </activity>

    </navigation>
    
<!-- Graph B -->
    <navigation xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        app:startDestination="@id/product_details">

        <fragment
            android:id="@+id/product_details"
            android:name="com.example.android.persistence.ui.ProductDetailsFragment"
            android:label="Product Details"
            tools:layout="@layout/product_details_fragment">
            <argument
                android:name="product_id"
                app:argType="integer" />
        </fragment>

    </navigation>
    
```

您可以使用转到 Fragment 目的地所用的机制转到图 B 的托管 Activity：



```kotlin
    fun navigateToProductDetails(productId: String) {
        val directions = ProductListDirections.navigateToProductDetail(productId)
        findNavController().navigate(directions)
    }
    
```

### 将 Activity 目的地 args 传递到起始目的地 Fragment

如果目的地 Activity 接收 extra（如上例所示），您可以将这些 extra 作为参数直接传递到起始目的地，但您需要在托管 Activity 的 `onCreate()` 方法中手动设置宿主的导航图，以便可以将 Intent extra 作为参数传递到 Fragment，如下所示：



```kotlin
    class ProductDetailsActivity : AppCompatActivity() {

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.product_details_host)
            findNavController(R.id.main_content)
                    .setGraph(R.navigation.product_detail_graph, intent.extras)
        }

    }
    
```

**注意**：在这种情况下，您应该避免在 `NavHostFragment` 定义中设置 `app:NavGraph` 属性，因为这样做会导致再次扩充和设置导航图。

您可以使用生成的 args 类从 Fragment 参数 `Bundle` 中提取数据，如下例所示：



```kotlin
    class ProductDetailsFragment : Fragment() {

        val args by navArgs<ProductDetailsArgs>()

        override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
            val productId = args.productId
            ...
        }
        ...
    
```

## 合并 Activity

如果出现多个 Activity 共用同一个布局（例如包含单个 Fragment 的简单 `FrameLayout`）的情况，您可以合并导航图。在大多数情况下，您只能合并每个导航图中的所有元素，并将任何 Activity 目的地元素更新为 Fragment 目的地。

以下示例将合并上一部分的图 A 和图 B：

**合并前**：

```xml
<!-- Graph A -->
    <navigation xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/product_list_graph"
        app:startDestination="@id/product_list">

        <fragment
            android:id="@+id/product_list"
            android:name="com.example.android.persistence.ui.ProductListFragment"
            android:label="Product List Fragment"
            tools:layout="@layout/product_list">
            <action
                android:id="@+id/navigate_to_product_detail"
                app:destination="@id/product_details_activity" />
        </fragment>
        <activity
            android:id="@+id/product_details_activity"
            android:name="com.example.android.persistence.ui.ProductDetailsActivity"
            android:label="Product Details Host"
            tools:layout="@layout/product_details_host">
            <argument android:name="product_id"
                app:argType="integer" />
        </activity>

    </navigation>
    
<!-- Graph B -->
    <navigation xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/product_detail_graph"
        app:startDestination="@id/product_details">

        <fragment
            android:id="@+id/product_details"
            android:name="com.example.android.persistence.ui.ProductDetailsFragment"
            android:label="Product Details"
            tools:layout="@layout/product_details">
            <argument
                android:name="product_id"
                app:argType="integer" />
        </fragment>
    </navigation>
    
```

**合并后**：

```xml
<!-- Combined Graph A and B -->
    <navigation xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/product_list_graph"
        app:startDestination="@id/product_list">

        <fragment
            android:id="@+id/product_list"
            android:name="com.example.android.persistence.ui.ProductListFragment"
            android:label="Product List Fragment"
            tools:layout="@layout/product_list">
            <action
                android:id="@+id/navigate_to_product_detail"
                app:destination="@id/product_details" />
        </fragment>

        <fragment
            android:id="@+id/product_details"
            android:name="com.example.android.persistence.ui.ProductDetailsFragment"
            android:label="Product Details"
            tools:layout="@layout/product_details">
            <argument
                android:name="product_id"
                app:argType="integer" />
        </fragment>

    </navigation>

    
```

如果合并时保持操作名称不变，可确保该过程无缝完成，并且无需更改现有代码库。例如，`navigateToProductDetail` 在此处保持不变。唯一的区别在于此操作现在表示转到同一 `NavHost` 内的 Fragment 目的地，而非 Activity 目的地：



```kotlin
fun navigateToProductDetails(productId: String) {   
  val directions = ProductListDirections.navigateToProductDetail(productId)    		         findNavController().navigate(directions) 
                                                }  
```

## 其他资源

要详细了解 Navigation，请参阅下面列出的其他资源：



### 示例

- [Android 架构组件导航基本示例](https://github.com/googlesamples/android-architecture-components/tree/master/NavigationBasicSample)





### Codelab

- [Navigation](https://codelabs.developers.google.com/codelabs/android-navigation/index.html?index=..%2F..%2Findex#0)





### 视频

- [Android Jetpack：使用导航控制器管理界面导航（2018 年 Google I/O 大会）](https://www.youtube.com/watch?v=8GCXtCjtg40)
