# Jenkins获取Git commit记录

Linux下：

官方插件市场暂时无法下载，故采取自行导入方法：



#### 先安装Maven

```java
创建一个目录并进入
mkdir usr/local/maven ; cd usr/local/maven
 
为了方便创建文件夹并直接cd过去，故自定义命令
alias mkcd='function __mkcd(){ if [ $# == 1 ]; then mkdir $1; cd $1; unset -f __mkcd; elif [ $# == 2 ]; then mkdir $1 $2; cd $2; unset -f __mkcd; fi }; __mkcd'

使用时
mkcd usr/local/maven

安装maven
wget wget http://mirrors.cnnic.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
如果提示command...，不识别命令，那先安装wget后，再安装maven
yum -y install wget

解压maven安装包
tar -zxvf apache-maven-3.5.4-bin.tar.gz


配置 maven
vi /etc/profile
给配置文件中添加这两句
export MAVEN_HOME=/usr/local/maven/apache-maven-3.5.4
export PATH=$MAVEN_HOME/bin:$PATH
上面的安装路径如果不同，自行更换。

配置文件生效
source /etc/profile

验证是否成功
查看版本
mvn -version
```



