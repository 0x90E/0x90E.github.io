---
title: YARN Basics
date: 2017-08-30 11:41:58
tags: [Hadoop, Big data, YARN]
---

## YARN

### Introduction
- [Apache Hadoop YARN官網](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html "YARN")
- 分布式資源調度框架，用以提高分布式集群環境下的資源利用率。
- 將資源管理功能(ResourceManager)與作業調度和監控(ApplicationMaster)分別使用不同的Daemon進行。
- 集群資源統一管理，支援大部份大數據處理框架，例如MapReduce、Spark、Storm等，降低運維成本。


### Core components
- Client
    - 提交啟動/結束一個應用程式/作業。
- ResourceManager
    - 全局主節點。
    - 可以做到主備，但Active的只有一個，另一個做備援。
    - 負責整個集群的資源管理和調度。
    - 處理Client請求(啟動/殺死一個作業)。
- NodeManager
    - 每個節點上的資源管理和Task的運行情況。
    - 接收來自ApplicationMaster的Container的啟動/停止的各種命令。
    - 定期透過Heartbeat向ResourceManager匯報當前節點的資源情況。
- ApplicationMaster
    - 每個應用程式/作業對應一個ApplicationMaster。
    - 負責應用程式/作業的管理。
    - 數據切分。
    - 為應用程式/作業向ResourceManager申請資源(Container)，並分配給Task。
    - 與NodeManager溝通以啟動/停止Container。
    - Task的監控與容錯。
- Container
    - 動態資源分配單元。
    - 對Task的運行狀況的資源量進行描述，包含cpu、memory、硬碟空間、網路等。


### Application submission in YARN
<img src="/2017/08/30/yarn-basics/YARN_Process.svg" width="800">

1. Client向ResourceManager提交應用程式，其中包含ApplicationMaster程式、啟動ApplicationMaster的命令、應用程式等。
2. ResourceManager為該應用程式分配一個Container。
3. 
    - ResourceManager與對應的NodeManager溝通，要求NodeManager在這個Container啟動應用程式的ApplicationMaster。
    - 若運行過程中，ApplicationMaster異常終止了，NodeManager將通知ResourceManager。ResourceManager將會重啟一個Container執行ApplicationMaster。
4. 
    - ApplicationMaster向ResourceManager註冊自己，並且與ResourceManager保持Heartbeat。
    - ApplicationMaster採用輪詢的方式透過RPC協議向ResourceManager申請和領取Container。，並監控Task的運行狀態，直到運行結束。
5. 
    - Container申請成功後，由ApplicationMaster進行初始化，然後ApplicationMaster與對應的NodeManager溝通，要求NodeManager啟動Container。
    - ApplicationMaster與NodeManager保持Heartbeat，用以對Task進行監控與管理。
6. 
    - NodeManager啟動對應的Container。
    - Container透過RPC協議向對應的ApplicationMaster匯報任務狀態與進度，讓ApplicationMaster掌握各個Task的運行狀態，並在Task失敗時重新啟動之。
    - 應用程式運行結束後，ApplicationMaster向ResourceManager註銷自己，並允許屬於它的Container被回收。


