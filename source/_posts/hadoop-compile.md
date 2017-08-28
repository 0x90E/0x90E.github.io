---
title: Compile Hadoop source code
date: 2017-08-23 12:33:33
tags: [Hadoop, Big data]
---

##  Compile Hadoop source code 

###  編譯步驟

#### 基本訊息
- OS: CentOS 6.5 64bit
- Hadoop: 2.8.1
- JDK: 8u144
- Maven: 3.3.9
- ProtocolBuffer: 2.5.0
- Findbugs: 1.3.9


####設置Hadoop目錄
- Hadoop source code:
    - [Hadoop官網下載](http://hadoop.apache.org/releases.html "Hadoop releases")
``` bash
[root@hadoop-01 ~]# cd /opt/sourcecode/
[root@hadoop-01 sourcecode]# ls
hadoop-2.8.1-src.tar.gz  ## 已下載的hadoop 2.8.1 source code壓縮包
[root@hadoop-01 sourcecode]# tar -xzvf hadoop-2.8.1-src.tar.gz
[root@hadoop-01 sourcecode]# ls
hadoop-2.8.1-src
hadoop-2.8.1-src.tar.gz
[root@hadoop-01 sourcecode]# cd hadoop-2.8.1-src
[root@hadoop-01 hadoop-2.8.1-src]# ll
total 228
-rw-rw-r--.  1 root root 15623 May 24 07:14 BUILDING.txt
drwxr-xr-x.  4 root root  4096 Aug 21 00:38 dev-support
drwxr-xr-x.  4 root root  4096 Aug 21 02:01 hadoop-assemblies
drwxr-xr-x.  4 root root  4096 Aug 21 02:01 hadoop-build-tools
drwxrwxr-x.  3 root root  4096 Aug 21 02:13 hadoop-client
drwxr-xr-x. 11 root root  4096 Aug 21 02:04 hadoop-common-project
drwxr-xr-x.  3 root root  4096 Aug 21 02:13 hadoop-dist
drwxr-xr-x.  9 root root  4096 Aug 21 02:07 hadoop-hdfs-project
drwxr-xr-x. 10 root root  4096 Aug 21 02:11 hadoop-mapreduce-project
drwxr-xr-x.  4 root root  4096 Aug 21 02:01 hadoop-maven-plugins
drwxr-xr-x.  3 root root  4096 Aug 21 02:13 hadoop-minicluster
drwxr-xr-x.  4 root root  4096 Aug 21 02:01 hadoop-project
drwxr-xr-x.  3 root root  4096 Aug 21 02:01 hadoop-project-dist
drwxr-xr-x. 19 root root  4096 Aug 21 02:13 hadoop-tools
drwxr-xr-x.  4 root root  4096 Aug 21 02:09 hadoop-yarn-project
-rw-rw-r--.  1 root root 99253 May 24 07:14 LICENSE.txt
-rw-------.  1 root root   289 Aug 21 00:45 nohup.out
-rw-rw-r--.  1 root root 15915 May 24 07:14 NOTICE.txt
drwxrwxr-x.  2 root root  4096 Jun  2 14:24 patchprocess
-rw-rw-r--.  1 root root 20477 May 29 06:36 pom.xml
-rw-r--r--.  1 root root  1366 May 20 13:30 README.txt
-rwxrwxr-x.  1 root root  1841 May 24 07:14 start-build-env.sh
```
BUILDING.txt文件中列出編譯的基本需求，以及編譯時需要注意的地方，可以在編譯的過程中對照著看。


####  Java的安裝與配置
``` bash
[root@hadoop-01 ~]# ls cd /opt/software/
[root@hadoop-01 software]# ls
jdk-8u144-linux-x64.tar  ## 已下載的JDK壓縮包
[root@hadoop-01 software]# mkdir -p /usr/java
[root@hadoop-01 software]# mv jdk-8u144-linux-x64.tar /usr/java
[root@hadoop-01 software]# cd /usr/java
[root@hadoop-01 java]# tar -vxf jdk-8u144-linux-x64.tar
[root@hadoop-01 java]# ll
drwxr-xr-x. 8 uucp  143      4096 Jul 22 13:11 jdk1.8.0_144
-rw-r--r--. 1 root root 377835520 Aug 20 22:13 jdk-8u144-linux-x64.tar
[root@hadoop-01 java]# chown -R root:root jdk1.8.0_144
[root@hadoop-01 java]# ll
drwxr-xr-x. 8 root root      4096 Jul 22 13:11 jdk1.8.0_144
-rw-r--r--. 1 root root 377835520 Aug 20 22:13 jdk-8u144-linux-x64.tar
[root@hadoop-01 java]# vim /etc/profile
## 新增下列兩行在文件中的任一位置，設置Java環境變量
export JAVA_HOME=/usr/java/jdk1.8.0_144
export PATH=$JAVA_HOME/bin:$PATH
[root@hadoop-01 java]# source /etc/profile
[root@hadoop-01 java]# which java  ## 確認Java環境變量設置成功
/usr/java/jdk1.8.0_144/bin/java
[root@hadoop-01 java]# java -version
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```

#### Maven的安裝與配置
``` bash
[root@hadoop-01 ~]# cd /opt/software/
[root@hadoop-01 software]# ls
apache-maven-3.3.9-bin.tar.gz  ## 已下載的Maven壓縮包
[root@hadoop-01 software]# tar -zxvf apache-maven-3.3.9-bin.tar.gz
[root@hadoop-01 software]# ls
apache-maven-3.3.9 
apache-maven-3.3.9-bin.tar.gz
[root@hadoop-01 software]# vim /etc/profile
## 新增下列兩行在文件中的任一位置，設置Maven環境變量
export MAVEN_HOME=/opt/software/apache-maven-3.3.9
export PATH=$MAVEN_HOME/bin:$PATH
[root@hadoop-01 software]# source /etc/profile
[root@hadoop-01 software]# mvn -version  ## 確認Maven環境變量設置成功
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /opt/software/apache-maven-3.3.9
Java version: 1.8.0_144, vendor: Oracle Corporation
Java home: /usr/java/jdk1.8.0_144/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.32-431.el6.x86_64", arch: "amd64", family: "unix"
```


####  Autoconf的安裝
``` bash
[root@hadoop-01 ~]# cd /opt/software/
[root@hadoop-01 software]# wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz
[root@hadoop-01 software]# ls
autoconf-2.69.tar.gz
[root@hadoop-01 software]# tar -zxvf autoconf-2.69.tar.gz
[root@hadoop-01 software]# ls
autoconf-2.69
autoconf-2.69.tar.gz
[root@hadoop-01 software]# cd autoconf-2.69 
[root@hadoop-01 autoconf-2.69]# ./configure
[root@hadoop-01 autoconf-2.69]# make & make install
[root@hadoop-01 autoconf-2.69]# autoconf --version  ## 確認Autoconf安裝成功
autoconf (GNU Autoconf) 2.69
```


####  Automake的安裝
``` bash
[root@hadoop-01 ~]# cd /opt/software/
[root@hadoop-01 software]# wget http://ftp.gnu.org/gnu/automake/automake-1.14.tar.gz
[root@hadoop-01 software]# ls
automake-1.14.tar.gz
[root@hadoop-01 software]# tar -xvzf automake-1.14.tar.gz
[root@hadoop-01 software]# ls
automake-1.14
automake-1.14.tar.gz
[root@hadoop-01 software]# cd automake-1.14
[root@hadoop-01 automake-1.14]# ./configure
[root@hadoop-01 automake-1.14]# make & make install
[root@hadoop-01 automake-1.14]# automake --version  ## 確認Automake安裝成功
automake (GNU automake) 1.14
```

#### Protobuf安装
``` bash
[root@hadoop-01 ~]# cd /opt/software/
[root@hadoop-01 software]# ls
protobuf-2.5.0.tar.gz  ## 已下載的Protobuf壓縮包
[root@hadoop-01 software]# tar -zxvf protobuf-2.5.0.tar.gz
[root@hadoop-01 software]# ls 
protobuf-2.5.0 
protobuf-2.5.0.tar.gz
[root@hadoop-01 software]# cd protobuf-2.5.0
[root@hadoop-01 protobuf-2.5.0]# ls
[root@hadoop-01 protobuf-2.5.0]# yum install -y gcc gcc-c++ make cmake
[root@hadoop-01 protobuf-2.5.0]# ./configure --prefix=/usr/local/protobuf
[root@hadoop-01 protobuf-2.5.0]# make && make install
[root@hadoop-01 protobuf-2.5.0]# vim /etc/profile
## 新增下列兩行在文件中的任一位置，設置Protobuf環境變量
export PROTOC_HOME=/usr/local/protobuf
export PATH=$PROTOC_HOME/bin:$PATH
[root@hadoop-01 protobuf-2.5.0]# source /etc/profile
[root@hadoop-01 protobuf-2.5.0]# protoc --version   ## 確認Protobuf環境變量設置成功
libprotoc 2.5.0
```


#### Findbugs安装
``` bash
[root@hadoop-01 ~]# cd /opt/software
[root@hadoop-01 software]# ls
findbugs-1.3.9.zip  ## 已下載的Findbugs壓縮包
[root@hadoop-01 software]# unzip findbugs-1.3.9.zip
[root@hadoop-01 software]# ls
findbugs-1.3.9
findbugs-1.3.9.zip
[root@hadoop-01 software]# vi /etc/profile
## 新增下列兩行在文件中的任一位置，設置Findbugs環境變量
export FINDBUGS_HOME=/opt/software/findbugs-1.3.9
export PATH=$FINDBUGS_HOME/bin:$PATH
[root@hadoop-01 software]# source /etc/profile
[root@hadoop-01 software]# findbugs -version   ## 確認Findbugs環境變量設置成功
1.3.9
```


#### 安装其他依賴
``` bash
[root@hadoop-01 ~]# yum install -y openssl openssl-devel svn ncurses-devel zlib-devel libtool
[root@hadoop-01 ~]# yum install -y snappy snappy-devel bzip2 bzip2-devel lzo lzo-devel lzop
```


#### 進行編譯
``` bash
[root@hadoop-01 ~]# cd /opt/sourcecode/hadoop-2.8.1-src
[root@hadoop-01 hadoop-2.8.1-src]# mvn clean package -Pdist,native -DskipTests -Dtar
## 編譯過程中，需保證網路暢通，因為maven會自動下載依賴的jar檔
## 若在下載的過程中有某個jar卡住，可以使用Ctrl+C退出後，重新輸入編譯命令
## 編譯完成的Hadoop jar放置在 /opt/sourcecode/hadoop-2.8.1-src/hadoop-dist/target/hadoop-2.8.1.tar.gz

```
