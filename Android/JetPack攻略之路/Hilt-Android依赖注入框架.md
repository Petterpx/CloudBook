# Hilt-依赖注入框架上手指南
![image-20200615204639816](https://user-gold-cdn.xitu.io/2020/6/15/172b804d22e840b1?w=1032&h=799&f=jpeg&s=49040)

Hilt 是Google 最新的依赖注入框架，其是基于Dagger研发，但它不同于Dagger。对于Android开发者来说，Hilt可以说专门为Android 打造，提供了一种将Dagger依赖项注入到Android应用程序的标准方法，而且创建了一组标准的组件和作用域，这些组件会自动集成到Android应用程序的各个生命周期中，以简化开发者的上手难度。



> 在学习本文之前，假定大家已经了解依赖注入是什么，如果没有了解过，可以先了解概念。Hilt 的目的是降低Android 开发者使用依赖注入框架的上手成本，但是基本的理念大家还是要明白。
>


<br/>


### 相应的一些注解如下：

- **@HiltAndroidApp**

  触发Hilt的代码生成，包括适用于应用程序的基类，可以使用依赖注入，应用程序容器是应用程序的父容器，这意味着其他容器可以访问其提供的依赖项。

- **@AndroidEntryPoint**

  其会创建一个依赖容器，该容器遵循Android类的生命周期

- **@Inject**

  用来注入的字段，**其类型不能为Private**

  如果要告诉 Hilt  如何提供相应类型的实例，需要将  **@Inject**  添加到要注入的类的构造函数中。

  **Hilt有关如何提供不同类型的实例的信息也称为**绑定**。**

- **@Install(xx)**

  Install 用来告诉 Hilt 这个模块会被安装到哪个组件上.
  

<br/>


## 组件(Compenent)

> Hitl默认有以下标准组件，只需要在类上增加 **[`@AndroidEntryPoint`](https://dagger.dev/hilt/android-entry-point.html)** 即可支持以下类的注入

| Compenent                       | **Injector for**                                             |
| ------------------------------- | ------------------------------------------------------------ |
| **`ApplicationComponent`**      | `Application`                                                |
| **`ActivityRetainedComponent`** | `ViewModel`（请参阅[JetPack-ViewModel扩展](https://developer.android.com/training/dependency-injection/hilt-jetpack)） |
| **`ActivityComponent`**         | `Activity`                                                   |
| **`FragmentComponent`**         | `Fragment`                                                   |
| **`ViewComponent`**             | `View`                                                       |
| **`ViewWithFragmentComponent`** | `View` 与 `@WithFragmentBindings`                            |
| **`ServiceComponent`**          | `Service`                                                    |

需要注意的是，Hilt仅支持扩展[`FragmentActivity`](https://developer.android.com/reference/androidx/fragment/app/FragmentActivity)（如[`AppCompatActivity`](https://developer.android.com/reference/kotlin/androidx/appcompat/app/AppCompatActivity)）的活动和扩展Jetpack库的片段`Fragment`，而不支持`Fragment`Android平台（现已弃用）的 片段 。


<br/>


## 组件(Compenent)的生命周期

- 它限制了在创建组件和生成组件范围绑定的生命周期
- 它指示合适可以使用成员注入的值。(例如：当@Inject 字段不为null时)

| Component                       | 作用范围                 | Created at                                                   | Destroyed at                                                 |
| ------------------------------- | ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`ApplicationComponent`**      | `@Singleton`             | `Application#onCreate()`                                     | `Application#onDestroy()`                                    |
| **`ActivityRetainedComponent`** | `@ActivityRetainedScope` | `Activity#onCreate()`[1](https://dagger.dev/hilt/components#fn:1) | `Activity#onDestroy()`[1](https://dagger.dev/hilt/components#fn:1) |
| **`ActivityComponent`**         | `@ActivityScoped`        | `Activity#onCreate()`                                        | `Activity#onDestroy()`                                       |
| **`FragmentComponent`**         | `@FragmentScoped`        | `Fragment#onAttach()`                                        | `Fragment#onDestroy()`                                       |
| **`ViewComponent`**             | `@ViewScoped`            | `View#super()`                                               | `View` destroyed                                             |
| **`ViewWithFragmentComponent`** | `@ViewScoped`            | `View#super()`                                               | `View` destroyed                                             |
| **`ServiceComponent`**          | `@ServiceScoped`         | `Service#onCreate()`                                         | `Service#onDestroy()`                                        |

> 默认情况下，所有的绑定都是无作用域，也就是说，每次绑定时，都会创建一个新的绑定实例；
>
> 但是，Dagger 允许绑定作用域到特定组件，如上表所示，在指定组件范围内，实例都只会创建一次，并且对该绑定的所有请求都将共享同一实例。

例如：

```kotlin
@Singletion
class TestCompenent @Inject constructor()
```

> 其中@Singleton 就代表 TestComponent 实例在整个app中是唯一的，当后续某个类想要注入其时，将共享这个实例。

<br/>

## 如何使用？

**先导入依赖**

```groovy
implementation 'com.google.dagger:hilt-android:2.28-alpha'
kapt 'com.google.dagger:hilt-android-compiler:2.28-alpha'
```

```groovy
classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
```

**相应的model下增加**

```groovy
apply plugin: 'dagger.hilt.android.plugin'
```

### 举个🌰：

> 我们有一个 NetDataSource的 远程数据类，然后我们可能需要在Activity中调用,代码如下
>
> ```kotlin
> 
> class NetDataSource{
>     fun test(){
>         println("我只是一个测试方法")   
>     }
> }
> ```
>
> ```kotlin
> class MainActivity : AppCompatActivity() {
>     lateinit var netDataSource: NetDataSource
>     override fun onCreate(savedInstanceState: Bundle?) {
>         ...
>         netDataSource = NetDataSource()
>     }
> }
> ```

<br/>

 这样用没有什么问题，我们大多数时候都是这样干的，当然在kt中也可以使用 by lazy，不过具体看你自己的场景了。但如何将上面的代码用Hilt 改造呢？
>
> 
>
> **改造后**
>
> ```kotlin
> class NetDataSource @Inject constructor(){
>     fun test(){
>         println("我只是一个测试方法")
>     }
> }
> ```
>
> ```kotlin
> @AndroidEntryPoint
> class MainActivity : AppCompatActivity() {
>     @Inject
>     lateinit var netDataSource: NetDataSource
>     override fun onCreate(savedInstanceState: Bundle?) {
>        	...
>         netDataSource.test()
>     }
> }
> ```

 这样就结束了吗，如果这样使用，那么就会直接报错，因为Hilt在代码生成时需要访问所有模块，所以必须使用 **@HiltAndroidApp** 标注你的基类Application.类似如下：
>
> ```kotlin
> @HiltAndroidApp
> class KtxApplication : Application()
> ```
>
> 同样，添加了@HiltAndroidApp 之后，你也可以在后续任意位置使用到  **@ApplicationContext** 


<br/><br/>


## @Module

**模块用于向 Hlit 添加绑定**，换句话说，是告诉 Hlit 如何提供不同类型的实例。

增加了@Module注解的类，其代表着相当与一个模块，并通过指定的组件来告诉在哪个容器中可以使用绑定安装。

> *对于Hilt可以注入的每个Android类，都有一个关联的 Hilt Component，例如，Application 容器与之关联 ApplicationComponent ,并且Fragmenet容器与关联的 FragmentComonent*，就是最开始我们讲的组件生命周期


### 举个🌰：

 ##### 创建一个模块：
>
> ```kotlin
> @InstallIn(ApplicationComponent::class)
> @Module
> object TestModule {
> 	
>   //每次都是新的实例
>   @Provides
>   fun bindBook():Book{
>     	return Book()
>   }
>   
>   //全局复用同一个实例
>   @Provides
>   @Singleton
>   fun bindBook():BookSingle{
>     	return Book()
>   }
> }
> ```
>
> Install 用来告诉hilt 这个模块会被安装到哪个组件上。

>**一个常见的误解是，模块中声明的所有绑定都将作用于安装该模块的组件。但是，事实并非如此。仅使用范围注释注释的绑定声明将被限制范围**。

<br/>

 那什么时候添加注入范围呢？
>
> **对绑定进行作用域限定会在生成的代码大小和其运行时性能上付出代价，因此请谨慎使用作用域。确定绑定是否应限制作用域的一般规则是，仅在代码正确性需要绑定作用域时才对绑定进行作用域。如果您认为绑定仅出于性能方面的考虑而作用域，请首先验证性能是否存在问题，然后考虑使用`@Reusable`而不是组件作用域。**

注意：在Kotlin中，仅包含@Provides函数的模块可以是object类。这样，提供程序就可以得到优化，并且几乎可以内联在生成的代码中。 

<br/>
<br/>



## 使用@Provides告诉Hilt如何获得具体实例

>用来告诉Hilt 如何提供不能被构造函数注入的类型

>每当 Hilt 需要提供该类型的实例时，将执行带注释的函数的函数主体。**@Provides** 常用于模块中

### 举个🌰：

##### room的常规用法

 我们使用room,有一个数据库表和相应的Dao
>
> ```kotlin
> @Entity(tableName = "book")
> class Book(val name: String) {
> 	 @PrimaryKey(autoGenerate = true)
>     var id: Long = 0
> }
> 
> @Dao
> interface BookDao {
>     @Insert
>     suspend fun insertAll(vararg books: Book)
> 
>     @Query("SELECT COUNT(*) FROM book")
>     suspend fun queryBookAll(): Int
> }
> ```

 接着，我们还有一个**AppDataBase** 和 一个单例的 **roomDatabase**
>
> ```kotlin
> @Database(entities = [Book::class], version = 1)
> abstract class AppDataBase : RoomDatabase() {
>     abstract fun bookDao(): BookDao
> }
> 
> object RoomSingle {
>     lateinit var context: Context
>     val roomDatabase by lazy {
>         Room.databaseBuilder(
>             context,
>             AppDataBase::class.java,
>             "logging.db"
>         ).build()
>     }
> }
> ```

 一般我们在使用时，都采用如下写法：
>
> ```kotlin
> class MainActivity : AppCompatActivity() {
>     val bookDao by lazy {
>         RoomSingle.roomDatabase.bookDao()
>     }
> }
> ```

#### **如何利用 Hilt改造呢？**

 我们创建一个BookModule，并使用 **@Model** 注明这是一个模块，**@InstallIn** 声明这个模块的生命范围为APP级别
>
> ```kotlin
> @InstallIn(ApplicationComponent::class)
> @Module
> object BookModule {
>  	@Provides
>     @Singleton
>     fun provideDatabase(@ApplicationContext appContext: Context): AppDatabase {
>         return Room.databaseBuilder(
>             appContext,
>             AppDatabase::class.java,
>             "book.db"
>         ).build()
>     }
>   
>     @Provides
>     fun provideBookDao(database: AppDatabase): BookDao {
>         return database.bookDao()
>     }
> }
> ```
>
 我们为其中的 **provideBookDao** 增加了@Provides，其意义是告诉Hilt 提供实例`BookDao`时，需要执行`database.bookDao()`。由于我们具有`AppDatabase`传递依赖关系，因此我们还需要告诉Hilt如何提供该类型的实例。
>
 由于`AppDatabase`是由Room生成的，因此是项目不拥有的另一个类，因此我们直接复制原方法即可，这里的 **@Singleton** 标志这个其方法只会被调用一次，类似于一个单例。


 **更改 MainActivity 中的代码如下**：
>
> ```kotlin
> @AndroidEntryPoint
> class MainActivity : AppCompatActivity() {
>     @Inject
>     lateinit var bookDao: BookDao
> }
> ```
>
到现在为止，我们就算改造完成了，这样的话，我们在任意位置都可以获得这个BookDao.而且免除手动构造


<br/>
<br/>

## 使用@Binds为接口提供注入

对于接口，无法使用构造函数进行注入，我们需要告诉Hilt使用哪种实现。Binds的作用就在于此。

需要注意以下使用条件：

- **Binds** 必须注释一个抽象函数，抽象函数的返回值是我们为其提供实现的接口。通过添加具有接口实现类型的唯一参数来指定实现。

### 举个🌰：

我们有一个 IBook 接口，用来存储及查询书本数据
>
> ```kotlin
> interface IBook {
>     suspend fun saveBook(name: String)
> 
>     suspend fun getBookAllSum(): Int
> }
> ```

 接着如果我们想在别的地方拿到这个接口对象，常规的实现方式可能就是 你的某个具体实现类实现了其，然后在需要使用的地方 再  ***val iBook=xxxImpl()***
>


 **如果用Hint呢？**，继续代码演示
>
 接着有一个具体的实现类 **BookImpl** ，这里我们使用构造函数注入 并且注入了  **BookDao** 用来处理具体的数据存储。这里我们用到了挂起函数，对于这块不怎么熟系的同学，可以理解为，其相当于一个标记位，提示编译器这块可能会有耗时操作，挂起函数即逻辑上的一个处理。具体理解可参考扔物线等大佬的解释，这里不做过多解释。
>
> ```kotlin
> class BookImpl @Inject constructor() : IBook {
>     @Inject
>     lateinit var dao: BookDao
>     override suspend fun saveBook(name: String) {
>         //这里可能会有一些自己的处理...
>         dao.insertAll(Book(name))
>     }
>   
>    override suspend fun getBookAllSum(): Int {
>         return dao.queryBookAll()
>     }
> }
> ```

 然后我们需要一个新的模块，用来实现 **IBook** 接口的注入
>
> ```kotlin
> @InstallIn(ApplicationComponent::class)
> @Module
> abstract class BookModel {
>  @Binds
>     abstract fun getIBook(impl: BookImpl): IBook
>    }
> ```
> 

在某个Activity使用时(Demo示例，实际开发我们更推荐数据处理放在Repository中，由ViewModel管理)：
>
> ```kotlin
> @AndroidEntryPoint
> class MainActivity : AppCompatActivity() {
>     @Inject
>     lateinit var iBook: IBook
> 
>     override fun onCreate(savedInstanceState: Bundle?) {
>         super.onCreate(savedInstanceState)
>         setContentView(R.layout.activity_main)
>         btnSave.setOnClickListener {
>             lifecycleScope.launch(Dispatchers.IO) {
>                 iBook.saveBook("Android艺术")
>                 val sum = iBook.getBookAllSum()
>                 Log.e("petterp", "当前长度-----$sum")
>             }
>         }
>     }
> }
> ```


<br/>
<br/>


## **Qualifiers**（限定）

我们在上面的例子中，只有一个具体实现类，但是往往实际开发，我们是存在多个具体实现。而且他们的作用域也都不同，有些可能只是某个Activity使用，有些是全局使用，对于这种问题我们如何解决呢？

我们可以为两个具体实现定义不同的模块并使用Qualifers规定。

### 举个🌰：

 依然以上面的 代码延续。此时有另一个实现，想实现有特殊条件的存储。
>
> 另一个实现类为：**BookConditionImpl**
>
> ```kotlin
> class BookConditionImpl @Inject constructor() : IBook {
>     @Inject
>     lateinit var dao: BookDao
>     override suspend fun saveBook(name: String) {
>         if (name == "Android艺术") {
>             dao.insertAll(Book("Petterp"))
>         }
>     }
> 
>     override suspend fun getBookAllSum(): Int {
>         return dao.queryBookAll()
>     }
> 
> }
> ```

> 然后我们更改 BookModel,新加入**BookConditionModel** 模块 
>
> ```kotlin
> @InstallIn(ApplicationComponent::class)
> @Module
> abstract class BookModel {
>     @Binds
>     @Singleton
>     abstract fun getIBook(impl: BookImpl): IBook
> }
> 
> @InstallIn(ActivityComponent::class)
> @Module
> abstract class BookConditionModel {
>     @Binds
>     @ActivityScoped
>     abstract fun getConditionIBook(impl: BookConditionImpl): IBook
> }
> ```

> 接着添加两个注解，其中使用到了 **@Qualifier**
>
> ```kotlin
> @Qualifier
> annotation class BookModelSingle
> 
> @Qualifier
> annotation class BookModelCondition
> 
> @InstallIn(ApplicationComponent::class)
> @Module
> abstract class BookModel {
>     @BookModelSingle
>     @Binds
>     @Singleton
>     abstract fun getIBook(impl: BookImpl): IBook
> }
> 
> //其模块范围为Activity
> @InstallIn(ActivityComponent::class)
> @Module
> abstract class BookConditionModel {
>     @Binds
>     @ActivityScoped
>     @BookModelCondition
>     abstract fun getConditionIBook(impl: BookConditionImpl): IBook
> }
> ```

> 然后更改我们的Activity
>
> ```kotlin
> @AndroidEntryPoint
> class MainActivity : AppCompatActivity() {
>     @BookModelCondition
>     @Inject
>     lateinit var iBookCondition: IBook
> 
>     @BookModelSingle
>     @Inject
>     lateinit var iBook: IBook
>    
> }
> ```
>

<br/>
<br/>



## 与JetPack

> 作为Google推荐的依赖注入组件，目前Hilt 可以与ViewModel配合使用

**导入依赖**

```groovy
    implementation 'androidx.hilt:hilt-lifecycle-viewmodel:1.0.0-alpha02'
    kapt 'androidx.hilt:hilt-compiler:1.0.0-alpha02'
		
		
		//viewModel的数据恢复，可以不导入，这里只是为了演示
    implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:2.2.0"
		//便于 使用ViewModel-ktx扩展
    implementation 'androidx.activity:activity:1.1.0'
    implementation 'androidx.fragment:fragment-ktx:1.2.5'
```

### 举个🌰

**我们创建一个MainActivty**

```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    private val viewModel by viewModels<TestViewModel>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel.test()
    }
}
```

**ViewModel**

```kotlin
class TestViewModel @ViewModelInject constructor(
    private val repository: TestRepository,
    @Assisted val savedState: SavedStateHandle
) : ViewModel() {
    fun test() {
        repository.test()
    }
}
```

**TestRepository**

```kotlin
@ActivityScoped
class TestRepository @Inject constructor() {
    fun test() {
        Log.e("petterp", "一个测试方法")
    }
}
```

### 补充Java中注入ViewModel

```java
public class TestViewModel extends ViewModel {
    @ViewModelInject
    public TestViewModel() {

    }
    public void test(){
        Log.e("petterp","我是测试方法");
    }
}
```



到目前为止，Hilt相关内容基本就结束了，目前对于Hilt的资料较少，而且处于alpha，如果想要用到实际项目中，建议多试多看文档，再做决定。

参考资料,按照优先级：(以下内容需要梯子)

[new  JetPack](https://www.youtube.com/watch?v=R3caBPj-6Sg)

[Hilt-代码实验室](https://codelabs.developers.google.com/codelabs/android-hilt/#0)

[DaggerDev](https://dagger.dev/hilt/)

[Google-iosched](https://github.com/google/iosched)


