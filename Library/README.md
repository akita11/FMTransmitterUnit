# KT0803L ライブラリの使い方

KT Micro KT0803L FM ステレオトランスミッタを I2C で制御する Arduino ライブラリを、Arduino IDE と PlatformIO で使う手順です。関数の説明や配線の詳細は、ライブラリに含まれる `README.md` を見てください。

## 配布ファイル

`KT0803L-1.0.0.zip` の中身は次のとおりです。Arduino IDE でも PlatformIO でも、この zip をそのまま使えます。

```
KT0803L/
├─ src/KT0803L.h, KT0803L.cpp        ライブラリ本体
├─ examples/KT0803L_FM_Transmitter/   スケッチ例（Arduino UNO R3 用）
├─ library.properties                  Arduino IDE 用の情報
├─ library.json                        PlatformIO 用の情報
├─ keywords.txt                        Arduino IDE の色分け用
├─ README.md                           関数・配線・レジスタの説明
├─ usage.md                            この文書
└─ LICENSE                             MIT License
```

## Arduino IDE

Arduino IDE 2.x と 1.8.x で同じ手順です。

### インストール

**方法 1：zip から追加する（おすすめ）**

1. メニューの「スケッチ」→「ライブラリをインクルード」→「.ZIP形式のライブラリをインストール...」を選びます。
2. `KT0803L-1.0.0.zip` を選びます。zip は展開しないでください。
3. 「ライブラリが追加されました」と表示されれば完了です。

ライブラリは、スケッチブックの保存場所（「ファイル」→「基本設定」の「スケッチブックの場所」）の `libraries/KT0803L` に入ります。

**方法 2：フォルダをコピーする**

1. zip を展開します。
2. できた `KT0803L` フォルダを、スケッチブックの保存場所の `libraries` フォルダにコピーします（例：`ドキュメント/Arduino/libraries/KT0803L`）。
3. Arduino IDE を起動し直します。

`libraries/KT0803L/library.properties` という配置になっていれば正しく入っています。`libraries/KT0803L/KT0803L/...` のように 1 段深くなっていると認識されません。

### スケッチ例を動かす

1. 「ファイル」→「スケッチ例」→「KT0803L」→「KT0803L_FM_Transmitter」を開きます（「カスタムライブラリのスケッチ例」の下にあります）。
2. 「ツール」→「ボード」で「Arduino Uno」を選び、ポートを選びます。
3. 「書き込み」を押します。
4. シリアルモニタを 115200bps、改行コード「LF」または「CRLF」にすると、状態表示とコマンド入力ができます。

スケッチ例は、初期設定（`SLEEP_AFTER_SETUP = 1`）では、送信を始めてから 1 秒後にマイコンを止めます。ボタンやシリアルコマンドで操作したいときは、スケッチ先頭の `SLEEP_AFTER_SETUP` を `0` にしてください。

### 自分のスケッチで使う

「スケッチ」→「ライブラリをインクルード」→「KT0803L」を選ぶと、`#include <KT0803L.h>` が入ります。最小のスケッチは次のとおりです。

```cpp
#include <Wire.h>
#include <KT0803L.h>

KT0803L fmtx(Wire);

void setup() {
    Serial.begin(115200);
    Wire.begin();
    Wire.setClock(100000);

    if (!fmtx.begin(88100000UL)) {  // 88.1MHz で送信開始
        Serial.println("KT0803L not found");
        return;
    }
    fmtx.setRfGain(0);                              // 送信出力 最小
    fmtx.setPga(0);                                 // 入力ゲイン 0dB
    fmtx.preEmphasis(KT0803L::PRE_EMPHASIS_50_US);  // 日本
}

void loop() {
}
```

Arduino UNO R3 などの 5V ボードでは、`Wire.begin()` の直後に次の 2 行を入れて、マイコンの内蔵プルアップ（5V）を切ってください（理由は後述の「5V ボードで使うとき」）。

```cpp
    digitalWrite(SDA, LOW);  // ATmega328P の内蔵プルアップを無効にする
    digitalWrite(SCL, LOW);
```

### 更新・削除

