---
title: "KaTeXに対応させた話"
date: 2022-03-23T08:26:59+09:00

author: "chizuchizu"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
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

私は，HugoでPaperModと呼ばれるテーマを使ってこのブログを作っているのですが，デフォルトでは数式の表示に対応していないらしいです．

## どうなるか
### インラインモード
```
$x$と$y$という変数を足したらどうなるかな．$\sum_{k=0}^n \frac{x^k}{y}$にはならないよ．
```

$x$と$y$という変数を足したらどうなるかな．$\sum_{k=0}^n \frac{x^k}{y}$にはならないよ．
### ディスプレイモード
```
$$
x + y \neq \sum_{k=0}^n \frac{x^k}{y}
$$
```
$$
x + y \neq \sum_{k=0}^n \frac{x^k}{y}
$$

## どうやるのか

PaperModのデモサイト[^1]では，数式表示に対応させる方法が書いてあります．
[^1]: https://adityatelange.github.io/hugo-PaperMod/posts/math-typesetting/

`layouts/partials/extend_head.html`に以下のコードを追加してください．KaTeXのドキュメント[^2]のコードをコピペです．
[^2]: https://katex.org/docs/autorender.html

```javascript
{{ if or .Params.math .Site.Params.math }}
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.15.3/dist/katex.min.css" integrity="sha384-KiWOvVjnN8qwAZbuQyWDIbfCLFhLXNETzBQjA/92pIowpC0d2O3nppDGQVgwd2nB" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.15.3/dist/katex.min.js" integrity="sha384-0fdwu/T/EQMsQlrHCCHoH10pkPLlKA1jL5dFyUOvB3lfeT2540/2g6YgSi2BL14p" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.15.3/dist/contrib/auto-render.min.js" integrity="sha384-+XBljXPPiv+OzfbB3cVmLHf4hdUFHlWNZN5spNQ7rmHTXpd7WvJum6fIACpNNfIR" crossorigin="anonymous"
    onload="renderMathInElement(document.body);"></script>
<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
          // customised options
          // • auto-render specific keys, e.g.:
          delimiters: [
              {left: '$$', right: '$$', display: true},
              {left: '$', right: '$', display: false},
              {left: '\\(', right: '\\)', display: false},
              {left: '\\[', right: '\\]', display: true}
          ],
          // • rendering keys, e.g.:
          throwOnError : false
        });
    });
</script>
{{ end }}

```


すると，デフォルトではKaTeXがオフになりますが，markdownファイルのトップに`math: true`とすることでKaTeXを有効化させることができます．

```markdown
---
title: "KaTeXに対応させた話"
date: 2022-03-23T08:26:59+09:00

author: "chizuchizu"
...
math: true

---
```

KaTeXの読み込みは遅いらしいので，デフォルトはオフにしたほうが良いらしいです．