# PHP学习

## API

### Get

```
<?php
//$data = array('age' => 20, 'name' => '景天');
//$response = array('code'  => 200,'message' => '请求成功','data'  => $data,);
//echo json_encode($response);



$_name=isset($_GET['name'])?$_GET['name']:'';
$_pswd=isset($_GET['pswd'])?$_GET['pswd']:'';

if ($_name&&$_pswd){
    $response = array('code'  => 200,'message' => '请求成功','data'  => 'Petterp',);
    //后面是转utf-8
    echo json_encode($response,JSON_UNESCAPED_UNICODE);
}else{
    echo json_encode("null數據",JSON_UNESCAPED_UNICODE);
}
```



### Post

```php
<?php
/**
 * Created by PhpStorm.
 * User: Pettepr
 * Date: 2019/6/24
 * Time: 20:58
 */
$name='';
if (!empty($_POST)){
    $name=$_POST['name'];
}else{
    $json = file_get_contents("php://input");
    $data = json_decode($json, true);
    $name=$data['name'];
}
if ($name){
    $response = array('code'  => 200,'message' => '请求成功','data'  => $name);
    echo json_encode($response,JSON_UNESCAPED_UNICODE);
}
```

```php
/**
第二种，表单形式形式提交
 * Created by PhpStorm.
 * User: Pettepr
 * Date: 2019/6/25
 * Time: 10:01
 */
header('content-type:text/html;charset-utf-8');
$info1=isset($_POST['data'])?$_POST['data']:'123';
//这句话只能转义json串，所以会客户端将不会接收到返回的信息
echo json_decode("数据为".$info1);
//这句话会组合成json格式，前提是后面带上相应的格式
echo json_encode("数据为".$info1,JSON_UNESCAPED_UNICODE);
```





## 数据库操作

### 连接数据库

```php
<?php
$mysql_conf = array(
    'host'    => '127.0.0.1:3306',
    'db'      => 'book',
    'db_user' => 'root',
    'db_pwd'  => '',
);

$mysqli =new mysqli($mysql_conf['host'], $mysql_conf['db_user'], $mysql_conf['db_pwd'],$mysql_conf['db']);
if ($mysqli->connect_error){
    echo '连接失败';
}else{
    echo '连接成功';
}

//}
```



### 查询数据库

```php
<?php
header('content-type:text/html;charset-utf-8');
$mysql_conf = array(
    'host' => '127.0.0.1:3306',
    'db' => 'book',
    'db_user' => 'root',
    'db_pwd' => '',
);
$error = json_encode(array(
    'code' => 200,
    'message' => '请求失败',
), JSON_UNESCAPED_UNICODE);

//连接数据库
$mysqli = new mysqli($mysql_conf['host'], $mysql_conf['db_user'], $mysql_conf['db_pwd'], $mysql_conf['db']);

//更改数据库编码
mysqli_query($mysqli, "set names 'utf8'");
if ($mysqli->connect_error) {
    echo $error;
} else {
    $query = "select *from User";
    // or die指的是如果前面失败，后面将不会执行
    $result = mysqli_query($mysqli, $query);
    if ($result) {
        //mysqli_fetch_assoc 返回关联数组，键为字段名,值为对应的数据
        while ($row = mysqli_fetch_assoc($result)) {
            $emp_info[] = $row;
        }
        //转成json
        echo json_encode($emp_info, JSON_UNESCAPED_UNICODE);
    } else {
        echo $error;
    }
    //@指的是错误抑制符号,这里是释放内存
    @mysqli_free_result($result);
}
```



### 数据库添加

