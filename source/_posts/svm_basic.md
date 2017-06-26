---
title: SVM basic
date: 2017-06-26 13:33:41
tags: [Data mining, Machine learning, SVM]
---

### SVM(Support Vector Machines)

## 運作原理
使用分隔超平面(separating hyperplane)，將線性可分的數據分隔開來；在超平面同側的資料，屬於同一個類別。

## 如何挑選分隔超平面
找到離超平面最近的點，並且讓它們離分隔面越遠越好。支援向量(support vector)及為離超平面最近的那些點；而支援向量到超平面的距離稱為間隔(margin)。在挑選超平面時，追求一個擁有最大間隔的超平面，保證在處理未知資料時的分類或回歸效果最佳。

## 演算法參數
- 鬆弛變量(slack variable)
    - 許多時候數據並非能切分的很乾淨，引入鬆弛變量後，允許有些數據點可以被切分至錯誤的一側。
- RBF kernel(Radial basis function kernel)
    - gamma: 影響每個支持向量對應的高斯的作用範圍。當gamma越大，則支援向量越少。而支援向量的數量影響訓練與預測的速度。


## 核函數(kernel)
在數據集非線性可分的情況下，可以透過核函數將數據集原本所在的特徵空間，映射至另一個更高維度的特徵空間中，在高維空間中，透過SVM解決線性可分的問題。目前應用較廣的核函數為線性核與高斯核。


## 優點
- 泛化錯誤率低
- 容易解释
- 計算複雜度較低


## 缺點
- 對優化參數、核函數以及核函數中的參數較敏感
- 解決多分類問題時，需用額外的方式進行擴展
- 對噪聲敏感


## Python 示例
```python
    from sklearn.svm import SVC
    from sklearn.metrics import accuracy_score, precision_score, recall_score

    # Type of data is DataFrame
    X = data.iloc[:, data.columns != "label"].values
    y = data.iloc[:, data.columns == "label"].values

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=0)

    # Using the linear kernel to build the SVC classifier
    # classifier = SVC(C=1, kernel='linear', random_state=0)

    # Using the RBF kernel to build the SVC classifier
    classifier = SVC(C=1, kernel='rbf', random_state=0)
    classifier.fit(X_train, y_train.ravel())

    y_pred = classifier.predict(X_test)
    print "accuracy: %f" %accuracy_score(y_test, y_pred)
    print "precision: %f" %precision_score(y_test, y_pred, average="macro")
    print "recall: %f" %recall_score(y_test, y_pred, average="macro")
```
