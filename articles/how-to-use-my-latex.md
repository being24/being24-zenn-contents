---
title: VSCodeとDockerでLaTeXを用いた多機能論文執筆環境を整える
emoji: 😸
type: tech
topics: [LaTeX, Docker, VSCode, Github, Deploy, CLI]
published: false
---
## はじめに

面倒な$\LaTeX$の環境構築と論文執筆テンプレートを作った

https://github.com/being24/latex-template-ja

https://github.com/being24/latex-docker

## 前提

$\LaTeX$を使っていますか？
自分が所属する研究室では基本的に$\LaTeX$を使用する事が前提です。また、学会に論文を投稿するときにスタイルファイルでテンプレートを渡されることもあります。

自分は学部と大学院で研究室を移っているのですが、学部時代の研究室は卒業論文をワードで執筆するようにと指定されました。控えめに言って地獄でした。(自分が使えこなせていなかったのかもしれませんが、図表が勝手に移動したりキャプションがずれたりフォントが勝手に変わったり変更してるのに変わらなかったり…)

移動後の研究室では$\LaTeX$を使用しており、これまでは各々が独自に環境を構築していました。
TeX Liveを自分でインストールする人もいればクラウド系のコンパイラを使う人もいて、それぞれ使用できるライブラリなどが微妙に食い違っている気持ちが悪い状態でした。

また、TeX Liveは毎年新バージョンがリリースされ、特に理由がなければ最新版を常に使いたい病の私は割とめんどくさい環境構築と初期設定を行うことを強いられていました。複数台PCをもっているためその分手間も増えましたし、何故かうまくインストールできないこともあり、もう嫌になったのでdockerに環境を構築することにしました。

さらに、今回の環境は研究室の後輩などが使用することを前提に構築したため使用者が$\LaTeX$以外の勉強をしなくてもとりあえず動くことを目標にしています。

### 2023/01/11追記

