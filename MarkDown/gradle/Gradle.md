# Gradle

## 1、下载安装

* 下载地址 https://services.gradle.org/distributions/
* 安装：添加一个 gradle 的环境变量即可
    * 注意：gradle 在环境变量中的位置必须在 JDK 之后



## 2、配置文件 .gradle

* **repositories 标签**
    * 指定使用的仓库

```groovy

repositories {
    mavenCentral()
}
```

* **dependencies 标签**
    * 依赖坐标

```groovy
// testCompile 表示在测试类中起作用，compile 表示在整个项目中起作用
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

## 3、添加阿里云仓库

* 在 gradle 目录下的 init.d 目录中新建 init.gralde 文件，内容如下

```groovy
allprojects {
    repositories {
        maven { 
			url 'http://maven.aliyun.com/nexus/content/repositories/central/' 
		}
    }
}
```

## 4、jar 包存放路径

```java
F:\Gradle\gradle-6.4\gradle-6.4\caches\modules-2\files-2.1
```

 

