> 增加语句(第一种)：INSERT INTO cs_user VALUES (null,'阿里','wodemima','男',22);
>
> 增加语句(第二种)：INSERT INTO cs_user(username,password) VALUES ('巴巴','baba');
>
> **第一种：这个表有5个字段，那么你就要写5个数据，并且数据的顺序要对应字段的顺序，id 是主键并自增长，所以在 VALUES 里写入 NULL 即刻，自动增加，每次+1，数字类型，可以直接写数字，字符串类型，比如带单引号或双引号(不能使用中文的)**
>
> **	第二种：在cs_user 表的后面加入了字段名(以逗号隔开)，因为id是自动增长，所以我们可以不用写，这里我们写的字段是 username和password，所以添加数据，也只写入这两个就可以了，其他的字段，不需要理会**
>
> 两种写法的差异：
>
> 1：表名后面有没有跟字段名；
>
> 2：第一种数据添加必须写全，第二种只写表名后面跟的字段(id自增属性 NULL都不用写)；
>
> 3：第一种必须按照表字段的顺序，第二种可以把字段顺序打乱(比如：上面的username和password交换位置也没问题)，但数据也必须对应写入字段的顺序；
>
> 4：第一种写全数据才能添加，否则失败，第二种只添加指定的字段，没有指定的字段则为空(NULL)，或是为你的默认值(DEFAULT设置字段属性，默认某个值)
>
>  
>
> 两种写法的共同点：
>
> 1：必须按字段的顺序添加数据，若添加的数据和字段属性不匹配，则会添加失败(比如：数字类型，不能写入字符串)

```php
<?php
$id =isset($_POST['id'])?$_POST['id']:'';;
$name =isset($_POST['name'])?$_POST['name']:'';;
$pswd =isset($_POST['pswd'])?$_POST['pswd']:'';;

//造连接对象
$db = @new MySQLi("localhost", "root", "", "book");
//编码
mysqli_query($db, "set names utf8 ");
//判断是否出错
if (mysqli_connect_error()) {
    echo json_encode("失败", JSON_UNESCAPED_UNICODE);
} else {
    $sql = "insert into user values({$id},'$name','$pswd') ";
    if ($db->query($sql)) {
        echo json_encode("成功", JSON_UNESCAPED_UNICODE);
    } else {
        echo json_encode("失败", JSON_UNESCAPED_UNICODE);
    }
}
```





### 修改语句

```php
$id =isset($_POST['key'])?$_POST['key']:'';;
$date =isset($_POST['date'])?$_POST['date']:'';;
$money =isset($_POST['money'])?$_POST['money']:'';;
$type =isset($_POST['type'])?$_POST['type']:'';;

//造连接对象
$db = @new MySQLi("localhost", "root", "", "book");
//编码
mysqli_query($db, "set names utf8 ");
//判断是否出错
if (mysqli_connect_error()) {
    echo json_encode("失败", JSON_UNESCAPED_UNICODE);
} else {
    $id=$id+10000;
    //key为关键字，所以需要 加 ``这两个字符
    $sql = "update info set money=$money,date='$date',type='$type' where `key`=$id";
    echo $sql;
    if ($db->query($sql)) {
        echo json_encode("成功", JSON_UNESCAPED_UNICODE);
    } else {
        echo json_encode("失败", JSON_UNESCAPED_UNICODE);
    }
}
```



### 删除语句

```php
$id =isset($_POST['key'])?$_POST['key']:'';;

//造连接对象
$db = @new MySQLi("localhost", "root", "", "book");
//编码
mysqli_query($db, "set names utf8 ");
//判断是否出错
if (mysqli_connect_error()) {
    echo json_encode("失败", JSON_UNESCAPED_UNICODE);
} else {
    $id=$id+10000;
    //key为关键字，所以需要 加 ``这两个字符
    $sql = "delete from info where `key`=$id";
    if (@mysqli_num_rows($db->query($sql))>0) {
        echo json_encode("删除成功", JSON_UNESCAPED_UNICODE);
    } else {
        echo json_encode("删除失败", JSON_UNESCAPED_UNICODE);
    }
}
```



## 实例Demo

### 登录

```php
<?php
/**
 * Created by PhpStorm.
 * User: Pettepr
 * Date: 2019/6/25
 * Time: 16:52
 */

$name = isset($_GET['name']) ? $_GET['name'] : '';
$pswd = isset($_GET['pswd']) ? $_GET['pswd'] : '';
$db = new mysqli("localhost", "root", "", "book");
mysqli_query($db, "set names 'utf8'");
if (mysqli_connect_error()) {
    echo json_decode("数据库异常");
    exit();
} else {
    $sql = "select * from user where name='$name' and pswd='$pswd'";
    $result = mysqli_query($db, $sql);
    if (mysqli_num_rows($result)==0) {
        echo json_decode("输入有误");
        exit();
    } else {
        $row = mysqli_fetch_assoc($result);
        echo json_decode($row['id']);
    }
}
```



