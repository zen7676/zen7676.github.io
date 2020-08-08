---
layout: single
title:  "物体検知パイプラインをEnd to Endに学習する(論文メモ)"
date:   2020-07-26 22:45:00 +0900
categories: artifical-intelligence computer-vision
---
"End-to-End Object Detection with Transformers"  
Nicolas Carion, Francisco Massa, Gabriel Synnaeve, Nicolas Usunier, Alexander Kirillov and Sergey Zagoruyko  
https://arxiv.org/abs/2005.12872v3

従来の深層学習による物体検出において、Bipertite matching lossをロス関数とするTransformerを使うことによって、Anchorの設計やNMS(Non Maximum Suppression)といったヒューリスティックな部分をなくし、End-to-Endに学習可能にした、という論文です。

シンプルながら、COCO datasetにおいてFaster R-CNNと同程度の精度を達成し、panoptic segmentationにも容易に適用可能という特徴があります。

## End-to-Endな物体検知
まず、物体検知がやろうとしていることは、画像を入力として、

(物体1のクラス, 物体1の位置座標)  
(物体2のクラス, 物体2の位置座標)  
・・・

を出力することです。  
従来、物体検知の深層学習による学習は、何種類のanchorに対する出力をし、NMSをし、と複雑なことをごちゃごちゃやっていたわけですが、  
素直に、この入力と出力のペアをEnd-to-Endに学習できないか、と考えます。

## Bipertite matching loss 
End-to-Endの物体検知で難しい点は、ニューラルネットワークの予測と正解のロスをどう設計するか、ということです。  
ニューラルネットワークの予測出力として、(物体1のクラス, 物体1の位置座標)、(物体2のクラス, 物体2の位置座標)、...の羅列が出てきますが、順番はどうでもいいので、どういう順番で出力されても正しい、と言えるようにしなければなりません。  
そこで、Bipertite matching lossを使います。Bipertite matching lossは、どの予測がどの正解に対応するかを計算します。  
まず、予測した値\\( {\hat{y}}\_i = ({\hat{c}}\_i, {\hat{b}}\_i) \\)がどの正解の値\\(y\_i = (c\_i, b\_i)\\)に対応するかという1対1の対応関係\\(\sigma(i)\\)を、コスト
\begin{equation}
{\mathcal{L}}\_{\rm match} (y_i,{\hat{y}}\_{\sigma(i)}) = -{\mathbb{1}}\_{\{c\_i \neq \varnothing \}}{\hat{p}}\_{\sigma (i)}(c\_i) + {\mathbb{1}}\_{\{c\_i \neq \varnothing\}} \mathcal{L}\_{\rm box}(b\_i, {\hat{b}}\_{\sigma(i)})
\end{equation}
を最小にするように探します。ここで、
\\[
{\mathbb{1}}\_{\{c\_i \neq \varnothing \}} = 
\begin{cases}
0 & ({\rm if}\ c\_i = \varnothing) \\\
1 & ({\rm otherwise})
\end{cases}
\\]
です。探索方法は、ハンガリー法を使います。  
注意すべきポイントとして、バウンディングボックスの数\\(N\\)は、学習データの1枚の画像に含まれるバウンディングボックスの数以上にしておく必要があるということです。そして、正解のバウンディングボックスが\\(N\\)個未満の場合は、足りない部分を\\(\varnothing\\)(no object)で埋めます。そうすることにより、1対1の対応関係を計算することができます。
<!--例として、5つの予測を出す場合を考えます(Fig.1)。-->
ここで得た1対1の対応関係\\(\hat{\sigma}(i)\\)を使って、学習する際のロスを以下のようにします。
\begin{equation}
{\mathcal{L}}(y,\hat{y}) = \sum\_{i=1}^N [-{\log}\hat{p}\_{\hat{\sigma}(i)}(c\_i) + {\mathbb{1}}\_{\{c\_i \neq \varnothing \}}\mathcal{L}\_{\rm box}(b\_i, {\hat{b}}\_{\hat{\sigma}(i)})]
\end{equation}

## Transformer
![transformer](/assets/endtoend_object_detection.JPG)
Fig.1 Trasnformerを用いた物体検出パイプライン([1]のFig.2)

深層学習のアーキテクチャとして、自然言語処理でスタンダードとなっている、Trasnformerと呼ばれるエンコーダ・デコーダモデルを用います(Fig.1)。

まず、CNNを用いてサイズ\\(d\times H\times W\\)の特徴マップを得ます。これを\\(d\times HW\\)にリシェイプし、Trasnformerのエンコーダに入力します。こうすることで、特徴マップのピクセルとピクセルとの関係をエンコードすることができます。エンコーダはそのままでは入力の順番を無視してしまいますので、"positional encodings"と呼ばれる位置情報も同時に入力します。

次に、デコーダでは\\(N\\)個のバウンディングボックスの予測を同時に出力します。デコーダは、そのままでは\\(N\\)個の異なる出力ができないので、\\(\N\)個の"object queries"と呼ばれる「種」をデコーダに入力します。

最後に、デコーダの出力を全結合ネットワークによって処理してバウンディングボックスの座標とクラスの確率を得ます(Fig.1のFFN(Feed Forward Network)の部分)。

## 精度
COCO dataset(COCO 2017 detection)でAP44.9と、Faster R-CNN(AP44.0)と同程度の精度です。

## Attentionの可視化
![encoder_vis](/assets/encoder_visualize.JPG)
Fig.2 Encoderの可視化([1]のFig.3)

![decoder_vis](/assets/decoder_visualize.JPG)
Fig.2 Decoderの可視化([1]のFig.6)

本手法の面白いポイントとして、エンコーダとデコーダのattentionを可視化することができます(Fig.2,3)。  
Fig.2はエンコーダの最終層のattentionを可視化したもので、物体を一つ一つ区別できていることがわかります。  
Fig.3はデコーダのattentionを可視化したもので、動物を区別するうえで、鼻や足などの部位に注目していることがわかります。

## 所感
現状ではState-of-the-artに比べて性能面では劣るかもしれませんが、これまでの物体検知アルゴリズムよりも細かく作りこむ部分が少なく、シンプルで洗練されており、本手法をベースラインとして進化していくのではないかと感じました。また、自然言語認識で活躍するTransformerが画像認識でも活用できる可能性があり、Transformer×画像認識の研究が促進されるのではないでしょうか。

[1]"End-to-End Object Detection with Transformers"
Nicolas Carion, Francisco Massa, Gabriel Synnaeve, Nicolas Usunier, Alexander Kirillov, Sergey Zagoruyko
https://arxiv.org/abs/2005.12872v3