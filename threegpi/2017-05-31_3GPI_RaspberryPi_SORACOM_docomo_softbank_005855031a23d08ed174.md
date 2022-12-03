<!--
title:   デュアル3GPiで3G回線の冗長化に挑戦してみた
tags:    3GPI,RaspberryPi,SORACOM,docomo,softbank
id:      005855031a23d08ed174
private: false
-->
# はじめに

今回、[メカトラックス][メカトラックスさんのURL]さんからドコモ網向けの[3GPi Ver.2][3GPIのURL]とソフトバンク網向けの3GPi-SB Ver.2、さらにソフトバンク系のSIMをお借り出来ましたので、ラズパイ3にそれぞれの3GPiを接続して、デュアル3GPiを試してみました。

3GPi Ver.2：
![PIC_20170503_215724_.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/da31d2fb-72a0-ca3d-f5c5-e13264e01036.jpeg)

Ver.2の3GPiは使用GPIOをジャンパで切り替えられるようになっています。使用ポートの詳細は[公式Wiki][3GPi Ver.2使用ポートのURL]を参照してください。

しかも4本のGPIOのジャンパは個別に切り替え可能です。
これによって、他のボードやハードウェアと組み合わせて使うときの自由度が格段に高くなったように思います。

また、4本すべてを切り替えられるので、2枚の3GPiをジャンパ設定のみでひとつのラズパイに接続することができます。

## 使用機材

この記事では以下の機材を用います。

- ラズパイ3
- 3GPI Ver.2(ドコモ網用)
- 3GPi-SB Ver.2(ソフトバンク網用)
- PiConsole I/F
- ソフトバンク系SIMカード
- [SORACOM Air][SORACOM AirのURL] SIMカード

今回PiConsole I/Fは、PCとシリアルで接続するために用います。USBシリアル変換ケーブルなどでもOKです。


# 準備

## SIMを手配する

今どきはMVNOが当たり前になってきましたので、家電量販店にでも行って、スタータキット的なものを買ってきて、アクティベーションすれば使えるSIMがすぐに手に入ります。

今回はソフトバンク系のSIMをお借りできたこともあり、ドコモ系として[SORACOM Air][SORACOM AirのURL]のSIMを契約してみました。すでにドコモ系のMVNOのSIMがあるのに、あえて[SORACOM Air][SORACOM AirのURL]を使用したのは、ユーザーコンソールからセッションを切断できるからです。
(私は知らなかったのですが、[メカトラックスさん][メカトラックスさんのURL]に教えてもらいました。)

SORACOM Air SIM
![PIC_20170503_215906_.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/f1501f20-82fa-805d-e11a-f3bbb1dae2c9.jpeg)

## 3GPiを1枚ずつ動作確認する

今回使用する3GPiに名前を付けておきます。

- 3GPi-1：ドコモ網用：SORACOM Air：ジャンパは工場出荷のまま
- 3GPi-2：ソフトバンク網用：ソフトバンク系：ジャンパは工場出荷の反対側設定

3GPiの上にPiConsole I/Fを載せてフタをしてしまったので、アンテナにマジックで「SB」と書いておきました。

アンテナにSB：
![PIC_20170504_104415_.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/ee79bc94-1378-dcad-f8f4-8e70a5a346c7.jpeg)

ちなみに3GPiがドコモ網用なのかソフトバンク網用なのかは、外見で簡単に見分けることができます。通信モジュールの型番の末尾が異なるからです。

3GPi（ドコモ版）SIM5320J：
![PIC_20170530_212849_J.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/e578ddda-1641-9f7c-1936-351bcc8ab0da.jpeg)

3GPi-SB（SB版）SIM5320JE：
![PIC_20170530_212830_JE.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/7acc5b2d-1657-3762-966e-0560267bc1a1.jpeg)

3GPi付属のSDカードでブートすれば、3GPiをすぐに使用できる状態のRaspbianが起動します。

