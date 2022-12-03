<!--
title:   PiConsole I/Fでカップ麺タイマー
tags:    3GPI,ADT7410,PiConsole,Python,RaspberryPi
id:      cd9e4ff021a0db60f9a1
private: true
-->
# はじめに

[3GPI][3GPIのURL]に続き、[メカトラックスさん][メカトラックスさんのURL]からPi Console I/F(仮)をお借りすることができましたので基本的な動作確認をしつつカップ麺タイマを作ってみます。

この記事ではPi Console以外は手持ちの機材を使用しますが、Pi Console I/Fは[メカトラックスさん][メカトラックスさんのURL]のラズパイIoTスタータキット「anyPi」専用の同梱品だそうで、単体での販売はないそうです。ご注意いただければ幸いです。

## 使用機材

この記事では以下の機材を用います。はんだ付けや加工は不要です。

- Pi Console I/F
- ラズパイ3
- 3GPI
- USB mini Bケーブル(シリアル用)
- ACアダプタ
(↑↑↑ここまではanyPiに同梱されるもの)
(↓↓↓以下はanyPiに同梱されないもの)

- PC(Windowsマシン)
- I2C接続温度センサ( http://akizukidenshi.com/catalog/g/gM-06675/ )
- PiCamera
- ブレッドボード
- ジャンパケーブル適量

ラズパイに直結できるヘッダコネクタが付いていますので、同様のヘッダコネクタ付の3GPIとともに3階建て構成です。古株さんならPC/104やGP-IB(HP-IB)を思い出されるのではないでしょうか。

![PIC_20160924_113657.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/0eec6b70-3550-959e-b6bf-4c224124c4c2.jpeg)

サンプルとして作成したスクリプトは以下のgithubにアップロードしてあります。

(TODO:githubのURL)

写経が面倒な場合はcloneなどしてお使いください。

## パソコン（Win系）からシリアル接続でラズパイにログイン

（有線/無線LANで接続してssh経由で端末エミュレータを使用する場合はここは読み飛ばして基本編へ進んでください。）

Pi Console I/FにはUSB/シリアル(UART)変換回路が搭載されています。USB mini BケーブルでPCと接続するだけでラズパイのシリアルにアクセスすることができます。

さらにデフォルトでシリアルがttyに設定されていますので、つなぐだけで何もしなくて良いということです。

何もしなくて良いことは、端末エミュレータ(ここではTeraTermを使います)を立ち上げてからラズパイの電源を投入すればすぐに分かります。115,200bps、データ8ビット、ストップビット1、パリティなし、です。

・ラズパイブート時のシリアル出力
![20160924134713.png](https://qiita-image-store.s3.amazonaws.com/0/48424/785bd86c-34ee-e458-b13e-7f8cdb9e166f.png)

ただし、はじめてPi Console I/FとUSBで接続したときにはドライバがインストールされるまでしばらく時間がかかりますので、このブート時のログは見られないかもしれません。手元のWindows8.1ならOS付属のドライバで認識されましたが、それでも数十秒かかりました。

・COMポート認識した状態
![20160924160537.png](https://qiita-image-store.s3.amazonaws.com/0/48424/098d20a6-fd55-1462-38b9-4506321a8241.png)

というわけではじめて見る端末エミュレータの画面は以下のようになるかもしれません。

・はじめてつないだら真っ暗
![20160924135104.png](https://qiita-image-store.s3.amazonaws.com/0/48424/918a9e26-1a95-6b4c-b551-7601ad151609.png)

この場合、Enterキーを叩いてください。ログインプロンプトが表示されるはずです。

・ログイン可能
![20160924135113.png](https://qiita-image-store.s3.amazonaws.com/0/48424/c670767f-8f9b-a2bd-adc1-3b1ac1075c1d.png)

ラズパイのデフォルトのユーザでログインしてみます。

ユーザ名：pi
パスワード：raspberry

```shell-session:端末エミュレータ上
Raspbian GNU/Linux 8 raspberrypi ttyAMA0

raspberrypi login: pi
Password:
Last login: Mon Feb  8 16:03:40 JST 2016 on ttyAMA0
Linux raspberrypi 4.1.17-v7+ #834 SMP Mon Feb 1 15:17:54 GMT 2016 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
```

普段LAN経由のsshでつなぐのに慣れている方は、シリアル接続の端末で戸惑う面もあるかと思います。面倒な部分は慣れるしかないですが、以下の2点のみ対策をします。(参考：[Qiita:RaspberryPi3でシリアル通信を行う][RaspberryPi3でシリアル通信を行うのURL])

- 端末サイズ
- 256色対応

まず問題点を確認します。

・端末サイズが残念な感じ
![20160924141306.png](https://qiita-image-store.s3.amazonaws.com/0/48424/2d0960da-ed00-2f6b-bd8f-393ec429a08a.png)

```shell-session:端末エミュレータ上
pi@raspberrypi:~$ stty rows 36; stty columns 128
pi@raspberrypi:~$ export TERM=xterm-256color; source ~/.bashrc
```

・広々＆256色
![20160924162812.png](https://qiita-image-store.s3.amazonaws.com/0/48424/1aab746d-c85a-c49d-6696-f356a75ee7c3.png)

行数や桁数は環境に合わせて変更してください。


あとはsshでつないだ端末と同じように使えますが、パッケージの更新や追加などの作業は、有線/無線LAN経由で行った方が良いでしょう。DHCPが有効なLAN環境をお使いなら、有線LANケーブルをさすだけです。

シリアルで接続した場合の利点のひとつは、ラズパイの電源を切り入りしても端末エミュレータは開きっぱなしにできることだと思います。端末エミュレータの機能次第ですが、ssh経由では再接続が面倒に感じることがありませんでしょうか。


# 基本編

ここからはPi Console I/Fに搭載されている各機能を個別に動作確認します。

## GPIO

まず、ボタン、LED、およびブザーの動作確認をします。

gpioコマンドを使います。gpioコマンドはインストールしなくても使えますが、3GPI用イメージの3gpi-20160208-2gb.imgでは、以下のようにラズパイ3に対応していないようです。

```shell-session:端末エミュレータ上
pi@raspberrypi:~ $ gpio readall
Oops - unable to determine board type... model: 8
・・・
pi@raspberrypi:~ $ gpio -v
gpio version: 2.31
Copyright (c) 2012-2015 Gordon Henderson
This is free software with ABSOLUTELY NO WARRANTY.
For details type: gpio -warranty

Raspberry Pi Details:
  Type: Unknown08, Revision: 02, Memory: 1024MB, Maker: Sony
  Device tree is enabled.
  This Raspberry Pi supports user-level GPIO access.
    -> See the man-page for more details
```

「Type: Unknown08」となっていますね。そこで、wiringpiパッケージを更新します。

```shell-session:端末エミュレータ上
pi@raspberrypi:~ $ sudo apt-get install wiringpi
・・・
pi@raspberrypi:~ $ gpio -v
gpio version: 2.32
Copyright (c) 2012-2015 Gordon Henderson
This is free software with ABSOLUTELY NO WARRANTY.
For details type: gpio -warranty

Raspberry Pi Details:
  Type: Pi 3, Revision: 02, Memory: 1024MB, Maker: Sony
  * Device tree is enabled.
  * This Raspberry Pi supports user-level GPIO access.
    -> See the man-page for more details
    -> ie. export WIRINGPI_GPIOMEM=1
```

「Type: Pi 3」と認識されました。ラズパイのgpioにはいろいろな設定があるので、まずは元の設定を確認します。

```shell-session:端末エミュレータ上
pi@raspberrypi:~ $ gpio readall
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 | ALT0 | 1 |  3 || 4  |   |      | 5V      |     |     |
 |   3 |   9 |   SCL.1 | ALT0 | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | ALT0 | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 1 | ALT0 | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |  OUT | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |  OUT | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 1 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI |   IN | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO |   IN | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK |   IN | 0 | 23 || 24 | 1 | IN   | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | IN   | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |   IN | 1 | 35 || 36 | 1 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | IN   | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 0 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+---Pi 3---+---+------+---------+-----+-----+
```

上記の表について少しだけ説明しておきます。Physicalの列がラズパイのヘッダピンのピン番号に対応しています。この記事ではgpioの番号はいわゆる「GPIO番号」を用いますので、上記の表では「BCM」の列に書かれた数字に対応します。スクリプト内でもGPIO番号を用います。例えばGPIO20は、ヘッダピンの38番ピンに出ています。

GPIOの番号の対応付けの確認方法が分かったところで、まずはLピカです。LEDを点灯させます。

```shell-session:端末エミュレータ上
pi@raspberrypi:~ $ gpio -g mode 20 out
pi@raspberrypi:~ $ gpio -g mode 21 out
pi@raspberrypi:~ $ gpio -g write 20 1
(赤LED点灯)
pi@raspberrypi:~ $ gpio -g write 21 1
(赤・黄LED点灯)
pi@raspberrypi:~ $ gpio -g write 20 0
pi@raspberrypi:~ $ gpio -g write 21 0
```

・赤・黄LED点灯状態
![PIC_20160924_194759.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/2c5c123c-1ce3-893e-4344-a6be5d25ea98.jpeg)

残りの確認の前にPi Console I/FのIOを確認しておきます。

表1:Pi Console I/Fで使用するGPIO

|GPIO名|PIN番号|設定|機能|
|:-----|:-----|:----|:---|
|GPIO20 | 38 | 出力 | LED1(赤) |
|GPIO21 | 40 | 出力 | LED2(黄) |
|GPIO25 | 22 | 出力 | ブザー |
|GPIO19 | 35 | 入力 | SW1(白) |
|GPIO16 | 36 | 入力 | SW2(黒) |

以下はピンを直接操作しないものです。

表2:Pi Console I/Fで使用するGPIO

|GPIO名|PIN番号|機能|
|:-----|:-----|:-------|
|GPIO2 |3 |液晶SDA|
|GPIO3 |5 |液晶SCL |
|GPIO14 |8 |USB232変換(TXD) |
|GPIO15 |10 |USB232変換(RXD) |

では続きです。ブザーです。結構大きな音が鳴ります。

```shell-session:端末エミュレータ上
pi@raspberrypi:~ $ gpio -g mode 25 out
pi@raspberrypi:~ $ gpio -g write 25 1
pi@raspberrypi:~ $ gpio -g write 25 0
```

続いてSWです。SWは押下状態で'0'となります。(いわゆる負論理)

```shell-session:端末エミュレータ上
pi@raspberrypi:~ $ gpio -g read 19
1
pi@raspberrypi:~ $ gpio -g read 19
0
（SW1白ボタン押下状態）
pi@raspberrypi:~ $ gpio -g read 16
1
pi@raspberrypi:~ $ gpio -g read 16
0
（SW2黒ボタン押下状態）
```

GPIOの確認は以上です。

## I2C

液晶モジュールはI2Cバス経由でラズパイと接続されています。I2Cバスを使用するには、raspi-configでI2Cを有効化する必要があります。

> raspi-config>Advanced Options>I2C

動作確認のためのツールをインストールします。

```shell-session:端末エミュレータ上
pi@raspberrypi:~ $ sudo apt-get install i2c-tools
```

I2Cはバスなので複数のデバイスを数珠つなぎにできます。ということはデバイスを特定する必要があります。さっそくインストールしたツールでデバイスの認識状態を確認します。

```shell-session:端末エミュレータ上
pi@raspberrypi:~ $ sudo i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- 3e --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

この'3e'が液晶モジュールのアドレスです。液晶モジュールの動作確認は「[Qiita:WiringPi-Pythonを使ってAQM0802A / ST7032i LCD表示][WiringPi-Pythonを使ってAQM0802A / ST7032i LCD表示のURL]」のスクリプトを使わせていただきます。wiringpiのpythonモジュールが必要なのでpip3でインストールします。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $  sudo pip3 install wiringpi
```

上記Qiitaの記事のスクリプトの以下の行を変更します。

>import wiringpi2 as wp

となっているところを

>import wiringpi as wp

に変更します。

実行すると液晶画面にtickerが表示されます。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ python3 st7032i.py
```

![2016-09-26-12h16m46s770.png](https://qiita-image-store.s3.amazonaws.com/0/48424/7aa979d6-30c3-8056-cc5d-ffd867b2cff1.png)

I2Cの動作確認は以上です。

## サンプル1:ボタンを押すとLEDが光る

wiringpiを使って簡単なpythonスクリプトを書いてみます。まずはボタンとLEDの連携です。

・ボタンを押したらLEDが光るサンプル

```py3:b1.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import wiringpi as wp
PIN_SW1_WHITE = 19
PIN_SW2_BLACK = 16
PIN_LED1_RED = 20
PIN_LED2_YELLOW = 21
PIN_BUZZER = 25

if __name__ == '__main__':
    wp.wiringPiSetupGpio()
    wp.pinMode(20,1)
    wp.pinMode(21,1)
    wp.pinMode(25,1)
    while True:
        sw1 = wp.digitalRead(PIN_SW1_WHITE)
        sw2 = wp.digitalRead(PIN_SW2_BLACK)

        # swが負論理なので反転してLED出力に設定
        wp.digitalWrite(PIN_LED1_RED, ~sw1 & 1)
        wp.digitalWrite(PIN_LED2_YELLOW, ~sw2 & 1)
        wp.delay(250)
```

以下のように実行します。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ sudo python3 b1.py
```

SW1(白)を押すとLED1(赤)が点灯、SW2(黒)を押すとLED2(黄)が点灯します。

## サンプル2:ボタンを長押しするとリブート/シャットダウン

pythonスクリプトでできることならなんでも連携できます。このサンプルではSW押下の検出をwiringPiISRで割り込み風に記述しています。

```py3:b2.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import wiringpi as wp
PIN_SW1_WHITE = 19
PIN_SW2_BLACK = 16
PIN_LED1_RED = 20
PIN_LED2_YELLOW = 21
PIN_BUZZER = 25

import os

# 1秒以上長押しでTrueを返す
def is_long_pushed(pin):
    count = 0;
    while count < 4:
        state = wp.digitalRead(pin)
        if state != 0:
            return False
        count = count + 1
        wp.delay(250)
    return True

# sw1が押されたコールバック
def sw1_callback():
    if is_long_pushed(PIN_SW1_WHITE):
        print("reboot")
        os.system("sudo reboot")
    else:
        print("reboot cancel")

# sw2が押されたコールバック
def sw2_callback():
    if is_long_pushed(PIN_SW2_BLACK):
        print("shutdown")
        os.system("sudo poweroff")
    else:
        print("shutdown cancel")

if __name__ == '__main__':
    wp.wiringPiSetupGpio()
    wp.pinMode(20,1)
    wp.pinMode(21,1)
    wp.pinMode(25,1)
    # 負論理なので立下りトリガ
    wp.wiringPiISR(PIN_SW1_WHITE, wp.GPIO.INT_EDGE_FALLING, sw1_callback)
    wp.wiringPiISR(PIN_SW2_BLACK, wp.GPIO.INT_EDGE_FALLING, sw2_callback)
    while True:
        wp.delay(250)
```

実行します。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ sudo python3 b1.py
(SW1(白)長押し)
reboot
[ 3388.810715] reboot: Restarting system
・・・
```

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ sudo python3 b1.py
(SW2(黒)長押し)
shutdown
[  123.228773] reboot: Power down
・・・
```

## サンプル3:ボタンを押すと液晶に現在のIPアドレスを表示

液晶モジュールの動作確認で利用したクラスをimportして使うのでシンプルです。st7032i.pyという名前で保存しておきます。

```py3:b3.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import wiringpi as wp
PIN_SW1_WHITE = 19
PIN_SW2_BLACK = 16
PIN_LED1_RED = 20
PIN_LED2_YELLOW = 21
PIN_BUZZER = 25

import sys, socket, struct
from fcntl import ioctl
SIOCGIFADDR = 0x8915

from st7032i import St7032iLCD as LCD
I2C_ADDR_LCD = 0x3e

def get_ip(interface):
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    try:
        ifreq  = struct.pack(b'16s16x', interface)
        ifaddr = ioctl(s.fileno(), SIOCGIFADDR, ifreq)
    finally:
        s.close()
    _, sa_family, port, in_addr = struct.unpack(b'16sHH4s8x', ifaddr)
    return (socket.inet_ntoa(in_addr))

if __name__ == '__main__':
    wp.wiringPiSetupGpio()
    wp.pinMode(20,1)
    wp.pinMode(21,1)
    wp.pinMode(25,1)
    while True:
        sw1 = wp.digitalRead(PIN_SW1_WHITE)
        sw2 = wp.digitalRead(PIN_SW2_BLACK)
        if sw1 == 0:
            ip_addr_eth0 = get_ip(b'eth0')
            lcd = LCD(I2C_ADDR_LCD)
            lcd.clear()
            wp.delay(500)
            lcd.set_cursor(0, 0)
            lcd.print(ip_addr_eth0)
        wp.delay(250)
```

実行してみます。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ sudo python3 b3.py
```

黒SWを押せば、液晶にeth0インタフェースのアドレスが表示されます。

・IPaddress表示
![PIC_20160925_151334.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/43b79d03-8a1a-14ec-5bf2-391833c5c5d7.jpeg)

script上の'eth0'の部分を'ppp0'に変更すれば3Gのpppのアドレスが表示されます。

## サンプル4:ボタンを押すと液晶に現在の温度を表示

（I2C接続の温度センサモジュールをお持ちでない場合はこの項目は読み飛ばしてください。）

I2C接続の温度センサモジュール( http://akizukidenshi.com/catalog/g/gM-06675/ )の資料を参考にしながら配線します。4本だけなので簡単です。

・ピン対応表(モジュール側のPIN番号は仮のものです)

|温度センサモジュール側|(PIN番号)|PIN番号|ラズパイ側|
|:-----|:-----|:-----|:-------|
|VDD |(1) |1 |3.3v|
|SDA |(3)|3 |SDA|
|SCL |(2)|5 |SCL |
|GND |(4)|6 |GND |

・温度センサモジュールの配線図
![adt7410配線.png](https://qiita-image-store.s3.amazonaws.com/0/48424/64dda654-41aa-e3da-14e6-5d770e4abb09.png)

上記画像ではラズパイになっていますが、実際の配線はPi Console I/F上のヘッダピンにおこないます。

![PIC_20160925_124627.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/9596bd44-77df-166d-08d5-96d91719d80d.jpeg)

この際使うのは、片方がオスもう片方がメスのジャンパピンですが、両方がメスのジャンパピンがあればブレッドボードなしですみます。温度センサを引き回して実験する場合、こちらの方が便利かもしれません。

(TODO:写真)
![PIC_20160925_124742.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/77f8a5fe-af1d-216b-0be8-6ab6e58d1437.jpeg)

まずはI2Cバスで認識されているか確認します。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ gpio i2cdetect
0	1	2	3	4	5	6	7	8	9	a	b	c	d	e	f
00:	--	--	--	--	--	--	--	--	--	--	--	--	--
10:	--	--	--	--	--	--	--	--	--	--	--	--	--	--	--	--
20:	--	--	--	--	--	--	--	--	--	--	--	--	--	--	--	--
30:	--	--	--	--	--	--	--	--	--	--	--	--	--	--	3e	--
40:	--	--	--	--	--	--	--	--	48	--	--	--	--	--	--	--
50:	--	--	--	--	--	--	--	--	--	--	--	--	--	--	--	--
60:	--	--	--	--	--	--	--	--	--	--	--	--	--	--	--	--
70:	--	--	--	--	--	--	--	--
```

'48'が温度センサモジュールのアドレスです。ここではgpioコマンドで確認しました。正常認識されているようです。

さらに準備が必要です。ここで使用する温度センサモジュールはちょっとした設定をすることで本来の16ビット分解能で測定が可能です。設定は「[Raspberry Pi で I2C の Repeated Start Condition を有効化][Raspberry Pi で I2C の Repeated Start Condition を有効化のURL]」を参考に行います。

```shell-session:/etc/modprobe.d/i2c.conf
options i2c_bcm2708 combined=1
```

上記ファイルを作成したら、ラズパイを再起動します。

それではスクリプトです。まず温度センサモジュール単体の動作確認です。

```py3:adt7410.py
#!/usr/bin/python
# -*- coding: utf-8 -*-

import wiringpi as wp
import time

I2C_ADDR_THERMO = 0x48

# 2の補数表現変換
# http://stackoverflow.com/questions/1604464/twos-complement-in-python
def twos_comp(val, bits):
    """compute the 2's compliment of int value val"""
    if (val & (1 << (bits - 1))) != 0: # if sign bit is set e.g., 8bit: 128-255
        val = val - (1 << bits)        # compute negative value
    return val                         # return positive value as is

class ADT7410:
    """
    ADT7410 thermometer control class
    """
    def __init__(self, i2c_addr = I2C_ADDR_THERMO):
        self.i2c = wp.I2C()
        self.fd = self.i2c.setup(i2c_addr)
        # 分解能を16ビットに設定
        self._write(0x003, 0x80)

    def _write(self, offset, data):
        self.i2c.writeReg8(self.fd, offset, data)

    def _read(self, offset):
        datum = self.i2c.readReg8(self.fd, offset)
        return datum

    def read_temp(self):
        msb = self._read(0x00)
        lsb = self._read(0x01)
        temp = (msb << 8 | lsb)  # MSB, LSB
        temp = twos_comp(temp, 16)
        return temp

if __name__ == '__main__':
    thermo = ADT7410(I2C_ADDR_THERMO)
    while True:
        temp = thermo.read_temp()
        print("Temperature:%6.2f" % (temp / 128.0))
        time.sleep(1)
```

上記スクリプトは単体でも動作するので実行して動作確認を行います。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ python3 adt7410.py
Temperature: 27.98
Temperature: 28.02
Temperature: 27.98
・・・
```

1秒ごとに温度が表示されますので、室温と近い値が表示されていればOKです。指でICに触れるなどして温度が変化することも確認しておきます。

これで温度センサモジュールが使えるようになりました。「ボタンを押したら～」のスクリプトは以下です。

```py3:b4.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import wiringpi as wp
PIN_SW1_WHITE = 19
PIN_SW2_BLACK = 16
PIN_LED1_RED = 20
PIN_LED2_YELLOW = 21
PIN_BUZZER = 25

from st7032i import St7032iLCD as LCD
I2C_ADDR_LCD = 0x3e

from adt7410 import ADT7410 as THERMO
I2C_ADDR_THERMO = 0x48

if __name__ == '__main__':
    wp.wiringPiSetupGpio()
    wp.pinMode(20,1)
    wp.pinMode(21,1)
    wp.pinMode(25,1)
    lcd = LCD(I2C_ADDR_LCD)
    thermo = THERMO(I2C_ADDR_THERMO)
    while True:
        sw1 = wp.digitalRead(PIN_SW1_WHITE)
        sw2 = wp.digitalRead(PIN_SW2_BLACK)
        if sw2 == 0:
            temp = thermo.read_temp() / 128.0
            lcd.clear()
            wp.delay(250)
            lcd.set_cursor(0, 0)
            lcd.print("{0:6.2f}".format(temp))
        wp.delay(250)
```

実行して、SW2(黒)を押すと液晶画面に温度が表示されます。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ sudo python3 b4.py
```

![PIC_20160925_155544.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/a4a99426-b7e4-9d6f-f2f6-e0ac80c301cb.jpeg)

## サンプル5:温度センサからの値が閾値超えるとブザーが鳴る

（I2C接続の温度センサモジュールをお持ちでない場合はこの項目は読み飛ばしてください。）

```py3:b5.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import wiringpi as wp
PIN_SW1_WHITE = 19
PIN_SW2_BLACK = 16
PIN_LED1_RED = 20
PIN_LED2_YELLOW = 21
PIN_BUZZER = 25

from adt7410 import ADT7410 as THERMO
I2C_ADDR_THERMO = 0x48

if __name__ == '__main__':
    wp.wiringPiSetupGpio()
    wp.pinMode(20,1)
    wp.pinMode(21,1)
    wp.pinMode(25,1)
    thermo = THERMO(I2C_ADDR_THERMO)
    t1 = 32.0   # 閾値1
    print("threshold t1:{0:6.2f}".format(t1))
    while True:
        sw1 = wp.digitalRead(PIN_SW1_WHITE)
        sw2 = wp.digitalRead(PIN_SW2_BLACK)
        temp = thermo.read_temp() / 128.0
        print("{0:6.2f}".format(temp))
        if temp > t1:
            print("over t1")
            wp.digitalWrite(PIN_BUZZER, 1)
        else:
            wp.digitalWrite(PIN_BUZZER, 0)
        wp.delay(500)
```

それでは実行します。閾値は32℃に設定していますので、指で触れるとすぐに閾値を超えてブザーが鳴ります。指を離して閾値を下回ればブザーは止まります。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ sudo python3 b5.py
threshold t1: 32.00
 30.59
 30.50
 30.49
 30.99
 31.42
 31.74
 31.97
 32.15
over t1
 32.28
over t1
 32.40
over t1
 32.52
over t1
 32.42
over t1
 32.17
over t1
 31.98
 31.70
```

## サンプル6:3GPIの接続状況を液晶に表示

3GPIの接続状況を液晶に表示します。3GPIの3G接続のAPN設定などについては、手前味噌ですが「[Qiita:猫Piカメラ～DMM.comのSIMで接続テスト][猫Piカメラ～DMM.comのSIMで接続テストのURL]」などを参考に設定しておきます。

3GPIでは接続の管理にNetworkManagerが使われていますので、これをPythonから利用する準備をします。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ sudo apt-get install python3-dbus
・・・
pi@raspberrypi:~/sandbox $ sudo pip3 install python-networkmanager
```

ちなみにですが、3GPI用イメージの3gpi-20160208-2gb.imgをベースにした手元の環境では、pip3からdbusモジュールをインストールできませんでした。

インタプリタで確認しておきます。

```pycon:NetworkManagerの確認
Python 3.4.2 (default, Oct 19 2014, 13:31:11)
[GCC 4.9.1] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import NetworkManager
>>> NetworkManager.NetworkManager.Version
'0.9.10.0'
>>> [(dev.Interface, dev.SpecificDevice().__class__.__name__)
...  for dev in NetworkManager.NetworkManager.GetDevices()]
[('lo', 'Generic'), ('eth0', 'Wired'), ('ttyUSB3', 'Modem')]
```

接続状況はpython-networkmanagerのexampleにあるinfo.pyの以下の部分を参考にします。

> print("%-30s %s" % ("Overall state:", c('state', NetworkManager.NetworkManager.State)))

スクリプトはサンプル3をベースにして作成しました。

```py3:b6.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import wiringpi as wp
PIN_SW1_WHITE = 19
PIN_SW2_BLACK = 16
PIN_LED1_RED = 20
PIN_LED2_YELLOW = 21
PIN_BUZZER = 25

from st7032i import St7032iLCD as LCD
I2C_ADDR_LCD = 0x3e

import NetworkManager
c = NetworkManager.const

if __name__ == '__main__':
    wp.wiringPiSetupGpio()
    wp.pinMode(20,1)
    wp.pinMode(21,1)
    wp.pinMode(25,1)
    lcd = LCD(I2C_ADDR_LCD)
    while True:
        sw1 = wp.digitalRead(PIN_SW1_WHITE)
        sw2 = wp.digitalRead(PIN_SW2_BLACK)
        if sw1 == 0:
            lcd.clear()
            wp.delay(250)
            lcd.set_cursor(0, 0)
            lcd.print("state:")
            state = "{0}".format(c('state', NetworkManager.NetworkManager.State))
            lcd.set_cursor(0, 1)
            lcd.print(state)
        wp.delay(500)
```

実行してSW1(白)を押す度に以下のように表示されます。

> state:
> disconnected

3gpictlユーティリティで3gpiの電源を投入すれば、表示内容が変化するはずです。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ 3gpictl --poweron
```

端末がシリアルのものだけなら、スクリプトをバックグラウンド実行しておいて、フォアグラウンドで3gpictlを実行しても良いでしょう。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/sandbox $ sudo python3 b6.py &
```

手元の環境では切断状態から以下のように変化します。

・切断→接続済みへの変化
![2016-09-26-14h35m56s985.png.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/4b079ffe-4a5d-2743-0427-047f41563c5a.jpeg)
↓↓↓
![2016-09-26-14h36m21s407.png.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/6364470d-a18d-0f9f-6d18-768628490ea1.jpeg)
↓↓↓
![2016-09-26-14h36m25s606.png.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/373c372d-544d-a1e1-d4e0-09e486472ab8.jpeg)

# 応用編

基本編よりも少しだけ複雑なサンプルを作成します。

## 応用1:カップ麺タイマを実装(ボタンとブザーと液晶)

シンプルなカップ麺タイマを実装します。

### 仕様

- ボタン2白押下で3,4,5分を切り替える
- ボタン1黒押下でスタート
- 液晶に残り時間を表示
- 時間が来たらブザー鳴動
- ボタン1/2押下でブザー停止

### 実装

ちょっと長いです。

```py3:a1.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import wiringpi as wp
PIN_SW1_WHITE = 19
PIN_SW2_BLACK = 16
PIN_LED1_RED = 20
PIN_LED2_YELLOW = 21
PIN_BUZZER = 25

from st7032i import St7032iLCD as LCD
I2C_ADDR_LCD = 0x3e

from datetime import timedelta, datetime

class RamenTimer:
    """
    ramen-timer class
    """
    def __init__(self):
        self.lcd = LCD(I2C_ADDR_LCD)
        self.lcd.clear()
        self.timer = 3
        self.sw1_old = 1
        self.sw2_old = 1

    def beep(self, duration = 100):
        wp.digitalWrite(PIN_BUZZER, 1)
        wp.delay(duration)
        wp.digitalWrite(PIN_BUZZER, 0)

    def proc_inputs(self):
        sw1 =  wp.digitalRead(PIN_SW1_WHITE)
        if sw1 == 0 and self.sw1_old == 1:
            sw1_pushed = True
        else:
            sw1_pushed = False
        self.sw1_old = sw1
        sw2 =  wp.digitalRead(PIN_SW2_BLACK)
        if sw2 == 0 and self.sw2_old == 1:
            sw2_pushed = True
        else:
            sw2_pushed = False
        self.sw2_old = sw2
        return (sw1_pushed, sw2_pushed)

    def timer_select(self):
        while True:
            (sw1, sw2) = self.proc_inputs()
            if sw1:
                return
            if sw2:
                self.beep()
                self.timer = self.timer + 1
            if self.timer > 5:
                self.timer = 3
            self.lcd.clear()
            self.lcd.set_cursor(0, 0)
            self.lcd.print("{0}:00".format(self.timer))
            wp.delay(100)

    def timer_run(self):
        start = datetime.now()
        while True:
            (sw1, sw2) = self.proc_inputs()
            if sw1:
                return False
            delta = datetime.now() - start
            elapse = delta.seconds
            self.lcd.clear()
            self.lcd.set_cursor(0, 0)
            self.lcd.print("{0}:{1:02d}".format(int(elapse/60), elapse % 60))
            if elapse >= self.timer*60:
                return True
            wp.delay(250)

    def timer_buzz(self):
        count = 0
        while True:
            (sw1, sw2) = self.proc_inputs()
            if sw1 or sw2:
                wp.digitalWrite(PIN_BUZZER, 0)
                return
            if count in [0, 2, 4]:
                wp.digitalWrite(PIN_BUZZER, 1)
            else:
                wp.digitalWrite(PIN_BUZZER, 0)
            if count in [0, 2, 4, 6, 8]:
                wp.digitalWrite(PIN_LED1_RED, 1)
            else:
                wp.digitalWrite(PIN_LED1_RED, 0)
            count = (count + 1) % 10
            wp.delay(100)

if __name__ == '__main__':
    wp.wiringPiSetupGpio()
    wp.pinMode(20,1)
    wp.pinMode(21,1)
    wp.pinMode(25,1)
    wp.digitalWrite(PIN_BUZZER, 0)

    rt = RamenTimer()
    rt.timer_select()
    ret = rt.timer_run()
    if ret:
        rt.timer_buzz()
```

ボタンは「押して離したとき」をトリガとしています。ブザーは3回短く鳴る感じに実装しています。タイマーカウント中にボタンを押すと途中終了します。
GPIOでUIを構築する面倒さがお分かりいただけるかと思います。

## 応用2:PiCamera

（PiCameraモジュールをお持ちでない場合はこの項目は読み飛ばしてください。）

ボタンを押すと写真を撮影するサンプルです。

PiCameraを使うにはraspi-configでカメラを有効化する必要があります。

> raspi-config＞Enable Camera

再起動後にraspistillやraspividコマンドでカメラが使えるようになります。

スクリプトはカップ麺タイマを切り貼りして作成しました。

```py3:a2.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import wiringpi as wp
PIN_SW1_WHITE = 19
PIN_SW2_BLACK = 16
PIN_LED1_RED = 20
PIN_LED2_YELLOW = 21
PIN_BUZZER = 25

from st7032i import St7032iLCD as LCD
I2C_ADDR_LCD = 0x3e

import os
from datetime import timedelta, datetime

class DamnCamera:
    """
    damn camera class
    """
    def __init__(self):
        self.lcd = LCD(I2C_ADDR_LCD)
        self.lcd.clear()
        self.sw1_old = 1
        self.sw2_old = 1

    def beep(self, duration = 100):
        wp.digitalWrite(PIN_BUZZER, 1)
        wp.delay(duration)
        wp.digitalWrite(PIN_BUZZER, 0)

    def proc_inputs(self):
        sw1 =  wp.digitalRead(PIN_SW1_WHITE)
        if sw1 == 0 and self.sw1_old == 1:
            sw1_pushed = True
        else:
            sw1_pushed = False
        self.sw1_old = sw1
        sw2 =  wp.digitalRead(PIN_SW2_BLACK)
        if sw2 == 0 and self.sw2_old == 1:
            sw2_pushed = True
        else:
            sw2_pushed = False
        self.sw2_old = sw2
        return (sw1_pushed, sw2_pushed)

    def run(self):
        while True:
            (sw1, sw2) = self.proc_inputs()
            if sw1:
                return False
            if sw2:
                self.lcd.clear()
                self.lcd.set_cursor(0, 0)
                self.lcd.print("<capturing...>")
                self.beep()
                fn = datetime.now().strftime('%m%d_%H%M%S.jpg')
                os.system("raspistill -o {0}".format(fn))
                self.lcd.clear()
                self.lcd.set_cursor(0, 0)
                self.lcd.print("{0}".format(fn))
                self.beep(200)
            wp.delay(250)

if __name__ == '__main__':
    wp.wiringPiSetupGpio()
    wp.pinMode(20,1)
    wp.pinMode(21,1)
    wp.pinMode(25,1)
    wp.digitalWrite(PIN_BUZZER, 0)

    cam = DamnCamera()
    cam.run()
```

SW2(黒)を押すと短くブザーが鳴って撮影します。撮影画像の保存が完了したら長めにブザーが鳴って、液晶画面にファイル名が表示されます。SW1(白)を押すと終了します。

## 応用3:チャタリング対策(debounce)

基本編のサンプルのように、python上で100[msec]オーダーのdelayでループを回せば、ある程度チャタリング対策となります。あまり気にもならないと思います。一方、サンプル2のようにwiringPiISRを使うとチャタリング現象に悩まされます。

とにかくまぁ、pythonスクリプトでチャタリング現象を捉えたものが以下です。適当に改行を追加しています。

```
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111110000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000000000000000000000110000000000000000000000000011111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111
```

スクリプトは以下のようなものです。これを数回実行して、ボタンを数回押しただけで上記現象が採れました。

```py3:チャタリング観測
    while True:
        sw1 = wp.digitalRead(PIN_SW1)

        wp.digitalWrite(PIN_LED_RED, ~sw1 & 1)

        print(sw1, end='')
```

whileループ1周は1msec未満で実行されます。上記結果の「・・・0001100・・・」の部分がチャタリング現象と考えられます。
何も考えずにスクリプトを書けば、ボタンは実際には1回しか押されていないのに、2回押されたことになるわけです。

UI操作におけるSWのチャタリングは操作性を気にしなければ大した問題ではないと思われるかもしれませんが、チャタリングのせいで存在しないプログラムのバグを追いかけるハメに陥ることもおおいにあるので、ソフトで対応しておくべきでしょう。入力のチャタリングが出力にそのまま出るとより深刻な問題を引き起こします。例えばギヤードモーターなどの寿命が極端に短くなったりします。

このように面倒なチャタリング対策ですが、ハードであってもソフトであっても対策によって入力の応答性は悪化します。例えばタイマを入れたり、CR回路でフィルタすれば、入力がそれだけ遅れます。ポチポチ高速連打するようなボタンやセンサなどでは問題になるということです。

そのため、「どんな場合でも有効なユニバーサルなチャタリング対策」は実はありません。トレードオフがあります。重要なのは「ヒステリシスがあるから心配ない」とか「水銀スイッチにはチャタリングは生じない」などの話を鵜呑みにしないことではないでしょうか。チャタリングはせいぜいmsecオーダーの現象ですので、安物のオシロでも充分観測できると思います。

いずれにしても、実際に必要な対策はソフトだけでは決まらないので、ハード、システム全体で検討するべきでしょう。

# 最後に

Pi Console I/FはLED、ボタン、ブザー、液晶というシンプルな構成ですが、ラズパイ直結でかつシリアルUSB変換も搭載しているので、非常にコンパクトに収まります。ノートPCとセットにして持ち歩いてデモするのも楽そうです。

・3階建て再掲
![PIC_20160924_113657.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/4bef4ce2-0291-3b96-ab7f-eef68d12667d.jpeg)

試しにブレッドボードで似たようなI/Oを組んでみた写真を以下にお示しします。

・スパゲティ載せ
![PIC_20160924_112139.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/1ce1d997-3c37-eafd-d8dc-3c695966d7cf.jpeg)

なかなか厄介です。ジャンパ線が邪魔してボタンが押せません。Pi Console I/Fなら、つないですぐに使えるプラグアンドプレイというわけです。

また、Pi Console I/Fには3GPIと同様にヘッダピンが出ていますので、この記事のようにデバイスを追加して簡単に拡張することができます。

この記事がIoT機器開発のお役に立てれば幸いです。

ここまでお読みいただきありがとうございました。

# リンク
[メカトラックスさんのサイト][メカトラックスさんのURL]
[3GPI][3GPIのURL]
[猫Piカメラの記事][ラズパイと3GPIで猫PiカメラのURL]
[遠隔操作OAタップの記事][遠隔操作OAタップURL]
[3GPIとOBDIIでPimetryシステム][3GPIとOBDIIでPimetryシステムのURL]
[3GPI付属SDイメージ][3GPI付属SDイメージのURL]
[Qiita:RaspberryPi3でシリアル通信を行う][RaspberryPi3でシリアル通信を行うのURL]
[Qiita:WiringPi-Pythonを使ってAQM0802A / ST7032i LCD表示][WiringPi-Pythonを使ってAQM0802A / ST7032i LCD表示のURL]
[Raspberry Pi で I2C の Repeated Start Condition を有効化][Raspberry Pi で I2C の Repeated Start Condition を有効化のURL]
[Qiita:猫Piカメラ～DMM.comのSIMで接続テスト][猫Piカメラ～DMM.comのSIMで接続テストのURL]


[メカトラックスさんのURL]:http://www.mechatrax.com/
[3GPIのURL]:http://www.mechatrax.com/products/3gpi
[ラズパイと3GPIで猫PiカメラのURL]:http://qiita.com/syasuda/items/5def33db425aaf05bf6d
[遠隔操作OAタップURL]:http://qiita.com/syasuda/items/f683a1c0fe9a7e51fb68
[3GPIとOBDIIでPimetryシステムのURL]:http://qiita.com/syasuda/items/3a08851dbe4ca1876409
[3GPI付属SDイメージのURL]:http://mechatrax.com/data/3gpi/
[RaspberryPi3でシリアル通信を行うのURL]:http://qiita.com/yamamotomanabu/items/33b6cf0d450051d33d41
[WiringPi-Pythonを使ってAQM0802A / ST7032i LCD表示のURL]:http://qiita.com/f_nishio/items/b1b99b4763993a9239c6
[Raspberry Pi で I2C の Repeated Start Condition を有効化のURL]:http://rabbit-note.com/2015/02/15/raspberry-pi-i2c-repeated-start/
[猫Piカメラ～DMM.comのSIMで接続テストのURL]:http://qiita.com/syasuda/items/5def33db425aaf05bf6d#dmmcom%E3%81%AEsim%E3%81%A7%E6%8E%A5%E7%B6%9A%E3%83%86%E3%82%B9%E3%83%88