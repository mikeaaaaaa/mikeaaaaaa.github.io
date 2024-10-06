---

title: objection手册
date: 2024-01-12 11:00:00 +0800
categories: [Android]
tags: [Objection,Frida]

---
## objection执行手册
### 连接fr-server：

（1）objection -g 包名 explore

   (2)  objection -N -h 192.168.1.1 -p 8080 -g 包名 explore

（如果报名了不存在会自动帮我们启动）

#### 连接并执行命令

使用-s参数，

![1699193193614](/assets/image/2024-01-12-objection手册/1699193193614.png)

#### 指定充满指令的文件执行

```shell
objection -g com.cz.babySister  explore -c ".\2.txt"
```

![1699270132281](/assets/image/2024-01-12-objection手册/1699270132281.png)

### 内存搜索



#### 列出app所有的类/服务/活动

基本不用，因为涉及到所有app的，结果太多了

android hooking list classes/services/activities

#### 关键词搜索相关类

android hooking search  classes xxx(display)（搜索包含display关键词的类）

#### 关键词搜索方法

**因为是在所有类中搜索，因此特别花费时间，不推荐使用**（可以ctrl + c停止）

android hooking search methods xxx

#### 查看类的所有方法

android hooking list class_methods xxx(类名)

####  启动指定activity/service：

android intent launch_activity  xxx(activity_name)

android intent launch_service  xxx(activity_name)

#### 搜索类的实例

Android heap search instances xxx（类名）



#### 查看内存中加载的库 

memory list modules 

#### 查看库的导出函数 

memory list exports so名称 

#### 解除证书绑定
android sslpinning disable




### instance使用

#### 搜索类的实例

Android heap search instances xxx（类名）

#### 主动调用方法

android heap execute  xxx(实例) xxx（方法名）    **无参数**方法

android heap evaluate xxx(实例)                                **有参数方法**（需要进入编辑器环境）![1699188192004](/assets/image/2024-01-12-objection手册/1699188192004.png)![1699188234514](/assets/image/2024-01-12-objection手册/1699188234514.png)![1699188224744](/assets/image/2024-01-12-objection手册/1699188224744.png)



####  插件 wallbreaker使用

再objection 开启命令 最后加上 -p /objection_plugin 即使用-p参数指定 wallbreaker目录

##### 1. 搜索类

```
plugin wallbreaker classsearch <pattern>
```

##### 2.搜索对象

```
plugin wallbreaker objectsearch <classname>
```

##### 3. 输出类的结构

```
plugin wallbreaker classdump <classname> [--fullname]

 
输出类的结构， 若加了 --fullname 参数，打印的数据中类名会带着完整的包名。
```

##### 4. 输出对象的具体数据（！！！）

```
plugin wallbreaker objectdump <handle> [--fullname]
```
### Hook

#### 查看hook作业jobs：

jobs list

#### 关闭hook作业

jobs kill [task id]

#### hook整个类的方法

android hooking watch class xxx(类名)

#### hook具体某个方法

输出 **参数、返回值、调用栈** 

android hooking watch class_method xxx(包名.类.method)--dump-args --dump-backtrace --dump-return 

#### hook 类的构造方法 

android hooking watch class_method 类名.$init 

#### hook 方法的所有重载 

android hooking watch class_method 类名.方法名 



### 勾取整个程序的堆栈：

#### ZenTracer

 [项目地址](https://github.com/hluwa/ZenTracer) 

缺点:无法打印调用栈

无法`hook`构造函数

#### r0tracer

 [项目地址](https://github.com/r0ysue/r0tracer) 

使用：

frida -U 包名 -l r0tracer.js



### 加载插件

方法一：运行`objection`时加载插件：

`objection -g com.app.name explore -P ~/.objection/plugins`  

方法二：首先进入`objection`，

之后运行命令： `plugin load xxx/Wallbreaker-master`,这里以`wallbreaker`为例，这是一个可以打印类结构的`objection`插件