なのでドコモ用の3GPi-1はラズパイ3とつないで電源を入れるだけで動作確認終了です。[SORACOM Air][SORACOM AirのURL]のAPNは最初から設定されているからです。

しかしジャンパ切り替えをした3GPiではそのままというわけにはいかないようです。(2017年5月現在)

お借りしたソフトバンク網用の3GPi-SB Ver.2にはありがたいことにAPNが設定されていましたので、必要な作業はジャンパの変更のみでした。

ちなみにAPNの追加は以下のような感じでできます。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/dual $ sudo nmcli con add type gsm ifname "ttyUSB3" con-name  gsm-3gpi-softbank apn YYY.YYYY user YYY password YYY
・・・（勝手につながる）
pi@raspberrypi:~/dual $ nmcli c
NAME               UUID                                  TYPE     DEVICE
ppp0               e267166a-4f5e-4918-becf-8765ade76b62  generic  ppp0
gsm-3gpi-softbank  028bed8f-956b-496e-91ac-8a944e601433  gsm      ttyUSB3
ppp0               bbde2a85-f544-44dd-bad8-fe625b751fb0  generic  --
```

まずはジャンパ設定をデフォルトのままで上記のようにAPNに接続できることを確認しておきます。
それからpoweroffして、ジャンパを切り替えます。

ジャンパを切り替えた状態：
![PIC_20170504_104249_.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/36f803f7-f7ac-f444-8244-72f5ce4d6f7c.jpeg)

写真を見れば分かりますが、4本のGPIOのうち1本はオープンのままにしています。PiConsole I/FとGPIOがバッティングするので、念のためこうしています。

次にGPIOの設定を変更します。変更するファイルは/etc/default/3gpi-utilsです。

 POWER_PIN=17
 STATUS_PIN=22
 RESET_PIN=27
となっているところを、
 POWER_PIN=12
 STATUS_PIN=13
 RESET_PIN=6
と変更します。rebootして、ジャンパを変更する前と同じように動作したら動作確認OKです。poweroffして、次は組み立てです。

## 4階建てを組み立てる

3GPiはVer.2で電源を強化したらしいです。
3GPiの普通の使い方では

- 3GPiにACアダプタで給電して、ラズパイにはヘッダコネクタで供給する

となると思います。

しかし2枚刺しの場合は、3GPi-1と3GPi-2の両方から供給して大丈夫なんか？と疑問がわいてきます。
そういうことを気にしはじめると、仕事で何枚もの試作基板を「焼いた」経験のある私は、「グラウンドループでモクモク・・・」などの苦い経験を思い出して不安になってしまいます。

そこでまずは、3GPi Ver.2のJP1(+5V 供給選択ピン)のジャンパを外して使用することにしました。
2枚とも外しておきます。
こうすることでラズパイに別途電源を供給する必要があります。つまり電源供給は以下のような感じとなります。

- ラズパイにはUSBマイクロコネクタへ給電
- 3GPi-1/3GPi-2は付属のACアダプタで給電

ラズパイ用5V、3GPi Ver.2付属ACアダプタx2：
![PIC_20170504_125803_.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/39b6ab24-9658-d1de-87ab-908908b98561.jpeg)

その後、[メカトラックスさん][メカトラックスさんのURL]から以下のように教えていただきました。

- 「3GPiからラズパイと、もう一つの3GPiへの給電が可能」

つまり、2枚の3GPiのJP1は出荷時のショートのままで良かったわけです。
そしてACアダプタは片方だけONでOK。

ACアダプタ１つでラズパイと3GPix2に給電：
![PIC_20170529_220401_.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/fb08b82c-a52a-43d9-8c9e-05d917ac83dd.jpeg)
1つで充分ですよ、ダンナ(c)ブレードランナー

そして電源投入手順は以下の通り。

- 先に3GPi-1/3GPi-2の電源ON
- ラズパイの電源ON

切断は逆順としておきます。

それでは組み立てます。

- ラズパイ3の上に3GPi-1(ドコモ網用)を刺し
- その上に3GPi-2(ソフトバンク網用)を刺し
- その上にPi Console I/Fを刺した
状態が以下の写真です。

4階建て:
![PIC_20170525_001202_.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/991acd5d-92aa-b589-de3e-d93459cc8a30.jpeg)

3GPi-ラズパイ間のUSBケーブルも忘れずに刺します。アンテナを取り付けて火入れの前に休憩です。

# 検証

準備では2枚の3GPiを別々にラズパイに接続して動作確認を行いました。
ここでは2枚同時に接続した状態で3G通信の切り替えを検証します。

OSは3GPiの公式Raspbianを使用しています。

以下では3gpictlコマンドを使用せず直接gpioを操作します。
gpio操作の基本については [anypiの記事][anypiの記事のURL]などを参照してください。

まず起動後に必要な操作です。
3GPiのサービス(3gpi-utils.service)は起動時に必要なgpioの初期設定をおこなってくれますが、2枚目のgpioは自前でおこないます。
本格的にシステムにデュアル3GPiを組み込む場合にはサービス自体を改造するのが良いのかもしれません。（例えばexport.shを改造する、など。）

以下がコマンドです。

```shell-session:端末エミュレータ上
 $ gpio export 12 out
 $ gpio export 6 out
 $ gpio export 13 in
