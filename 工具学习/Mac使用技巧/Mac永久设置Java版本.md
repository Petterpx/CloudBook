# Mac永久设置Java版本



查看当前Java版本

> **java -version**

查看当前存在的java版本

> 目录 /Library/Java/JavaVirtualMachines

临时更改Java 版本

> ***export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/***

永久更改Java版本

> **echo 'export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/' >> ~/.bash_profile source ~/.bash_profile**

