---
title: NLP N-gram
date: 2017-07-03 00:38:06
tags: [NLP]
---
## N-gram Language Model

### 定義
為一種統計語言模型(Statistical Language Model)，統計語言模型定義:
{% img /2017/07/03/nlp-gram/img_1.svg 500 %}

而N-gram又稱為N元模型，N-gram是指一段語句中包含N個Token，譬如abcde，則2-gram依次為:
ab, bc, cd, de
<br>
### 馬可夫假設(Markov chain)
根據鏈式法則將統計語言模型展開後可得:
{% img /2017/07/03/nlp-gram/img_2.svg 500 %}

這裡利用馬可夫假設進行，縮減參數空間。馬可夫假設的概念為，任一個詞出現的機率，只與前幾個詞有關西。
N-gram模型的N指出要取前幾個詞(包含自己)來計算。N越大，則模型越準確，但也越複雜，且計算量也大。
最常使用的為Bigram，N取≥4的情况的情況較少。
常用的N-gram模型的縮減後為:
1-gram(Unigram):
{% img /2017/07/03/nlp-gram/img_3.svg 300 %}
2-gram(Bigram):
{% img /2017/07/03/nlp-gram/img_4.svg 400 %}
3-gram(Trigram):
{% img /2017/07/03/nlp-gram/img_5.svg 500 %}

學習過程中，利用最大似然法將上述算式的參數求出，並追求將訓練樣本概率的最大值。
<br>
### 平滑方法(Smoothing)
對於語言處理，被證實數據稀疏(Data Sparseness)的存在，故提出數據平滑(Data Smoothing)技術來解決。
數據稀疏指出在測試過程中的語料，沒有在訓練階段出現過。數據平滑技術是增強語言模型的重要手段，而且其效果與訓練集的規模有關，當規模越小，則數據平滑的效果越好，反之，則不顯著，甚至可以忽略。

目前已知效果最好的數據平滑演算法為Modified kneser-Ney Smoothing:

{% img /2017/07/03/nlp-gram/img_6.svg 500 %}
D1、D2、和D3分別用來對應於 Unigram、Bigram 與 Trigram
{% img /2017/07/03/nlp-gram/img_7.svg 100 %}
{% img /2017/07/03/nlp-gram/img_8.svg 150 %}
{% img /2017/07/03/nlp-gram/img_9.svg 150 %}
{% img /2017/07/03/nlp-gram/img_10.svg 150 %}
