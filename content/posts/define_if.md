---
title: "SNN寄りなIFモデルの定義とPythonのデモ"
date: 2022-09-03T16:29:59+09:00

author: "chizuchizu"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
math: true

---

Integrate-and-Fire(以下IF) neuron modelは、ニューロンの動きを簡単にモデル化したものです。

このチュートリアルでは次の事を書きます。
- SNNよりなIFモデルの定義
- IFモデルをシミュレートしたコードの紹介

## IFモデルの定義
そもそも、IFモデルの定義は色々あります。こういうニューロンモデルは医学寄り（神経科学的な視点）と工学寄り（NN的な視点）のものがあるので、一意に定まりません。工学寄りでも論文によって定義が違うこともよくあります。もちろん、基本となる考え方は同じですが。

参考までに医学寄りと工学寄りなIFモデルの差を示しておきます。これは主観です。

医学寄りなモデルの特徴[^1]
- 時間は連続的
	- 積分で表現される
- 入力は時間依存した電流
- 適切な初期値を割り当てる
	- 静止膜電位は-75mVみたいに

工学寄りなモデルの特徴
- 時間は離散的
	- 総和で表現される
- 入力は過去のシナプスの発火ベクトルと重みベクトルの内積
	- NNと同じ
- 計算しやすい定数を用いる
	- 静止膜電位は0にすることが多い


あるSNNの論文[^2]に登場したIFモデルの定義です。
[^2]: Wu, H., Zhang, Y., Weng, W., Zhang, Y., Xiong, Z., Zha, Z.-J., Sun, X. and Wu, F. 2021. Training Spiking Neural Networks with Accumulated Spiking Flow. _Proceedings of the AAAI Conference on Artificial Intelligence_. 35, 12 (May 2021), 10320-10328.



$$
x^{t+1,n}_{i} = \sum_j w\_{ij}^{n} o^{t+1,n-1}_i
$$
$$
u^{t+1,n}_i = u^{t,n}_i (1-o^{t,n}_i) + x^{t+1,n}_i
$$

$$
o^{t+1,n}_i = H(u^{t+1,n}_i - V\_{th})
$$


式中に出てくる変数は次のとおりです。
- $u^{t,n}_i$は時刻$t$における、$n$層の$i$番目のニューロンの膜電位を表す。
- $x^{t,n}_i$は時刻$t$における、$n$層の$i$番目のニューロンの入力(電流)を表す。
- $w^{n}\_{ij}$は、$n-1$層の$i$番目のニューロンから、$n$層の$j$番目のニューロンへ伝播する際の重みを表す。
- $o^{t,n}_i$は時刻$t$において、$n$層の$i$番目のニューロンが発火したかを表す。(0か1)
- $V_{th}$はニューロンがどれだけの膜電位に達したらニューロンが発火するかを表す。
- $H$はHeaviside関数を表す。

## IFモデルのシミューレーション

IFモデルのシミュレーションを次に示します。

![](/img/simulation_of_if_model.png#center)

定義に従って書きました。入力は端折って乱数使ってます。

```python
import numpy as np  
import matplotlib.pyplot as plt  
  
n_timestep = 100  
# これがx_tに相当する  
random_input = np.random.random(n_timestep) / 15  
V = 0  # 膜電位に初期値  
threshold = 1  
spikes = np.zeros_like(random_input)  # 時刻tで発火したか  
V_list = np.zeros_like(random_input)  # 時刻tの膜電位  
  
  
def Heaviside(x):  
    return int(x > 0)  
  
  
for t in range(n_timestep):  
    V = V * (1 - spikes[t - 1]) + random_input[t]  
    V_list[t] = V  
  
    spikes[t] = Heaviside(V - threshold)  
  
print(V_list)  
plt.plot(V_list)  
plt.hlines(1, 0, n_timestep, color="red", linestyles="dotted")  
plt.xlabel("Timestep")  
plt.ylabel("Membrane potential")  
plt.title("Simulation of IF model")  
plt.show()
```

[^1]: Wulfram Gerstner. 2014. 1.3 Integrate-And-Fire Models | Neuronal Dynamics online book. _Epfl.ch_. Retrieved September 3, 2022 from https://neuronaldynamics.epfl.ch/online/Ch1.S3.html

‌