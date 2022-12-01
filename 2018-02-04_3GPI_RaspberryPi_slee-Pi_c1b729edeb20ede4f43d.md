<!--
title:   slee-Piを使った間欠動作（タイマー動作）と死活監視
tags:    3GPI,RaspberryPi,slee-Pi
id:      c1b729edeb20ede4f43d
private: true
-->
# はじめに

[以前の記事](2018-01-12_3GPI_RaspberryPi_SMS_SORACOM_slee-Pi_26bbe73bc62ee5856188.md)では、[メカトラックス][メカトラックスさんのURL]さんの[slee-Pi 2][sleepi2のURL]のSMS着信起動を試しましたが、この記事では[slee-Pi 2][sleepi2のURL]のタイマー動作と死活監視の基本的な使い方を確認します。

![IMG_20180102_170158 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/2c966820-db7d-31b6-dc03-e45236400b3f.jpeg)

## 使用機材

この記事では以下の機材を用います。

- ラズパイ3B(含マイクロSDカード 4GB以上)
- [slee-Pi 2][sleepi2のURL]
- 12V DCアダプター([3GPi Ver.2][3GPIのURL]付属のものなど)
- 直流安定化電源(なくても可)
- [Pi カメラ][Pi CameraのURL](なくても可)

# 準備

## 組み立て

40ピンヘッダを向きを合わせて刺すだけです。
以下の写真の通り、下がラズパイ、上に[slee-Pi 2][sleepi2のURL]が来るように接続します。

![IMG_20180321_133343 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/eb839caf-252e-c147-aeb5-59b29fc1e2e6.jpeg)

ラズパイへの電源供給はヘッダピン経由で行われますので、電源は[slee-Pi 2][sleepi2のURL]付属の電源ハーネス経由で[slee-Pi 2][sleepi2のURL]のみ供給するようにします。ラズパイの電源用USBには給電しないようにします。
以下の写真の通り、[slee-Pi 2][sleepi2のURL]の電源ハーネスだけを接続します。

![IMG_20180321_133427 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/b0b02c93-c3a2-0b0f-de53-47f88272afa0.jpeg)

## ジャンパ設定

デフォルトのまま使用します。
変更して使用する場合はGPIOの番号を読み替えます。

| GPIO 名 | 設定 | 機能 | 備考 |
|:------|:-------|:---------|:---|
| GPIO18 / GPIO16 | IN   | RI        | JP1 で選択 |
| **GPIO26** / GPIO23 | IN   | IRQ       | JP2 で選択 |
| **GPIO5** / GPIO24 | OUT  | HEARTBEAT | JP3 で選択 |


## 電源投入

[slee-Pi 2][sleepi2のURL]の電源ハーネスにACアダプタを接続し、ACアダプタをコンセントやACタップに刺すとラズパイが起動します。

## Raspbianの準備

公開されているイメージをマイクロSDカードに書き込みます。

[Raspbianのダウンロード][raspbianのURL]からイメージをダウンロードして使います。3月13日の最新のものです。

> 2018-03-13-raspbian-stretch-lite.zip

書き込みには以下のものが必要です。

* マイクロSDカードリーダー
* PC

ここではWindowsマシンとSDカードアダプタとUSBカードリーダを使います。

![IMG_20180317_151643 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/272b54b5-2399-5980-7044-6c8917835843.jpeg)


PCに接続します。

![IMG_20180317_151949 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/eef60fb2-cc5f-3a90-0012-f34f299b7b11.jpeg)


"フォーマットしますか？"などのダイアログはすべてキャンセルします。

[Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/)を使って、イメージを書き込みます。

"2018-03-13-raspbian-stretch-lite.img"を指定して"Drive"でUSBカードリーダーを指定してから、"Write"をクリックします。カードリーダーが複数のドライブレター（例 D:と E:など）を持つ場合は、先頭を指定します。


