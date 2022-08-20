---
title: "LIFモデルの解き方（微分方程式）"
date: 2022-07-11T16:05:07+09:00

author: "chizuchizu"
showToc: true
TocOpen: false
draft: false
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

LIFモデルの解き方をまとめました。
# LIFモデルの定義
basic integrate-and-fire model[^3]と呼ばれるモデルを使います。定義は次のとおりです。


$$
\tau \frac{du(t)}{dt} = -u(t) + RI(t)
$$

ここで、解きやすくするために次の条件を課します。
- 一定の入力電流
	- $I(t) = I_0$
- 静止膜電位は0
	- $u_r = 0$
- $t=t^{(1)}$のとき、スパイクが発火する
	- $u(t^{(1)})=0$

条件を課すと、LIFモデルは次のように書けます。
$$
\tau \frac{du(t)}{dt} = -u(t) + RI_0
$$
# LIFモデルを解く
定数変化法[^1]という手法を用いて微分方程式を解きます。
LIFモデルを変形すると、次のような非同次1階線形法微分方程式の形になります。
$$
\frac{du(t)}{dt} + \frac{u(t)}{\tau} = \frac{RI_0}{\tau}
$$

## 1. 同次形にして解く
まずは上で得られた式の右辺を無視し、同次形にした方程式を解きます。
$$
\frac{du(t)}{dt} + \frac{u(t)}{\tau} =0
$$
変数分離形なので、次のように変形します。
$$
\frac{1}{u(t)}du = -\frac{1}{\tau}dt
$$
両辺をそれぞれ積分すると次のようになります。$C_1$は任意定数です。
$$
\log{u(t)} = \frac{t}{\tau} + C_1
$$
uについてまとめます。$C$は任意定数ですが、ここで$C$を$t$の関数だとみなします。
$$
u(t) = \pm e^{-\frac{t}{\tau}}e^{C_1}\\
=Ce^{-\frac{t}{\tau}}\\
=C(t)e^{-\frac{t}{\tau}}
$$
上で得られた結果を用いて、次のように$U$を$t$で微分します。
$$
\frac{du(t)}{dt} = C'(t)e^{-\frac{t}{\tau}} + C(t)(-\frac{t}{\tau}e^{-\frac{t}{\tau}}) \\
= e^{-\frac{t}{\tau}}(C'(t) - \frac{1}{\tau}C(t))
$$
## 2. 得られた結果を元の方程式に代入する
先程得られた$\frac{du(t)}{dt}$を非同次形の方程式に代入します。
$$
e^{-\frac{t}{\tau}}(C'(t) - \frac{1}{\tau}C(t)) + \frac{C(t)e^{-\frac{t}{\tau}}}{\tau} = \frac{RI_0}{\tau}
$$
$$
e^{-\frac{t}{\tau}}C'(t) = \frac{RI_0}{\tau}
$$
$C'(t)$について次のようにまとめます。
$$
C'(t) = \frac{RI_0}{\tau}e^{\frac{t}{\tau}}
$$
$C'(t)$を積分して、$C(t)$を求めます。$C$は任意定数です。
$$
C(t) = RI_0 e^{-\frac{t}{\tau}} + C
$$
結局、LIFモデルの解は次のように表されます。
$$
u(t) = C(t) e^{-\frac{t}{\tau}}\\
= (RI_0 e^{-\frac{t}{\tau}} + C)e^{\frac{t}{\tau}}\\
= RI_0 + Ce^{-\frac{t}{\tau}} 
$$

## 3. 初期条件を課す
ここでは、時刻$t$における膜電位を$u(t)$と表しています。
膜電位$u(t^{(1)})=0$という初期条件を課すと、次のような解が得られます。
$$
U=RI(t)[1 - e^{-\frac{t - t^{(1)}}{\tau}}]
$$

このようにして、微分方程式として与えられていたLIFモデルを解くことができました。

[^1]: [一階線形微分方程式 - EMANの物理数学](https://eman-physics.net/math/differential07.html)
[^3]: Gerstner, W., & Kistler, W. (2002). _Spiking Neuron Models: Single Neurons, Populations, Plasticity_. Cambridge: Cambridge University Press. doi:10.1017/CBO9780511815706