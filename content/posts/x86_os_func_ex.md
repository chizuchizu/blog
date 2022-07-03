---
title: "[NASM]基本的な構文の記述例とその意味"
date: 2022-07-02T10:37:09+09:00

author: "chizuchizu"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "いくつかのアセンブラ構文の記述例と，その意味についてまとめました．
- if文
- switch ~ case文
- do ~ while文
- for文
- 関数
- メモリのコピー
- メモリの比較"
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
[作って理解するOS x86系コンピュータを動かす理論と実装：書籍案内｜技術評論社](https://gihyo.jp/book/2019/978-4-297-10847-2)
を読んだメモです．

# TL;DR
いくつかのアセンブラ構文の記述例と，その意味についてまとめました．
- if文
- switch ~ case文
- do ~ while文
- for文
- 関数
- メモリのコピー
- メモリの比較

# if文の記述
このようなC言語のコードを書きたいとします．
## 記述例
C言語
```c
if (AX < 3){
	BX = 2;
} else {
	BX = 1;
}
```
アセンブリ
```nasm
Test:
	cmp ax, 3
	jae .Fase
.True:
	mov bx, 2
	jmp .End
.False:
	mov bx, 1
.End:
```

アセンブリは上から実行されるので，`False`を飛ばす仕様になっています．
# switch ~ case文
## 記述例
C言語
```c
switch (AX){
	case 10:
		BX = 1;
		break;
	case 15:
		BX = 2;
		break;
	case 18:
		BX = 3;
		break;
	default:
		BX = 4;
		break;
}
}
```
アセンブリ
```nasm
Test:
	cmp ax, 10
	je .C10
	cmp ax, 15
	je .C15
	cmp ax, 18
	je .C18
	jmp .D
.C10:
	mov bx, 1
	jmp .E
.C15:
	mov bx, 2
	jmp .E
.C18:
	mov bx, 3
	jmp .E
.D:
	mov bx, 4
	jmp .E
.E:
	
```
if文と似た構文です．
# do ~ while文
## 記述例
```c
CX = 5;
do {
	// 様々な処理
} while (--CX);
```
アセンブリ
```nasm
	mov cx, 5
Test:
.L:
	; 様々な処理
	loop .L
```

予めカウントレジスタにループ回数を入れておき，0になるまでデクリメントするやり方ですね．

# for文
C言語
```c
for (short int CX = 0; CX < 5; CX++){
	// 様々な処理
}
```
アセンブリ
```nasm
Test:
	mov cx, 0
.L:
	cmp cx, 5
	jge .E
	; 様々な処理
	inc cx
	jmp .L
.E:
```
loopを使わずにwhile文を実装したような構文です．最初に終了条件を入れておきます．
# 関数
### 記述例
C言語
```c
int func_16(int x, int y){
	short i = 10;
	short j = 20;
	short AX = x;
	AX += y;
	return 1;
}
```
アセンブリ
```nasm
func_16:
	; スタックフレームの構築
	push bp
	mov bp, sp
	sub sp, 2
	push 0
	; 処理の開始
	mov [bp - 2], word 10
	mov [bp - 4], word 20
	
	mov ax, [bp + 4]
	add ax, [bp + 6]
	
	mov ax, 1

	; スタックフレームの破棄
	mov sp, bp
	pop bp
	
	ret
```
### 関数を呼び出したときのスタックフレームの動作
色々な記事や本を漁りましたが，わかりやすい表がなかったので自分がまとめました．

|メモリ番地|意味|関数呼び出し前|関数呼び出し直後|ローカル変数の定義後|SPレジスタの復帰|スタックフレームの破棄後|CALL命令終了時|スタック上の変数の削除後|
|:----|:----|:----|:----|:----|:----|:----|:----|:----|
|BP + 4|変数2| | | | | | | |
|BP + 2|変数1| |↑未定義|↑定義済み|↑未定義| | | |
|BP|BP| |SP|SP|SP|↑未定義| | |
|BP - 2|戻り番地| | | | |SP|↑未定義| |
|BP - 4|引数1| | | | | |SP| |
|BP - 6|引数2|↑未定義| | | | | |↑未定義|
|BP - 8| |SP| | | | | |SP|

用語
- スタックフレーム
	- スタック上にに関数内で使用するローカル変数を定義する構造
- ローカル変数
	- 関数内で使う変数
# メモリコピー
### 記述例
```c
void memcpy(dst, src, size){
	// dst: コピー先
	// src: コピー元
	// size: バイト数
}
```
アセンブリ
```nasm
memcpy:
	; スタックフレームの構築
	push bp
	mov bp, sp

	; レジスタの保存(レジスタをローカル変数として使う)
	push cx
	push si
	push di

	; バイト単位でのコピー
	cld  ; DF = 0 (順方向)
	mov di, [bp + 4]  ; コピー先
	mov si, [bp + 6]  ; コピー元
	mov cx, [bp + 8]  ; 実行回数

	rep movsb  ; バイト単位でコピー

	; レジスタの復帰
	pop di
	pop si
	pop cx

	; スタックフレームの破棄
	mov sp, bp
	pop bp

	ret
```

# メモリの比較
### 記述例
```c
void memcmp(src0, src1, size){
	// src0: アドレス0
	// src1: アドレス1
	// size: バイト数
}
```
アセンブリ
```nasm
memcmp:
	; スタックフレームの構築
	push bp
	mov bp, sp

	; レジスタの保存
	push bx
	push cx
	push dx
	push si
	push di

    ; 引数の取得
    cld  ; DF = 0(順方向)
    mov si, [bp + 4]
    mov di, [bp + 6]
	mov cx, [bp + 8]

	; バイト単位での比較
	repe cmpsb  ; ZFが1になる or ZFが1のまま最後までループされる
	jnz .10Z  ; ZFが1なら飛ばす
	; ZF == 0
	mov ax, 0 
	jmp .10E
.10F:
	; ZF == 1
	mov ax, -1
.10E:

	; レジスタの復帰
	pop di
	pop si
	pop dx
	pop cx
	pop bx

	; スタックフレームの破棄
	mov sp, bp
	pop bp

	ret
```



# 命令の解説
## `cmp`: 整数値の比較
`cmp A B`
- A - Bの結果を使ってフラグレジスタを立てる
	- subとは違ってAに演算結果が書き込まれない
フラグレジスタの変化は以下の通り．
- OF
	- overflow flag
- SF
	- sign flag
- ZF
	- zero flag
	- A == Bのとき1
- CF
	- carry flag
	- A < Bのとき1

### 符号なし演算
命令
- JB
	- jump if below
		- CF == 1
		- A < B
- JBE
	- Jump if Below or Equal
		- CF == 1 || ZF == 1
		- A <= B
- JAE
	- Jump if Above or Equal
		- CF == 0
		- A >= B
- JA
	- Jump if Above
		- CF == 1 && ZF == 0
		- A > B
それぞれの否定命令
A > Bの否定は，A <= Bになりますね．
- JB
	- JNB
- JBE
	- JNA
- JAE
	- JNB
- JA
	- JNBE
### 符号あり演算
A < Bを判定するのには，A - Bが**オーバーフローせず，SFが1**か，**オーバーフローして, SFが0**なので，**SF != OF**とかける．このように，少しややこしくなります．

- JL
	- Jump if Less
		- SF != OF
		- A < B
- JLE
	- Jump if Less or Equal
		- ZF == 1 || SF != OF
		- A <= B
- JGE
	- Jump if Greater or Equal
		- SF == OF
		- A >= B
- JG
	- Jump if Greater
		- ZF == 0 && SF == OF
		- A > B
それぞれの否定演算
- JL
	- JNGE
- JLE
	- JNG
- JGE
	- JNL
- JG
	- JNLE
## `LOOP`: 繰り返し命令
`loop A`
CXをデクリメントし，Aにジャンプします．
|命令名|終了条件|
|:----|:----|
|`loop`|`CX==0`|
|`loppe`/`loopz`|`CX==0` or `ZF==0`|
|`loopne`/`loopnz`|`CX==0` or `ZF==1`|
## `MOVS`: データコピー命令
`movs`

- 使うレジスタ
	- DF
		- Direction Flag
		- 0のときは加算
			- 前向き
		- 1のときは減算
			- 後ろ向き
	- SI
		- 転送元
	- DI
		- 転送先

- サイズ
	- `movsb`
		- 1バイト
	- `movsw`
		- 2バイト
	- `movsd`
		- 4バイト

```
[DI] = [SI];
SI += SI + <size>;
DI += DI + <size>;
```
## `CMPS`: データ比較命令
movsと同じレジスタを使う．
```
FLAGS = CMP([DI], [SI])
SI += SI + <size>;
DI += DI + <size>;
```
## `REP`: リピートプレフィックス
`rep A`
CXをデクリメントしながら0になるまでAを実行します．
`loop`は，do ~ whileのように，Aを実行してからCXの値をみてジャンプしますが，`rep`は，先にCXの値をみます．したがって，CX=0のとき，`rep`では一度もAが実行されません．

- 終了条件
	- `rep`
		- CX == 0
	- `repe`
		- CX == 0 or ZF == 0
	- `repne`
		- CX == 0 or ZF == 1

## フラグ操作命令(`CLD`, `STD`, `CLI`, `STI`, `CLC`, `STC`)
|命令|DF|IF|CF|
|:----|:----|:----|:----|
|`CLD`|0| | |
|`STD`|1| | |
|`CLI`| |0| |
|`STI`| |1| |
|`CLC`| | |0|
|`STC`| | |1|
