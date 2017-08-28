---
title: HDFS Basics
date: 2017-08-28 12:15:01
tags: [Hadoop, Big data, HDFS]
---

## HDFS(Hadoop Distributed File System)

### Introduction
- [Apache HDFS官網](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html "HDFS")
- 可運行在普通且廉價的硬體上
- 高容錯能力
- 提供應用程式對數據的高吞吐量(high throughput)訪問，適用於具有大數據集的應用程式

### Assumptions and Goals
- Hardware Failure
    - 快速硬體錯誤偵測與自動復原機制
- Streaming Data Access
    - 供數據批次處理的應用程式使用，並追求數據訪問的高吞吐量
- Large Data Sets
    - 支持大文件存儲，並支持高聚合數據带寬及高擴展性
- Simple Coherency Model
    - 支持write-once-read-many access的文件存取模型
- Moving Computation is Cheaper than Moving Data
    - 將應用程式的計算移動到數據所在節點附近進行運算，降低網路阻塞，並提高數據吞吐量
- Portability Across Heterogeneous Hardware and Software Platforms
    - 支持平台間的可移植性


### Core components
HDFS是一個主從式架構(master/slave)，一個HDFS集群(cluster)內，包含一個NameNode與多個DataNode。HDFS提供文件系統命名空間(File System Namespace)給使用者，讓使用者將數據儲存在文件中。下列分別說明NameNode與DataNodes。
- NameNode
    - Master(單個)
    - 管理File System Namespace，例如文件開啟、關閉、重新命名文件或資料夾   
    - 決定Block儲存的DataNode
    - 負責響應客戶端的請求，以及文件存取權限控制
    - 負責管理以下數據:
        - 元數據(Metadata)管理:
            - 文件名稱
            - 文件屬性(權限、創建時間、副本係數)
        - 文件目錄結構
        - 文件對應的Block，以及Block所存放的DataNode
    
- DataNodes
    - Slaves(一個以上)
    - 一般是一個節點執行一個DataNode
    - 存放Block
    - 執行Block的創建、刪除與副本操作
    - 每隔3秒向NameNode發送Heartbeat
        - Heartbeat表示該DataNode運作正常
    - 每隔10個Heartbeat(30秒)，向NameNode發送BlockReport
        - BlockReport包含該DataNode上所有的Block列表


- 其他說明:
    - NameNode與DataNodes皆可運行在普通的機器上。
    - NameNode與DataNodes皆使用JAVA實作，故任何支援JAVA運行的機器皆可執行。
    - NameNode與DataNodes一般是運行在不同的機器上


### Blocks
- 在HDFS中，文件會根據hdfs-default.xml或hdfs-site.xml中的dfs.blocksize參數進行切割成Blocks。
- dfs.blocksize預設為128M，當檔案大於128M時，則會進行切割，直到最後一個Block等於或小於128M。
- 寫入時，文件會拆分成多個Block儲存在不同的DataNode。
- 讀取時，則從DataNodes中取出Blocks進行合併。


### File System Namespace
- File System Namespace的結構與傳統檔案系統相同，並且支持對檔案的創建、刪除、搬移，重新命名等操作。
- 支持用戶容量配額與訪問權限控制。
- 不支持硬連結與軟連結。
- NameNode負責管理File System Namespace。
- 任何File System Namespace與其屬性的改變，都記錄在NameNode中。
- 應用程序可以設置文件的副本係數，副本係數的資訊儲存在NameNode中。



### Data Replication
- HDFS設計目的是為了在跨機器的集群中，提供高可靠的超大文件儲存服務。
- 為了高容錯能力，將Blocks進行多副本方式儲存。
- 每個文件的Block大小與副本係數都是可以設置的。
- 副本係數是指一個Block副本的數量。
- 預設副本係數為3。生產環境設為5或7。
- 副本係數可以在文件創建設置，也可以在之後進行修改。
- HDFS文件都是一次性寫入的(write-once)，並嚴格要求一次只能有一個Client進行寫入。
- NameNode管理Block的複製，並週期性的接收DataNode的Heartbeat與BlockReport，
- 根據BlockReport，對寫入文件的副本進行合理的安排。


### Replica Selection
- 優先將一個Block的副本放置在不同的機器上。
- 分為積架環境與一般環境說明:
    - 一般環境:
        - 在DataNode中執行文件寫入:
            - 挑選當前DataNode進行第一個Block副本放置處，節省網路傳輸成本。
            - 其他副本則以不重複原則，進行隨機挑選。
        - 在其他節點執行文件寫入:
            - 以不重複原則，進行隨機挑選。
    - 積架環境:
        - 在DataNode中執行文件寫入:
            - 挑選當前DataNode進行第一個Block副本放置處。節省網路傳輸成本。
            - 優先挑選不同的積架中的DataNode進行寫入，直到所有副本處理完畢。
        - 在其他節點執行文件寫入:
            - 根據資源狀況挑選DataNode。
            - 優先挑選不同的積架中的DataNode進行寫入，直到所有副本處理完畢。
