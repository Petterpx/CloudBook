# Koltin实际开发中的那些事

### 使用kotlin读入文件

```kotlin
val builder = StringBuilder()
            assets.open(fileName).buffered().reader().use {
                builder.append(it)
            }
```

