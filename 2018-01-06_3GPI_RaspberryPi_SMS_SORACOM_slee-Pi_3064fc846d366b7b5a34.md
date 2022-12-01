<!--
title:   3GPi Ver.2とslee-Pi2を使用して、SMS着信でラズパイ遠隔起動～シャットダウン
tags:    3GPI,RaspberryPi,SMS,SORACOM,slee-Pi
id:      3064fc846d366b7b5a34
private: true
-->
3GPi Ver.2とslee-Pi2を使用して、SMS着信でラズパイ遠隔起動～シャットダウン

# はじめに

今回、[メカトラックス][メカトラックスさんのURL]さんから[slee-Pi 2][sleepi2のURL]と[3GPi Ver.2][3GPIのURL]をお借りすることが出来ましたので、ラズパイ3のSMS着信起動を試します。

着信起動デモ動画は[メカトラックス][メカトラックスさんのURL]さんがYoutubeで公開しておられます。

[![着信起動デモ動画](http://img.youtube.com/vi/8NuvaZc4vLs/0.jpg)](http://www.youtube.com/watch?v=8NuvaZc4vLs)

これを手元でやってみるというのが本記事のヘソです。

さて、[slee-Pi][sleepi2のURL]はラズパイ用の電源制御・RTCモジュールです。ラズパイの電源をタイマー制御したり、フリーズしたラズパイにリセットをかけたりすることができます。詳細はメカトラックスさんのサイトでご確認ください。

[slee-Pi 2][sleepi2のURL]はRaspberry Pi3に対応し、GPIO切替等可能な最新版です。[slee-Pi 2][sleepi2のURL]を[3GPi Ver.2][3GPIのURL]と組み合わせてSMSでラズパイを起動させてみます。

[SORACOM Air][SORACOM AirのURL]のSIMを用い、[ユーザーコンソール][ユーザーコンソールのURL](ウェブコンソール)からSMSを送信します。


## 使用機材

この記事では以下の機材を用います。

- ラズパイ3
- [3GPI Ver.2][3GPIのURL]
- [slee-Pi 2][sleepi2のURL]
- [SORACOM Air][SORACOM AirのURL] SIMカード
- 12V DCアダプター([3GPi Ver.2][3GPIのURL]付属のもの2式・または分岐ケーブル)
- 2mmピッチジャンパーピン[^1]x2個(http://akizukidenshi.com/catalog/g/gP-03902/)
- [Pi カメラ][Pi CameraのURL](なくても可)
- I2C接続温度センサ( http://akizukidenshi.com/catalog/g/gM-06675/ なくても可)


# 準備

## SIMを手配する

今回使用するSIMは[SORACOM][SORACOM AirのURL]さんの"グローバル向け Air SIM"です。

SORACOM Air SIM（黒SIM）：
![IMG_20171228_232202.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/c56deb3a-cf00-2d3d-ec41-2794276dcbfc.jpeg)



このSIMは[SORACOM][SORACOM AirのURL]さんのユーザーコンソール(Web管理画面)と認証済みのAPIからのみSMSを送ることができるそうです。

IoT用途では、SMSの送信者を限定しないと使い物にならないわけですが、その辺りは[SORACOM][SORACOM AirのURL]さんのプレスとブログに説明がありますので以下ご参照ください。

[SMS APIのプレスリリース][SORACOM SMS APIプレスのURL]

> この度ソラコムが提供開始する SMS 機能は、API コールで SMS を送ることができます。これにより、携帯電話通信網外のサーバーからも、Air SIM が挿入されている IoT 機器に対して SMS を送信したり、プログラムに組み込んで送信を自動化することが可能になります。なお、本機能は予め認証を行った「SORACOM」の API 経由で、お客様のアカウントが保有している SIM 宛しか利用できないため、外部からの SMS による不正アクセスを防ぎ、セキュアな SMS 機能利用が可能となります。

[SMS APIの解説][SORACOM SMS APIのURL]

> 1つの特徴は、デバイスにSMSを送信可能なのはそのSIMを所有するOperatorあるいはそのSAMユーザのみとしている点です。所有していないSIMを指定してAPIコールをしてもリクエストは拒否されます。また、先述のセキュリティ面の懸念を考慮し、例えデバイスの電話番号が漏れたり推測されたりとしても、外部からSMSを受信することはない設計のため、想定していないメッセージを受信して誤動作やバッテリ消耗をする心配も減らせます。


本当にIoTデバイスを遠隔管理しようとしたなら必ずぶつかるであろう壁を華麗に突破してくれるソリューションですね。例えば、"IoTデバイスのネットワーク設定をネットワークに接続せずににどうやっておこなうか、そもそもどうやって必要な時だけ電源を入れるか"のような問題です。

SMSの送信には無料枠もあり、我々のような零細試作業者は大変助かります。

[グローバル向けAir SIM(plan01s)のSMS機能がPublic Betaから正式リリースとなります][SORACOM SMS APIパブリックのURL]
> IoT デバイスへの SMS 送信：0.005USD/通
> IoT デバイスからの SMS 送信：0.4USD/通
・・・
> SMS 機能では IoT デバイスへの SMS 送信を対象に 1 アカウントにつき毎月10通分の SMS 送信を無料でご利用いただけます。


さて、グローバルSIMはAmazon.comで買うか、[SORACOM][SORACOM AirのURL]にユーザー登録済みなら、[ユーザーコンソール][ユーザーコンソールのURL]から発注することができます。
[ユーザーコンソール][ユーザーコンソールのURL]での発注時の操作方法は以下のページにヘルプがあります。非常に簡単です。

[Air SIM/その他商品の発注][Air SIM/その他商品の発注のURL]

私の場合、年末に注文したら二日ほどで届きました。SIMを受け取ったら、同じく発注画面で「受け取り」操作をしておきます。


## ジャンパ設定

SMS着信起動のためには、[slee-Pi 2][sleepi2のURL]および[3GPi Ver.2][3GPIのURL]それぞれのジャンパ設定を変更する必要があります。

- [3GPi Ver.2][3GPIのURL]のJP1をはずす(2つ共)
- [3GPi Ver.2][3GPIのURL]のJP2をつける(GPIO16を選択)
- [slee-Pi 2][sleepi2のURL]のJP1をつける(GPIO16を選択)

やや見にくいですが、JP設定と設定済みの写真を張り付けておきます。ご参考にしていただければ幸いです。

表:着信起動のための[3GPi Ver.2][3GPIのURL]ジャンパ設定

|GPIO ポート|設定|機能|備考|
|:-----|:-----|:----|:---|
|**GPIO16** / GPIO18|OUT|着信通知|JP2 で選択|

![20180103180309.png](https://qiita-image-store.s3.amazonaws.com/0/48424/c719ae6b-977f-3a61-b7ea-6aa3a3a964eb.png)


表:着信起動のための[slee-Pi 2][sleepi2のURL]ジャンパ設定

|GPIO 名|設定|機能|備考|
|:-----|:-----|:----|:---|
|**GPIO16** / GPIO18|IN|RI|JP1 で選択|

![20180103180611.png](https://qiita-image-store.s3.amazonaws.com/0/48424/b83ad6fa-6ab5-0917-35ff-8693b43693a6.png)


## 組み立て

以下のように3階建てに組み立てます。[3GPi Ver.2][3GPIのURL]のSIMソケットには組み立て後にもアクセスできます。届いた[SORACOM][SORACOM AirのURL]のSIMをセットしておきます。ちなみにですが、黒SIMはいわゆる標準サイズ(ミニSIM)ですが、切れ目が入っているので、マイクロSIM、ナノSIMとしても使えるようです。SIMカッターいらずです。[3GPi Ver.2][3GPIのURL]は標準サイズなので、そのまま使います。

|slee-Pi 2|
|:-----|
|3GPi Ver.2|
|ラズパイ3B|

![20180103182013.png](https://qiita-image-store.s3.amazonaws.com/0/48424/61d082bd-55cf-b106-7c02-4f264a3821d0.png)

電源は、[3GPi Ver.2][3GPIのURL]、[slee-Pi 2][sleepi2のURL]の両方に給電します。わたしは[3GPi Ver.2][3GPIのURL]用のACアダプタを2つ使いました。
[slee-Pi][sleepi2のURL]の内部はあまりよく分かっていませんが、先述の[3GPi Ver.2][3GPIのURL]のジャンパ設定により電源の系統はおよそ以下のようになっているはずです。

> 系統1＞slee-Pi 2＞ラズパイ3
> 系統2＞3GPi Ver.2

[slee-Pi 2][sleepi2のURL]は系統1でラズパイ3の電源を制御します。一方シャットダウン中も系統2で[3GPi Ver.2][3GPIのURL]は生きたままです。この際、[3GPi Ver.2][3GPIのURL]は低消費電力になるようです。

## raspbianの準備

[3GPi Ver.2][3GPIのURL]付属のSDカードはjessieだったので、[3GPi Ver.2のstretch][3GPIのstretchのURL]からイメージをダウンロードして使います。


SDカードをラズパイ3に刺して電源を投入します。系統2→系統1の順で投入します。

まずは[3GPi Ver.2][3GPIのURL]と[slee-Pi 2][sleepi2のURL]のパッケージを更新/インストールします。


```bash:コンソール
$ sudo apt-get update
$ sudo apt-get install 3gpi-utils 3gpi-network-manager
$ sudo apt-get install sleepi2-firmware sleepi2-utils sleepi2-monitor
```

この際、ラズパイのコンソールにアクセスするには、以下の様な選択肢があります

 - HDMI出力の画面とUSBキーボード
 - シリアル端末
 - SSH

私はSDカードにイメージを焼いてから最初の立ち上げ時はHDMI/USBキーボードを使って、あとはSSHで接続しています。

その他、[slee-Piのセットアップ][slee-PiのセットアップのURL]辺りを参照してセットアップします。上記セットアップで1点補足が必要かと思います。
「1.3 時刻の反映」にhwclock コマンドでシステム時刻をRTCに反映しますが、この時ラズパイのシステム時刻は正しいものに設定されている必要があります。有線LANやWiFi経由でインターネットに接続したらntpで勝手に時刻設定されるかもしれませんが、そうでない場合は以下のようなコマンドで設定します。

```bash:コンソール
$ date
2018年  1月  7日 日曜日 01:36:38 JST
$ sudo date -s "2018-1-7 12:07:00"
$ sudo hwclock -w
```


最後に、バージョン組み合わせの確認です。

```bash:コンソール
pi@raspberrypi:~/sandbox $ 3gpictl -v
3gpictl version 2.0
pi@raspberrypi:~/sandbox $ sleepi2ctl -v
sleepi2ctl version 1.1
pi@raspberrypi:~/sandbox $
```

## 3GPi Ver.2の動作確認

[3GPi Ver.2][3GPIのURL]の公式raspbianを用い、忘れずにUSBケーブルを接続していれば、[SORACOM][SORACOM AirのURL]のSIMなら勝手に接続していると思います。

```bash:コンソール
pi@raspberrypi:~ $ nmcli c
NAME              UUID                                  TYPE  DEVICE
gsm-3gpi-soracom  25afee3e-fb56-41be-8e34-0169f7a98456  gsm   ttyUSB3
pi@raspberrypi:~ $ ifconfig ppp0
ppp0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.136.X.XXX  netmask 255.255.255.255  destination 0.0.0.0
        ppp  txqueuelen 3  (Point-to-Point Protocol)
        RX packets 11  bytes 194 (194.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12  bytes 208 (208.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

上記の状態なら、[3GPi Ver.2][3GPIのURL]の青色LEDが高速点滅しているかと思います。
さらに、[SORACOMユーザーコンソール][ユーザーコンソールのURL]でもオンラインが確認できるかと思います。

![20180106193211.png](https://qiita-image-store.s3.amazonaws.com/0/48424/76364e03-2c2f-f0e2-a333-a48e157657db.png)


## 設定ファイルの変更

ジャンパ設定に合わせて以下の設定ファイルを変更します。

```bash:/etc/default/3gpi-utils
・・・
# Power off 3GPi at shutdown
#  AUTO_OFF=0: disable auto power off
#  AUTO_OFF=1: enable auto power off
#AUTO_OFF=1
AUTO_OFF=0
・・・
# Enable the modem to wake up on ring during standby
#  WAKE_ON_RING=0: disable Wake-on-Ring
#  WAKE_ON_RING=1: enable Wake-on-Ring (AUTO_OFF=0 is required)
#WAKE_ON_RING=0
WAKE_ON_RING=1
・・・
```

↑↑↑こちらはSMS着信起動のために必須の設定です。

メカトラックスさんからの言付けで、以下に注意が必要だそうです。
> SORACOM グローバル向け Air SIMを使用する場合はSTORE_SMS_ON_RING=0のままとすること

上記を"1"に設定すると、通信モジュール内に保存した SMS が削除できなくなってしまうそうです。
※搭載通信モジュール（SIM5320）ファームウエアの仕様だそうです

設定変更したらsudo rebootで再起動しておきます。

また、以下は必須ではありませんが、スクリプトなどでGPIOの状態を確認などするときに必要な設定です。
ジャンパ設定のメモにもなります。

```bash:/etc/default/3gpi-utils
# GPIO pins
#  RI_PIN: 16 or 18
#  POWER_PIN: 17(default) or 12
#  STATUS_PIN: 22(default) or 13
#  RESET_PIN: 27(default) or 6
#  Disabled if not set
#RI_PIN=
RI_PIN=16
・・・
```

## slee-Pi 2の挙動を確認

sudo poweroffでラズパイをシャットダウンすると、シャットダウン中に[slee-Pi 2][sleepi2のURL]の赤LEDがしばらく点滅してから、ラズパイの赤LEDが消灯します。シャットダウン状態ですね。

その状態で[slee-Pi 2][sleepi2のURL]のプッシュスイッチを押下すると、ラズパイの電源が入ります。以下のコマンドで起動要因を確認すると、switchと表示されると思います。

```bash:コンソール
pi@raspberrypi:~ $ sleepi2ctl -g wakeup-flag
switch
```

sleepi2alarm -sでタイマー起動時刻を設定して起動した場合はalaramとなり、SMS着信ではriとなります。

以上で[slee-Pi 2][sleepi2のURL]が正常に動作し、ラズパイの電源を制御できていることが確認できます。


# 本題

本記事の本題です。以下をシェルスクリプトでおこないます。

- [SORACOMユーザーコンソール][ユーザーコンソールのURL]からSMSを送信してラズパイの電源を投入
- Piカメラで撮影
- 温度センサで温度測定
- シャットダウン

Piカメラと温度センサを配線すると以下のような構成となります。

![20180107001907.png](https://qiita-image-store.s3.amazonaws.com/0/48424/d83df802-f988-0c07-8a94-4cdab4246952.png)

配線の詳細などは、[anypiの記事][anypiの記事のURL]を参照していただけると幸いです。

Piカメラを使用するためにraspiconfigで有効化します。

```bash:コンソール
pi@raspberrypi:~ $ sudo raspi-config
```

セットアップの詳細は以下がよくまとまっています。

[Piカメラのセットアップ][PiカメラのセットアップURL]


## SMS着信でラズパイ起動を確認

ラズパイをシャットダウンしておきます。

```bash:コンソール
$ sudo poweroff
```

この状態で[SORACOMユーザーコンソール][ユーザーコンソールのURL]からSMSを送信します。

対象のSIMにチェックを入れて、「操作」-「SMS送信」です。

![20180106234418.png](https://qiita-image-store.s3.amazonaws.com/0/48424/9f3066d2-4267-ee21-1e61-1c97f2239b25.png)


適当な文字列を入れて「SMSを送信」ボタンを押します。

![20180106234854.png](https://qiita-image-store.s3.amazonaws.com/0/48424/c348b95b-4f84-7fc0-41dd-33bfe88c5b15.png)


数秒で[3GPi Ver.2][3GPIのURL]の青色LEDが点滅し、すぐに[slee-Pi 2][sleepi2のURL]とラズパイの電源が入ります。

## ラズパイ起動時に実行するスクリプトを作成する

書いたスクリプトは以下のようなものです。冒頭の起動要因のガード以外はごく普通のシェルスクリプトです。

```bash:~sandbox/smspi.sh
#!/bin/sh -
set -xv
while :
do
    wakeup_flag=`sleepi2ctl -g wakeup-flag`
    if [ ! $wakeup_flag = "ri" ]; then
	# 着信起動でない場合は処理しない
	break
    fi

    sleep 5
    # 撮影
    /usr/bin/raspistill -o /home/pi/sandbox/sentinel.jpg -w 320 -h 240
    # 温度取得
    /usr/bin/python3 get_temp.py >temp.txt
    # 温度を撮影画像に書き込む
    /usr/bin/convert  -pointsize 24 -gravity southeast -annotate 0 "`cat /home/pi/sandbox/temp.txt`" -fill green sentinel.jpg sentinel-o.jpg
    # ウェイト
    sleep 30

    # TODO: ここでメールやFTPで画像を送信

    # シャットダウン
    sudo poweroff
    break
done
```

get_temp.py は、[anypiの記事][anypiの記事のURL]で使用したもので、温度センサで温度を測定します。
測定した温度はImageMagickで撮影した画像に書き込みます。

「TODO」部分には、画像のアップロード処理などをお好みで追加します。

スクリプトを実行する前に、温度センサとImageMagickを使うためのパッケージをインストールします。

```bash:コンソール
$ sudo pip3 install wiringpi
$ sudo apt-get install imagemagick
```

「sudo poweroff」をコメントアウトしておいて、スクリプト単体で動作確認をおこないます。
`bash:コンソール
pi@raspberrypi:~/sandbox $ ./smspi.sh
`

以下のような画像が保存されていればOKです。

sentinel-o.jpg：
![20180107002918.png](https://qiita-image-store.s3.amazonaws.com/0/48424/5f4f399a-fc8e-09c2-6ef8-99a78d6a8696.png)

## スクリプトの起動時実行

いろいろやり方はありますが、systemdで設定します。
上記スクリプトを呼び出すようにsystemdの設定ファイルを作成します。

```bash:/etc/systemd/system/smspi.service
[Unit]
Description=SMS Pi Script Service
After=multi-user.target

[Service]
Type=idle
ExecStart=/home/pi/sandbox/smspi.sh

[Install]
WantedBy=multi-user.target
```

そして有効化します。

```bash:コンソール
 $ sudo systemctl enable smspi
```

## 動作確認

sudo poweroffでシャットダウンした状態で、[SORACOMユーザーコンソール][ユーザーコンソールのURL]からSMSを送信すると、ラズパイが起動して撮影をおこない、自動的にシャットダウンします。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">smspi：SMS着信起動～Piカメラ撮影～シャットダウン <a href="https://t.co/yxuYE9IHGi">pic.twitter.com/yxuYE9IHGi</a></p>&mdash; syasuder (@syasuder0) <a href="https://twitter.com/syasuder0/status/948096375421407232?ref_src=twsrc%5Etfw">2018年1月2日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


# 間欠起動

[slee-Pi 2][sleepi2のURL]にはキャパシタでバックアップされたRTCが搭載されており、時刻を指定してラズパイを起動することができます。
これを応用すると、定期的にラズパイを起動させることができます。
先に作成したスクリプトのように、起動スクリプトでシャットダウンすれば、ラズパイがシャットダウンしている間の消費電力を低く抑えることが可能となります。

そこで先ほどのスクリプトを間欠起動に対応させたものが以下です。
タイマーで起動した場合のみ「sleepi2alarm」コマンドにて3分後に起動予約します。
スクリプトの最後でpoweroffします。

```bash:~sandbox/smspi.sh
#!/bin/sh -
set -xv
wakeup_flag=`sleepi2ctl -g wakeup-flag`
echo $wakeup_flag

case $wakeup_flag in
    "alarm" | "ri")
	# 間欠起動 or 着信起動
	echo "間欠起動 or 着信起動"
        sleep 1
	cd /home/pi/sandbox/
	# 撮影
	/usr/bin/raspistill -o sentinel.jpg -w 320 -h 240
	# 温度取得
	/usr/bin/python3 get_temp.py >temp.txt
	# 温度を撮影画像に書き込む
	/usr/bin/convert -pointsize 24 -gravity southeast -annotate 0 "`cat /home/pi/sandbox/temp.txt`" -fill green sentinel.jpg sentinel-o.jpg
	# ウェイト
	sleep 10

	# TODO: ここでメールやFTPで画像を送信

	if [ $wakeup_flag = "alarm" ]; then
	    # 間欠起動予約
	    sudo sleepi2alarm --set "+3min"
	fi

	# シャットダウン
	sudo poweroff
	;;
    *)
        # ほかの場合は処理しない
	echo "何もしない";;
esac
```

間欠起動の最初の１回はコマンドで実行します。

```bash:コンソール
$ sudo sleepi2alarm --set "+3min"
$ sudo poweroff
```

これで約3分+アルファごとに起動を繰り返すはずです。手持ちの環境で24時間放置しましたが、ずっと繰り返していました。


# まとめ

[slee-Pi 2][sleepi2のURL]と[3GPi Ver.2][3GPIのURL]を組み合わせることで、SMS送信によるラズパイの電源投入が簡単確実におこなえることが確認できました。

[SORACOM][SORACOM AirのURL]さんのグローバル向け Air SIMを用いることで、SMS送信者を限定することができますので、間違いSMSやスパムSMSによる誤作動を避けることができるでしょう。
さらに、[SMS APIの解説][SORACOM SMS APIのURL]で説明されているように、このSMSはAPIやデバイスから送信することもできるようです。

遠隔監視や遠隔保守システムの構築で非常に有用と思われます。

# 落穂ひろい

## slee-Piの起動要因について
[slee-Pi 2][sleepi2のURL]についてはまだいろいろ試せていませんが、起動要因について以下の点に注意が必要かもしれません。

- rebootコマンドで再起動した場合、起動要因が前回起動時のままとなる
- sleepi2alarmで予約した時刻までにシャットダウンしないと起動予約が消える

## スクリプトで起動要因をチェックする理由
上記スクリプトでは着信起動やタイマ起動を場合分けしていますので、[slee-Pi 2][sleepi2のURL]のプッシュスイッチや電源投入で起動した場合はスクリプトは途中で終了してpowroffしません。
このようにしておくとデバッグや動作確認が少し楽になります。

# リンク

[anyPi（エニーパイ）][anyPiのURL]
[メカトラックスさんのサイト][メカトラックスさんのURL]
[3GPi Ver.2][3GPIのURL]
[slee-Pi][sleepi2のURL]
[SORACOM Air][SORACOM AirのURL]
[SMS APIの解説][SORACOM SMS APIのURL]
[SMS APIのプレスリリース][SORACOM SMS APIプレスのURL]
[3GPIのstretch][3GPIのstretchのURL]
[Air SIM/その他商品の発注][Air SIM/その他商品の発注のURL]
[slee-Piのセットアップ][slee-PiのセットアップのURL]
[anypiの記事][anypiの記事のURL]
[3GPi Ver.2使用ポート][3GPi Ver.2使用ポートのURL]
[Pi Camera][Pi CameraのURL]
[SORACOMユーザーコンソール][ユーザーコンソールのURL]

[anyPiのURL]:https://mechatrax.com/products/anypi/
[メカトラックスさんのURL]:https://mechatrax.com/
[3GPIのURL]:https://mechatrax.com/products/3gpi/
[sleepi2のURL]:https://mechatrax.com/products/slee-pi/
[Pi CameraのURL]:https://www.raspberrypi.org/products/camera-module-v2/

[SORACOM AirのURL]:https://soracom.jp/services/air/
[SORACOM SMS APIのURL]:https://blog.soracom.jp/blog/2017/10/11/sms-api/
[SORACOM SMS APIプレスのURL]:https://soracom.jp/press/2017101104/
[Air SIM/その他商品の発注のURL]:https://dev.soracom.io/jp/start/console/#order
[SORACOM SMS APIパブリックのURL]:https://blog.soracom.jp/blog/2017/11/08/sms-ga/

[anypiの記事のURL]:https://qiita.com/syasuda/items/5e1da8f3315e3031684a
[3GPi Ver.2使用ポートのURL]:https://github.com/mechatrax/3gpi/wiki/%E5%9F%BA%E6%9D%BF%20Ver.2#4-%E4%BD%BF%E7%94%A8%E3%83%9D%E3%83%BC%E3%83%88

[slee-PiのセットアップのURL]:https://github.com/mechatrax/slee-pi2/wiki/%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2#12-%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB

[3GPIのstretchのURL]:https://github.com/mechatrax/3gpi/blob/master/release/3gpi2-stretch-20171102.md

[ユーザーコンソールのURL]:https://console.soracom.io/
[PiカメラのセットアップURL]:https://www.rs-online.com/designspark/raspberry-pi-camera


# 脚注

[^1]:3GPi Ver.2のJP1から取り外したジャンパピンは2.54mmピッチなので[ジャンパ設定](#ジャンパ設定)では使えないので別途用意します。