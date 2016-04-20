title: 单点Hadoop搭建过程中错误
date: 2014-11-05 16:49:29
categories: 笔记
tags: Hadoop
---
Hadoop 安装过程中问题 

## A fatal error has been detected by the Java Runtime Environment:  

参考: [http://blog.163.com/wf_shunqiziran/blog/static/17630720920125286153575/](http://blog.163.com/wf_shunqiziran/blog/static/17630720920125286153575/)中的方法二

<!--more-->
## java.io.IOException: File /user/root/input could only be replicated to 0 nodes, instead of 1

参考: [http://www.linuxidc.com/Linux/2012-03/56220.htm](http://www.linuxidc.com/Linux/2012-03/56220.htm)

```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com hadoop-1.2.1]$ bin/stop-all.sh
Warning: $HADOOP_HOME is deprecated.

stopping jobtracker
jiangnan04@localhost's password: 
localhost: stopping tasktracker
stopping namenode
jiangnan04@localhost's password: 
localhost: no datanode to stop
jiangnan04@localhost's password: 
localhost: stopping secondarynamenode
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com hadoop-1.2.1]$ bin/start-all.sh
Warning: $HADOOP_HOME is deprecated.

starting namenode, logging to /home/users/jiangnan04/tools/hadoop-1.2.1/libexec/../logs/hadoop-jiangnan04-namenode-cp01-rdqa-dev361.cp01.baidu.com.out
jiangnan04@localhost's password: 
localhost: starting datanode, logging to /home/users/jiangnan04/tools/hadoop-1.2.1/libexec/../logs/hadoop-jiangnan04-datanode-cp01-rdqa-dev361.cp01.baidu.com.out
jiangnan04@localhost's password: 
localhost: starting secondarynamenode, logging to /home/users/jiangnan04/tools/hadoop-1.2.1/libexec/../logs/hadoop-jiangnan04-secondarynamenode-cp01-rdqa-dev361.cp01.baidu.com.out
starting jobtracker, logging to /home/users/jiangnan04/tools/hadoop-1.2.1/libexec/../logs/hadoop-jiangnan04-jobtracker-cp01-rdqa-dev361.cp01.baidu.com.out
jiangnan04@localhost's password: 
localhost: starting tasktracker, logging to /home/users/jiangnan04/tools/hadoop-1.2.1/libexec/../logs/hadoop-jiangnan04-tasktracker-cp01-rdqa-dev361.cp01.baidu.com.out
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com hadoop-1.2.1]$ bin/hadoop fs -put conf input
Warning: $HADOOP_HOME is deprecated.

[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com hadoop-1.2.1]$ bin/hadoop jar hadoop-examples-*.jar grep input output 'dfs[a-z.]+'
Warning: $HADOOP_HOME is deprecated.

14/11/05 16:42:23 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
14/11/05 16:42:23 WARN snappy.LoadSnappy: Snappy native library not loaded
14/11/05 16:42:23 INFO mapred.FileInputFormat: Total input paths to process : 21
14/11/05 16:42:24 INFO mapred.JobClient: Running job: job_201411051640_0001
14/11/05 16:42:25 INFO mapred.JobClient:  map 0% reduce 0%
14/11/05 16:42:30 INFO mapred.JobClient:  map 9% reduce 0%
14/11/05 16:42:32 INFO mapred.JobClient:  map 19% reduce 0%
14/11/05 16:42:33 INFO mapred.JobClient:  map 28% reduce 0%
14/11/05 16:42:35 INFO mapred.JobClient:  map 38% reduce 0%
14/11/05 16:42:37 INFO mapred.JobClient:  map 47% reduce 12%
14/11/05 16:42:39 INFO mapred.JobClient:  map 57% reduce 12%
14/11/05 16:42:41 INFO mapred.JobClient:  map 66% reduce 12%
14/11/05 16:42:42 INFO mapred.JobClient:  map 76% reduce 12%
14/11/05 16:42:44 INFO mapred.JobClient:  map 85% reduce 12%
14/11/05 16:42:46 INFO mapred.JobClient:  map 95% reduce 22%
14/11/05 16:42:48 INFO mapred.JobClient:  map 100% reduce 22%
14/11/05 16:42:52 INFO mapred.JobClient:  map 100% reduce 33%
14/11/05 16:42:54 INFO mapred.JobClient:  map 100% reduce 100%
14/11/05 16:42:54 INFO mapred.JobClient: Job complete: job_201411051640_0001
14/11/05 16:42:54 INFO mapred.JobClient: Counters: 30
14/11/05 16:42:54 INFO mapred.JobClient:   Job Counters 
14/11/05 16:42:54 INFO mapred.JobClient:     Launched reduce tasks=1
14/11/05 16:42:54 INFO mapred.JobClient:     SLOTS_MILLIS_MAPS=36382
14/11/05 16:42:54 INFO mapred.JobClient:     Total time spent by all reduces waiting after reserving slots (ms)=0
14/11/05 16:42:54 INFO mapred.JobClient:     Total time spent by all maps waiting after reserving slots (ms)=0
14/11/05 16:42:54 INFO mapred.JobClient:     Launched map tasks=21
14/11/05 16:42:54 INFO mapred.JobClient:     Data-local map tasks=21
14/11/05 16:42:54 INFO mapred.JobClient:     SLOTS_MILLIS_REDUCES=23896
14/11/05 16:42:54 INFO mapred.JobClient:   File Input Format Counters 
14/11/05 16:42:54 INFO mapred.JobClient:     Bytes Read=37815
14/11/05 16:42:54 INFO mapred.JobClient:   File Output Format Counters 
14/11/05 16:42:54 INFO mapred.JobClient:     Bytes Written=180
14/11/05 16:42:54 INFO mapred.JobClient:   FileSystemCounters
14/11/05 16:42:54 INFO mapred.JobClient:     FILE_BYTES_READ=82
14/11/05 16:42:54 INFO mapred.JobClient:     HDFS_BYTES_READ=40197
14/11/05 16:42:54 INFO mapred.JobClient:     FILE_BYTES_WRITTEN=1305144
14/11/05 16:42:54 INFO mapred.JobClient:     HDFS_BYTES_WRITTEN=180
14/11/05 16:42:54 INFO mapred.JobClient:   Map-Reduce Framework
14/11/05 16:42:54 INFO mapred.JobClient:     Map output materialized bytes=202
14/11/05 16:42:54 INFO mapred.JobClient:     Map input records=1061
14/11/05 16:42:54 INFO mapred.JobClient:     Reduce shuffle bytes=202
14/11/05 16:42:54 INFO mapred.JobClient:     Spilled Records=6
14/11/05 16:42:54 INFO mapred.JobClient:     Map output bytes=70
14/11/05 16:42:54 INFO mapred.JobClient:     Total committed heap usage (bytes)=4421976064
14/11/05 16:42:54 INFO mapred.JobClient:     CPU time spent (ms)=10760
14/11/05 16:42:54 INFO mapred.JobClient:     Map input bytes=37815
14/11/05 16:42:54 INFO mapred.JobClient:     SPLIT_RAW_BYTES=2382
14/11/05 16:42:54 INFO mapred.JobClient:     Combine input records=3
14/11/05 16:42:54 INFO mapred.JobClient:     Reduce input records=3
14/11/05 16:42:54 INFO mapred.JobClient:     Reduce input groups=3
14/11/05 16:42:54 INFO mapred.JobClient:     Combine output records=3
14/11/05 16:42:54 INFO mapred.JobClient:     Physical memory (bytes) snapshot=4446228480
14/11/05 16:42:54 INFO mapred.JobClient:     Reduce output records=3
14/11/05 16:42:54 INFO mapred.JobClient:     Virtual memory (bytes) snapshot=11182075904
14/11/05 16:42:54 INFO mapred.JobClient:     Map output records=3
14/11/05 16:42:54 INFO mapred.FileInputFormat: Total input paths to process : 1
14/11/05 16:42:54 INFO mapred.JobClient: Running job: job_201411051640_0002
14/11/05 16:42:55 INFO mapred.JobClient:  map 0% reduce 0%
14/11/05 16:42:59 INFO mapred.JobClient:  map 100% reduce 0%
14/11/05 16:43:07 INFO mapred.JobClient:  map 100% reduce 33%
14/11/05 16:43:08 INFO mapred.JobClient:  map 100% reduce 100%
14/11/05 16:43:09 INFO mapred.JobClient: Job complete: job_201411051640_0002
14/11/05 16:43:09 INFO mapred.JobClient: Counters: 30
14/11/05 16:43:09 INFO mapred.JobClient:   Job Counters 
14/11/05 16:43:09 INFO mapred.JobClient:     Launched reduce tasks=1
14/11/05 16:43:09 INFO mapred.JobClient:     SLOTS_MILLIS_MAPS=4483
14/11/05 16:43:09 INFO mapred.JobClient:     Total time spent by all reduces waiting after reserving slots (ms)=0
14/11/05 16:43:09 INFO mapred.JobClient:     Total time spent by all maps waiting after reserving slots (ms)=0
14/11/05 16:43:09 INFO mapred.JobClient:     Launched map tasks=1
14/11/05 16:43:09 INFO mapred.JobClient:     Data-local map tasks=1
14/11/05 16:43:09 INFO mapred.JobClient:     SLOTS_MILLIS_REDUCES=8602
14/11/05 16:43:09 INFO mapred.JobClient:   File Input Format Counters 
14/11/05 16:43:09 INFO mapred.JobClient:     Bytes Read=180
14/11/05 16:43:09 INFO mapred.JobClient:   File Output Format Counters 
14/11/05 16:43:09 INFO mapred.JobClient:     Bytes Written=52
14/11/05 16:43:09 INFO mapred.JobClient:   FileSystemCounters
14/11/05 16:43:09 INFO mapred.JobClient:     FILE_BYTES_READ=82
14/11/05 16:43:09 INFO mapred.JobClient:     HDFS_BYTES_READ=301
14/11/05 16:43:09 INFO mapred.JobClient:     FILE_BYTES_WRITTEN=116951
14/11/05 16:43:09 INFO mapred.JobClient:     HDFS_BYTES_WRITTEN=52
14/11/05 16:43:09 INFO mapred.JobClient:   Map-Reduce Framework
14/11/05 16:43:09 INFO mapred.JobClient:     Map output materialized bytes=82
14/11/05 16:43:09 INFO mapred.JobClient:     Map input records=3
14/11/05 16:43:09 INFO mapred.JobClient:     Reduce shuffle bytes=82
14/11/05 16:43:09 INFO mapred.JobClient:     Spilled Records=6
14/11/05 16:43:09 INFO mapred.JobClient:     Map output bytes=70
14/11/05 16:43:09 INFO mapred.JobClient:     Total committed heap usage (bytes)=401997824
14/11/05 16:43:09 INFO mapred.JobClient:     CPU time spent (ms)=1750
14/11/05 16:43:09 INFO mapred.JobClient:     Map input bytes=94
14/11/05 16:43:09 INFO mapred.JobClient:     SPLIT_RAW_BYTES=121
14/11/05 16:43:09 INFO mapred.JobClient:     Combine input records=0
14/11/05 16:43:09 INFO mapred.JobClient:     Reduce input records=3
14/11/05 16:43:09 INFO mapred.JobClient:     Reduce input groups=1
14/11/05 16:43:09 INFO mapred.JobClient:     Combine output records=0
14/11/05 16:43:09 INFO mapred.JobClient:     Physical memory (bytes) snapshot=325132288
14/11/05 16:43:09 INFO mapred.JobClient:     Reduce output records=3
14/11/05 16:43:09 INFO mapred.JobClient:     Virtual memory (bytes) snapshot=1023143936
14/11/05 16:43:09 INFO mapred.JobClient:     Map output records=3
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com hadoop-1.2.1]$ bin/hadoop fs -get output output
Warning: $HADOOP_HOME is deprecated.

[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com hadoop-1.2.1]$ cat output/*
cat: output/_logs: Is a directory
1	dfs.replication
1	dfs.server.namenode.
1	dfsadmin
```