```

そもそもですが、3GPi-1に[SORACOM Air][SORACOM AirのURL]のSIMを刺していれば、Raspbianが起動した時点で、3GPiのステータスLED(青色LED)が高速点滅しているかもしれません。自動的に接続した状態ですね。

## 切り替えスクリプト

とりあえずnmcliを叩いて、接続先を切り替えるスクリプトを書いてみました。
gpioの番号から接続情報までハードコードでみっともないですが。

```bash:swcon.sh
#!/bin/sh
#set -xv

CON_3GPI_1=gsm-3gpi-soracom
CON_3GPI_2=gsm-3gpi-softbank

APN_3GPI_1=soracom.io
APN_3GPI_2=YYY.YYYY
USER_3GPI_1=sora
USER_3GPI_2=YYY
PASS_3GPI_1=sora
PASS_3GPI_2=YYY

disdev=`nmcli d | grep -oP 'ttyUSB\d'`
#echo $disdev
sudo nmcli d disconnect $disdev
sudo nmcli c delete id $CON_3GPI_1
sudo nmcli c delete id $CON_3GPI_2

mode=/home/pi/dual/mode
mode_value=`cat $mode`

#
# Usage: stat_gpio [GPIO_PIN]
#
stat_gpio ()
{
    case "$(gpio -g read ${1})" in
        0)
            echo "off"
            ;;
        1)
            echo "on"
            ;;
        *)
            echo "unknown"
            return 1
            ;;
    esac
}

power_on ()
{
    while :
    do
        STATUS=$(stat_gpio ${2})
        if [ $STATUS = "on" ]; then
            break
        fi
        gpio -g write ${1} 1
        sleep 3
        gpio -g write ${1} 0
        sleep 10
    done
}

power_off ()
{
    while :
    do
        STATUS=$(stat_gpio ${2})
        if [ $STATUS = "off" ]; then
            break
        fi
        gpio -g write ${1} 1
        sleep 3
        gpio -g write ${1} 0
        sleep 10
    done
}


if [ ${mode_value} -eq 1 ]; then
    echo '1->2'
    power_off "17" "22"
    power_on "12" "13"
    sleep 5
    condev=`nmcli d | grep -oP 'ttyUSB\d'`
    sudo nmcli con add type gsm ifname $condev con-name $CON_3GPI_2 apn $APN_3GPI_2 user $USER_3GPI_2 password $PASS_3GPI_2
    sleep 5
#    sudo nmcli c up id $CON_3GPI_2
    echo 2 > $mode
