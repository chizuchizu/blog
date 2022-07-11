---
title: "LIFモデルの解き方（微分方程式）"
date: 2022-07-11T16:05:07+09:00

author: "chizuchizu"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "LIFモデルを解きます。"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
math: true

---

簡略化したLIFモデルの解き方をまとめました。

LIFモデルは、[Tutorial 2 - The Leaky Integrate-and-Fire Neuron — snntorch 0.5.1 documentation](https://snntorch.readthedocs.io/en/latest/tutorials/tutorial_2.html#lapicque-step-input)を参考にしました。

このモデルは本来のLIFモデル[^2]を簡略化したものです。LIFモデルを次のように定義します。
$$
\tau \frac{dU}{dt} = -U + RI(t)
$$

**定数変化法**[^1]という手法を用いて微分方程式を解きます。

LIFモデルを式変形すると、次のような非同次1階線型微分方程式の形になります。
$$
\frac{dU}{dt} + \frac{U}{\tau} = \frac{RI(t)}{\tau}
$$

## 1. 同次形にして解く
まずは上で得られた式の右辺を無視し、同次形にした方程式を解きます。
$$
\frac{dU}{dt} + \frac{U}{\tau} =0
$$
変数分離形なので、次のように変形します。
$$
\frac{1}{U}dU = -\frac{1}{\tau}dt
$$
両辺をそれぞれ積分すると次のようになります。$C_1$は任意定数です。
$$
\log{U} = \frac{t}{\tau} + C_1
$$
Uについてまとめます。$C$は任意定数ですが、ここで$C$を$t$の関数だとみなします。
$$
U = \pm e^{-\frac{t}{\tau}}e^{C_1}\\
=Ce^{-\frac{t}{\tau}}\\
=C(t)e^{-\frac{t}{\tau}}
$$
上で得られた結果を用いて、次のように$U$を$t$で微分します。
$$
\frac{dU}{dt} = C'(t)e^{-\frac{t}{\tau}} + C(t)(-\frac{t}{\tau}e^{-\frac{t}{\tau}}) \\
= e^{-\frac{t}{\tau}}(C'(t) - \frac{1}{\tau}C(t))
$$
## 2. 得られた結果を元の方程式に代入する
先程得られた$\frac{dU}{dt}$を非同次形の方程式に代入します。
$$
e^{-\frac{t}{\tau}}(C'(t) - \frac{1}{\tau}C(t)) + \frac{C(t)e^{-\frac{t}{\tau}}}{\tau} = \frac{RI(t)}{\tau}
$$
$$
e^{-\frac{t}{\tau}}C'(t) = \frac{RI(t)}{\tau}
$$
$C'(t)$について次のようにまとめます。
$$
C'(t) = \frac{RI(t)}{\tau}e^{\frac{t}{\tau}}
$$
$C'(t)$を積分して、$C(t)$を求めます。$C$は任意定数です。
$$
C(t) = RI(t) e^{-\frac{t}{\tau}} + C
$$
結局、LIFモデルの解は次のように表されます。
$$
U = C(t) e^{-\frac{t}{\tau}}\\
= (RI(t) e^{-\frac{t}{\tau}} + C)e^{\frac{t}{\tau}}\\
= RI(t) + Ce^{-\frac{t}{\tau}} 
$$

## 3. 初期条件を考える
ここでは、時刻$t$における膜電位を$U(t)$と表しています。
膜電位$U(0)=U_0$という初期条件を課すと、次のような解が得られます。
$$
U=RI(t) + [U_0 - RI(t)]e^{-\frac{t}{\tau}}
$$
もし、$U_0=0$ならば、更に簡素な表現が得られます。
$$
U=RI(t)[1 - e^{-\frac{t}{\tau}}]
$$

このようにして、微分方程式として与えられていたLIFモデルを解くことができました。

[^1]: [一階線形微分方程式 - EMANの物理数学](https://eman-physics.net/math/differential07.html)
[^2]: [Leaky integrate-and-fire モデル — Juliaで学ぶ計算論的神経科学](https://compneuro-julia.github.io/neuron-model/lif.html)に詳しく書かれています。今回私が例に上げているのは静止膜電位が0のケースを考えています。