# gradle语法学习 #
1.配置gradle环境  gradle -v 

2.新建一个文件夹 里面新建一个文件 命名为build.gradle  在cmd命令行中执行  gradle  -q hello  默认会执行本文件夹下面 build.gradle 脚本文件  如果想执行 别的文件 gradle -b build2.gradle -q hello     //-q表示输出日志的级别 

task hello{

	doLast{
		println 'hello world i come'
	}
}

## gradle wrapper  ##
gardle wrapper 与 gradle（本地）类似 作为一个包装类 跟随项目 
在build.gradle 也可以手动配置

task wrapper(type : Wrapper){

gradleVersion = '2.4'

archiveBase = 'GRADLE_USER_HOME'

archivePath = 'wrapper/dists'

distributionBase = 'GRADLE_USER_HOME'

distributionPath = 'wrapper/dists'


distributionUrl =''

}

## gradle日志 ##

级别递减  和log一样   会输出 在当前级别之上 的日志

error 错误消息

quiet重要消息

waring警告消息


lifecycle进度消息

info 信息消息

debug调试消息

println ===quiet 

logger.quiet()

logger.error()

## 查看gradle命令 ##
gradle -h   --help  help  

gradle tasks 查看所有可执行的task


强制刷新依赖 gradle --refresh --dependencies assemble 


通过任务名称的缩写来执行task     eg：addTaskMouldle   gradle  -q aTM 

## Groovy基础 ##

每个build.gradle都是groovy脚本  groovy兼容 java代码 所以可以在build.gradle 里面写java代码

1.字符串 

''  ""  单引号 与双引号 都标示字符串  区别 ''单纯的字符串 ""  可参与运算  gradle pS

task printlnString <<{


def name ="zhangsan"

println "cou:${name}"


}

def  表示变量

""具有计算的能力  二'' 没有

${}  {}里面放表达式 里面只有一个变量的时候可以省去{}  $name

**list操作 **

task printlnList <<{

def numList = [1,2,3,4];
println numList[1]
println numList[-1]

numList.each{
println it
}
}


**map操作**

task printMap <<{

def map1  = ['width':1024,'height':720];

println map1.getClass().name;

println map1['width'];

println map1.height


map1.each{
println "Key:${it.key},Value:${it.value}"

}


}


**方法 method**



task methodTask <<{
methodTest(1,2);

methodTest 1,2;

def result = methodTest1 3,4;

println 'result'+"${result}";  //单引号 单纯的字符串 双引号 内部可参与计算

}

def methodTest(int a ,int b){

println a+b;
}

def methodTest1(int a ,int b){

return a+b;

//可以不加 return  直接a+b
}


method调用 可以 不加括号 method 1,2

method  方法内 可以不写 return   默认最后一行作为返回值


**代码块（闭包）可作为参数传递的**

当有一个参数的时候  用it代替  当有多个参数的时候 就得用->一一列出来 eg  k,v ->

ask  closeureTest <<{

methodClose{
println it
}

}

def methodClose(haha) {
for(int i in 1..10){
haha(i);


}
}


eg2:

task closeTestTwo <<{

methodCloseTwo({k,v->

println "k:${k},v:${v}"
})

}

def methodCloseTwo(close){

def map1 =["width":1024,"height":720];
map1.each{
close(it.key,it.value);
}

}


**闭包委托**

没太理解 

## gradle构建脚本基础 ##

setting.gradle  是为了配置子工程  一个子工程modle只有在setting中做了配置  才能被识别 

创建task的两种方式 

task customTask {

doFirst{
println 'doFirst'
}
println 'do'
doLast{
println 'dolast'
}

}

tasks.create("customTask2"){

doFirst{
println 'doFirst'
}
println 'do'
doLast{
println 'dolast'
}
}


任务依赖

task customTask {

doFirst{
println 'customTaskdoFirst'
}
println 'customTaskdo'
doLast{
println 'customTaskdolast'
}

}

task  customTask2{

dependsOn customTask
doFirst{
println 'doFirst'
}
println 'do'
doLast{
println 'dolast'
}

}


//可以使用任务名称调用方法

customTask.doFirst{

println 'customTaskdoFirst-----'
}


**自定义属性**

//自定义一个属性
ext.age = 18

//自定义多个属性

ext{
name ="zhang"
phone = 111
}


task customProperty{

println "age:${age}"

println "name:${name}"

println "phone:${phone}"

}

与java代码互掉  获取当前的日期

task customProperty{

println "age:${age}"
println "name:${name}"
println "phone:${phone}"

def time = buildTime();
println "${time}"


}


def buildTime(){

def data = new Date();

def formatData = data.format("yyyyMMdd");
return formatData;


}

**task**

**配置项**

