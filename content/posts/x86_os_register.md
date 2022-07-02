---
title: "汎用レジスタとフラグレジスタ"
date: 2022-07-02T11:03:32+09:00

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

# 汎用レジスタ
- AX
	- Accumulator Register
	- 算術演算に使われる
- BX
	- Base Register
	- ポインタレジスタとして使う
- CX
	- Count Register
	- 繰り返すプログラムで，暗黙的にカウンタとして使われる
- DX
	- Data Register
		- 乗算，除算で使われる
## サイズ
- AX(16bit)
	- AH
		- 上位8bit
	- AL
		- 下位8bit
- BX
	- BH
	- BL
- CX
	- CH
	- CL
- DX
	- DH
	- DL
# フラグレジスタ
- DF
	- Direction Flag
- IF
	- Interrupt Enable Flag
- TF
	- Trap Flag
- **OF**
	- Overflow Flag
	- 算術結果の結果がビット幅に収まらなかった時
- **SF**
	- Sign Flag
	- 演算結果の最上位bitが入る(sign)
- **ZF**
	- Zero Flag
	- 演算結果がゼロのとき
- AF
	- Auxiliary Carry Flag
- PF
	- Parity Flag
- **CF**
	- Carry Flag
	- 符号なし演算
		- AX < BXのとき