else
    echo '2->1'
    power_off "12" "13"
    power_on "17" "22"
    sleep 5
    condev=`nmcli d | grep -oP 'ttyUSB\d'`
    sudo nmcli con add type gsm ifname $condev con-name $CON_3GPI_1 apn $APN_3GPI_1 user $USER_3GPI_1 password $PASS_3GPI_1
    sleep 5
#    sudo nmcli c up id $CON_3GPI_1
    echo 1 > $mode
fi
```

長くなるので細かい説明は割愛しますが、以下のようなことをやっています。

- nmcli dでttyUSBxを指定して切断
- mncli c deleteで接続先情報を削除
- 3GPiの電源ピンを操作して切り替える
- nmcli c addで接続先を作成
- (自動的に接続される)

3GPi-1と3GPi-2のどちらで接続するかはファイルに書いた数字をフラグとして使っています。
これだけです。ゼロから試してみれば分かることですが、地味にノウハウが詰まっています。

power_on ()、power_off ()は3gpictlの処理を真似て書いています。
異なるのは、「電源の状態が変化するまで待つ、リトライする」点です。
3gpictlではタイムアウトでエラーとして処理してしまいますが、このスクリプトでは失敗すると困るので、リトライし続けます。

上記のスクリプトを10回くらい繰り返し実行して正常に切り替わることを確認しておきます。

## 接続が切れたら自動的に切り替える

ここまでで、好きなタイミングで接続先を切り替えることができるようになりました。
次に回線冗長化のために必要な「切断されたら切り替える」動作を試します。

3GPiはNetworkManager(NM)で接続を管理していますので、NMの仕組みを利用します。
[ラズパイと3GPIで猫Piカメラ][ラズパイと3GPIで猫PiカメラのURL]では接続をトリガーにしましたが、今回は切断をトリガーにしてスクリプトを実行します。

```bash:/etc/network/if-post-down.d/333gpi
#!/bin/sh
set -e

if [ "$IFACE" != ppp0 ]; then
        exit 0
        fi

if [ "$DEVICE_IFACE" != ppp0 ]; then
        exit 0
        fi

#if [ "$MODE" != start ]; then
#        exit 0
#       fi

if [ "$ADDRFAM" != inet ] && [ "$ADDRFAM" != inet6 ]; then
        exit 0
        fi

echo "`date`: ppp0 if-post-down" >> /home/pi/3gpi.log
# 以降に必要な処理を書く
#echo "`set`" >> /home/pi/3gpi.log
echo "`echo $CONNECTION_UUID`" >> /home/pi/3gpi.log

/home/pi/dual/swcon.sh
```

if-downではなくif-post-downを使うところががミソといえばミソです。

接続状態で、sudo nmcli d disconnect ttyUSB3 などを実行すれば、先ほどのスクリプトがキックされて回線が切り替わることが確認できます。

しかしそれは「明示的な切断」の動作ですね。必要なのは「急に接続が切れた時に切り替わる」という動作です。

3G回線シミュレータが使える幸せな環境ではありませんので、ベタな方法が必要です。
少し無線の分かる方なら小さいファラデーケージ（例：スチールデスクの引き出しの中）の利用を考えるかもしれませんが、それだと2枚の3GPiが両方とも通信できない状況になったり、そもそもいまどきの3Gはスチールデスクくらいでは圏外状態にならなくて困るだけかもしれません。

幸い[SORACOM Air][SORACOM AirのURL]には「セッション切断」機能があるので、これを利用します。

セッション切断：
![20170524235611.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/9f1cee11-bb61-90cf-1920-bda051a7e8cb.jpeg)
MVNOオペレータの気分：
![20170524235708.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/9a614252-a21f-0760-8c9b-3ea69e58da77.jpeg)

swcon.shスクリプトを手でたたいて、[SORACOM Air][SORACOM AirのURL]の接続状態にしておき、「ユーザーコンソール」で「セッション切断」するとソフトバンクに切り替わることが確認できます。

今回の記事は簡単に切り替えを行うための実験ですので上記で済ませておりますが、主回線・副回線の構成で冗長化する場合には
スクリプトにもうひと工夫が必要になると思います。
普段は安い主回線を使い、障害発生時のみ信頼性の高い副回線を利用することで、トータルの通信費を抑えつつ信頼性の高いシステムを構築することも可能になるというわけです。


# 実用っぽいテスト

## リモート温度計

[anyPiの記事][anypiの記事のURL]で使用したI2C接続の温度センサモジュールで測定した温度をサーバーにアップロードし続けるだけのスクリプトを書きました。

```py3:b2.py
#!/usr/bin/python
# -*- coding: utf-8 -*-

