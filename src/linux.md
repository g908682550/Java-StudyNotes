## 

# Linux

## 基本

### 入门

Linux：Linux is not Unix

Linux里面一切皆是文件

Linux里面没有后缀名这一说

Linux与Windows对比

![image-20191030151530300](linux.assets/image-20191030151530300.png)

### 查看SSH服务

/etc/init.d/sshd

### 启动SSH

/etc/init.d/sshd start

### 运行级别

![image-20191030184618600](linux.assets/image-20191030184618600.png)



### 启动过程

1. 内核引导

   ![image-20191030185310443](linux.assets/image-20191030185310443.png)

2. 运行init

   ![image-20191030185349437](linux.assets/image-20191030185349437.png)

   RunLevel

   ![image-20191030185751189](linux.assets/image-20191030185751189.png)

3.系统初始化

![image-20191030190159101](linux.assets/image-20191030190159101.png)

4.建立终端

5.用户登录系统

![image-20191030190057482](linux.assets/image-20191030190057482.png)

### 关机

![image-20191030190330672](linux.assets/image-20191030190330672.png)

## 命令

### VI和VIM的三大模式

#### 1、一般模式

​	![image-20191030195845390](linux.assets/image-20191030195845390.png)

#### 2、编辑模式

​	i ：当前光标前

​	a：当前光标后

​	o：下一行

#### 3、指令模式

​		冒号![image-20191030201331706](linux.assets/image-20191030201331706.png)

/:![image-20191030201519297](linux.assets/image-20191030201519297.png)

?:

![image-20191030201549086](linux.assets/image-20191030201549086.png)

### 用户与用户组

![image-20191030202159547](linux.assets/image-20191030202159547.png)

- [ ] ![image-20191030203752080](linux.assets/image-20191030203752080.png)


![image-20191030203651605](linux.assets/image-20191030203651605.png)

![image-20191030204155253](C:\Users\gy136\AppData\Roaming\Typora\typora-user-images\image-20191030204155253.png)

### 常用基本命令

#### 1、时间日期类

#### 2、文件目录类

##### ls

![image-20191030210413096](linux.assets/image-20191030210413096.png)

##### mkdir

-p 创建多级目录

##### rmdir

删除空文件目录

##### touch

创建文件

##### cd

切换目录

##### cp

复制

##### rm

-r 代表删除这个下面的一切

-f 表示不需要用户确认，直接执行 

##### mv

移动文件与目录或修改名称

##### cat

![image-20191030212141625](linux.assets/image-20191030212141625.png)

##### tac

与cat相反

##### more

![image-20191030212754614](linux.assets/image-20191030212754614.png)

##### less

![image-20191030212819992](linux.assets/image-20191030212819992.png)

##### tail

看文件后几行 tail -n #文件

##### head

看文件头几行 head -n #文件

##### history

所敲命令历史

##### 重定向命令

![image-20191030213447619](linux.assets/image-20191030213447619.png)

#### 3、文件权限类



#### 4、网络配置类

ifconfig

#### 5、磁盘分区类

##### fdisk -l

![image-20191030213827433](linux.assets/image-20191030213827433.png)

##### 挂载/卸载概念

![image-20191030214839952](linux.assets/image-20191030214839952.png)

##### 磁盘

![image-20191030215734861](linux.assets/image-20191030215734861.png)

#### 6、搜索查找类

##### find

find #目录 -name #名字

##### grep

![image-20191030220148049](linux.assets/image-20191030220148049.png)

#### 7、进程和线程类

##### ps

ps -ef

ps -aux

![image-20191030221131880](linux.assets/image-20191030221131880.png)

##### netstat

![image-20191030221808321](linux.assets/image-20191030221808321.png)

#### 8、压缩和解压类

##### 第一组（gzip+gunzip）

![image-20191030222245545](linux.assets/image-20191030222245545.png)



##### 第二组(最重要)

![image-20191030222440801](linux.assets/image-20191030222440801.png)

##### 第三组

![image-20191030222825354](linux.assets/image-20191030222825354.png)

## Linux文件与目录结构

### 目录结构

![image-20191030223434633](linux.assets/image-20191030223434633.png)

![image-20191030223724804](linux.assets/image-20191030223724804.png)

### 文件属性

#### 文件概述

![image-20191030224552119](linux.assets/image-20191030224552119.png)

作用到文件或目录的区别

![image-20191030225048736](linux.assets/image-20191030225048736.png)

#### 0首位表示类型

![image-20191030225208364](linux.assets/image-20191030225208364.png)

#### 文件详细说明

![image-20191030225325620](linux.assets/image-20191030225325620.png)

### 文件权限类

#### chmod

![image-20191030225444477](linux.assets/image-20191030225444477.png)

## RPM

### 是什么？

![image-20191031170932551](linux.assets/image-20191031170932551.png)

### 查询

rpm -qa: 查询所安装的所有rpm软件包 ex：rpm -qa|more；rpm-qa|grep X

rpm -q 软件包名：查询软件包是否安装

rpm -qi 软件包名：查询软件包信息

rpm -ql 软件包名：查询软件包中的文件

### 安装

![image-20191031172213498](linux.assets/image-20191031172213498.png)

### 卸载

![image-20191031172549592](linux.assets/image-20191031172549592.png)