#### Run a application on YARN
下面說明提交一個應用程序/作業到YARN上面執行。
``` bash
[hadoop@hadoop-01 ~]$cp /opt/software/hadoop/sbin  ## 進入Hadoop的sbin目錄
[hadoop@hadoop-01 sbin]$ ./start-yarn.sh  ## 執行YARN
starting yarn daemons
starting resourcemanager, logging to /opt/software/hadoop-2.8.1/logs/yarn-hadoop-resourcemanager-hadoop-01.out
hadoop-01: starting nodemanager, logging to /opt/software/hadoop-2.8.1/logs/yarn-hadoop-nodemanager-hadoop-01.out
[hadoop@hadoop-01 sbin]$ ./start-dfs.sh  ## 執行HDFS
Starting namenodes on [localhost]
localhost: starting namenode, logging to /opt/software/hadoop-2.8.1/logs/hadoop-hadoop-namenode-hadoop-01.out
hadoop-01: starting datanode, logging to /opt/software/hadoop-2.8.1/logs/hadoop-hadoop-datanode-hadoop-01.out
Starting secondary namenodes [hadoop-01]
hadoop-01: starting secondarynamenode, logging to /opt/software/hadoop-2.8.1/logs/hadoop-hadoop-secondarynamenode-hadoop-01.out
[hadoop@hadoop-01 sbin]$ jps  ## 透過jps命令，確認YARN與HDFS相關行程正常啟動
21888 NameNode
22001 DataNode
21105 NodeManager
22212 SecondaryNameNode
20996 ResourceManager
22325 Jps
``` 
{% img /2017/08/30/yarn-basics/yarn_process_1.png 1000 %}
透過Web頁面，確認YARN已正常運行中，並等待應用程式/作業上傳。
<br>
``` bash
[hadoop@hadoop-01 sbin]$ cd ../bin  ## 進入Hadoop bin目錄
[hadoop@hadoop-01 bin]$ hadoop
Usage: hadoop [--config confdir] [COMMAND | CLASSNAME]
  jar <jar>            run a jar file
                       note: please use "yarn jar" to launch
                             YARN applications, not this command.
## 透過hadoop jar <jar> 命令，上傳一個mapreduce應用程式
## hadoop-mapreduce-examples-2.8.1.jar為Hadoop自帶的示例
[hadoop@hadoop-01 bin]$ hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar pi 10 10
17/08/30 01:32:51 INFO mapreduce.JobSubmitter: number of splits:10
17/08/30 01:32:51 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1504026603603_0001
17/08/30 01:32:52 INFO impl.YarnClientImpl: Submitted application application_1504026603603_0001
17/08/30 01:32:52 INFO mapreduce.Job: The url to track the job: http://hadoop-01:8088/proxy/application_1504026603603_0001/
17/08/30 01:32:52 INFO mapreduce.Job: Running job: job_1504026603603_0001
17/08/30 01:33:06 INFO mapreduce.Job: Job job_1504026603603_0001 running in uber mode : false
17/08/30 01:33:06 INFO mapreduce.Job:  map 0% reduce 0%
17/08/30 01:33:49 INFO mapreduce.Job:  map 60% reduce 0%
17/08/30 01:34:10 INFO mapreduce.Job:  map 80% reduce 0%
17/08/30 01:34:11 INFO mapreduce.Job:  map 90% reduce 0%
17/08/30 01:34:12 INFO mapreduce.Job:  map 100% reduce 0%
17/08/30 01:34:13 INFO mapreduce.Job:  map 100% reduce 100%
17/08/30 01:34:13 INFO mapreduce.Job: Job job_1504026603603_0001 completed successfully
17/08/30 01:34:13 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=226
		FILE: Number of bytes written=1503073
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=2650
		HDFS: Number of bytes written=215
		HDFS: Number of read operations=43
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=3
	Job Counters 
		Launched map tasks=10
		Launched reduce tasks=1
		Data-local map tasks=10
		Total time spent by all maps in occupied slots (ms)=324597
		Total time spent by all reduces in occupied slots (ms)=21100
		Total time spent by all map tasks (ms)=324597
		Total time spent by all reduce tasks (ms)=21100
		Total vcore-milliseconds taken by all map tasks=324597
		Total vcore-milliseconds taken by all reduce tasks=21100
		Total megabyte-milliseconds taken by all map tasks=332387328
		Total megabyte-milliseconds taken by all reduce tasks=21606400
	Map-Reduce Framework
		Map input records=10
		Map output records=20
		Map output bytes=180
		Map output materialized bytes=280
		Input split bytes=1470
		Combine input records=0
		Combine output records=0
		Reduce input groups=2
		Reduce shuffle bytes=280
		Reduce input records=20
		Reduce output records=0
		Spilled Records=40
		Shuffled Maps =10
		Failed Shuffles=0
		Merged Map outputs=10
		GC time elapsed (ms)=5918
		CPU time spent (ms)=12010
		Physical memory (bytes) snapshot=2271346688
		Virtual memory (bytes) snapshot=22685822976
		Total committed heap usage (bytes)=1648820224
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=1180
	File Output Format Counters 
		Bytes Written=97
Job Finished in 83.85 seconds
Estimated value of Pi is 3.20000000000000000000
``` 
{% img /2017/08/30/yarn-basics/yarn_process_2.png 1000 %}
剛開始執行作業時的Web頁面，對應的Status欄位會顯示Running
<br>
{% img /2017/08/30/yarn-basics/yarn_process_2.png 1000 %}
作業執行完畢時的Web頁面，對應的Status欄位會顯示FINISHED，FinalStatus會顯示SUCCEEDED。生產環境上，則會顯示對應的作業運行狀態。

