---
title: YDLIDAR SDM15のPythonライブラリを作った
emoji: ⚙️
type: tech
topics: [Python, YDLIDAR, SDM15]
published: false
---
## はじめに

YDLIDAR SDM15のPythonライブラリを作りました。

https://github.com/being24/YDLIDAR-SDM15_python

## このライブラリは？

[公式のマニュアル](https://www.ydlidar.com/service_support/download.html?gid=24)を参考に、SDM15をPySerialを用いて接続するライブラリです。

マニュアルに記載されているすべてのコマンドを実装しています。
また、設定のパラメータはEnumで定義しているので、コード補完が効きます。

## 使い方

sample.pyを参考にしてください。

## 愚痴

SDM15はBaudRateを230400、460800、512000、921600、1500000から選択できます。
しかしながら、SDM15用に案内されているUSB-UART変換モジュールはCP2102を採用しており、センサで使用できる選択肢は230400、460800、921600のみであり、512000、1500000は使用できません。
ライブラリ開発中にBaudRateを1500000に設定したところセンサとの通信ができなくなり、FT231Xを引っ張り出す羽目になりました。
付属品みたいなものなのに、センサとBaudRateが合わないのは勘弁してほしい……。