![20180318165415.png](https://qiita-image-store.s3.amazonaws.com/0/48424/ae3b49af-7c15-84a4-79e1-5371d223ae5e.png)

"全部消えますよ"という警告ダイアログが出ますので、HDDやSSDの大事なデータが入ったドライブではないことを確認して"Yes"をクリックします。

![20180318165521.png](https://qiita-image-store.s3.amazonaws.com/0/48424/097fc40c-14eb-08ef-6539-99b8c63711ab.png)

書き込みには数分かかります。

![20180318165622.png](https://qiita-image-store.s3.amazonaws.com/0/48424/35bbf581-c4d5-18ff-d5ee-50118ddb03e0.png)

![20180318170614.png](https://qiita-image-store.s3.amazonaws.com/0/48424/bd3863a0-c33e-3f59-c2f2-3cfdaf56904d.png)


PCからとりはずして、マイクロSDカードをラズパイに刺します。裏返しです。

![IMG_20180317_152103 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/911ec2a5-ba02-e03a-21de-d6fa5c422184.jpeg)

![IMG_20180317_152133 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/28104ea9-35c8-d6c7-820c-85c9f43f809b.jpeg)

電源ハーネスを接続します。ACアダプタにはまだ接続しません。

![IMG_20180317_152216 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/6e6b7ac6-dab6-054f-f499-4586cc5524d8.jpeg)

USBキーボード、HDMIディスプレイに接続します。LANもつないでよいです。

![IMG_20180317_152356 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/893944bd-41fc-c2b2-21ba-bc216a549bdd.jpeg)

電源を投入するとHDMIディスプレイにこんなのが出ます。

![vlcsnap-2018-03-18-17h30m49s228.png](https://qiita-image-store.s3.amazonaws.com/0/48424/e3045937-0ff9-cda1-4e50-8c5a019bd702.png)

![20180317152740.PNG](https://qiita-image-store.s3.amazonaws.com/0/48424/64849830-d985-8d31-0fb0-c31d413d1bf6.png)

ログインプロンプトが出たら、pi/<パスワード>でログインします。

![20180317152838.PNG](https://qiita-image-store.s3.amazonaws.com/0/48424/5aeadb03-d5f3-0ef0-7725-3787cbe5e780.png)

DHCPのLANを使用している場合は、ここでLANケーブルを接続してifconfigコマンドでIPアドレスを確認しておきます。

![20180317153005.PNG](https://qiita-image-store.s3.amazonaws.com/0/48424/4796423c-48ea-d175-1478-c208025c12ba.png)

## タイムゾーンなどの設定と、SSH有効化

長くなるので、別記事にしておきました。ご参照ください。

[Raspbian stretch Liteで初期設定](https://qiita.com/syasuda/private/fec9fe2486f6a6536ef2)

## パッケージのインストール

[slee-Piのセットアップ][slee-PiのセットアップのURL]の手順に従って、[slee-Pi 2][sleepi2のURL]のパッケージを更新/インストールします。

```bash:コンソール
pi@raspberrypi:~ $ sudo bash -c 'echo "deb http://mechatrax.github.io/raspbian/ stretch main contrib non-free" > /etc/apt/sources.list.d/mechatrax.list'
pi@raspberrypi:~ $ wget http://mechatrax.github.io/raspbian/pool/main/m/mechatrax-archive-keyring/mechatrax-archive-keyring_2016.12.19.1_all.deb
--2018-03-18 22:29:51--  http://mechatrax.github.io/raspbian/pool/main/m/mechatrax-archive-keyring/mechatrax-archive-keyring_2016.12.19.1_all.deb
Resolving mechatrax.github.io (mechatrax.github.io)... 151.101.73.147
Connecting to mechatrax.github.io (mechatrax.github.io)|151.101.73.147|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4880 (4.8K) [application/octet-stream]
Saving to: ‘mechatrax-archive-keyring_2016.12.19.1_all.deb’

mechatrax-archive-keyring_2016.12 100%[============================================================>]   4.77K  --.-KB/s    in 0s

2018-03-18 22:29:51 (97.2 MB/s) - ‘mechatrax-archive-keyring_2016.12.19.1_all.deb’ saved [4880/4880]

pi@raspberrypi:~ $ sudo dpkg -i mechatrax-archive-keyring_2016.12.19.1_all.deb
Selecting previously unselected package mechatrax-archive-keyring.
(Reading database ... 34533 files and directories currently installed.)
Preparing to unpack mechatrax-archive-keyring_2016.12.19.1_all.deb ...
Unpacking mechatrax-archive-keyring (2016.12.19.1) ...
Setting up mechatrax-archive-keyring (2016.12.19.1) ...
pi@raspberrypi:~ $
```

```bash:コンソール
pi@raspberrypi:~ $ sudo apt-get update
Get:1 http://mechatrax.github.io/raspbian stretch InRelease [5,026 B]
Hit:2 http://mirrordirector.raspbian.org/raspbian stretch InRelease
Hit:3 http://archive.raspberrypi.org/debian stretch InRelease
Get:4 http://mechatrax.github.io/raspbian stretch/main armhf Packages [2,009 B]
Get:5 http://mechatrax.github.io/raspbian stretch/contrib armhf Packages [802 B]
Get:6 http://mechatrax.github.io/raspbian stretch/non-free armhf Packages [858 B]
Fetched 8,695 B in 1s (6,283 B/s)
Reading package lists... Done
pi@raspberrypi:~ $ sudo apt-get install sleepi2-firmware sleepi2-utils sleepi2-monitor
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  i2c-tools python3-sleepi python3-smbus read-edid
Suggested packages:
  libi2c-dev python-smbus
The following packages will be REMOVED:
  fake-hwclock
The following NEW packages will be installed:
  i2c-tools python3-sleepi python3-smbus read-edid sleepi2-firmware sleepi2-monitor sleepi2-utils
0 upgraded, 7 newly installed, 1 to remove and 7 not upgraded.
Need to get 101 kB of archives.
After this operation, 364 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://mechatrax.github.io/raspbian stretch/main armhf python3-sleepi all 1.0.2 [4,110 B]
Get:2 http://mechatrax.github.io/raspbian stretch/non-free armhf sleepi2-firmware armhf 1.1.3 [3,558 B]
Get:3 http://mechatrax.github.io/raspbian stretch/contrib armhf sleepi2-utils all 1.1.0 [7,096 B]
Get:4 http://mirrordirector.raspbian.org/raspbian stretch/main armhf python3-smbus armhf 3.1.2-3 [9,774 B]
Get:5 http://mechatrax.github.io/raspbian stretch/contrib armhf sleepi2-monitor all 1.0.3 [4,360 B]
Get:6 http://mirrordirector.raspbian.org/raspbian stretch/main armhf read-edid armhf 3.0.2-1 [15.4 kB]
Get:7 http://mirrordirector.raspbian.org/raspbian stretch/main armhf i2c-tools armhf 3.1.2-3 [56.7 kB]
Fetched 101 kB in 3s (31.2 kB/s)
(Reading database ... 34538 files and directories currently installed.)
Removing fake-hwclock (0.11) ...
Selecting previously unselected package python3-smbus:armhf.
(Reading database ... 34532 files and directories currently installed.)
Preparing to unpack .../0-python3-smbus_3.1.2-3_armhf.deb ...
Unpacking python3-smbus:armhf (3.1.2-3) ...
Selecting previously unselected package python3-sleepi.
Preparing to unpack .../1-python3-sleepi_1.0.2_all.deb ...
Unpacking python3-sleepi (1.0.2) ...
Selecting previously unselected package read-edid.
Preparing to unpack .../2-read-edid_3.0.2-1_armhf.deb ...
Unpacking read-edid (3.0.2-1) ...
Selecting previously unselected package sleepi2-firmware.
Preparing to unpack .../3-sleepi2-firmware_1.1.3_armhf.deb ...
Unpacking sleepi2-firmware (1.1.3) ...
Selecting previously unselected package sleepi2-utils.
Preparing to unpack .../4-sleepi2-utils_1.1.0_all.deb ...
Unpacking sleepi2-utils (1.1.0) ...
Selecting previously unselected package sleepi2-monitor.
Preparing to unpack .../5-sleepi2-monitor_1.0.3_all.deb ...
Unpacking sleepi2-monitor (1.0.3) ...
Selecting previously unselected package i2c-tools.
Preparing to unpack .../6-i2c-tools_3.1.2-3_armhf.deb ...
Unpacking i2c-tools (3.1.2-3) ...
Setting up read-edid (3.0.2-1) ...
Setting up i2c-tools (3.1.2-3) ...
Setting up sleepi2-firmware (1.1.3) ...
Setting up python3-smbus:armhf (3.1.2-3) ...
Setting up python3-sleepi (1.0.2) ...
Processing triggers for man-db (2.7.6.1-2) ...
Setting up sleepi2-utils (1.1.0) ...
Created symlink /etc/systemd/system/sysinit.target.wants/sleepi2-start.service → /lib/systemd/system/sleepi2-start.service.
Created symlink /etc/systemd/system/poweroff.target.wants/sleepi2-stop.service → /lib/systemd/system/sleepi2-stop.service.
Created symlink /etc/systemd/system/reboot.target.wants/sleepi2-restart.service → /lib/systemd/system/sleepi2-restart.service.
Created symlink /etc/systemd/system/shutdown.target.wants/sleepi2-systohc.service → /lib/systemd/system/sleepi2-systohc.service.
Created symlink /etc/systemd/system/sysinit.target.wants/sleepi2-heartbeat.service → /lib/systemd/system/sleepi2-heartbeat.service.
Setting up sleepi2-monitor (1.0.3) ...
Created symlink /etc/systemd/system/multi-user.target.wants/sleepi2-monitor.service → /lib/systemd/system/sleepi2-monitor.service.
pi@raspberrypi:~ $
```

パッケージのインストール後に再起動します。

```bash:コンソール
$ sudo shutdown -r now
```

最後に時刻を設定します。

```bash:コンソール
pi@raspberrypi:~ $ date
Sun 18 Mar 23:02:53 JST 2018
pi@raspberrypi:~ $ sudo hwclock -w
pi@raspberrypi:~ $ date
Sun 18 Mar 23:03:02 JST 2018
pi@raspberrypi:~ $
```

これで[slee-Pi 2][sleepi2のURL]上のRTCのカレンダーにラズパイの現在時刻が設定されます。
RTCはラズパイの電源を切っていてもキャパシタに蓄えられた電荷によって時を刻み続けます。

# 本題

やっと準備が終わりました。ここから本題です。

## 最初の一歩

[slee-Pi 2][sleepi2のURL]によってできることを確認していきます。
まずはSW1(起動用スイッチ)の使い方を確認します。

ここまでの手順通りなら、ラズパイは起動していると思います。
赤LEDが点灯、緑LEDがSDカードへのアクセスランプのように時々点灯しているはずです。
一方[slee-Pi 2][sleepi2のURL]の赤LEDは点灯、緑LEDが定期的に点滅しているはずです。

この状態でSW1を3秒間(デフォルト)長押しします。電源コネクタCN1/CN2の横にある黄色いスイッチ(プッシュスイッチ)です。
先に[slee-Pi 2][sleepi2のURL]の緑LEDの点滅が停止してから、赤LEDが数秒間点滅して消灯します。
それからラズパイの赤LEDが消灯するはずです。
これでラズパイがシャットダウンしました。これを"スリープ状態"と呼ぶようです。

つづいてSW1を押すと、[slee-Pi 2][sleepi2のURL]とラズパイの赤LEDが点灯してラズパイが起動するはずです。
SW1は便利な電源ボタンとして使えるわけです。

長押し時間の3秒は以下のファイルで変更できます。

```bash:/etc/sleepi2-monitor.conf
pi@raspberrypi:~ $ sudo vi /etc/sleepi2-monitor.conf
・・・
[switch]
command = "shutdown -h now"
oneshot = true
threshold = 3
・・・
```

ただし変更を反映するにはシャットダウンが必要ですのでいったんシャットダウンします。(スリープ状態)
この状態でSW1を押さずに、[slee-Pi 2][sleepi2のURL]の電源ハーネスをACアダプタから外してみます。

再び電源ハーネスをACアダプタに接続してもラズパイは起動しません。
[さきほど](## 電源投入)はいきなり起動しましたが、今度はSW1を押すと起動します。

起動したら、SW1を1秒間長押ししてシャットダウンすることを確認します。

デフォルトではSW1の長押しで実行されるコマンドは"shutdown -h now" ですが、以下のファイルで変更可能です。

```bash:コンソール
pi@raspberrypi:~ $ cat /etc/sleepi2-monitor.conf
・・・
[switch]
command = "shutdown -h now"
・・・
```

sleepi2-monitor.confの詳細は以下のページで確認することができます。

[sleepi2-monitorのマニュアル][sleepi2-monitorのURL]


以上で最初の一歩は完了です。

つづいて、間欠動作(タイマー動作)を確認します。

## タイマー動作

[slee-Pi 2][sleepi2のURL]はラズパイがシャットダウンした状態(スリープ状態)でも時を刻み続け、指定の時間になるとラズパイを起動することができます。

設定はスクリプトを実行するだけです。例えば5分後に起動する設定は以下です。

```bash:コンソール
pi@raspberrypi:~ $ sudo sleepi2alarm --set "+5min"
```

SW1を長押しし、シャットダウンしてから5分ほどまつとラズパイが勝手に起動します。
設定内容は同じスクリプトで確認することができます。

```bash:コンソール
pi@raspberrypi:~ $ sudo sleepi2alarm --get
Mon 19 Mar 09:06:57 JST 2018
```

しかしこの設定は一度起動すると消えてしまいます。

```bash:コンソール
pi@raspberrypi:~ $ sudo sleepi2alarm --get
```

ここで、起動要因を確認しておきます。

```bash:コンソール
pi@raspberrypi:~ $ sudo sleepi2ctl --get wakeup-flag
alarm
```

起動要因とは、文字通り起動した要因のことです。以下の種類があります。

* poweron : 電源接続が起動要因です。
* watchdog : ウォッチドッグタイマのタイムアウトによる再起動が起動要因です。
* alarm : リアルタイムクロックのアラーム発生が起動要因です。
* switch : プッシュスイッチの押下が起動要因です。
* extin : 外部入力の検出が起動要因です。(slee-Pi 2 Plusのみ)
* ri : RI 信号の検出が起動要因です。

上記のextinについては、以下の説明を参照願います。
[slee-Pi][sleepi2のURL]

> slee-Pi 2 Plus：slee-Pi 2 Plus は slee-Pi 2 に外部入出力回路を追加したものです。設定すれば、外部入力による起動と強制電源断が可能です。
> ※使用には電子回路の知識が必要です、詳細はお問い合わせください。

extinについては次に書く記事でも説明する予定です。

間欠動作の実システムでは起動するたびにスクリプトを手で実行するわけにはいかないので、ラズパイが起動したときに自動的にアラームを設定するようにします。
まずは以下のようなスクリプトを書いて実行属性をつけておきます。このスクリプトを実行すると3分後のアラームを設定してからシャットダウンします。

```bash:~/int.sh
cat int.sh
#!/bin/sh -
set -xv
while :
do
    # TODO: ここに間欠動作したい処理を記述
    # ウェイト
    sleep 60

    # 間欠起動予約
    sudo sleepi2alarm --set "+3min"

    # シャットダウン
    sudo poweroff
    break
done
```

```bash:コンソール
pi@raspberrypi:~ $ chmod 755 int.sh
```

上記のスクリプトがラズパイ起動のたびに実行されるように設定します。
そのために以下のような、systemdの設定ファイルを作成します。

```bash:/etc/systemd/system/int.service
[Unit]
Description=slee-Pi 2 Script Service
After=multi-user.target

[Service]
Type=idle
ExecStart=/home/pi/int.sh

[Install]
WantedBy=multi-user.target
```

そして有効化します。

```bash:コンソール
$ sudo systemctl enable int
Created symlink /etc/systemd/system/multi-user.target.wants/int.service → /etc/systemd/system/int.service.
pi@raspberrypi:~ $
```

以下を実行すると、3分+アルファごとに起動～シャットダウンを繰り返します。

```bash:コンソール
pi@raspberrypi:~ $ sudo sleepi2alarm --set "+3min"
pi@raspberrypi:~ $ sudo poweroff
```

この繰り返しを強制的に止めるには、スクリプトの起動中のウェイト期間に以下を実行します。

```bash:コンソール
pi@raspberrypi:~ $ sudo systemctl disable int
Removed /etc/systemd/system/multi-user.target.wants/int.service.
pi@raspberrypi:~ $
```

スクリプト内で起動要因をチェックするなどすれば、もう少しスマートに間欠動作を停止させることも可能です。


## 死活監視(電源電圧)

[slee-Pi 2][sleepi2のURL]にはいくつもの死活監視機能がありますが、ここでは電源電圧とウォッチドッグの動作を確認します。

まず、バッテリ駆動時に必要な電源電圧監視です。
[slee-Pi 2][sleepi2のURL]に供給されている電源電圧は以下のコマンドで確認することができます。
供給電源を変化させながら実行した結果の例が以下です。

![IMG_20180317_152716 - コピー.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/376a1a08-cab5-46de-ec05-334c164c05a9.jpeg)

```bash:コンソール
pi@raspberrypi:~ $ sleepi2ctl --get voltage
7480
pi@raspberrypi:~ $ less /etc/sleepi2-monitor.conf
pi@raspberrypi:~ $ sleepi2ctl --get voltage
6096
pi@raspberrypi:~ $ sleepi2ctl --get voltage
13248
pi@raspberrypi:~ $
```

この電圧が低下したときに自動的にシャットダウンすることができます。

以下のように設定ファイルを変更します。

```bash:/etc/sleepi2-monitor.conf
pi@raspberrypi:~ $ sudo vi /etc/sleepi2-monitor.conf
・・・
[voltage]
command = "shutdown -h now"
oneshot = true
threshold = 10000
・・・
```

変更後に再起動して設定を反映してから安定化電源の供給電圧を下げると、ラズパイがシャットダウンします。

DMMで読んだCN1/CN2の端子間電圧と、"sleepi2ctl --get voltage"の出力には0.05[V]程度のズレがありましたので、シビアな場合は機差も含めて検証が必要かもしれません。

つづいて、ウォッチドッグの確認します。

## 死活監視(ウォッチドッグ)

ウォッチドッグは複数の設定が絡みますので、まとめてsleepi2ctlを実行してデフォルト設定を確認しておきます。

```bash:コンソール
pi@raspberrypi:~ $ cat getall.sh
sleepi2ctl -g extin-count
sleepi2ctl -g extin-powerdown
sleepi2ctl -g extin-trigger
sleepi2ctl -g extout
sleepi2ctl -g restart
sleepi2ctl -g ri-trigger
sleepi2ctl -g sleep
sleepi2ctl -g switch-count
sleepi2ctl -g timeout
sleepi2ctl -g voltage
sleepi2ctl -g wakeup-delay
sleepi2ctl -g wakeup-flag
sleepi2ctl -g wakeup-status
pi@raspberrypi:~ $ ./getall.sh
0
0
0
0
1
0
0
0
20
12000
40
switch
8
```

各設定の詳細は以下のページを参照願います。

[sleepi2-utilsのマニュアル][sleepi2-utilsのURL]

関係しそうな設定だけ抜き出すと以下です。

> restart
> システムが無応答と判定された場合の動作設定を表示します。
>
> 0 : 電源断を行います。
> 1 : 電源の再投入を行います。
→"1"なので無応答で再起動。

> timeout
> システムの応答を検知するタイムアウト値を表示します。
> 単位は [秒] です。 最大値は 255 です。
>
> 0 : 応答を検知しません。
> 1..255 : 設定値を超えても応答を検知できない場合は無応答と判定されます。
→"20なので無応答が20秒継続で無応答と判定。

まとめると、以下のようになるかと思います。

> 20秒間ラズパイが応答しないと再起動する

まずはカーネルパニックで試します。

```bash:コンソール
pi@raspberrypi:~ $ cat /proc/sys/kernel/panic
0
pi@raspberrypi:~ $ sudo bash
root@raspberrypi:/home/pi# cat /proc/sys/kernel/panic
0
root@raspberrypi:/home/pi# echo c > /proc/sysrq-trigger

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14435.934225] Internal error: Oops: 817 [#1] SMP ARM

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.032581] Process bash (pid: 20624, stack limit = 0xb6986210)

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.036005] Stack: (0xb6987e70 to 0xb6988000)

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.039434] 7e60:                                     b6987ea4 b6987e80 804ccd00 804cc314

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.042937] 7e80: b6986000 00000002 00000000 00000000 b9eaf880 00000002 b6987ebc b6987ea8

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.046477] 7ea0: 804cd240 804ccc54 b6987f80 00000001 b6987edc b6987ec0 802da348 804cd204

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.050052] 7ec0: b906d0c0 b6987f80 013f4a08 b6987f80 b6987f4c b6987ee0 8026ff40 802da2e8

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.053642] 7ee0: 00000000 8026f778 ffffff9c 0000000a b9705000 80290cf4 b6987f3c 00000000

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.057264] 7f00: b92e2a80 b9705000 b92e2a80 8026e378 00000000 00000000 b6987f44 80270e08

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.060923] 7f20: 8026e378 80273724 b6987f4c 00000002 b906d0c0 013f4a08 b6987f80 80108244

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.064610] 7f40: b6987f7c b6987f50 80270e48 8026ff14 8026e378 80290ff4 b906d0c0 b906d0c0

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.068339] 7f60: 00000002 013f4a08 80108244 b6986000 b6987fa4 b6987f80 80271f88 80270da0

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.072112] 7f80: 00000000 00000000 00000002 013f4a08 76ef8d50 00000004 00000000 b6987fa8

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.075908] 7fa0: 801080c0 80271f40 00000002 013f4a08 00000001 013f4a08 00000002 00000000

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.079701] 7fc0: 00000002 013f4a08 76ef8d50 00000004 00000002 00000004 00000000 000fe77c

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.083516] 7fe0: 00000000 7eaae354 76e24f4c 76e7e15c 60000010 00000001 00000000 00000000

Message from syslogd@raspberrypi at Mar 19 13:07:37 ...
 kernel:[14436.114834] Code: e3a02001 e5832000 f57ff04e e3a03000 (e5c32000)
```

カーネルパニックが発生してから約20秒後に[slee-Pi 2][sleepi2のURL]の赤LEDが点滅し、その後シャットダウン～再起動します。
上記ではパニック前に確認していますが、Linuxには、「カーネルパニックが発生したら再起動する設定」、がありますので、[slee-Pi 2][sleepi2のURL]の監視を使うか検討が必要でしょう。


次にハートビート信号で試してみます。ラズパイが起動した状態で[slee-Pi 2][sleepi2のURL]のJP3を外して20秒間待ちます。
無事に？再起動します。

この際、HDMI出力側にはちょっと違う出力が出ます。以下画像のような感じです。

![vlcsnap-2018-03-21-15h02m02s708.png](https://qiita-image-store.s3.amazonaws.com/0/48424/5d79e04a-d023-b32f-3b51-d2625996b480.png)

# まとめ

[slee-Pi 2][sleepi2のURL]の間欠動作と死活監視機能の基本動作を確認しました。
設定ファイルやスクリプトについても一通り説明できたと思います。

RTCとアラームによる起動だけでなく、ウォッチドッグによって謎のフリーズ現象発生時に"再起動してみる"という一次対応がスタンドアロンで可能となります。

次の記事では、slee-Pi 2 Plusを用いた外部入力による電源投入、切断を確認する手順をまとめる予定です。


# 落穂ひろい

## キャパシタ

[slee-Pi 2][sleepi2のURL]への電源供給がない状態でもスリープ状態を維持できるようです。
メカトラックスさんによると、保証値ではないようですが4時間程度、維持できるようです。


# リンク

[メカトラックスさんのサイト][メカトラックスさんのURL]
[slee-Pi][sleepi2のURL]
[3GPi Ver.2][3GPIのURL]
[Raspbianのダウンロード][raspbianのURL]
[Pi カメラ][Pi CameraのURL]

[slee-Piのセットアップ][slee-PiのセットアップのURL]
[sleepi2-monitorのマニュアル][sleepi2-monitorのURL]
[sleepi2-utilsのマニュアル][sleepi2-utilsのURL]
[TeraTerm][TeraTermのURL]

[メカトラックスさんのURL]:https://mechatrax.com/
[3GPIのURL]:https://mechatrax.com/products/3gpi/
[Pi CameraのURL]:https://www.rs-online.com/designspark/raspberry-pi-camera
[sleepi2のURL]:https://mechatrax.com/products/slee-pi/
[slee-PiのセットアップのURL]:https://github.com/mechatrax/slee-pi2/wiki/%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2#12-%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB
[sleepi2-monitorのURL]:https://github.com/mechatrax/sleepi2-monitor
[sleepi2-utilsのURL]:https://github.com/mechatrax/sleepi2-utils
[raspbianのURL]:https://www.raspberrypi.org/downloads/raspbian/

[TeraTermのURL]:https://ttssh2.osdn.jp/