# JUnit单元测试框架使用

> 在我以往的Android开发生涯中，几乎没有使用过单元测试，也没有见过有人去介绍过，好像这个东西在国内开发者眼里并不是很重要，或者说大多数开发同学没有专门的时间去使用单元测试框架，也许更重要的原因应该是我个人的 `孤陋寡闻` 。

## 背景

#### 什么是单元测试？

单元测试是针对最小的单元编写测试代码。在 **Java** 中,最小的功能单位是方法，因此，对**Java** 程序进行单元测试就是针对单个 **Java** 方法的测试。

#### 为什么要做单元测试

在国外，实际开发流程往往是，先编写测试，测试写完后，再开始真正编写实现代码。在具体实现过程中，一边写一边测，什么时候测试全部通过，就代表开发任务完成。这也就是我们常说的  **TDD**(`测试驱动开发`)。



---

## 简介

`Junit` 是一个开源的Java语言的单元测试框架，专门为 **Java** 设计，使用也最为广泛。(当然 **Kotlin** 使用也没问题，注意一些**小细节**即可)

### 依赖方式

#### Maven

```xml
<dependencies>
 	<dependency>
    	<groupId>junit</groupId>
    	<artifactId>junit</artifactId>
    	<version>4.12</version>
    	<scope>test</scope>
	</dependency>
</dependencies>
```

#### Gradle

```groovy
dependencies {
	testImplementation 'junit:junit:4.12'
}
```



---

### 主要方法

Assert类中主要方法如下：

|      方法名       |                方法描述                |
| :---------------: | :------------------------------------: |
|   assertEquals    |    断言传入的预期值与实际值是相等的    |
|  assertNotEquals  |   断言传入的预期值与实际值是不相等的   |
| assertArrayEquals |  断言传入的预期数组与实际数组是相等的  |
|    assertNull     |          断言传入的对象是为空          |
|   assertNotNull   |         断言传入的对象是不为空         |
|    assertTrue     |              断言条件为真              |
|    assertFalse    |              断言条件为假              |
|    assertSame     | 断言两个对象引用同一个对象，相当于“==” |
|   assertNotSame   | 断言两个对象引用不同的对象，相当于“!=” |
|    assertThat     |      断言实际值是否满足指定的条件      |

`注意` 上面的所有方法,都有对应的重载方法，可以在前面加一个 **String** 类型的参数，表示断言失败时的提示。

---

### 常用注解

执行顺序：@BeforeClass –> @Before –> @Test –> @After –> @AfterClass

|     注解名      |                          含义                           |
| :-------------: | :-----------------------------------------------------: |
|      @Test      |                  表示此方法为测试方法                   |
|     @Before     |          在每个测试方法前执行，可做初始化操作           |
|     @After      |         在每个测试方法后执行，可做释放资源操作          |
|     @Ignore     |                     忽略的测试方法                      |
|  @BeforeClass   | 在类中所有方法前运行。此注解修饰的方法必须是static void |
|   @AfterClass   |    在类中最后运行。此注解修饰的方法必须是static void    |
|    @RunWith     |               指定该测试类使用某个运行器                |
|   @Parameters   |                指定测试类的测试数据集合                 |
|      @Rule      |               重新制定测试类中方法的行为                |
| @FixMethodOrder |               指定测试类中方法的执行顺序                |

---

## 使用方式

### 基础使用

比如我们有一个等效括号的这样一个算法。

`StackExample.kt`

```kotlin
/** 等效括号
 *  如题：给定一个字符串所表示的括号序列，包含以下字符： '(', ')', '{', '}', '[' and ']'， 判定是否是有效的括号序列。
 *  括号必须依照 "()" 顺序表示， "()[]{}" 是有效的括号，但 "([)]" 则是无效的括号。
 *
 *  解法思路：
 *  使用栈存储，将字符串切割为char遍历，先存储指定方向的符号，如'(','{','['。
 *  如果属于右方向的，如'}'等，进入判断，如果栈顶符号与当前char相等并且栈不会null，即为正确，否则直接return false
 * */
fun isBrackets(str: String): Boolean {
    if (str.length < 2) return false
    val stack = Stack<Char>()
    str.forEach {
        if ("{([".contains(it)) {
            stack.push(it)
        } else {
            if (stack.isNotEmpty() && isBracketChart(stack.peek(), it)) {
                stack.pop()
            } else return false
        }
    }
    return stack.isEmpty()
}


private fun isBracketChart(str1: Char, str2: Char): Boolean =
    (str1 == '{' && str2 == '}') ||
            (str1 == '[' && str2 == ']') || (str1 == '(' && str2 == ')')
```

#### 代码测试(未使用Junit)

如果是没有使用 **Junit**，我们可能会写出下面这样的测试代码：

```kotlin
fun main() {
    println(isBrackets("{}"))
  	xxxx...
}
```

相比来说我们如果我们增加别的方法，就需要频繁修改main()方法，而且对于测试的正确性也不能做到直观。

#### 使用Junit

我们在相应的test包下，新建 `StackExampleKtTest` 这样的类,或者直接使用如下快捷方式，在相应的方法前使用mac(**option+回车**),windows(**ctrl+回车**)，如图所示