従来は[LaTeX workshop](https://marketplace.visualstudio.com/items?itemName=James-Yu.latex-workshop)の機能を用いてdockerコマンドを直接叩いていましたが、[dev container](https://code.visualstudio.com/docs/devcontainers/containers)を使用したほうがパフォーマンスが向上することが判明したため、そちらに移行しています。

現在のmasterブランチはそちらに移行しており、従来のバージョンはcommand_arcブランチに移動していますが、要望がない限り保守はしない予定です。

## $\LaTeX$の環境構築はDockerを使おう

Dockerって便利ですよね、可搬性も高いですしアーキテクチャが同じならどの環境でも同じように動きます。一回docker imageをbuild・pushしてしまえばあとは

```bash
docker pull ghcr.io/being24/latex-docker:latest
```

で環境が構築できます。

### 今回使用するimageについて

[このリポジトリ](https://github.com/pddg/latex-docker)をお借りし一部変更した[このリポジトリ](https://github.com/being24/latex-docker)のimageを使用します。個人的に必要だと思った[minted](https://ctan.org/pkg/minted)や[latexindent](https://www.ctan.org/pkg/latexindent)、[siunitx](https://ctan.org/pkg/siunitx)が動くように改変してあります。

Dockerのインストールや仕組みの解説は専門の方々にお任せします。

### VSCodeとgithubによる執筆環境の構築・共有

上で述べた通り、初学者でもとりあえず動くことを目標に環境を構築していきます。
構築した環境のテンプレートは[こちら](https://github.com/being24/latex-template-ja)においてあります。

VSCodeは.vscode内にsettings.jsonが存在する場合、個人設定を上書きしてくれます。
よって、必要な環境を構築した上でこのリポジトリのテンプレートを用いることですべての設定が完結します。

## $\LaTeX$のbuildができる環境の構築

### Buildソフトウェアのインストール

早速$\LaTeX$ソースをビルドできる環境を整えていきます。

**この記事の内容を使用する場合は、必ずdockerが使用できる状態にしておいてください！！**

必要なソフトウェアは

* [VSCode](https://code.visualstudio.com/)
  * [LaTeX Workshop](https://marketplace.visualstudio.com/items?itemName=James-Yu.latex-workshop)(必須)
  * [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)(必須)
  * [Git Graph](https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph)等、あとはお好みで
* [Docker](https://www.docker.com/)
  * Windowsの方は[Docker Desktop for Windows](https://docs.docker.com/desktop/windows/install/)
  * **sudoコマンド無しでdockerを使用できるようにしておいてください**
* [git](https://git-scm.com/)

です。
また、githubのアカウントを作成しgitの初期設定を行い、

```bash
docker pull ghcr.io/being24/latex-docker:latest
```

を実行して環境構築済みのimageをローカルにpullしてください。

### テンプレートのclone

github上に存在するリポジトリはcloneによってローカル環境にダウンロードできます。

今回使用する[リポジトリ](https://github.com/being24/latex-template-ja)はテンプレートリポジトリにしてありますので画面右上の

![Use this template](https://storage.googleapis.com/zenn-user-upload/9cb185badd01-20220601.png)

Use this templateをクリックし、

![Repository name](https://storage.googleapis.com/zenn-user-upload/82b99299f7d1-20220601.png)

Repository nameに適当な名前を入力してリポジトリを作成、GitHubの処理が終了した後、自分のローカル環境にcloneしてください。vscodeを使用してクローンしたリポジトリを開くとvscodeが右下のポップアップで、推奨される拡張機能のインストールと、dev containerを使用するかどうかの質問をしてくれます。すべてyesで進めてください。

基本的にこの手順でリポジトリを作成し、文章を作成していきます。

## 構築したシステムでの$\LaTeX$ソースのbuild方法

たまに(よく？)main.texにすべて記述し、rootディレクトリに画像を置き、参考文献は直接ソースに記述するという論文を見ることがあります。
たしかに出力されるpdfファイルの体裁は整っているかもしれませんが、可読性が低いしルートディレクトリはごちゃっとするし、参考文献の管理はおざなりになります。

### 今回使用するテンプレートの構造

こういったの問題を解消するため、自分が作成したテンプレートでは以下の前提で構築されています。

* subfileを使用し、章(Chapter)ごとにファイル分割
* 図や章ごとのtexファイルを専用のフォルダに格納
* BibLaTeXを[Mendeley](https://www.mendeley.com/)等の論文管理ソフトウェアから出力されるbibファイルで使用

これらによって、できるだけ整理された状態を維持します。

### build方法

texファイルにソースを書いたあと、pdfファイルを作成するにはいくつかの方法があります。
build時にはdockerを実行可能にしておく必要があります。

#### 保存時に自動でbuild

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

#### GUIでbuild

いちいち入力時にbuildが行われるとエラーメッセージが煩わしく、またbuildに時間がかかってしまうので、buildを行いたいときに実行するためこちらの設定をおすすめします。
LaTeX-workshopのbuildボタンを押すと、buildが行われます。

![up-right](https://storage.googleapis.com/zenn-user-upload/d7fe347040e6-20220604.png)

上記画像(エディタの右上)の場合は緑色の再生ボタンを、

![left-side](https://storage.googleapis.com/zenn-user-upload/f52605233ba1-20220604.png)

上記画像(LaTeX Workshopの拡張機能)の場合は`Recipe: compile`をクリックしてください。
どちらの場合も、buildが完了すると、pdfファイルが生成されます。

また、執筆が一区切りするごとに変更をcommitして置くことを推奨します。
進行度の管理や差し戻しも簡単になりますし、また、指導教官に複数回確認してもらうときワンタッチで差分ファイルを生成できます。

### latexindentを用いたフォーマット

LaTeXにおけるコードのインデントは文章構造の理解などに役立ちます。
というか、コードを記述するときにはインデントは必要不可欠なものです。
ただ、文章を考えてキーボードを叩いているときにインデントまで気にする余裕はないかもしれません。

例えば、pythonにはautopep8というパッケージが存在し、自動でコードを整形してくれます。
同じように、LaTeXにもlatexindentというパッケージがあり、自動で整形をしてくれます。

![menu](https://storage.googleapis.com/zenn-user-upload/c86728fa2124-20220604.png)

上記画像の`ドキュメントをフォーマット  Shift+Alt+F`をクリックするか、そこに表示されたショートカットキーを押下してください。設定に従って自動で整形されます。

だたし、latexindentの設定ファイルである`localSettings.yaml`は発展途上です。バグや改善提案はissueに投げてください。

### github actionsにおける文章校正

LaTeXで記述した文章に校正をかけることができます。
記述した文章を[textlint](https://textlint.github.io/)とgithub actionsを使用して校正します。
これはwordなどで行われる自動校正を代替することを目的としており、日本語的におかしな文章や表記ゆれをPRの形で指摘してくれます。

こういった自動ツールで機械的な修正点を指摘することで、内容の推敲に集中できるようになります、が、**過信は禁物です**。あくまで機械的に指摘しているため精度は完璧とは言い難いです。このため、自分としては印刷した上で音読し、違和感がある言い回しなどを修正することを強く推奨します。

使い方は単純で、PR時に自動でコメントをしてくれるので普段はdevブランチで作業し、一段落したらmain/masterブランチにPRを出しLinterで指摘点を確認、修正したり無視したりを決定するといいと思います。
ただし、一回無視した指摘点は再度指摘されないので注意してください。

### 文章記述中のtextlint

文章記述中にもtextlintを使用することができます。
まず最初に

```shell
sh ./bin/install_textlint.sh
```

を実行しtextlintをインストールした上で、VSCodeの拡張機能である[vscode-textlint](https://marketplace.visualstudio.com/items?itemName=taichi.vscode-textlint)をインストールしてください。

### github actionsを用いたbuildとrelease

論文のバージョンを自分で指定し、releaseの形式でgithub上においておくことができます。
バージョンの数字の付け方自体は自由ですが、git上でvから始まるタグをつけてpushするとgithub actionsによってソースがbuildされ、[このように](https://github.com/being24/latex-template-ja/releases/tag/v1.0)その名前のreleaseが作成されます。

## 最後に

ここまで、自分が作成・改造した論文執筆環境の構築方法、使い方をざっくりまとめました。
この文章はgithubで管理しているので、issueやPRは歓迎です。質問もそちらからお願いします。

## 補遺

研究室内で使って見た結果、いくつかの問題が発生したのでそれつにいて追記します。

### 画像などを貼るときにエラーが起きる問題

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

### スラッシュ・バックスラッシュ問題

VSCodeのエクスプローラから、対象のファイルを右クリックすることで相対パスを取得する事ができます。(絶対パスは使用しないでください)

Windowsでは、その方法で取得される文字列は`figures\screenshot.png`となります。
しかし、これをそのままLaTeXに渡すとエラーが起きてしまいます。これはlinuxとwindowsのパスの表し方の違いであるため、`figures/screenshot.png`と修正する必要があります。(バックスラッシュをスラッシュにする)

### shell scriptの改行コード問題

![エラー画面](https://storage.googleapis.com/zenn-user-upload/20e25614588e-20220702.png)

```shell
[19:32:10] Format with command: docker
[19:32:10] Format with command args: ["run","--rm","-v","%DIR%:/workdir","ghcr.io/being24/latex-docker","sh","/workdir/bin/linter.sh"]
[19:32:12] Formatting failed with exit code 2
[19:32:12] stderr: /workdir/bin/linter.sh: 2:
: not found
/workdir/bin/linter.sh: 5: Syntax error: word unexpected (expecting "do")
```

```shell
./bin/build.sh: 2:
: not found
./bin/build.sh: 8: Syntax error: end of file unexpected (expecting "then")
```

このエラーは、shell scriptの改行コードがLFからCRLFに変更されている場合に生じます。改行コードをLFに変更してください。
windows版gitのインストール時のオプションで、改行コードの自動変更が行われることが原因である可能性があります。
最近のエディタは基本どの改行コードでも対応できると思うため、この設定を切ってしまうことをおすすめします。

### 画像の幅の問題

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

### svgファイルを貼り付けるとき

svgファイルを貼り付けるとき、svg packageを使用する方法や、pdfに変換して貼り付ける方法があります。

この環境ではどちらも対応できますが、inkscapeを使用してPDFに変換するスクリプトを用意しています。

```shell
sh ./bin/svg2pdf.sh
```

このスクリプトを使用すると、`figures`ディレクトリ以下のsvgファイルをすべてPDFに変換します。このとき、図のテキストはすべてパスに変換されます。パスに変換する必要ない場合は、`--export-text-to-path`オプションを削除してください。

### 学会提出用の論文作成時のエラー

実際に後輩が遭遇していた例ですが、論文を学会に提出する際に”未対応の圧縮形式”というエラーが出ることがあります。
画像の圧縮を解除する等の方法が提示されていますが、PDFのバージョンを変更することで対応できました。

#### (u)pLaTeX使用時

dvipdfmxのオプションを変更して対応します。

`.latexmkrc`の中にある

```shell
$dvipdf = 'dvipdfmx %O -o %D %S';
```

を

```shell
$dvipdf = 'dvipdfmx -V 4 %O -o %D %S';
```

に変更してください。

#### LuaLaTeX使用時

bxpdfverパッケージを使用して対応します。

main.texのdocumentclass指定の後に

```latex
\usepackage[1.4]{bxpdfver}
```

を追加してください。bxpdfverの使い方は[こちら](https://github.com/zr-tex8r/BXpdfver/blob/master/README-ja.md)を参照してください。

### 学会指定のスタイルファイルがbuildできないとき

本テンプレートは、LuaLaTeXでのビルドを前提としていますが、学会によっては、(u)pLaTeXでのビルドを指定している場合があります。

そういった場合は、`.latexmkrc`の中の

```perl
$pdf_mode = 4;
```

を

```perl
$pdf_mode = 3;
```

に変更してください。
この変更により、`latexmk`実行時のコマンドが `$lualatex`から`$latex`に変更され、upLaTeXでのビルドが行われます。

学会によってはplatexを使用する場合もありますが、そういった場合は`$latex`の`uplatex`を`platex`に変更してください。

そろそろ(u)pLaTeXは不味そうなので、各学会の迅速な対応をお願いしたいところです。

このuplatexへ変更を行っても

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

に変更することで解決できます。

### 保存時にエラーが出る、buildが通らない等

このテンプレートを自分のアカウントにforkして使っていると、保存時にエラーが出たり、buildが通らなかったりすることがあります。

これは、fork後にこちらが更新した際に、fork元のテンプレートとの差分が発生してしまうためです。
このため、fork変更を取り込む、fetch upstreamを行う必要があります。
ただし、本リポジトリはtemplateとして使用できるようにしてあるため、PRを出す時以外はforkを行う必要はありません。

### docker for windowsでbuild時に"user declined directory sharing"のエラーが出る場合

docker for windowsのアップデートにより、localのディレクトリをdockerコンテナにマウントする際に、ユーザーの許可が必要になりました。
これは、docker for windowsの設定画面から許可を与えることで解決できます。
右上の歯車マークから設定を開き、Settings -> Resources -> FILE SHARINGに、sourceのディレクトリを追加してください。
Apply & Restartを押すと、docker for windowsが再起動され、設定が適応できます。

### gitの追跡が遅い、もしくは行われない場合

WSL2側の問題のようです、手動でsyncボタンを押すと反映されます。

### 画像が表示されない

更新したclsファイルでは**あえて**pngやjpgなどを貼り付けると表示されないようにしています。
画像を直接貼ると画像が劣化したり、buildが遅くなったりするためです。
画像も一度PDFファイルした上で貼り付けることをおすすめします。

### reference.bibに文献を追加したらエラーが出る、reference以降のページが表示されない、など

logファイルを確認すると

  ```shell
  ! Arithmetic overflow.
  <to be read again>
                    =
  l.5 \printbibliography[title=参考文献]
  ```

や

  ```shell
  ! Improper alphabetic or KANJI constant.
  <to be read again> 
                    \q__text_recursion_tail 
  l.5 \printbibliography[title=参考文献]
  ```

などのエラーが出ることがあります。
これは、最近biblatexもしくはbiberに関連するなにかの仕様が変更された事によるものだと思われます。
この場合、reference.bibのtitleの日本語部分を{}で囲うことで解決できます。例えば

  ```latex
  title   = {あいうえお},
  ```

  を

  ```latex
  title   = {{あいうえお}},
  ```

  に変更してください。

### 参考文献(biblatex)の書式変更について

参考文献の記述方法は多岐にわたりますが、本環境ではバックエンドにbiberを使用しているため、junsrtなどをそのまま使用することができません。
biber使用時は以前の記述方法は使用できず、biber用のjunsrtは提供されていないため、互換用として提供されいてる[trad-unsrt style](https://github.com/moewew/biblatex-trad)を使用し、細かい設定を自分で修正する必要があります。

```latex
\usepackage[backend=biber,style=ieee]{biblatex} % biblatexを使用するためのパッケージ
\addbibresource{references.bib}
```

これを以下のように変更します。

```latex
\usepackage[backend=biber,style=trad-unsrt]{biblatex} % biblatexを使用するためのパッケージ
\addbibresource{references.bib}
\DeclareFieldFormat{journaltitle}{\textit{#1}}
```

trad-unsrtだけでは、日本語の文献名が太字になってしまうため、`journaltitle`の書式を上書きしています。

[参考文献](https://okumuralab.org/tex/mod/forum/discuss.php?d=3336#p20271)

### 参考文献(biblatex)の書式変更について2

学会によっては、参考文献のリストにおいて、1)などの片括弧を使用し、かつ、引用箇所の右肩に1)を表示することを求めています。その場合は、

```latex
\DeclareFieldFormat{labelnumberwidth}{{#1})\hspace{.5\jlreq@zw}} % 参考文献番号の後ろに)をつける

% \mkbibparens のコピーを作成して片括弧のマクロを作成
\let\onebrackets\mkbibparens
\renewcommand*{\onebrackets}[1]{\textsuperscript{#1)}}

\DeclareCiteCommand{\cite}[\onebrackets]
    {\usebibmacro{prenote}}
    {\usebibmacro{citeindex}%
    \usebibmacro{cite}}
    {\multicitedelim}
    {\usebibmacro{postnote}}
```

を追加してください。

### 昔のバージョンのTeX Liveを使用したい場合

互換性の問題などから、昔のバージョンのTeX Liveを使用したい場合があります。
本当は専用のリポジトリなりを作成すべきですが、frozenのTeX Liveをimageにしておくことで対応しています。

<!-- 表 -->
| frozen version | frozen tag |
|:---:|:---:|
| 2022 | 3.2.6 |
| 2023 | 2023-frozen |

本来は2022などのタグを付けることが好ましいですが、自分の理解不足かうまくいかなかったため、3.2.6になっています。
2023からは2023-frozenというタグを付けています。

### CloudLaTeX, Overleafなどのクラウドサービスを使用したい場合

クラウドサービスを使用する場合、dockerを使用する必要はありません。
その場合は、リポジトリ右上のCodeをクリックし、Download ZIPをクリックしてダウンロードしてください。

CloudLaTeXの場合は、新規プロジェクトをクリックし、ZIPをアップロードからzipファイルをアップロードしてください。（プロジェクト名は適したものに変更してください）

この場合、コンパイラの設定は自動で行われますが、もし行われない場合は、設定からコンパイラをLuaLaTeXに変更してください。

Overleafの場合は、新規プロジェクトをクリックし、プロジェクトのアップロードからzipファイルをアップロードしてください。（プロジェクト名は適したものに変更してください）

その後、メニューの設定欄のコンパイラをLuaLaTeXに変更し、主要文章がmain.texであることを確認した上で、コンパイルを行ってください。

ただし、Overleafのプラン変更により、フリープランではコンパイル完了までにタイムアウトしてしまう可能性があります。
そういった場合はOverleaf以外のサービスや、ローカル環境にdockerやインストーラを用いてTexlive環境を構築することをおすすめします。

### docker imageを更新したらパーミッションエラーが出る場合

特にLinux環境においては、devcontainer内で作成したファイルがroot権限になってしまい、色々と不便でした。
このため、新たなdocker image からはlatexユーザーでコンテナを起動するようにしています。

この修正により、旧バージョンのdocker imageを使用していた文章を新しいdocker imageでbuildしようとするとエラーが出ます。
ただしこのエラーはパーミッションエラーなどとは表示されないことが多いです。

確認するためには

```shell
ls -l
```

を実行して、ファイルの所有者がrootになっているか確認してください。
この場合の解決方法は2つあります。

#### ファイルのパーミッションを変更する

ファイルのオーナーを変更すればこの問題は解決しますが、rootユーザーがオーナーのファイルを変更するためには、root権限が必要です。
dockerコンテナ内でroot権限を取得するのはできないので、コンテナを起動した状態のときにホストコンピュータのターミナルで

```shell
docker ps
```

を実行してコンテナのIDを取得し、

```shell
docker exec -it -u root コンテナID /bin/bash
```

を実行してroot権限でコンテナに入り

```shell
chown -R latex:latex /workdir
```

を実行してオーナーを変更してください。

#### フォルダをダウンロードし直す

githubに現状の内容をpushしておいて、再度cloneして起動してください。
この場合、ファイルのオーナーはlatexユーザーになっているため、エラーが出ないはずです。

### 突然github actionsでのビルドが通らなくなった場合

docker imageのユーザを変更した影響で、従来通っていたリポジトリでgithub actionsでのbuildが通らなくなる事象が確認されています。
これは、github actionsの仕様によるものです。こういった場合は、`.github/workflows/build.yml`を確認してください。

`.github/workflows/build.yml`の

```yaml
# 実行されるジョブの定義
jobs:
  # PDFのビルドジョブ
  build:
    runs-on: ubuntu-20.04
    container:
      # もし独自のDockerイメージを変更したい場合、ここを変更する
      image: ghcr.io/being24/latex-docker:latest
```

の`image`の後に`options: --user root`を追加して、

```yaml
# 実行されるジョブの定義
jobs:
  # PDFのビルドジョブ
  build:
    runs-on: ubuntu-20.04
    container:
      # もし独自のDockerイメージを変更したい場合、ここを変更する
      image: ghcr.io/being24/latex-docker:latest
      options: --user root
```

としてください。

### jlreqとupLaTeXを併用したときに文字化けが発生する場合

2024-08-23にjlreqが更新され、このバージョンのjlreq（clsやstyでの使用を含む）をupLaTeXでbuildすると文字化けが発生することを確認しています。
開発者様も確認されているようなので、そのうち解決させると思いますが、luaLaTeXを使用するか、jlreqのバージョンを下げる、docker imageを使用している場合は前のバージョンのimageを使用するなどで対応可能です。

自分のテンプレートでluaLaTeXを使用するには、`.latexmkrc`の

```perl
$pdf_mode = 3;
```

を

```perl
$pdf_mode = 4;
```

に変更してください。

docker imageの前のバージョンを使用する場合は、`ghcr.io/being24/latex-docker:24.08.0`を使用してください。

`.devcontainer/devcontainer.json`の

```json
"image": "ghcr.io/being24/latex-docker",
```

を

```json
"image": "ghcr.io/being24/latex-docker:24.08.0",
```

に変更すると正常にbuildできることを確認しています。
