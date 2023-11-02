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

```python
from SDM15 import SDM15, BaudRate

if __name__ == "__main__":
    lidar = SDM15("/dev/ttyUSB0", BaudRate.BAUD_460800) # change the port name to your own port

    version_info = lidar.obtain_version_info()
    print("get version info success")

    lidar.lidar_self_test()
    print("self test success")

    lidar.start_scan()

    while True:
        try:
            distance, intensity, disturb = lidar.get_distance()
            print(f"distance: {distance}, intensity: {intensity}, disturb: {disturb}")
        except KeyboardInterrupt:
            break
```

ポートとBaudRateを指定して、SDM15のインスタンスを作成します。

その後、`obtain_version_info`でバージョン情報を取得し、`lidar_self_test`でセンサの自己診断を行います。

`start_scan`で計測を開始し、`get_distance`で計測結果を取得します。

## Tips

- Scan中は、`stop_scan`で計測を停止するか、`get_distance`を呼び出すことしかできません。設定などを変更したい場合は、`stop_scan`で計測を停止してから行ってください。
- 設定変更後はセンサを再起動することをおすすめします。Baud rateの設定は再起動しなければ反映されませんし、その他の設定変更もその後の通信が不安定になることがあります。
- このセンサは`start_scan`を呼び出さなければ計測を開始しません。
- 開発マニュアルには、このセンサを使用するときは以下の手順を踏むことを推奨しています。
  - まず`obtain_version_info`でバージョン情報を取得する
  - 次に`lidar_self_test`でセンサの自己診断を行う
  - 最後に`start_scan`で計測を開始する
- このライブラリは、`atexit`を使用してプログラム終了時に`stop_scan`を呼び出したあとにシリアルポートを自動的に閉じます。

## 愚痴

SDM15はBaudRateを230400、460800、512000、921600、1500000から選択できます。
しかしながら、SDM15用に案内されているUSB-UART変換モジュールはCP2102を採用しており、センサで使用できる選択肢は230400、460800、921600のみであるため、512000、1500000は使用できません。
ライブラリ開発中にBaudRateを1500000に設定したところセンサとの通信ができなくなり、FT231Xを引っ張り出す羽目になりました。
付属品みたいなものなのに、センサのBaudRateに対応できないのは勘弁してほしい……。