import wiringpi as wp
import time
import adt7410
import requests

I2C_ADDR_THERMO = 0x48

base_path = 'http://exampple.com/dual_3gpi/'


def register(**params):
    try:
        res = requests.get('{0}'.format(base_path), params=params)
#        jres =res.json()
#        ret = jres['result']
#        pprint(ret)
        return 0
    except Exception:
        print ("request failed")
        return 1  # 登録失敗とする


if __name__ == '__main__':
    thermo = adt7410.ADT7410(I2C_ADDR_THERMO)
    while True:
        temp = thermo.read_temp()
        print("Temperature:%6.2f" % (temp / 128.0))
        register(temp='{0:6.2f}'.format(temp/128.0))
        time.sleep(30)
```

約30秒ごとにGETメソッドでリクエストを投げ続けます。

サーバー側のスクリプトは省略しました。GETなので404のログを見れば温度が分かるからです（笑）

以下のように、回線が切り替わったこともクライアントのIPアドレスの変化で分かります(苦笑)

```text:サーバーログ
54.250.XXX.XX - - [21/May/2017:22:14:22 +0900] "GET /dual_3gpi/?temp=27.9296875 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
54.250.XXX.XX - - [21/May/2017:22:14:53 +0900] "GET /dual_3gpi/?temp=29.0 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
54.250.XXX.XX - - [21/May/2017:22:18:02 +0900] "GET /dual_3gpi/?temp=+28.04 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
54.250.XXX.XX - - [21/May/2017:22:18:42 +0900] "GET /dual_3gpi/?temp=+28.88 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
54.250.XXX.XX - - [21/May/2017:22:19:13 +0900] "GET /dual_3gpi/?temp=+28.40 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
126.237.YYY.YY - - [21/May/2017:22:20:45 +0900] "GET /dual_3gpi/?temp=+28.12 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
126.237.YYY.YY - - [21/May/2017:22:21:18 +0900] "GET /dual_3gpi/?temp=+28.09 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
126.237.YYY.YY - - [21/May/2017:22:21:51 +0900] "GET /dual_3gpi/?temp=+28.09 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
126.237.YYY.YY - - [21/May/2017:22:22:23 +0900] "GET /dual_3gpi/?temp=+28.06 HTTP/1.1" 404 250 "-" "python-requests/2.4
```

## 一晩放置

以下のようにスクリプトを起動して屋内で一晩放置してみました。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/dual $ python3 up_temp.py  &
[1] 1066
pi@raspberrypi:~/dual $ Temperature: 26.72
・・・
```

結果は・・・1回しか切り替えは発生しませんでした（笑）

```text:サーバーログ
＜放置開始＞
54.250.xxx.98 - - [24/May/2017:23:00:41 +0900] "GET /dual_3gpi/?temp=+26.90 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
54.250.xxx.97 - - [24/May/2017:23:05:28 +0900] "GET /dual_3gpi/?temp=+26.90 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
＜切り替わり発生＞
126.200.yyy.16 - - [24/May/2017:23:07:00 +0900] "GET /dual_3gpi/?temp=+26.91 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
126.200.yyy.16 - - [24/May/2017:23:07:33 +0900] "GET /dual_3gpi/?temp=+26.93 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
・・・
126.200.yyy.16 - - [25/May/2017:07:42:12 +0900] "GET /dual_3gpi/?temp=+26.12 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
126.200.yyy.16 - - [25/May/2017:07:42:44 +0900] "GET /dual_3gpi/?temp=+26.06 HTTP/1.1" 404 250 "-" "python-requests/2.4.3 CPython/3.4.2 Linux/4.4.48-v7+"
＜放置終了＞
```

