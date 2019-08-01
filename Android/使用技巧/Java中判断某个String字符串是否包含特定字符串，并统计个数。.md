# Java中判断某个String字符串是否包含特定字符串，并统计个数。

方法1(最简单的)：

A 字符串    -> b字符串，统计A中b的个数。

```java
int sum=
  (res.length()-res.replace(b,"").length())/b.length();
```

原理很简单，用空格代替特定字符，然后计算与以前字符串的长度差,再除以 特定字符的长度，即可得出A中所占b的个数。



方法2：(利用indexOf与subString)

```java
String res="AAAAABBAAA";
String b="B";
int count=0;
while (res.contains(b)){
    res=res.substring(res.indexOf(b),res.length());
    ++count;
}
```

原理也很简单，就是不断判断相应字符串是否包含特定字符串，如果包含，+1，并找相应位置，切割剩余部分继续判断。