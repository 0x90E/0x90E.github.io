---
title: HDFS文件讀寫流程
date: 2017-08-28 12:20:51
tags: [Hadoop, Big data, HDFS]
---

## HDFS文件讀寫流程

### 文件寫入流程
{% img /2017/08/28/hdfs-read-write-procedure/HDFS_file_write_procedure.svg 800 %}
1. HDFS Client調用DistributedFileSystem的create()創建一个文件，DistributedFileSystem內部通過RPC呼叫NameNode。
2. NameNode對請求文件進行檢查:
    2.1 文件是否存在
    2.2 檢查Client權限是否合法
3. 通過檢查後，NameNode會創建不包含任何Block訊息的新文件，進行紀錄佔位。
4. 詳細描述如下:
    4.1 DistributedFileSystem會返回FSDataOutputStream，供客戶端寫入數據。
    4.2 FSDataOutputStream包裝了FSOutputStream，可對NameNode與DataNodes進行溝通。
    4.3 DFSOutputStream將文件分割成許多的小數據包，並寫入DataQueue。DataQueue由DataStreamer負責讀取。
    4.4 DataStreamer通知NameNode分配DataNode儲存Block及副本。
5. 根據副本放置策略，NameNode進行DataNode分配，並返回DataNode列表
6. 詳細描述如下:
    6.1 DataStreamer將分配好的DataNode會組成一個Pipeline。假設副本係數為3，所以在Pipeline中會有3個DataNode。
    6.2 DataStreamer將Block流式傳送到Pipeline中的第一個DataNode，第一個DataNode將數據往下傳遞給第二個DataNode，第二個DataNode再傳給第三個DataNode
7. DFSOutputStream維護ackqueue，ackqueue儲存等待DataNode確認的Block，只有當Pipeline內所有的DataNode皆返回寫入成功時，才會從ackqueue刪除，並開始寫入下一個Block。
8. 當目標文件所有的Block寫入完畢，Client通知NameNode寫入操作完畢。


### 文件存取流程
{% img /2017/08/28/hdfs-read-write-procedure/HDFS_file_read_procedure.svg 800 %}
1. HDFS Client調用FileSystem對象的靜態方法open()，打開一個目標文件的路徑。FileSystem對象(DistributedFileSystem對象)內部通過RPC呼叫NameNode。
2. NameNode返回Block列表，該列表視檔案大小情況，包含目標文件部分或全部的Block訊息，對於每個Block，NameNode返回該Block副本的DataNode位址，且DataNode地址會根據與Client的距離來排序。
3. 詳細描述如下:
    3.1 DistributedFileSystem返回一个FSDataInputStream對象供Client讀取數據。
    3.2 FSDataOutputStream包裝了FSOutputStream，可對NameNode與DataNodes進行溝通。
    3.3 使用FSDataInputStream的輸入流調用read()，輸入流會跟據Client與DataNode的距離，挑選DataNode來讀取Block，若Client本身就是DataNode，則從本地獲取數據。
    3.4 讀取完當前Block後，關閉與當前DataNode的連結，並讀取Block列表中下個Block訊息與尋找最佳DataNode。
    3.5 若Block列表讀取完畢，但文件讀取還沒結束，則Client需再次向NameNode獲取下一批Block列表。直到目標文件所有的Blocks讀取完畢。
    3.6 每當讀取完一個Block都會進行checksum校驗，若讀取DataNode時出現錯誤，則Client會通知NameNode，避免下一次錯誤再次發生。而Client則從下一個Block副本的DataNode中讀取數據。
5. 調用FSDataInputStream的close()，通知NameNode讀取操作完畢。