### 注册

```php
<?php
$name = isset($_POST['name']) ? $_POST['name'] : '';;
$pswd = isset($_POST['pswd']) ? $_POST['pswd'] : '';;

//造连接对象
$db = @new MySQLi("localhost", "root", "", "book");
//编码
mysqli_query($db, "set names utf8 ");
//判断是否出错
if (mysqli_connect_error()) {
    echo json_encode("失败", JSON_UNESCAPED_UNICODE);
} else {
    $sql1 = "select * from user where name='$name'";
    if (mysqli_num_rows($db->query($sql1))>0) {
        echo json_encode("注册失败，账号已存在",JSON_UNESCAPED_UNICODE);
        exit();
    } else {
        $sql2 = "insert into user(name,pswd) values('$name','$pswd') ";
        if ($db->query($sql2)) {
            echo json_encode("注册成功", JSON_UNESCAPED_UNICODE);
        } else {
            echo json_encode("注册失败", JSON_UNESCAPED_UNICODE);
        }
    }
}
```



## 处理文件上传

#### API

##### $FILES数组介绍

其数组中的error有7个值：

​	0表示上传成功

​	1表示文件大小超过了php.ini中的upload_max_filesize选项限制的值

​	2表示文件大小超过了表单中max_file_size选项指定的值，

​	3表示文件只有部分被上传

​	4表示没有文件被上传

​	6表示找不到临时文件夹

​	7表示写入失败

```PHP
array (size=1)
  'file' => 
    array (size=5)
      'name' => string '1_1.png' (length=7)
      'type' => string 'image/png' (length=9)
      'tmp_name' => string 'D:\wamp\wamp\tmp\php20B2.tmp' (length=28)
      'error' => int 0
      'size' => int 104398
```

##### $move_uploaded_file()介绍

该方法传递两个参数：

​	第一个参数：上传的文件的文件名。

​	第二个参数：上传后保存的文件位置

#### 实现保存多个上传文件

```php
<?php
//查看文件信息
var_dump($_FILES);
$error='';
$path = 'D:\wamp\wamp\www\image';
foreach($_FILES as $key=>$value){
    $file= $_FILES[$key];
    $newfilename = fileupload($file, $path, $error);
    echo $path.'\\'.$newfilename;
    //将上传的文件地址保存至数据库
}

/*
 * @param1 array $file 要上传的文件信息，包含5个元素
 * @param2 string $path 存储位置
 * @param3 string $error 错误信息
 * @param4 array $type=array() MIME类型限定
 * @param5 int $size=2000000 默认为2M
 * @return mixed，成功返回文件名字，失败返回false
 *
 * */
function fileupload($file,$path,&$error,$type=array(),$size=2000000){
    //判断文件本身是否是有效文件
    if (!isset($file['error'])){
        $error='文件无效';
        return false;
    }

    if (!is_dir($path)){
        $error='存储路径无效';
        return false;
    }
    //判断文件是否上传成功
    switch ($file['error']){
        case 1:
        case 2:
            $error='文件超过服务器允许大小';
            return false;
        case 3:
            $error='文件部分上传成功';
            return false;
        case 4:
            $error='用户没有选择上传的文件';
            return false;
        case 6:
        case 7:
            return false;
    }
    //类型判定
    if (!empty($type)&&!in_array($file['type'],$type)){
        $error='当前上传的文件类型不符合';
        return false;
    }

    //大小判定
    if ($file['size']>$size){
        $error='文件大小超过当前允许的大小';
    }

    //文件转存
    $newfilename = getNewName($file['name'],5);
   if ( move_uploaded_file($file['tmp_name'],$path.'\\'. $newfilename)){
       //成功
       return $newfilename;
   }else{
       return "存入失败";
   }
}


//随机生成文件名
function getNewName($filename,$rand){
    $newname=date('YmdHis');
    //将参数数组合并为一个数组
    $old=array_merge(range('a','z'),range('A','Z'),range('0','9'));
    //将数组打乱
    shuffle($old);
    for ($i=0;$i<$rand;$i++){
        $newname.=$old[$i];
    }
    //获取有效文件名
    return $newname.strstr($filename, '.');
}
```

