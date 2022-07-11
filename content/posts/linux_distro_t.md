---
title: "色々なLinuxディストロを入れた感想"
date: 2022-07-03T23:52:59+09:00

author: "chizuchizu"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Ubuntu, Arch, Gentoo, NetBSDを入れた感想を綴りました．"
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

## このポエムを書いたきっかけ
最近，様々なディストロを入れたけど，なんだかんだ安定しなくて困ってるからとりあえず今の気持ちをまとめたい．ポエムなのでです．ます．調はやめておく．
## この記事の対象者
誰でも（OSの説明はしないので各自調べてほしいです）
## 自分がOSに求めること
私は，機械学習エンジニア（自称）だからPythonとGPUでモデルをぶん回す環境を構築する必要がある．具体的には，
- Python
- Nvidia driver support
- GUI support
があれば何でもいい．
## 自分が今まで入れたOSと感想
### Ubuntu
初めて入れたときは，Windowsライクでとても使いやすいなぁなんて思ってたけど，aptはあんまり好きじゃない．全然新しいバージョンが入ってないし，`deb`ファイルを生で読み込むなんて今じゃ考えられない．nvidia driverにもバグがあって，autoinstallが使えなくてnvidia homepageからバイナリファイルとってきて入れてた記憶がある．もうやりたくない．
### EndeavourOS(Arch Linux)
今使ってるOS.なんだかんだこれも安定していて使いやすい．自分はデスクトップ環境を整えたいわけじゃないからArchはいらない．でも，Archのようなローリングリリースで気軽にパッケージを入れられる思想は気に入ったので入れてみた．AURのパッケージの網羅性，新しさは最強．
最近，Ubuntuの記事が多すぎて質の悪い記事に当たる確率が増えてきた印象がある．その一方で，Arch系は公式のWikiが充実しているし，使ってるユーザー層のレベルもある程度高いので，ググったらだいたい解決して嬉しい．Arch LinuxのSlackコミュニティも質問したらすぐ答えてくれるような優しい人がいて本当に助かってる（何度かお世話になった）．
Ubuntuを使っていたときは，OSとデスクトップ環境が対応するものだと思っていた（WindowsやMacのように）．でも実際にはそうじゃなくて，同じOSを使っていても白と黒くらい違う見た目のデスクトップ環境を使ってる人がいる事を知った．

もう一つ自分が学んだことは，configとの付き合い方．Ubuntu時代はGUI操作で完結したい一心だったが，config管理も中々良いことに気付いた．再現性が保てたり，バグの原因を探せたりするからだ．ついでに，configをいじるようになってvimが上達した（やはり必要にならないと上達しない気がする）．
### Gentoo Linux
こいつは挑戦中．思い出を語ると３年話せるが，とにかく憧れしかないOS．何度も入れたが，音がでない，日本語入力ができない（もうできた），インストールに時間かかる，苦しい思いばっかりしてきた．
こいつのおかげでLinux Kernel，パーティション，locale，その他諸々を学ぶことができた．
パッケージ管理システムも強いし，品揃えも豊富だから普通に実用レベル．だというのに，まだ満足できる環境を構築できてない．
### NetBSD
これは知り合いに布教されて挑戦したOS.結論から言うと，nvidia driverが入らなくて解像度がおかしいし，（機械学習の）開発ができないし，やめにした．技術的にはできると思うが，疲れてしまった．
メモリが12MBでも起動することで有名らしい．自宅に放置されてる最終処分場で買ったノートパソコンに入れる予定．さすがに古いパソコンのモニタを見続けるのは体に応えるので，サーバとして運用する予定．
### Void Linux
こいつはまだVirtualBox上でしか動かせていない．インストールはGentooやArchよりも簡単(GUIで完結する)．デスクトップ環境の構築に関してもWikiやYouTubeの動画を参考にすればできそう．
nvidia driverがちゃんと動くかどうか，パッケージマネージャがどれくらい使えるのかはまだわからないから，入れるだけ入れてみたい(NetBSD使ってる知り合いはなければビルドすればいいんだよ〜って言ってた)．

そこまできたらslackwareでも良い気がしてきたが，Gentooでもパッケージ管理つらい（USE Flagを考えてるだけでも）のでまだ使える気がしない．経験値が足りない．
## これからやること
今後もEndeavourOSを使い続けると思うが，長期休暇の間にGentooを入れて満足できたら移行しようと思う．
今になって思ったが，開発用のパソコンと趣味用のパソコンは分けるべきなんだなとしみじみ実感した．パソコンの性能を最大限活かすためにも，Linux lifeを楽しむためにも，CPU only(w/o GPU)のパソコンがほしいなと思うところ．正直BSD系は気になっている．
今はお金が無いから，そんなことはできないが．

## 余談
### M1 macの未来
正直M1 macで開発するのがつらすぎて誰かThinkPadと交換してくれないかなって毎日考えてる．Asahi Linuxを入れたが，結局Gentooを入れるのに失敗してアンマウントしてパーティション触れなくなって復旧に死ぬほど時間かかった悪い思い出がある．Macは一般人になるため（Office365, その他会議ツールなど）に必要なので，浅知恵でAsahi Linuxを入れて変にMac本体に影響があったらいけないと思って今はLinux計画やめてる．