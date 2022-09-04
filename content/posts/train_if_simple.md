---
title: "IFモデルを活性化関数としたSNNモデルを訓練する"
date: 2022-09-04T09:50:01+09:00

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

IFモデルを訓練させます。重みが狙った方向に変化することだけを確認したいので、意味のないタスクを設定しています。(出力がすべて0になることを期待)

## 要件
- 入力
	- 0 or 1の乱数
- ターゲット
	- すべて0

### モデルの構成
1. 入力
2. ループ(タイムステップ分)
	1. 全結合層
	2. IFモデルを挟む(活性化関数の役割)。
		- 0 or 1の値を伝播させる。
3. 得られたスパイク値の総和をとり、Loss計算に回す。

`torchinfo`というライブラリ[^1]で表示したモデルの構成を示しておきます。RNNモデルですね。
[^1]: TylerYep. 2022. TylerYep/torchinfo: View model summaries in PyTorch! _GitHub_. Retrieved September 4, 2022 from https://github.com/TylerYep/torchinfo

‌
```python
==========================================================================================
Layer (type:depth-idx)                   Output Shape              Param #
==========================================================================================
Model                                    [10, 500]                 --
├─Linear: 1-1                            [500]                     150,500
├─IF: 1-2                                [500]                     --
├─Linear: 1-3                            [500]                     (recursive)
├─IF: 1-4                                [500]                     --
├─Linear: 1-5                            [500]                     (recursive)
├─IF: 1-6                                [500]                     --
├─Linear: 1-7                            [500]                     (recursive)
├─IF: 1-8                                [500]                     --
├─Linear: 1-9                            [500]                     (recursive)
├─IF: 1-10                               [500]                     --
├─Linear: 1-11                           [500]                     (recursive)
├─IF: 1-12                               [500]                     --
├─Linear: 1-13                           [500]                     (recursive)
├─IF: 1-14                               [500]                     --
├─Linear: 1-15                           [500]                     (recursive)
├─IF: 1-16                               [500]                     --
├─Linear: 1-17                           [500]                     (recursive)
├─IF: 1-18                               [500]                     --
├─Linear: 1-19                           [500]                     (recursive)
├─IF: 1-20                               [500]                     --
==========================================================================================
Total params: 150,500
Trainable params: 150,500
Non-trainable params: 0
Total mult-adds (M): 752.50
==========================================================================================
Input size (MB): 0.01
Forward/backward pass size (MB): 0.00
Params size (MB): 0.60
Estimated Total Size (MB): 0.62
==========================================================================================
```

### Lossの定義
予測値(0 or 1)の平均が0になるようにしてMSEをLossとします。意味のないタスクです。
```python
loss = (pred.mean() - 0) ** 2
```

## IFモデルの実装
PyTorchのコードは次に示します。

```python
import torch  
import torch.optim as optim  
from torch import nn  
  
import matplotlib.pyplot as plt  
  
  
class Heaviside(torch.autograd.Function):  
    def forward(ctx, input_):  
        out = (input_ > 0).float()  
        ctx.save_for_backward(out.bool())  # 保存する  
        return out  
  
    def backward(ctx, grad_output):  
        (out,) = ctx.saved_tensors  # {0, 1}  
        grad = grad_output * out  # {0, 1}  
        return grad  
  
  
class IF(nn.Module):  
    def __init__(self, threshold=1):  
        super().__init__()  
  
        self.threshold = threshold  
  
    def forward(self, input_, v_previous, s_previous):  
        x = v_previous * (1 - s_previous) + input_  
        spikes = Heaviside.apply(x - self.threshold)  
        return x, spikes


class Model(nn.Module):  
    def __init__(self, size_input, size_hidden, n_time=10):  
        super(Model, self).__init__()  
        self.linear = nn.Linear(size_input, size_hidden)  
        nn.init.uniform_(self.linear.bias, 0, 1.0)  
        self.if_layer = IF()  
  
        self.n_time = n_time  
        self.size_hidden = size_hidden  
  
    def forward(self, x):  
        spikes = torch.zeros(size=(self.n_time + 1, self.size_hidden))  
  
        v = torch.tensor([0])  
  
        for timestep in range(self.n_time):  
            hidden_tensor = self.linear(x[timestep])  
            v, spikes[timestep + 1] = self.if_layer(hidden_tensor, v, spikes[timestep])  
  
        return spikes[1:]
```

途中で次のような記述がありました。
```python
nn.init.uniform_(self.linear.bias, 0, 1.0)
```
これをしないと、最初のステップでほとんどのニューロンが発火しないので学習が進まないことがわかっています。一つの解決策として、層のバイアスを0から1までの一様分布からサンプリングすることが挙げられます。

## IFモデルの訓練
次のように実装しました。入力は適当に1 or 0の乱数としています。
```python
model = Model(size_input=300, size_hidden=500)  
optimizer = optim.SGD(model.parameters(), lr=1e-1, momentum=0.9)  
loss_list = []  
  
for j in range(100):  
    X = torch.randint(low=0, high=2, size=(n_time_step, 300)).float()  
  
    pred = model(X)  
    loss = (pred.mean() - 0) ** 2  
    loss_list.append(loss.item())  
    print(loss.item())  
    loss.backward()  
    optimizer.step()  
    optimizer.zero_grad()  
  
    print(pred.mean())  
  
plt.plot(loss_list)  
plt.ylabel("Training Loss")  
plt.xlabel("Num iteration")  
plt.show()
```

学習が終わると、次のようなLossのグラフが表示されます。
![](/img/if_train_loss_1.png)

Lossが下がるように重みが更新されています。Heaviside関数で、めちゃくちゃな勾配を定義していましたが、こうして逆伝播されて勾配降下法が適用できていることを確認できました。