## 無通信放置

メカトラックスさんに教えていただいたのですが、SORACOMは無通信が連続すると切断されるようです。

[無通信の状態が続くと通信が切れるのはなぜですか？][無通信切断のURL]

> ソラコムでは無通信の状態が1時間ほど続くと
> セッションを切断する仕様があります
> （こちらの仕様は今後変更となる可能性もございます。
> あらかじめご了承ください）。

試しにSORACOM Airで接続した状態で放置したところ、以下のように1時間2分弱で切断されました。

![20170530210422.png](https://qiita-image-store.s3.amazonaws.com/0/48424/715d5aae-292d-b82e-5d9d-2f4b3bd8d37d.png)

放置テストでは、サーバーへの温度アップロードをしていましたが、それがkeep aliveの代わりになってしまっていたようです。（汗

また、ソフトバンクSIMでもおよそ1時間で同様に自動切断されました。NMのスクリプトからメモ代わりに吐いていた

```text:切断ログ
Tue May 30 20:15:39 JST 2017: ppp0 if-post-down
74cdddf2-2d75-44dd-b4fa-2a441d50071d
Tue May 30 21:16:54 JST 2017: ppp0 if-post-down
7b6bd560-2e72-4317-94d7-63ced2796e35
```

上2行が先ほど同じSORACOMの切断、下がソフトバンクの切断です。


# 最後に

3GPi Ver.2を2枚使えば、非常に簡単に3G回線の冗長化を実現できることがお示しできたと思います。スクリプトを書いただけですから。

また、今回のようにソフトバンク網用とドコモ網用を混ぜて使えば、自然災害対応冗長化を謳うことすら可能かもしれません。

[メカトラックスさん][メカトラックスさんのURL]では入手困難なソフトバンク網用のSIMと3GPiとのセット販売もしておられます。

[3GPiとソフトバンク網SIMのセット販売][3GPiセット販売のURL]
![20170531225125.png](https://qiita-image-store.s3.amazonaws.com/0/48424/cb29d74e-15b7-636c-f496-c9724f4ffdae.png)


せっかくMVNOが当たり前になったこのご時世です。この記事が、回線冗長化を気軽に試す一助となれば幸いです。

# リンク

[anyPi（エニーパイ）][anyPiのURL]
[メカトラックスさんのサイト][メカトラックスさんのURL]
[3GPI][3GPIのURL]
[SORACOM Air][SORACOM AirのURL]

[ラズパイと3GPIで猫Piカメラ][ラズパイと3GPIで猫PiカメラのURL]
[anypiの記事][anypiの記事のURL]
[3GPi Ver.2使用ポート][3GPi Ver.2使用ポートのURL]
[3GPiセット販売][3GPiセット販売のURL]

[anyPiのURL]:http://www.mechatrax.com/products/anypi
[メカトラックスさんのURL]:http://www.mechatrax.com/
[3GPIのURL]:http://www.mechatrax.com/products/3gpi
[SORACOM AirのURL]:https://soracom.jp/services/air/
[ラズパイと3GPIで猫PiカメラのURL]:http://qiita.com/syasuda/items/5def33db425aaf05bf6d
[anypiの記事のURL]:http://qiita.com/syasuda/items/5e1da8f3315e3031684a
[3GPi Ver.2使用ポートのURL]:https://github.com/mechatrax/3gpi/wiki/%E5%9F%BA%E6%9D%BF%20Ver.2#4-%E4%BD%BF%E7%94%A8%E3%83%9D%E3%83%BC%E3%83%88

[無通信切断のURL]:https://soracom.zendesk.com/hc/ja/articles/235781348
[3GPiセット販売のURL]:http://mtx.theshop.jp/items/3648734