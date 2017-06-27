---
title: RF Basic
date: 2017-06-28 00:20:02
tags: [Data mining, Machine learning, RF]
---
## Random Forsest

## 定義
RF為Bagging的擴展變形。RF以決策樹為基學習器，並在決策樹訓練時，引入隨機屬性選擇。

## 產生基學習器
建立每一顆決策樹皆包含行採樣、列採樣與完全分裂三個步驟:

1. 行採樣: 目的是挑選進行訓練的數據。採用有放回的方式，及為採樣得到的樣本集中，可能會有重複的樣本。假設，輸入N個樣本，那麼採樣的樣本也為N個。在訓練時，每一顆樹輸入樣本都不是全部的樣本，使得相對不容易over-fitting。

2. 列採樣: 目的是挑選進行訓練的特徵，從M個特徵中，挑m個(m << M)。

3. 完全分裂: 使用完全分裂的方式對採樣後的數據建立決策樹，這樣決策樹的某一個葉子節點要麼是無法繼續分裂，不然就是裡面的樣本全屬一同一個分類。

與其他決策樹算法不同的是，這裡不進行剪枝，因為前兩個隨機採樣的過程保證了隨機性，就算不進行剪枝，也不會出現over-fitting

## Bagging
為一種並行式集成學習方法，基於自助採樣法(bootstrap sampling)，在有N個樣本的數據集中，隨機挑選一個樣本，然後將樣本放回，下次隨機挑選時，仍有機會挑中相同的樣本。根據公式推算，約有63.2%的樣本會被用來進行訓練。
透過上述步驟，可採樣出T個包含N個樣本的子數據集，對每個子數據集進行訓練，則會得到T個基學習器，再將這些基學習器進行結合。
進行結合時，Bagging通常對分類問題採用簡單投票法，對回歸問題採用簡單平均法。

## 優點
- 在數據集上的表現良好
- 在當前很多數據集上，相對其他算法有著很大的優勢
- 能夠處理很高維度(feature很多)的數據，並且不用做特徵選擇
- 在訓練完後，它能給出哪些feature比較重要
- 在創建隨機森林時，對 generlization error使用的是無偏估計
- 訓練速度快
- 在訓練過程中，能夠檢測到feature間的互相影響
- 容易做成並行化方法
- 實現比較簡單

## 缺點
- 模型規模較大

## Python 示例
```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, precision_score, recall_score

# Type of data is DataFrame
X = data.iloc[:, data.columns != "label"].values
y = data.iloc[:, data.columns == "label"].values
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=0)
classifier = RandomForestClassifier(random_state=0)
classifier.fit(X_train, y_train.ravel())

y_pred = classifier.predict(X_test)
print "accuracy: %f" %accuracy_score(y_test, y_pred)
print "precision: %f" %precision_score(y_test, y_pred, average="macro")
print "recall: %f" %recall_score(y_test, y_pred, average="macro")
```