type:基于一个存在的task来创建 和我们的类继承差不多

overwriter是否替换存在的task 这个和type配合起来使用

dependsOn 用于配置任务的依赖

action 添加任务的一个action  或者一个闭包 

description 对task的描述

group 对配置任务的分组 


//创建task常见的两种方式
1.
task customTask{
description '描述信息 '

dependsOn task

doFirst{

}

doLast{

}

}

2.
//tasks 其实是所有task的一个map集合 

tasks.create('taskname'){

}

**多种方式访问task ** 

taskName.doLast();

tasks['taskname'].doLast();

**操作符 **

<<  是 doLast方法的短标记形式

task 内部维护一个 list<action>集合 

task  customTask2(type:MyRunnable){

dependsOn customTask

doFirst{
println 'doFirst'
}

println 'do'

doLast{
println 'dolast'
}


}

class MyRunnable extends DefaultTask{


//注解的意思是 这就是任务本身要执行的方法 

@TaskAction

def doSelf(){
println 'doself'
}

}

customTask.enable = false;表示禁用 任务 不让执行 

任务排序 ：：taskB.shouldRunAfter(taskA)  taskB.mustRunAfter(taskA)

任务的onlyif断言 
打包的时候 判断 当task.onlyif做判断 返回false则 不需要打包 

taskName.onlyif{
return false ;
}

gralde example48 : build 
gradle build_apps =shoufa :example48:build



## gradle插件 ##

插件:插件是为了解决 某一问题甚至各种问题 在gradle的基础上提供复用的扩展 

功能：

1.添加任务到项目 完成一些任务 ：eg:测试 编译 打包 
2：添加依赖配置到项目中 比如编译的时候依赖的的三方库 
。。。、

**如何应用插件 **

**二进制插件**

apply plugin "com.android.application"

**应用脚本插件**

//这并不是一个正真的插件 只是一个脚本 本质上是将脚本加载进来 脚本可以是本地的也可以是网络的 （http url）

apply from:'version.gradle'

应用第三方发布的插件

第三方发布的 作为jar的二进制插件我们的在应用的时候必须要在buildScript{}中配置classpath 才能使用 这个不像gradle内置的插件 eg:我们android gradle插件 就属于android 发布的三方插件 要使用就必须先配置 

buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.2'
        //热修复插件
        classpath 'com.dodola:rocoofix:1.2.6'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}


buildscript{}块是一个在构建项目之前为项目进行前期准备和初始化相关的配置依赖 配置好所需的依赖的、、，就应用插件了 

如果没有在buildscript{}中配置依赖的classpath会提示找不到插件


万能的开源社区 https://plugins.gradle.org/

自定义插件


## java gradle插件 ##

apply plugin :'java'

main 和test 是java插件为我们内置的默认的两个源码集合 

自己也可以手动添加

eg:新建一个vip文件夹 下面建java文件 res文件 

在build.gradle中配置 sourceSets{
vip{
}
}


**如何配置第三方的依赖 **

首先要配置依赖repositories  eg:mavenCentral  jcenter
也可以本地搭建maven服务器 配置url

maven{
url '';
}


denpendencies{

 compile  group:"com.quareup.okhttp3",name :"okhttp",version:"3.0.1"

}


每个依赖都由三部分组成的: group name version  

简写 compile "com.squareup.okhttp3:okhttp:3.0.1"

**几种依赖** 

compile  编译时依赖 
runtime  运行时依赖 
testCompile 测试时依赖 
archives  项目发布构建时依赖
default 默认依赖 
debugCompile 
也可指定源集在编译和运行的依赖

mainCompile ""  针对main源集下面的进行依赖

**项目依赖**

compile project ("");

**文件依赖**
依赖jar包 由于不能发布到 jceter端

//依赖一个jar文件

compile files('libs/layoutlib.jar')

依赖libs目录下所有的jar包 

compile fileTree(dir:'libs',include:['*.jar'])


**构建任务**

gradle build //构建整个项目 会打debug  release包

gradle clean  //删除build目录 以及其他构建生成的文件

gradle assembleDebug //构建debug包

gradle assembleRelease //构建release包 

check  只会执行单元测试 

javadoc 生成java格式的docapi文档

**源码集合 sourceSet概念**

sourceSets{
main{
//配置信息


}
}

name  String  main  

output.classesDir  File  源集编译后的class文件目录 

output.resourcesDir  file  编译生成的资源目录 

compileClassPath  编译源集所需要的classpath 

java  该源集的java源文件

java.srcDirs  该源集的java源文件所在目录 

resources 该源集的资源文件 

resources.srcDirs   该源集的资源文件所在目录 


sourceSets{
main{

java{
//这样就把java源集更改到此目录了 

srcDir 'src/java'

}
}

}