![image-20201115091238607](https://tva1.sinaimg.cn/large/0081Kckwgy1gkpl7ziv3ej31q60gxdrc.jpg)

**示例如下：**

`StackExampleKtTest`

```kotlin
class StackExampleKtTest {

    @Test
    fun testIsBrackets() {
        assertArrayEquals(
            booleanArrayOf(
                isBrackets(""),
                isBrackets("{"),
                isBrackets("{{}}"),
                isBrackets("{({})}"),
                isBrackets("{({}"),
            ), booleanArrayOf(
                false, false, true, true,false
            )
        )
    }
}
```



### 参数化测试

上述使用方法，如果我们每次测试一个方法都要去设置对应的值，相对比较繁琐，那如何用连续不同的值去测试同一个方法呢，这样就可以避免我们不去多次修改，节省部分时间。这时就要使用 `@RunWith` 和 `@Parameters`.

首先需要在测试类上添加 `RunWith(Paramterized.class)` 注解，在创建一个由 `@Paramters` 注解的 **static** 方法，让返回一个对应的测试数据合集，最后创建构造方法，方法的参数顺序和测试数据集合一一对应。

**示例如下：**

```kotlin
@RunWith(Parameterized::class)
class StackExampleKtTest(private val str: Pair<String, Boolean>) {

    // TODO: 2020/11/15 相应的，这种处理方式也容易造成对错误的难以寻找
    companion object {
        @Parameterized.Parameters
        @JvmStatic
        fun primeStrings(): Collection<Pair<String, Boolean>> =
            listOf(
                "" to false,
                "{" to false,
                "{}" to false,
                "{[]}" to true,
                "asd{()}" to false
            )
    }

    @Test
    fun testIsBrackets() {
      	//注意这里的错误提示
        assertEquals("出错了-当前 \"${str.first}\" to ${str.second}",
            str.second, isBrackets(str.first))
    }
}
```

**注意**：相应的 **@Parameterized.Parameters** 方法在Kotlin中使用需要增加 `@JvmStatic` 。使用过程中，这种参数化测试如果我们没有加`错误提示`，寻找问题时可能不容易找到那个测试用例出了问题，所以这点也需要注意。



### assertThat用法

用于为断言失败后的输出信息提高可读性。默认情况下，断言失败只会抛出 `AssertionError` ，我们无法知道到底是哪里出错，而 `assertThat` 的作用就是解决这个问题。

![image-20201115112159350](https://tva1.sinaimg.cn/large/0081Kckwly1gkpoykbd9zj312c036t9a.jpg)



**常用的匹配器整理：**

| 匹配器               | 说明                               | 例子                                              |
| -------------------- | ---------------------------------- | ------------------------------------------------- |
| is                   | 断言参数等于后面给出的匹配表达式   | assertThat(5, is (5));                            |
| not                  | 断言参数不等于后面给出的匹配表达式 | assertThat(5, not(6));                            |
| equalTo              | 断言参数相等                       | assertThat(30, equalTo(30));                      |
| equalToIgnoringCase  | 断言字符串相等忽略大小写           | assertThat(“Ab”, equalToIgnoringCase(“ab”));      |
| containsString       | 断言字符串包含某字符串             | assertThat(“abc”, containsString(“bc”));          |
| startsWith           | 断言字符串以某字符串开始           | assertThat(“abc”, startsWith(“a”));               |
| endsWith             | 断言字符串以某字符串结束           | assertThat(“abc”, endsWith(“c”));                 |
| nullValue            | 断言参数的值为null                 | assertThat(null, nullValue());                    |
| notNullValue         | 断言参数的值不为null               | assertThat(“abc”, notNullValue());                |
| greaterThan          | 断言参数大于                       | assertThat(4, greaterThan(3));                    |
| lessThan             | 断言参数小于                       | assertThat(4, lessThan(6));                       |
| greaterThanOrEqualTo | 断言参数大于等于                   | assertThat(4, greaterThanOrEqualTo(3));           |
| lessThanOrEqualTo    | 断言参数小于等于                   | assertThat(4, lessThanOrEqualTo(6));              |
| closeTo              | 断言浮点型数在某一范围内           | assertThat(4.0, closeTo(2.6, 4.3));               |
| allOf                | 断言符合所有条件，相当于&&         | assertThat(4,allOf(greaterThan(3), lessThan(6))); |
| anyOf                | 断言符合某一条件，相当于或         | assertThat(4,anyOf(greaterThan(9), lessThan(6))); |
| hasKey               | 断言Map集合含有此键                | assertThat(map, hasKey(“key”));                   |
| hasValue             | 断言Map集合含有此值                | assertThat(map, hasValue(value));                 |
| hasItem              | 断言迭代对象含有此元素             | assertThat(list, hasItem(element));               |



### @Rule

在测试过程中，我们也可以通过增加 `@Before` 或者 `@After` 从而做到测试前后的一个提示效果，但是每次都这样写也许有点麻烦。所以这个时候可以使用 `@Rule`.

**示例如下：**

`TestPrompt`

```kotlin
 class TestPormpt : TestRule {
    override fun apply(statement: Statement, description: Description): Statement {

        // 获取测试方法的名字
        val methodName: String = description.methodName
        //相当于 @Before
        println(methodName + "测试开始前！")

        // 运行的测试方法
        statement.evaluate()

        //运行结束，相当于@After
        println(methodName + "测试结束！")
        return statement
    }
}
```

```kotlin
class StackExampleKtTest {

    // TODO: 2020/11/15
    //  注意：在 Kotlin 中使用时，需要将@Rule 更改为 @get:Rule
    //  或者 使用 @Rule @JvmField
    //@get:Rule
    @Rule @JvmField
    public val prompt = TestPormpt()
  
//    @Before
//    fun testStart(){
//        println("测试开")
//    }
//
//    @After
//    fun testStop(){
//        println("测试关")
//    }

    @Test
    fun testThat() {
        assertThat("123", equalTo("123"))
    }
}
```



---



## 参考

- [廖雪峰-编写JUnit测试](https://www.liaoxuefeng.com/wiki/1252599548343744/1304048154181666)
- [Android单元测试(一)：JUnit框架的使用](https://blog.csdn.net/qq_17766199/article/details/78243176)













