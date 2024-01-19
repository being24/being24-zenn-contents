---
title: YDLIDAR SDM15のPythonとarduinoライブラリを作った
emoji: ⚙️
type: tech
topics: [Python, Arduino, YDLIDAR, SDM15]
published: false
---
## はじめに

YDLIDAR SDM15のPythonとarduinoライブラリを作りました。

https://github.com/being24/YDLIDAR-SDM15_python
https://github.com/being24/YDLIDAR-SDM15_arduino

## このライブラリは？

[公式のマニュアル](https://www.ydlidar.com/service_support/download.html?gid=24)を参考に、SDM15をシリアル通信を用いて接続するライブラリです。

Python版ではpyserialを使用しています。

Python版ではマニュアルに記載されているすべてのコマンドを実装しています。
また、設定のパラメータはEnumで定義しているので、コード補完が効きます。

Arduino版ではArduino R4上でHardwareSerialでの動作確認をしています。
自分が使う一部のコマンドだけを実装した状態で止まっていたのですが、Python版のissueにArduino版無い？って聞かれたので暫定公開しました。
少なくともうちのロボットでは動いています。

## 使い方

sample.pyとYDLIDAR-SDM15_arduino.inoを参考にしてください。

## 愚痴

SDM15はBaudRateを230400、460800、512000、921600、1500000から選択できます。
しかしながら、SDM15用に案内されているUSB-UART変換モジュールはCP2102を採用しており、センサで使用できる選択肢は230400、460800、921600のみであり、512000、1500000は使用できません。
ライブラリ開発中にBaudRateを1500000に設定したところセンサとの通信ができなくなり、FT231Xを引っ張り出す羽目になりました。
付属品みたいなものなのに、センサとBaudRateが合わないのは勘弁してほしい……。
