---
title: Hadoop Pseudo-Distributed Setup
date: 2017-08-23 12:23:23
tags: [Hadoop, Big data]
---

## Hadoop Pseudo-Distributed Setup

### 偽分布式模式(Pseudo-Distributed Mode)
- 在單個集群上運行Hadoop
- 所有的Hadoop daemon皆運行在不同的Java process
- 在單機模式上增加了程式調適功能，並允許檢查以下狀態:
    - 記憶體使用情況
    - HDFS的輸出/輸入
    - 與其他daemon的交互情況，如namenode，datanode，secondarynamenode，jobtracer，tasktrace


### 部署流程

#### 基本訊息
- OS: CentOS 6.5 64bit
- Hadoop: 2.8.1
- JDK: 8u144

#### 新增Hadoop user
``` bash
[root@hadoop-01 ~]# useradd hadoop
[root@hadoop-01 ~]# vi /etc/sudoers
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
hadoop ALL=(ALL)       ALL    ## 新增此行，賦予sudo權限給hadoop user 
```

#### 設置Hadoop目錄
- Hadoop壓縮包來源可以是:
    - 透過Hadoop source code自行編譯
    - [Hadoop官網下載](http://hadoop.apache.org/releases.html "Hadoop releases")
``` bash
[root@hadoop-01 ~]# cd /opt/software/
[root@hadoop-01 software]# ls
hadoop-2.8.1.tar.gz  ## 已下載的hadoop 2.8.1
[root@hadoop-01 software]# tar -xzvf hadoop-2.8.1.tar.gz
[root@hadoop-01 software]# ls
hadoop-2.8.1
hadoop-2.8.1.tar.gz
```

#### 建立Hadoop目錄軟連結
``` bash
[root@hadoop-01 software]# ln -s c /opt/software/hadoop
[root@hadoop-01 software]# chown hadoop:hadoop hadoop-2.8.1
[root@hadoop-01 software]# chown -R hadoop:hadoop hadoop-2.8.1/*
[root@hadoop-01 software]# chown hadoop:hadoop hadoop
[root@hadoop-01 software]# cd hadoop
[root@hadoop-01 hadoop]# ll
total 156
drwxr-xr-x. 2 hadoop hadoop  4096 Aug 21 02:13 bin
drwxr-xr-x. 3 hadoop hadoop  4096 Aug 21 02:13 etc
drwxr-xr-x. 2 hadoop hadoop  4096 Aug 21 02:13 include
drwxr-xr-x. 3 hadoop hadoop  4096 Aug 21 02:13 lib
drwxr-xr-x. 2 hadoop hadoop  4096 Aug 21 02:13 libexec
-rw-r--r--. 1 hadoop hadoop 99253 Aug 21 02:13 LICENSE.txt
drwxrwxr-x. 2 hadoop hadoop  4096 Aug 21 22:50 logs
-rw-r--r--. 1 hadoop hadoop 15915 Aug 21 02:13 NOTICE.txt
-rw-r--r--. 1 hadoop hadoop  1366 Aug 21 02:13 README.txt
drwxr-xr-x. 2 hadoop hadoop  4096 Aug 21 02:13 sbin
drwxr-xr-x. 3 hadoop hadoop  4096 Aug 21 02:13 share
```

#### 設置環境變量
``` bash
[root@hadoop-01 hadoop]# vi /etc/profile
export HADOOP_HOME=/opt/software/hadoop
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
[root@hadoop-01 hadoop]# source /etc/profile
```

#### 確認ssh service狀態
``` bash
[root@hadoop-01 hadoop]# service sshd status
openssh-daemon (pid  2086) is running...
## 若非上述這種狀態，需進行排查
```

#### 配置Hadoop user與ssh信任關係
``` bash
[root@hadoop-01 hadoop]# su - hadoop
[hadoop@hadoop-01 ~]$ cd /opt/software/hadoop
[hadoop@hadoop-01 hadoop]$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Created directory '/home/hadoop/.ssh'.
Your identification has been saved in /home/hadoop/.ssh/id_rsa.
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
f2:49:22:bd:fb:7b:ca:d1:1a:ef:81:45:74:d0:dc:28 hadoop@hadoop-01
...
[hadoop@hadoop-01 hadoop]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
[hadoop@hadoop-01 hadoop]$ chmod 0600 ~/.ssh/authorized_keys
[hadoop@hadoop-01 hadoop]$ ssh hadoop001 date
...
Mon Aug 21 22:10:12 CST 2017
[hadoop@hadoop-01 hadoop]$ ssh hadoop-01 date
Mon Aug 21 22:10:22 CST 2017
```

#### 配置Hadoop Pseudo-Distributed文件

[Hadoop官方建議配置](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation "Hadoop Pseudo-Distribute")
``` bash
[hadoop@hadoop-01 hadoop]$ vi etc/hadoop/core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>

[hadoop@hadoop-01 hadoop]$ vi etc/hadoop/hdfs-site.xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```


#### HDFS格式化與啟動Hadoop
``` bash
[hadoop@hadoop-01 hadoop]$  vi etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/java/jdk1.8.0_144 ## 將hadoop-env.sh內的JAVA_HOME指定到jdk位置
[hadoop@hadoop-01 hadoop]$ which hdfs
/opt/software/hadoop/bin/hdfs
[hadoop@hadoop-01 hadoop]$ hdfs namenode -format
...
17/08/21 22:22:37 INFO namenode.FSImage: Allocated new BlockPoolId: BP-42224246-192.168.16.91-1503325357194
17/08/21 22:22:37 INFO common.Storage: Storage directory /tmp/hadoop-hadoop/dfs/name has been successfully formatted.  ## 格式化成功訊息
...
[hadoop@hadoop-01 hadoop]$ which start-dfs.sh
/opt/software/hadoop/sbin/start-dfs.sh
[hadoop@hadoop-01 hadoop]$ start-dfs.sh
[hadoop@hadoop-01 hadoop]$ jps ## 使用jps確認hadoop相關process狀態
11586 NameNode
11718 DataNode
11882 SecondaryNameNode
11996 Jps
## 也可以使用瀏覽器訪問http://localhost:50070，確認Hadoop啟動狀態
```
