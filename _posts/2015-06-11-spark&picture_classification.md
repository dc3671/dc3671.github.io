---
layout: post
title:  "基于EMR及Spark运行Picture Classification"
date:   2015-06-11 12:00:00
categories: Spark
excerpt: AWS EMR Spark PictureClassification Amazon
---

* content
{:toc}


## 前期准备

### 配置spark环境

openjdk的问题，出现64位机器依赖32位包的情况

    sudo dpkg --add-architecture i386
    sudo apt-get update
    sudo apt-get install openjdk-7-jre:i386  

解决冲突，rpm强制删除（不删除依赖）之后再安装

    rpm -qa | grep openjdk
    rpm --erase --nodeps java-1.8.0-openjdk-headless...
    yum install zookeeper

在[这里](http://www.igniterealtime.org/downloads/index.jsp)下载了spark2.7.0的预编译版，可以直接使用，将之移动到/usr/local/spark下，加入环境变量即可。

### 数据转换成json

read all the images into memory as RDD and my method is as following:
    
    val images = spark.wholeTextFiles("hdfs://imag-dir/")

合并一维numpy.array：

    np.hstack((a,b))

合并二维numpy.array，第三个参数：

    np.append(a,b,0)

二维numpy.array转list，为了作为json存储：
    
    a.tolist()

运行过程出错，cPickle依赖pylearn2

安装scipy：

    yum install lapack lapack-devel blas blas-devel
    pip install numpy
    pip install scipy

过程中遇到跟新的gcc-4.6.2.6冲突，手动回退到4.6.2.1，利用上面的rpm --erase --nodeps删除，再用yum重装指定版本即可。

用union合并多个数据：

    scala> val rdd = sc.parallelize(List(('a',1),('a',2)))
    scala> val rdd2 = sc.parallelize(List(('b',1),('b',2)))
    scala> rdd union rdd2
    scala> res3.collect
    res4: Array[(Char, Int)] = Array((a,1), (a,2), (b,1), (b,2))

## 初步尝试

### 减少非必要输出

根据[教程](http://blog.jobbole.com/86232/)的指示，先配置一下。

Spark（和PySpark）的执行可以特别详细，很多INFO日志消息都会打印到屏幕。开发过程中，这些非常恼人，因为可能丢失Python栈跟踪或者print的输出。为了减少Spark输出 – 你可以设置$SPARK_HOME/conf下的log4j。首先，拷贝一份$SPARK_HOME/conf/log4j.properties.template文件，去掉“.template”扩展名。

    ~$ cp $SPARK_HOME/conf/log4j.properties.template $SPARK_HOME/conf/log4j.properties

编辑新文件，用WARN替换代码中出现的INFO。你的log4j.properties文件类似：

    # Set everything to be logged to the console
     log4j.rootCategory=WARN, console
     log4j.appender.console=org.apache.log4j.ConsoleAppender
     log4j.appender.console.target=System.err
     log4j.appender.console.layout=org.apache.log4j.PatternLayout
     log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n
    # Settings to quiet third party logs that are too verbose
     log4j.logger.org.eclipse.jetty=WARN
     log4j.logger.org.eclipse.jetty.util.component.AbstractLifeCycle=ERROR
     log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=WARN
     log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=WARN

现在运行PySpark，输出消息将会更简略！

### 代码过程

为了将输入的data正确地转化成RDD，需要先json.load，之后再把二维list进行sc.parallelize，从而可以对二维list里的每一行进行map操作。