- 新しい版に入れ替えるときは、スケッチブックの `libraries/KT0803L` フォルダを削除してから、新しい zip を追加します。
- 削除するときは、`libraries/KT0803L` フォルダを削除して Arduino IDE を起動し直します。

### arduino-cli を使う場合

zip からのインストールは、初期設定では無効になっています。一度だけ有効にしてからインストールします。

```sh
arduino-cli config set library.enable_unsafe_install true
arduino-cli lib install --zip-path KT0803L-1.0.0.zip
arduino-cli compile --fqbn arduino:avr:uno "<スケッチブック>/libraries/KT0803L/examples/KT0803L_FM_Transmitter"
```

## PlatformIO

### インストール

**方法 1：`lib_deps` に zip を指定する（おすすめ）**

`platformio.ini` の `lib_deps` に、zip の場所を `file://` で書きます。ビルドのときに自動で `.pio/libdeps/` に展開されます。

```ini
[env:uno]
platform = atmelavr
board = uno
framework = arduino
monitor_speed = 115200
lib_deps =
    file://C:/Users/<ユーザー名>/Downloads/KT0803L-1.0.0.zip
```

- パスの区切りは `/` で書きます（Windows でも `C:/...`）。
- zip をプロジェクトの中に置いた場合は、プロジェクトのフォルダからの相対パスでも書けます（例：`file://./KT0803L-1.0.0.zip`）。

**方法 2：プロジェクトの `lib` フォルダに置く**

zip を展開し、`KT0803L` フォルダをプロジェクトの `lib` フォルダにコピーします。`lib_deps` に書く必要はありません。

```
my_project/
├─ platformio.ini
├─ lib/
│   └─ KT0803L/
│       ├─ library.json
│       └─ src/KT0803L.h, KT0803L.cpp
└─ src/
    └─ main.cpp
```

### ソースを書く

PlatformIO では `src/main.cpp` に書きます。`.ino` と違って、先頭に `#include <Arduino.h>` が必要です。

```cpp
#include <Arduino.h>
#include <Wire.h>
#include <KT0803L.h>

KT0803L fmtx(Wire);

void setup() {
    Serial.begin(115200);
    Wire.begin();
    Wire.setClock(100000);
    if (!fmtx.begin(88100000UL)) Serial.println("KT0803L not found");
}

void loop() {
}
```

### スケッチ例を動かす

スケッチ例 `KT0803L_FM_Transmitter.ino` を、プロジェクトの `src` フォルダにそのままコピーしてもビルドできます（PlatformIO が `.ino` を自動で変換します）。`src` に `main.cpp` があるときは、どちらか一方にしてください。

```sh
pio run -e uno -t upload    # ビルドして書き込み
pio device monitor          # シリアルモニタ（115200bps）
```

zip を展開した `KT0803L` フォルダがあれば、プロジェクトを作らずに `pio ci` でビルドだけ確かめることもできます。

```sh
pio ci --board uno --lib KT0803L KT0803L/examples/KT0803L_FM_Transmitter/KT0803L_FM_Transmitter.ino
```

### ESP32（M5Stack など）で使う

ボードの指定以外は同じです。ESP32 では `Wire.begin(SDA, SCL, 周波数)` でピンを指定できます。

```ini
[env:m5stack-core2]
platform = espressif32
board = m5stack-core2
framework = arduino
monitor_speed = 115200
lib_deps =
    m5stack/M5Unified@^0.2.13
    file://C:/Users/<ユーザー名>/Downloads/KT0803L-1.0.0.zip
```

```cpp
Wire.begin(32, 33, 100000);  // Core2 の Grove ポート A（SDA = G32、SCL = G33）
```

### 更新

- 方法 1：`lib_deps` の zip のファイル名（版）を書き換えると、次のビルドで入れ替わります。同じファイル名のまま zip だけ差し替えたときは、`.pio/libdeps/<環境名>/KT0803L` フォルダを削除してからビルドしてください。
- 方法 2：`lib/KT0803L` フォルダを新しいものに置き換えます。


# Author:
Junichi Akita (@akita11, akita@ifdl.jp)
