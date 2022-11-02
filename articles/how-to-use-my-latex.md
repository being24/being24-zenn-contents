---
title: VSCodeとDockerでLaTeXを用いた多機能論文執筆環境を整える
emoji: 😸
type: tech
topics: [LaTeX, Docker, VSCode, Github, Deploy, CLI]
published: false
---
# はじめに

面倒な$\LaTeX$の環境構築と論文執筆テンプレートを作った

https://github.com/being24/latex-template-ja

https://github.com/being24/latex-docker

# 前提

$\LaTeX$を使っていますか？
自分が所属する研究室では基本的に$\LaTeX$を使用する事が前提です。また、学会に論文を投稿するときにスタイルファイルでテンプレートを渡されることもあります。

自分は学部と大学院で研究室を移っているのですが、学部時代の研究室は卒業論文をワードで執筆するようにと指定されました。控えめに言って地獄でした。(自分が使えこなせていなかったのかもしれませんが、図表が勝手に移動したりキャプションがずれたりフォントが勝手に変わったり変更してるのに変わらなかったり…)

移動後の研究室では$\LaTeX$を使用しており、これまでは各々が独自に環境を構築していました。
TeX Liveを自分でインストールする人もいればクラウド系のコンパイラを使う人もいて、それぞれ使用できるライブラリなどが微妙に食い違っている気持ちが悪い状態でした。

また、TeX Liveは毎年新バージョンがリリースされ、特に理由がなければ最新版を常に使いたい病の私は割とめんどくさい環境構築と初期設定を行うことを強いられていました。複数台PCをもっているためその分手間も増えましたし、何故かうまくインストールできないこともあり、もう嫌になったのでdockerに環境を構築することにしました。

さらに、今回の環境は研究室の後輩などが使用することを前提に構築したため使用者が$\LaTeX$以外の勉強をしなくてもとりあえず動くことを目標にしています。

## $\LaTeX$の環境構築はDockerを使おう

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

# $\LaTeX$のbuildができる環境の構築

## ソフトウェアのインストール

早速$\LaTeX$ソースをビルドできる環境を整えていきます。
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

Repository nameに適当な名前を入力してリポジトリを作成、GitHubの処理が終了した後、自分のローカル環境にcloneしてください。
基本的にこの手順でリポジトリを作成し、文章を作成していきます。

# 構築したシステムでの$\LaTeX$ソースのbuild方法

たまに(よく？)main.texにすべて記述し、rootディレクトリに画像を置き、参考文献は直接ソースに記述するという論文を見ることがあります。
たしかに出力されるpdfファイルの体裁は整っているかもしれませんが、可読性が低いしルートディレクトリはごちゃっとするし、参考文献の管理はおざなりになります。

## 今回使用するテンプレートの構造

こういったの問題を解消するため、自分が作成したテンプレートでは以下の前提で構築されています。

