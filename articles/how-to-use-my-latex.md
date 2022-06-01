# はじめに

面倒なLaTeXの環境構築と論文執筆テンプレートを作ったので使って欲しい

<https://github.com/being24/latex-template-ja>

<https://github.com/being24/latex-docker>

# 前提

LaTeXを使っていますか？
自分が所属する研究室では基本的にLaTeXを使用する事が前提ですし、学会に論文を投稿するときにスタイルファイルでテンプレートを渡されることもあります。

自分は学部と大学院で研究室を移っているのですが、学部時代の研究室は卒業論文をワードで執筆するようにと指定されました。控えめに言って地獄でした。（自分が使えこなせていなかったのかもしれませんが、図表が勝手に移動したりキャプションがずれたりフォントが勝手に変わったり変更してるのに変わらなかったり…）

移動後の研究室ではLaTeXを使用しており、これまでは各々が独自に環境を構築していました。
TeX Liveを自分でインストールする人もいればクラウド系のコンパイラを使う人もいて、それぞれ使用できるライブラリなどが微妙に食い違っている気持ちが悪い状態でした。

また、TeX Liveは毎年新バージョンがリリースされ、特に理由がなければ最新版を常に使いたい病の私は割とめんどくさい環境構築と初期設定を行うことを強いられていました。複数台PCをもっているためその分手間も増えましたし、何故かうまくインストールできないこともあり、もう嫌になったのでdockerに環境を構築することにしました。

さらに、今回の環境は研究室の後輩などが使用することを前提に構築したため仕様者がLaTeX以外の勉強をしなくてもとりあえず動くことを目標にしています。

## LaTeXの環境構築はDockerを使おう

Dockerって便利ですよね、可搬性も高いですしアーキテクチャが同じならどの環境でも同じように動きます。一回docker imageをbuild・pushしてしまえばあとは

```bash
docker pull ghcr.io/being24/latex-docker:latest
```

で環境が構築できます。

## 今回使用するimageについて

[このリポジトリ](https://github.com/pddg/latex-docker)をお借りし一部変更した[このリポジトリ](https://github.com/being24/latex-docker)のimageを使用します。個人的に必要だと思った[minted](https://ctan.org/pkg/minted)や[latexindent](https://www.ctan.org/pkg/latexindent)、[siunitx](https://ctan.org/pkg/siunitx)が動くように改変してあります。

Dockerのインストールや仕組みの解説は専門の方々にお任せします。

## VSCodeとgithubによる執筆環境の構築・共有

上で述べた通り、初学者でもとりあえず動くことを目標に環境を構築していきます。
構築した環境のテンプレートは[こちら](https://github.com/being24/latex-template-ja)においてあります。

VSCodeは.vscode内にsettings.jsonが存在する場合、個人設定を上書きしてくれます。
よって、必要な環境を構築した上でこのリポジトリのテンプレートを用いることですべての設定が完結します。

# LATeXのbuildができる環境の構築

## ソフトウェアのインストール

早速LaTeXソースをビルドできる環境を整えていきます。
必要なソフトウェアは

* [VSCode](https://code.visualstudio.com/)
  * [LaTeX Workshop](https://marketplace.visualstudio.com/items?itemName=James-Yu.latex-workshop)(必須)
  * [Git Graph](https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph)等、あとはお好みで
* [Docker](https://www.docker.com/)
  * Windowsの方は[Docker Desktop for Windows](https://docs.docker.com/desktop/windows/install/)
  * **sudoコマンド無しでdockerを使用できるようにしておいてください**
* [git](https://git-scm.com/)

また、githubのアカウントを作成しgitの初期設定を行い、

```bash
docker pull ghcr.io/being24/latex-docker:latest
```

を実行して環境構築済みのimageをローカルにpullしてください。

## テンプレートのclone

github上に存在するリポジトリはcloneによってローカル環境にダウンロードできます。

今回使用する[リポジトリ](https://github.com/being24/latex-template-ja)はテンプレートリポジトリにしてありますので画面右上の

![Use this template](https://storage.googleapis.com/zenn-user-upload/9cb185badd01-20220601.png)

Use this templateをクリックし、

![Repository name](https://storage.googleapis.com/zenn-user-upload/82b99299f7d1-20220601.png)

Repository nameに適当な名前を入力してリポジトリを作成してください。
GitHubの処理が終了した後、自分のローカル環境にcloneしてください。
基本的にこの手順でリポジトリを作成し、文章を作成していきます。

# 構築したシステムでのLaTeXソースのbuild方法

たまに（よく？）main.texにすべて記述し、rootディレクトリに画像を置き、参考文献は直接ソースに記述するという論文を見ることがあります。たしかに出力されるpdfファイルの体裁は整っているかもしれませんが、可読性が低いしルートディレクトリはごちゃっとするし、参考文献の管理はおざなりになります。

これらの問題を解消するため、自分が作成したテンプレートでは以下の前提で構築されています。

* subfileを用いた章(Chapter)を用いたファイル分割
* 図や章ごとのtexファイルを専用のフォルダに格納
* BibLaTeXを[Mendeley](https://www.mendeley.com/)等の論文管理ソフトウェアから出力されるbibファイルで使用

これらによって、できるだけ整理された状態を維持します。

## build方法(ここから下はWIP)

上記の前提で書いたとします、
ビルドして

* gitでバージョンごとにちゃんとcommitして置いたほうが良いよ、後で使うから

## latexindentを用いたフォーマット

* なんでいるのか
* どう使うのか
* 設定は発展途上

## latexdiff-vcを用いた差分表示

* 何が嬉しいのか
* どう使うのか

## Literの使い方

* linterくらい使え
* だからって盲信するな、印刷して音読しろ
* (パラグラフ・ライティングとか使うといいよ)

## github actionsを用いたbuildとrelease

* なぜ？

## コントリビュート方法
