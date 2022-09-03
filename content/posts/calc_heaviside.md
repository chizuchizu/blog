---
title: "SNNにおけるHeaviside関数の微分の構成(PyTorch実装)"
date: 2022-09-03T20:48:21+09:00

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
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
math: true

---

ここで語らないこと。
- なぜSNNでHeaviside関数が必要なのか
	- SNNをSurrogate Gradient法で学習させるという前提条件をおいてます。
- なぜ微分しなければいけないのか
	- 誤差逆伝播法を使いたいからです。

## Heaviside関数は微分できない

Heaviside関数は次のように定義されます。
$$
H(x)=
\begin{cases}
1 & (x > 0) \\
0 & (x \leq 0)
\end{cases}
$$

Heaviside関数はいわゆるステップ関数の一種で、原点で微分ができない関数です。

無理やり微分を構成しようと思っても、$x=0$で傾きが無限大に飛ぶようなものでしょうか。
$$
\dfrac{dH(x)}{dx} = \delta(x)
$$

## Heaviside関数の微分をどう構成するか
### より多くの場所で微分係数が0以外になってほしい
勾配消失してしまうのを防ぐためです。

### SNNの特性から仮定をおく
「ニューロンが発火するとき(H(x)のxが0を超える時)にはxは0に近い」という仮定をおきます。一見意味不明かもしれませんが、次のIFモデルのシミュレーションの図を見てください。
![](/img/simulation_of_if_model.png#center)

上のシミュレーションでは、閾値を1としています。ニューロンが発火する時点で膜電位はほぼ1です。つまり、この時点で何かしら意味のある傾きの値を与えても良さそうです。

このような仮定から、次のように構成するのが一つの解決策になります。snnTorchというライブラリもこの方法を採用しています[^1]。
$$
\dfrac{dH(x)}{x} = H(x)
$$
以上を踏まえて、$H(x)$と、その微分係数をプロットしてみました。意味不明ですが、これでいいんです。
![](/img/heaviside_calc.png)

## PyTorchでの構成
公式ドキュメントの実装[^2]を参考に自動微分に対応したHeaviside関数を作成してみました。
[^2]: torch.autograd.function.FunctionCtx.save_for_backward — PyTorch 1.12 documentation. _Pytorch.org_. Retrieved September 3, 2022 from https://pytorch.org/docs/stable/generated/torch.autograd.function.FunctionCtx.save_for_backward.html

```python
import torch


class Heaviside(torch.autograd.Function):  
    def forward(ctx, input_):  
        out = (input_ > 0).float()  
        ctx.save_for_backward(out.bool())  # 保存する  
        return out  
  
    def backward(ctx, grad_output):  
        (out,) = ctx.saved_tensors  # {0, 1}  
        grad = grad_output * out  # {0, 1}  
        return grad
```


[^1]: Tutorial 5 - Training Spiking Neural Networks with snntorch — snntorch 0.5.3 documentation. _Readthedocs.io_. Retrieved September 3, 2022 from https://snntorch.readthedocs.io/en/latest/tutorials/tutorial_5.html

‌