* subfileを使用し、章(Chapter)ごとにファイル分割
* 図や章ごとのtexファイルを専用のフォルダに格納
* BibLaTeXを[Mendeley](https://www.mendeley.com/)等の論文管理ソフトウェアから出力されるbibファイルで使用

これらによって、できるだけ整理された状態を維持します。

## build方法

texファイルにソースを書いたあと、pdfファイルを作成するにはいくつかの方法があります。
build時にはdockerを実行可能にしておく必要があります。

### 保存時に自動でbuild

VSCodeには自動保存を行う設定があり、また、LaTeX-Workshopには保存時に自動でbuildを行う設定があります。
標準では保存時の自動buildを行わない設定になっていますが、保存時に自動でbuildを行わせる場合、作成したリポジトリ内の.vscode/settings.jsonの

```json
  "latex-workshop.latex.autoBuild.run": "never",
```

となっているところを

```json
  "latex-workshop.latex.autoBuild.run": "onSave"
```

に変更してください。
この設定がonSaveになっていると保存時に自動的にbuildされますが、VSCodeの自動保存を有効にしている場合、構文の入力途中でbuildされエラーが出るなどの問題が生じるため手動でbuildする設定を標準にしています。

### GUIでbuild

いちいち入力時にbuildが行われるとエラーメッセージが煩わしく、またbuildに時間がかかってしまうので、buildを行いたいときに実行するためこちらの設定をおすすめします。
LaTeX-workshopのbuildボタンを押すと、buildが行われます。

![up-right](https://storage.googleapis.com/zenn-user-upload/d7fe347040e6-20220604.png)

上記画像(エディタの右上)の場合は緑色の再生ボタンを、

![left-side](https://storage.googleapis.com/zenn-user-upload/f52605233ba1-20220604.png)

上記画像(LaTeX Workshopの拡張機能)の場合は`Recipe: compile`をクリックしてください。
どちらの場合も、buildが完了すると、pdfファイルが生成されます。

また、執筆が一区切りするごとに変更をcommitして置くことを推奨します。
進行度の管理や差し戻しも簡単になりますし、また、指導教官に複数回確認してもらうときワンタッチで差分ファイルを生成できます。

## latexindentを用いたフォーマット

LaTeXにおけるコードのインデントは文章構造の理解などに役立ちます。
というか、コードを記述するときにはインデントは必要不可欠なものです。
ただ、文章を考えてキーボードを叩いているときにインデントまで気にする余裕はないかもしれません。

例えば、pythonにはautopep8というパッケージが存在し、自動でコードを整形してくれます。
同じように、LaTeXにもlatexindentというパッケージがあり、自動で整形をしてくれます。

![menu](https://storage.googleapis.com/zenn-user-upload/c86728fa2124-20220604.png)

上記画像の`ドキュメントをフォーマット  Shift+Alt+F`をクリックするか、そこに表示されたショートカットキーを押下してください。設定に従って自動で整形されます。

だたし、latexindentの設定ファイルである`localSettings.yaml`は発展途上です。バグや改善提案はissueに投げてください。

## latexdiff-vcを用いた差分表示

実際に指導を受けながら論文を作成するとき、前回との差分を示してほしいと言われることがありますし、そのそも自分が確認するときも差分で表示できるとわかりやすくて嬉しいです。
今回はgitでソースを管理することが前提なので、latexdiff-vcを用いて差分を表示できるようにします。

使い方は簡単で、

![left-side](https://storage.googleapis.com/zenn-user-upload/f52605233ba1-20220604.png)

この画像の`Recipe: create_diff`をクリックしてください。

実際にはdockerコンテナ内で`/bin/diff.sh`が実行されています。

```shell
#!/bin/bash

git config --global --add safe.directory /workdir
# 一つ前のコミットとこのコミットの差分
latexdiff-vc -e utf8 -t CFONT --git --flatten --force -r HEAD main.tex

# 現在と指定したIDの差分
# latexdiff-vc -e utf8 -t CFONT --git --flatten --force -r commit-ID main.tex

# 一つ前のタグと最新のタグの差分
# git tag | sort -V | tail -n 2 | xargs -n 2 bash -c 'latexdiff-vc -e utf8 -t CFONT --git --flatten --force -r $0 -r $1 -t CFONT main.tex'
```

中身はこのようになっており、いくつかの方法が用意されています。

現在は現コミットとその一つ前のコミットの差分が表示されますが、コミットIDを指定した差分表示や、タグ同士の差分表示ができます。この辺は適宜調整してください。

## Linterの使い方

LaTeXで記述した文章に校正をかけることができます。
記述した文章を[textlint](https://textlint.github.io/)とgithub actionsを使用して校正します。
これはwordなどで行われる自動校正を代替することを目的としており、日本語的におかしな文章や表記ゆれをPRの形で指摘してくれます。

こういった自動ツールで機械的な修正点を指摘することで、内容の推敲に集中できるようになります、が、**過信は禁物です**。あくまで機械的に指摘しているため精度は完璧とは言い難いです。このため、自分としては印刷した上で音読し、違和感がある言い回しなどを修正することを強く推奨します。

使い方は単純で、PR時に自動でコメントをしてくれるので普段はdevブランチで作業し、一段落したらmain/masterブランチにPRを出しLinterで指摘点を確認、修正したり無視したりを決定するといいと思います。
ただし、一回無視した指摘点は再度指摘されないので注意してください。

## github actionsを用いたbuildとrelease

論文のバージョンを自分で指定し、releaseの形式でgithub上においておくことができます。
バージョンの数字の付け方自体は自由ですが、git上でvから始まるタグをつけてpushするとgithub actionsによってソースがbuildされ、[このように](https://github.com/being24/latex-template-ja/releases/tag/v1.0)その名前のreleaseが作成されます。

# 最後に

ここまで、自分が作成・改造した論文執筆環境の構築方法、使い方をざっくりまとめました。
この文章はgithubで管理しているので、issueやPRは歓迎です。質問もそちらからお願いします。

# 補遺

研究室内で使って見た結果、いくつかの問題が発生したのでそれつにいて追記します。

## 画像などを貼るときにエラーが起きる問題

コード自体に問題が無いのに、エラーが生じることがあります。

その時に表示されるエラーの例を下に示します。

```shell
extractbb:warning: Can't find file (figures/screen), or it is forbidden to read ...skipping

extractbb:warning: Can't find file (shot.png), or it is forbidden to read ...skipping



! LaTeX Error: Cannot determine size of graphic in figures/screen shot.xbb (no 
BoundingBox).

See the LaTeX manual or LaTeX Companion for explanation.
Type  H <return>  for immediate help.
 ...                                              
                                                  
l.10 ...aphics[width=5cm]{figures/screen shot.png}
```

この例は、画像のファイル名に空白が含まれている場合に生じるエラーです。
LaTeX側がファイル名を正しく認識できないために生じるため、ファイル名に空白を使用しないでください。
その他にも、ファイル名・パスに日本語が用いられる場合などに同様の問題が生じます。
詳細は以下のリンクを確認してください。

<http://www.ic.daito.ac.jp/~mizutani/tex/texfile_name.html>

## スラッシュ・バックスラッシュ問題

VSCodeのエクスプローラから、対象のファイルを右クリックすることで相対パスを取得する事ができます。(絶対パスは使用しないでください)

Windowsでは、その方法で取得される文字列は`figures\screenshot.png`となります。
しかし、これをそのままLaTeXに渡すとエラーが起きてしまいます。これはlinuxとwindowsのパスの表し方の違いであるため、`figures/screenshot.png`と修正する必要があります。(バックスラッシュをスラッシュにする)

## linter.shの改行コード問題

![エラー画面](https://storage.googleapis.com/zenn-user-upload/20e25614588e-20220702.png)

```shell
[19:32:10] Format with command: docker
[19:32:10] Format with command args: ["run","--rm","-v","%DIR%:/workdir","ghcr.io/being24/latex-docker","sh","/workdir/bin/linter.sh"]
[19:32:12] Formatting failed with exit code 2
[19:32:12] stderr: /workdir/bin/linter.sh: 2:
: not found
/workdir/bin/linter.sh: 5: Syntax error: word unexpected (expecting "do")
```

このエラーは、`bin/linter.sh`の改行コードがLFからCRLFに変更されている場合に生じます。改行コードをLFに変更してください。
このエラーはwindows版gitのインストール時のオプションで、改行コードの自動変更が行われることが原因である可能性があります。
最近のエディタは基本どの改行コードでも対応できると思うため、この設定を切ってしまうことをおすすめします。

## 画像の幅の問題

これは問題、というわけではありませんが記述します。
多くのサンプルコードにおいて、画像を貼り付ける場合以下のようになっていると思います。

```latex
\begin{figure}
    \centering
    \includegraphics[width=5cm]{figures/screenshot.png}
    \caption{スクリーンショット}
\end{figure}

\begin{figure}
    \centering
    \includegraphics[scale=0.8]{figures/screenshot.png}
    \caption{スクリーンショット}
\end{figure}
```

確かにこれでも表示自体はされますが、表示サイズの指定を長さで指定するとカラムサイズの変更したときにうまくハマりませんし、画像サイズのスケールで指定すると、貼り付けたい画像のサイズで各々に調整が必要となり面倒です。このため、現在の行の幅を基準に画像の幅を指定すると良いと思います

```latex
\begin{figure}
    \centering
    \includegraphics[width=\linewidth]{figures/screenshot.png}
    \caption{スクリーンショット}
\end{figure}
```

(そもそもpngを貼るなと言う話もあるかもしれませんが…)

## 学会提出用の論文作成時のエラー

実際に後輩が遭遇していた例ですが、論文を学会に提出する際に”未対応の圧縮形式”というエラーが出ることがあります。
画像の圧縮を解除する等の方法が提示されていますが、我々の環境では`dvipdfmx`のオプションで出力されるPDFのバージョンを変更することで対応できました。

`.latexmkrc`の中にある

```shell
$dvipdf = 'dvipdfmx %O -o %D %S';
```

を

```shell
$dvipdf = 'dvipdfmx -V 4 %O -o %D %S';
```

に変更してください。

## 学会指定のスタイルファイルがbuildできないとき

```shell
! LaTeX Error: Encoding scheme `JY1' unknown.
```

というエラーが出てbuildできないことがあります。
これは、指定されたクラスがjarticleであれば

```latex
\documentclass{jarticle}
```

の部分を

```latex
\documentclass{ujarticle}
```

に変更することで、指定されたクラスがjsarticleであれば

```latex
\documentclass{jsarticle}
```

の部分を

```latex
\documentclass[uplatex]{jsarticle}
```

に変更することで解決します。

## 保存時にエラーが出る、buildが通らない等

このテンプレートを自分のアカウントにforkして使っていると、保存時にエラーが出たり、buildが通らなかったりすることがあります。

これは、fork後にこちらが更新した際に、fork元のテンプレートとの差分が発生してしまうためです。
このため、fork変更を取り込む、fetch upstreamを行う必要があります。
ただし、本リポジトリはtemplateとして使用できるようにしてあるため、PRを出す時以外はforkを行う必要はありません。

## docker for windowsでbuild時に"user declined directory sharing"のエラーが出る場合

docker for windowsのアップデートにより、localのディレクトリをdockerコンテナにマウントする際に、ユーザーの許可が必要になりました。
これは、docker for windowsの設定画面から許可を与えることで解決できます。
右上の歯車マークから設定を開き、Settings -> Resources -> FILE SHARINGに、sourceのディレクトリを追加してください。
Apply & Restartを押すと、docker for windowsが再起動され、設定が適応できます。
