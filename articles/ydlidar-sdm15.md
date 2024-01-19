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

```arduino
#include "SDM15.h"

SDM15 sdm15(Serial1);

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(9600);
  Serial1.begin(460800);

  // wait 1s
  delay(5000);

  Serial.println("get data");

  // get version info
  VersionInfo info = sdm15.ObtainVersionInfo();

  if (info.checksum_error) {
    Serial.println("checksum error");
    return;
  }

  // print version info
  Serial.print("model: ");
  Serial.println(info.model);
  Serial.print("hardware_version: ");
  Serial.println(info.hardware_version);
  Serial.print("firmware_version_major: ");
  Serial.println(info.firmware_version_major);
  Serial.print("firmware_version_minor: ");
  Serial.println(info.firmware_version_minor);
  Serial.print("serial_number: ");
  Serial.println(info.serial_number);

  // get self check test
  TestResult test = sdm15.SelfCheckTest();

  if (test.checksum_error) {
    Serial.println("checksum error");
    return;
  }

  if (test.self_check_result) {
    Serial.println("self check success");
  } else {
    Serial.println("self check failed");
    Serial.print("error code: ");
    Serial.println(test.self_check_error_code);
    return;
  }

  delay(1000);
}

void loop() {
  // set output freq
  // bool result2 = sdm15.SetOutputFrequency(Freq_100Hz);
  // if (!result2) Serial.println("set output freq checksum error");

  Serial.println("start scan");

  // start scan
  bool result = sdm15.StartScan();

  if (!result) Serial.println("start scan checksum error");

  // get scan data 50 times
  for (int i = 0; i < 50000; i++) {
    ScanData data = sdm15.GetScanData();

    if (data.checksum_error) Serial.println("checksum error");

    Serial.print("scan times: ");
    Serial.print(i);
    Serial.print(" distance: ");
    Serial.print(data.distance);
    Serial.print(" intensity: ");
    Serial.print(data.intensity);
    Serial.print(" disturb: ");
    Serial.println(data.disturb);
  }

  // stop scan
  result = sdm15.StopScan();

  if (!result) Serial.println("stop scan checksum error");

  Serial.println("stop scan");

  // wait 1s
  delay(1000);

  exit(0);
}
```

Arduino版では、`SDM15.h`をインクルードして、SDM15のインスタンスを作成します。

その後、`ObtainVersionInfo`でバージョン情報を取得し、`SelfCheckTest`でセンサの自己診断を行います。

`StartScan`で計測を開始し、`GetScanData`で計測結果を取得します。

## Tips

- Scan中は、`stop_scan`で計測を停止するか、`get_distance`を呼び出すことしかできません。設定などを変更したい場合は、`stop_scan`で計測を停止してから行ってください。
- 設定変更後はセンサを再起動することをおすすめします。Baud rateの設定は再起動しなければ反映されませんし、その他の設定変更もその後の通信が不安定になることがあります。
- このセンサは`start_scan`を呼び出さなければ計測を開始しません。
- 開発マニュアルには、このセンサを使用するときは以下の手順を踏むことを推奨しています。
  - まず`obtain_version_info`でバージョン情報を取得する
  - 次に`lidar_self_test`でセンサの自己診断を行う
  - 最後に`start_scan`で計測を開始する
- python版のライブラリは、`atexit`を使用してプログラム終了時に`stop_scan`を呼び出したあとにシリアルポートを自動的に閉じます。

## 愚痴

SDM15はBaudRateを230400、460800、512000、921600、1500000から選択できます。
しかしながら、SDM15用に案内されているUSB-UART変換モジュールはCP2102を採用しており、センサで使用できる選択肢は230400、460800、921600のみであるため、512000、1500000は使用できません。
ライブラリ開発中にBaudRateを1500000に設定したところセンサとの通信ができなくなり、FT231Xを引っ張り出す羽目になりました。
付属品みたいなものなのに、センサのBaudRateに対応できないのは勘弁してほしい……。
