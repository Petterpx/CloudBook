# Activity的回调和onActivityResult的使用

***onActivityResult这个回调方法在activity中用的非常多，并且一般与startActivityForResult这个方法配合使用。***

#### 1.startActivityForResult(Intent intent, int requestCode);

-  第一个参数：一个Intent对象
-  第二个参数：如果> = 0,当Activity结束时requestCode将归还在onActivityResult()中。以便确定返回的数据是从哪个Activity中返回的。

#### 2.onActivityResult(int requestCode, int resultCode, Intent data)；

-  第一个参数：这个整数requestCode提供给onActivityResult，是以便确认返回的数据是从哪个Activity返回的，就是在startActivityForResult设置的requestCode。
-  第二个参数：这整数resultCode是由子Activity通过其setResult()方法返回，就是setResult(int resultCode, Intent data)的第一个参数resultCode。



#### 一个🌰

>  **一般来说，resultCode主要指定为RESULT_CANCELED和RESULT_OK ，然后在onActivityResult获取到resultCode进行判断，如果是RESULT_CANCELED就不执行回调方法，如果是RESULT_OK 就执行回调方法**

***我们来举个栗子:***
有一个activity，它有诸多按钮可以跳转到不同的界面，就把这个activity先叫做 根activity 吧
 在跟activity上点击button1，可以跳转到activity1,同样点击button2，可以跳转到activity2，同时还都要获取到activity1或者activity2的返回值显示出来.

**button1的点击事件：**

```java
Intent intent = new Intent(RootActivity.this, Activity1.class);
context.startActivityForResult(intent, REQUEST_CODE_ACTIVITY1);
```

**button2的点击事件:**

```java
Intent intent = new Intent(RootActivity.this, Activity2.class);
context.startActivityForResult(intent, REQUEST_CODE_ACTIVITY2);
```

***REQUEST_CODE_ACTIVITY1***和***REQUEST_CODE_ACTIVITY2***是不相同的两个int值，标识出两个不同activity的返回值。

那么在activity1中，如果要设置回调，应该这样写

```kotlin
 Intent intent = new Intent(Activity1.this, RootActivity.class);
 intent.putExtra("pass_data", data);
 setResult(RESULT_OK, intent);
 finish();   
```

如果不想设置回调事件（比如在Activity1中什么都没操作点击返回），那么就设置为 ***setResult(RESULT_CANCELED, intent);***

 接下来就是重写RootActivity的**onActivityResult**方法

```java
@Override
   protected void onActivityResult(int requestCode, int resultCode, Intent intent) {
       super.onActivityResult(requestCode, resultCode, intent);
           switch (requestCode) {
               case REQUEST_CODE_ACTIVITY1:
                   if (resultCode == RESULT_OK) {
                               if（intent!=null）{
                                       //获取activity1传递的数据并显示出来
                                       dosomething;

                                }
                     }
                    break;
               case REQUEST_CODE_ACTIVITY2:
                   if (resultCode == RESULT_OK) {
                               if（intent!=null）{
                                       //获取activity2传递的数据并显示出来
                                       dosomething;
                                }
                     }
                    break; 
           }
   }
   
```



**接下来再说一种情况，那就是从RootActivity进入Activity1，再进入Activity2,再进入Activity3这种连续跨好几个界面的，要关闭Activity2后直接进入RootActivity，如果不需要传递数据，那么在跳转的时候就可以直接使用startActivity这个方法来跳转。在Activity3中只需要写如下代码：**

```java
Intent intent = new Intent(Activity3.this, RootActivity.class);
//在Activity栈中将在RootActivity之上的所有activity出栈。
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK|Intent.FLAG_ACTIVITY_CLEAR_TASK);
startActivity(intent);
```



**跨多个activity，并且传递数据的话，在跳转的时候也直接使用startActivity这个方法来跳转，不过在Activity3中则需要使用如下代码：**

```kotlin
 Intent intent = new Intent(Main3Activity.this, RootActivity.class);
 intent.putExtra("pass_data",data);
 intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP | 
 Intent.FLAG_ACTIVITY_SINGLE_TOP);
 setResult(RESULT_OK,intent);
 startActivity(intent);
```

在RootActivity中就不能通过**onActivityResult**接受数据了。而应该使用**onNewIntent**这个方法

```java
@Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        //从Main3Activity返回的数据
        String data = intent.getStringExtra("pass_data");
        dosomething;
    }
```

