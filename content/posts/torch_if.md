---
title: "IFモデルをPyTorchで動かす"
date: 2022-09-03T23:43:53+09:00

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
math: false

---

都合よく微分を構成したHeaviside関数の実装については、別記事で紹介しています。（リンクを貼る）
ここで使っているIFモデルについても、別記事で紹介しています。（リンクを貼る）
## IFモデルのPyTorch実装
IFモデルを実装すると次のようになります。

```python
import torch  
  
  
# https://pytorch.org/docs/stable/generated/torch.autograd.function.FunctionCtx.save_for_backward.html  
class Heaviside(torch.autograd.Function):  
    def forward(ctx, input_):  
        out = (input_ > 0).float()  
        ctx.save_for_backward(out.bool())  # 保存する  
        return out  
  
    def backward(ctx, grad_output):  
        (out,) = ctx.saved_tensors  # {0, 1}  
        grad = grad_output * out  # {0, 1}  
        return grad  
  
  
class IF(torch.nn.Module):  
    def __init__(self, threshold=1):  
        super().__init__()  
  
        self.threshold = threshold  
  
    def forward(self, v_previous, vt, s_previous):  
        x = v_previous * (1 - s_previous) + vt  
        spikes = Heaviside.apply(x - self.threshold)  
        return spikes
```

## デモ1. ある時刻だけで動かしてみる。
ここで、入力$x$と$o$は恒等的に0としています。ある時刻$t$におけるニューロンを40個用意し、膜電位を`arange`で初期化しています。IFモデルは閾値を超えるとスパイクが発火するので、Heaviside関数のグラフのようになることが期待されます。

次のようなコードを書きました。先程書いた`IF`モジュールはPyTorchで動く活性化関数として使えます。

```python
v_previous = torch.zeros(int(2 / 0.05), requires_grad=True)  
s_previous = torch.zeros(int(2 / 0.05), requires_grad=True)  
vt = torch.arange(0, 2, 0.05, requires_grad=True)  
IF_Layer = IF(threshold=1)  
o = IF_Layer(v_previous, vt, s_previous)  
# B = Heaviside.apply(A)  
print(f"{vt = }")  
print(f"{o = }")  
print(f"{vt.grad = }")  # ちなみにこの段階では、勾配は計算されていないので、Noneが出るはず  
C = torch.sum(o)  # 総和をとる（勾配を計算させために）  
C.backward()  # 勾配を計算  
print("backwardした")  
print(f"{vt.grad = }")
```

実行すると次のような出力が得られます。

```python
vt = tensor([0.0000, 0.0500, 0.1000, 0.1500, 0.2000, 0.2500, 0.3000, 0.3500, 0.4000,
        0.4500, 0.5000, 0.5500, 0.6000, 0.6500, 0.7000, 0.7500, 0.8000, 0.8500,
        0.9000, 0.9500, 1.0000, 1.0500, 1.1000, 1.1500, 1.2000, 1.2500, 1.3000,
        1.3500, 1.4000, 1.4500, 1.5000, 1.5500, 1.6000, 1.6500, 1.7000, 1.7500,
        1.8000, 1.8500, 1.9000, 1.9500], requires_grad=True)
o = tensor([0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
        0., 0., 0., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1.,
        1., 1., 1., 1.], grad_fn=<HeavisideBackward>)
vt.grad = None
backwardした
vt.grad = tensor([0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
        0., 0., 0., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1.,
        1., 1., 1., 1.])
```

`vt`が膜電位にあたるわけですが、今回設定した閾値は1になるので、真ん中あたりからニューロンが発火することが期待されます。出力は、その期待通りに真ん中あたりからニューロンが発火しています。更に、微分係数も0と1になっていることがわかります。ニューロンが発火しなければ勾配は0、 発火すれば1になります。

## デモ2. 時々刻々と変化させてみる。
