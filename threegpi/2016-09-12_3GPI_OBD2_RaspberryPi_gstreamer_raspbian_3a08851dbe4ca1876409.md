<!--
title:   3GPIとOBDIIでPimetryシステム 
tags:    3GPI,OBD2,RaspberryPi,gstreamer,raspbian
id:      3a08851dbe4ca1876409
private: false
-->
# はじめに

[猫Piカメラ][ラズパイと3GPIで猫PiカメラのURL]、[遠隔操作OAタップ][遠隔操作OAタップURL]につづき、[メカトラックスさん][メカトラックスさんのURL]にお借りしっぱなしの3GPIを使って簡単なテレメトリシステムを作ってみます。

乗用車のOBDIIポートから車両情報を取得し、リモートPCのダッシュボードアプリに転送して、タコメータや速度を表示します。

・ダッシュボードアプリ(Qtアプリ)
![20160902195114.png](https://qiita-image-store.s3.amazonaws.com/0/48424/f243070f-6833-4b30-2f19-50dfd296cf29.png)

## Pimetryシステムとは

2000年ごろから、乗用車にはOBDIIという共通ポートが搭載されるようになったそうです。
最近の車両にCANなどが搭載されていることは知っていましたし、そういう車載装置の仕事も複数やったことがあったのですが、わが愛車にもOBDIIのコネクタがあることを最近知りました。
カバーもなくむき出しで運転席の下方に見つけました。いままで気付かなかったのが不思議なくらいです。

・愛車のOBDポート
![PIC_20160723_171000.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/f3bf2a38-965a-c067-14fe-1592dfc443dd.jpeg)

このポートを使って車両の各種診断情報を吸い上げることができます。取得できる情報は車種によって異なるようです。

まずは愛車で取得可能な情報を確認し、ラズパイから取得し、それをリモートマシンに転送して画面に表示するアプリを製作します。

というわけで、ラズパイ＋テレメトリ=Pimetryです。

## 大事な注意

この記事ではOBDIIポートにやや怪しげなデバイスを接続しますが、車両側の回路などにダメージを与える可能性が少なからずあります。

この記事はあくまでも実験結果をまとめたものですので、実車にてお試しされる場合は自己責任でお願い致します。

# 準備

以下の段取りで進めます。

  - 既存スマホアプリによるOBDデータの確認(取れるデータを調べる)
  - ノートPCによる実車からのOBDデータの採取
  - ラズパイからのNAT越えでデータ転送
  - Windows用GUIアプリの実装

## 既存スマホアプリでトライ

まずはこのBluetooth接続のELM327アダプタをスマホアプリで試してみました。

・使用したBT接続のEML327アダプタ
![PIC_20160830_203926.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/e68ac8c7-bb13-8a92-d594-3e03608e094f.jpeg)

つまり以下のような構成です。
Androidスマホ＞(BT)＞ELM327アダプタ＞車両

使用したアプリは[OBD Info-san!][Google Play OBD Info-san!のURL]です。

・接続前のスクショ
![Screenshot_2016-08-30-20-03-24.png](https://qiita-image-store.s3.amazonaws.com/0/48424/cbf3b989-a388-3353-9ac3-ed1fc06cdbd4.png)

・接続後のスクショ
![Screenshot_2016-08-30-20-03-13.png](https://qiita-image-store.s3.amazonaws.com/0/48424/8a19f231-65dc-e7cc-a994-d56c996a9b5b.png)

バッテリ電圧が取れているので、とりあえずアダプタは動作しているようです。
しかし残念ながら愛車ではOBDII由来のデータは取得できませんでした。

Torqueというアプリも試したのですが、同様でした。

Google先生にお伺いを立てたところ、愛車は以下の魚拓で紹介されているように初期化してやらないと、OBDIIからデータが取れない車種に該当するようでした。

[Toruqueをアルテッツァで使う記事][魚拓のURL]

ただ残念ながらインストールしたTorqueでは上記記事の設定項目が見当たらず、どうにもならない感じでした。

一連の確認で分かったことは、

- わが愛車は特殊な初期化をしないとOBDIIの情報が取れなさそう

ということです。

## ノートPCによる実車からのOBDデータの採取にトライ

やや意気消沈しつつ、ノートPCによるトライをしました。

Bluetoothアダプタだと単体で電源が入らなくて不便なので、USBタイプのものを使用しました。

・使用したUSBタイプのELM327アダプタ
![PIC_20160830_203955.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/eaf19b20-d2bd-eb1c-e485-dd96718b3719.jpeg)

まず、さくっとPCにつないでみたのですが、COMポートとして認識されませんでした。

Google先生曰く、この手のアダプタの内部構造はいかのようになっているらしいです。

USB＞PL2303＞(UART)＞ELM327

ハズレなしでPL2303ぽいので、以下からPL2303のドライバをダウンロードしてインストールするとあっさりCOMポートとして認識されました。

[PL2303ドライバダウンロード][PL2303ドライバのURL]

ELM327デバイス(クローン・互換品含む)は、いわゆるATコマンドで制御します。メーカーサイトにATコマンドやデータシートが置いてあります。

[ELMデバイスのデータシート][ELMデバイスのデータシートのURL]

データシートはOBD自体についての解説も含んでいるので、みっちり読めばELM327デバイスを骨までしゃぶれそうです。ここではATコマンドで表層だけ撫でる程度を考えていますので、ATコマンド一覧だけで充分です。

アナログモデムでパソ通やってた世代なら、ATコマンドに抵抗ないかもしれません。まずはATZしておけば間違いなしというわけです。

COMポートの設定は38400bps、ノンパリ、ストップビット1です。

```shell-session:(ELM327)TeraTerm上
>atz

ELM327 v1.5
```

デバイスメーカーはELM327のV1.5をリリースしていないようで、これはいわゆる[クローン品とみなされている][V1.5はクローンだという情報のURL]ようです。一時そういう注意書きがデバイスメーカのサイトにあったらしいのですが、今は見当たりません。

もしかしてそれってOEMなんじゃないの？という気もしますし、チップのバージョンにV1.5がなくても製品としてのアダプタのバージョンに何を付けようが勝手な気もします・・・

いずれにしても、「ELM327 V1.5」というのは各種アプリの「動作確認デバイス」に挙げられるくらい広く使われている/いたこと、かつ最新のV2.1デバイスは古い車種では使い物にならないという情報もあるようなので、あまり深く考えずに進めることにしました。

気を取り直して、先ほど見つけておいた初期化コードをTeraTermから叩いてみました。

[Toruqueをアルテッツァで使う記事][魚拓のURL]

```shell-session:(ELM327)TeraTerm上
>atib 96
OK

>atiia 13
OK

>atsh8213f0
OK

>atsp4
OK

>0100
BUS INIT: ...OK
41 00 BE 1F B0 10

>atrv
13.5V
```

車両のIGN ON状態で確認しましたので、BUS INITの結果がOKとなりました。0100にて(mode1の00)車両がサポートしているPID(OBD情報のID)のビットマップも取得できています。上記ではBE1FB010がビットマップです。
以下のページにある表で各ビットの意味が分かります。

[OBDサポートビット一覧][OBDサポートビット一覧のURL]

この表から、車速やRPMなど最低でも使いたいと考えていた値が取れそうなのでやや安心しました。

ちなみにですが、車両がサポートしないPIDを指定すると、ちゃんとわかるようになっています。

```shell-session:(ELM327)TeraTerm上
>0120
NO DATA
```

典型的なスクリプト泣かせの「human readable」なインタフェースですね。

さらに使えそうなPIDを指定して実際の値を眺めてみました。「#」に続くコメントは後から書き足したものです。

```shell-session:(ELM327)TeraTerm上
>0104  # Calculated engine load
41 04 2E  # 0x2e = 46[%]

>0105  # Engine coolant temperature
41 05 7E  # 0x7e = 126, 126-40=86[℃]

>010c  # Engine RPM
41 0C 0D BE  # 0x0dbe = 3518, 3518/4= 879[rpm]

>010d  # Vehicle speed
41 0D 00  # 00 [km/h]
```

単位変換なども以下のページを見れば書いてあります。複数バイトのものは基本的にビッグエンディアンのようです。

[Mode01の一覧][Mode01の一覧のURL]

## ラズパイからのVPN接続にトライ

当初3G通信はUDPホールパンチでNAT越えを計画していたのですが、やや安定しなかったのと、送信側のスクリプトが複雑になるので、あきらめてVPNを使用することとしました。

VPNサーバはテスト用にいつでも使える状態のものがあるので、ラズパイをVPNクライアントに仕立てるだけで済ませました。いったん接続すれば宛先アドレスなどそのままでローカル環境と同じスクリプトが使えていい感じです。

いざ調べてみると、ラズパイをVPNサーバに仕立てる情報はたくさんありますが、クライアントしかも自動接続となるとあまりありませんでした。以下のUbuntuの記事を参考にしつつ設定ファイルを書きました。

[コマンドを使ってVPNクライアント接続する方法][コマンドを使ってVPNクライアント接続する方法のURL]

使用するVPNサーバは、Windows2012ServerのPPTPなので、以下の様な設定となりました。

```shell-session:/etc/NetworkManager/system-connections/VPN
pi@raspberrypi:/etc/NetworkManager/system-connections $ sudo cat VPN
[connection]
id=VPN
uuid=7747fb06-42aa-4062-a938-xxxxxxxxxxxxxxxxx
type=vpn
autoconnect=false

[vpn]
service-type=org.freedesktop.NetworkManager.pptp
gateway=example.com
user=XXXXXX
#password-flags=1
password-flags=0
require-mppe=yes
refuse-chap=yes
refuse-eap=yes
refuse-pap=yes

[ipv4]
method=auto

[vpn-secrets]
password=xxxxxxxxxxxxx
```

これで、nmcliからVPN接続/切断ができるようになりました。

```shell-session:ラズパイ上
# 接続
$ sudo nmcli con up id VPN
# 切断
$ sudo nmcli con down id VPN
```

以下のスクリプトを設置すれば自動接続もできました。これは[ラズパイと3GPIで猫Piカメラ][ラズパイと3GPIで猫PiカメラのURL]のときのものの応用です。もっとスマートなやり方がありそうな気もします。

```shell-session:/etc/network/if-up.d/998pimetry
#!/bin/sh
set -e

if [ "$IFACE" != ppp0 ]; then
	exit 0
fi

if [ "$DEVICE_IFACE" != ppp0 ]; then
	exit 0
fi

if [ "$MODE" != start ]; then
	exit 0
fi

if [ "$ADDRFAM" != inet ] && [ "$ADDRFAM" != inet6 ]; then
	exit 0
fi

nmcli con up id VPN

# エラーを起こさないように
```

# システム構築

## OBD情報取得＆転送スクリプト

pythonには[Python-OBD][Python-OBDのURL]という素敵なものがあります。
数行のテストスクリプトを書いて試してみましたが、やはり愛車ではBUS INITに失敗してしまいました。

```shell-session:ノートPC上
[obd.obd] ======================= python-OBD (v0.6.1) =======================
[obd.obd] Explicit port defined
[obd.elm327] Initializing ELM327: PORT=\.\COM7 BAUD=38400 PROTOCOL=4
[obd.elm327] write: b'ATZ\r\n'
[obd.elm327] wait: 1 seconds
[obd.elm327] read: b'ATZ\r\r\rELM327 v1.5\r\r>'
[obd.elm327] write: b'ATE0\r\n'
[obd.elm327] read: b'ATE0\rOK\r\r>'
[obd.elm327] write: b'ATH1\r\n'
[obd.elm327] read: b'OK\r\r>'
[obd.elm327] write: b'ATL0\r\n'
[obd.elm327] read: b'OK\r\r>'
[obd.elm327] write: b'ATTP4\r\n'
[obd.elm327] read: b'OK\r\r>'
[obd.elm327] write: b'0100\r\n'
[obd.elm327] read: b'BUS INIT: ...ERROR\r\r>'
[obd.elm327] Connected Successfully: PORT=\.\COM7 BAUD=38400 PROTOCOL=4
[obd.obd] querying for supported commands
[obd.obd] Sending command: b'0100': Supported PIDs [01-20]
[obd.elm327] write: b'01000\r\n'
[obd.elm327] read: b'BUS INIT: ...ERROR\r\r>'
[obd.OBDCommand] b'0100': Supported PIDs [01-20] did not recieve any acceptable
messages
[obd.obd] No valid data for PID listing command: b'0100': Supported PIDs [01-20]

[obd.obd] finished querying with 7 commands supported
[obd.obd] ===================================================================
```

このことから、巷で広く使われている「自動プロトコル認識手順」から外れた車種とあらためてあきらめが付きました。

もし最近の車種で試すなら、このPython-OBDがお勧めです。これを使えばOBD情報を直感的に取得できるスクリプトが簡単に書けそうだからです。厄介なhuman readableプロトコルもある程度最適化して高速に情報が取れるようです。(高速かどうかはインタフェースのbpsによるところが大きそうですが。)

最新のPython-OBDはPySerialのバージョン3以降が必要です。以下の様なwarningで知らせてくれますが、ちょっと分かりにくかったです。結局Python3が必要ということになるようです。

```py3:Python-OBDのテスト
>>> c = obd.OBD(p[0])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/python2.7/dist-packages/obd/obd.py", line 58, in __init__
    self.__connect(portstr, baudrate, protocol) # initialize by connecting and loading sensors
  File "/usr/local/lib/python2.7/dist-packages/obd/obd.py", line 85, in __connect
    self.interface = ELM327(portstr, baudrate, protocol)
  File "/usr/local/lib/python2.7/dist-packages/obd/elm327.py", line 143, in __init__
    self.__send(b"ATZ", delay=1) # wait 1 second for ELM to initialize
  File "/usr/local/lib/python2.7/dist-packages/obd/elm327.py", line 408, in __send
    return self.__read()
  File "/usr/local/lib/python2.7/dist-packages/obd/elm327.py", line 441, in __read
    data = self.__port.read(self.__port.in_waiting or 1)
AttributeError: 'Serial' object has no attribute 'in_waiting'

inWaiting()
	Deprecated since version 3.0: see in_waiting
```

ということで、Python-OBDが使えないのでPySerialでベタに進めるしかありません。
とりあえず以下の様なスクリプトを書きました。

```py3:test.py
#! /usr/bin/python
# -*- coding: utf-8  -*-
import serial
import socket
import json
import sys


# counte rpart
CP = {
    "address":"192.168.22.5",
    "port":5001,
    }

PIDS = {
    "SUPPORTED":b"00",
    "THROTTLE_POS":b"11",
    "RPM":b"0C",
    "SPEED":b"0D",
    "COOLANT_TEMP":b"05",
    "INTAKE_TEMP":b"0f",
    "ENGINE_LOAD":b"04",
    }

def mode1(pid, ser):
    res = cmd(b"01"+PIDS[pid], ser).split(b"\r")
    data = res[1].split()
#    print(data)
    if data[0] == b"41" and data[1] == PIDS[pid]:
        # 期待した応答
        vs = b"".join(data[2:]).decode('ascii')
        value = int(vs, 16)
#        print(pid, value)
        return value
    else:
        # エラー
#        print(pid, 0)
        return 0

# http://stackoverflow.com/questions/13017840/using-pyserial-is-it-possble-to-wait-for-data
def cmd(cmd, ser):
    out=b'';prev=b'101001011'
    ser.flushInput();ser.flushOutput()
    ser.write(cmd+b'\r');
    while True:
        out+= ser.read(1)
        if prev == out: return out
        prev=out
    return out


def init(port, ser):
    # 車種ごとに必要な初期化
    ser.timeout=0.3
    print( cmd(b"atz", ser) )
    print( cmd(b"atsh8213f0", ser) )
    print( cmd(b"atib 96", ser) )
    print( cmd(b"atiia 13", ser) )
    print( cmd(b"atsh8213f0", ser) )
    print( cmd(b"atsp4", ser) )
    print( cmd(b"atdp", ser) )

    ser.timeout=1.0 # ここは反応遅い
    # BUS初期化
    print( cmd(b"0100", ser) )
    ser.timeout=0.3

if __name__ == '__main__':
    port = sys.argv[1]
    print(port)
    ser = serial.Serial(port, 38400, timeout=1.0)
    init(port, ser)

    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    ser.timeout=0.3
    supported = mode1("SUPPORTED", ser)
    print( "PIDS supported: %s " % supported)
    while 1:
        rpm = mode1("RPM", ser) / 4
        print( "Engine RPM: %s [rpm]" % rpm)
        pos = mode1("THROTTLE_POS", ser) *100/255
        print( "Throttle position: %s [%%]" % pos)
        speed = mode1("SPEED", ser)
        print( "Vehicle speed: %s [km/h]" % speed)
# テストカーではほとんど変化しないので取らない
#        ctemp = mode1("COOLANT_TEMP", ser) -40
#        print( "Engine coolant temperature: %s [C]" % ctemp)
# テストカーでは取れない
#        itemp = mode1("INTAKE_TEMP", ser) -40
#        print( "Intake air temperature: %s [C]" % itemp)
        eload = mode1("ENGINE_LOAD", ser) *100/255
        print( "Engine load: %s [%%]" % eload)

        payload = {
            "rpm": rpm,
            "pos": pos,
            "speed": speed,
#            "ctemp": ctemp,
#            "itemp": itemp,
            "eload": eload,
            }
        msg = json.dumps(payload)
        s.sendto(b"\xe7" + msg.encode('ascii') + b"\xd7", (CP["address"], CP["port"]))

    ser.close()
```

車速やRPMなど必要最低限の情報を取得し、JSON文字列に固めて5001番ポートにUDPで投げるだけのスクリプトです。
それをひたすら繰り返します。
トライアンドエラーのやっつけ作業かつpython3環境不慣れなので、やや冗長な部分もあるかと思います。わが愛車と同様にプロトコル自動認識に失敗して困る皆様にてご参考にしていただければ幸いです。

ELM327の応答を解析する部分はタイムアウト頼みなのでのっそり動作しますが、1秒ごとに1メッセージをUDPで投げるくらいはできましたので、これで良しとしました。

最後にラズパイ上で確認です。ラズパイではPL2303は自動的に認識されました。

> [    5.667420] usbcore: registered new interface driver pl2303
> [    5.667480] usbserial: USB Serial support registered for pl2303
> [    5.667541] pl2303 1-1.4:1.0: pl2303 converter detected
> [    5.672396] usb 1-1.4: pl2303 converter now attached to ttyUSB0

ありがたいことに刺しっぱなしで電源投入すれば/dev/ttyUSB0で確定します。ということは以下で起動できるということです。

```shell-session:ラズパイ上
$ python3 test.py /dev/ttyUSB0
```

## Windows用GUIアプリの実装

ここまでで以下の構成ができました。

車両＞(OBD)＞ELM327アダプタ＞(USB)＞ラズパイ＞(UDP)＞(3G)＞(VPN)＞VPNサーバー

VPNサーバー配下のLAN上にあるWindowsマシンで動作させるアプリをさくっと書きました。たまたまQt4の環境があったので、Qtアプリにしておきました。

・（画面は開発中のものです）的なスクリーンショット
![20160902002938.png](https://qiita-image-store.s3.amazonaws.com/0/48424/0552d000-eca2-981c-810a-6d92b666e90a.png)
いまいち見えづらいので透明ウィンドウはお蔵入りになったのですが、Qtだと透明ウィンドウを作るのが楽ちんでした。

このアプリは主に以下の２つの要素からなります。

- UDP受信サーバ
- ダッシュボード描画

勢いに任せてPythonスクリプトではjson風文字列で情報を投げるようにしてしまったので、このアプリはjson文字列を受信して、データを取り出さないといけません。残念なことにQt4はjsonにネイティブ対応していませんのでQJsonを使いました。

[QJson][QJsonのURL]

QJsonのビルドにはCMakeが必要です。わたしのWindowsのQt環境はmingwベースのものなのですが、Qt CreatorからCMakeをうまく認識してくれませんでした。仕方がないのでQJsonのビルドはコマンドラインでおこないました。

3Gでは通信が断続的になると思われたので、オフライン状態が分かるようにしました。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">pimetry2：オフライン状態の表示 <a href="https://t.co/dJYAJMruxw">pic.twitter.com/dJYAJMruxw</a></p>&mdash; syasuder (@syasuder0) <a href="https://twitter.com/syasuder0/status/772326374396657664?ref_src=twsrc%5Etfw">2016年9月4日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

ソースは以下に置いてます。

https://github.com/syasuder/pimetry2

## ラズパイカメラ＋gstremaerでストリーミング

ダッシュボードだけだと寂しいので、ラズパイカメラの画像をストリーミングすることにしました。
gstreamerならラズパイ上で、ワンライナーを叩くだけで動画配信できました。

```shell-session:ラズパイ上
$ raspivid -n -w 640 -h 360 -b 250000 -fps 30 -t 0 -o - | \
gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=10 pt=96 \
! udpsink host=192.168.22.5 port=5000
```

これだけで192.168.22.5のマシンの5000番ポートにストリーミングできます。
MVNOの契約を考慮して、ビットレートを低く抑えました(苦笑)。

対向はWindowsマシンで、以下のコマンドを実行するだけでストリーミング受信、かつウィンドウが表示されます。

```shell-session:Windowsマシン上
C:\gstreamer\1.0\x86_64\bin> gst-launch-1.0.exe -v udpsrc port=5000 \
caps="application/x-rtp, media=(string)video, clock-rate=(int)90000,  \
encoding-name=(string)H264" ! rtph264depay ! avdec_h264 ! videoconvert ! \
autovideosink sync=false
```

ローカル環境で試した結果が以下です。

・デスクトップ171号線上の反対車線のリラクマ号
![20160903180512.png](https://qiita-image-store.s3.amazonaws.com/0/48424/adf5f488-da9d-013f-169b-1ad5a2359a2d.png)

実はgstreamer SDKでMFCアプリを書き換えてスケルトンは完成したのですが、パイプラインをC++で書くのが不慣れで時間切れとなってしまったのでお試しは上記ワンライナーで済ませました。

このワンライナーの良いところを一つ挙げておきます。受信側のgstreamerは送信側が停止しても終了することなく待ち受け状態となります。送信が再開されると、受信側はそのまま表示を再開してくれます。この挙動は実験ではありがたいです。走行中に3G通信が切断した時などに、Windowsマシンにリモートログインしてアプリを再度立ち上げたりするのは面倒だからです。

逆に、自前のアプリでそこまで面倒見るのは少々大変そうです。アプリを書く前からハードルがあがるgstremer、恐るべしです。

# 走行試験

例によって適当な箱に収めます。基板むき出しですと、いろいろアレですので。

・セリアのケーブルケースに収めた感じ
![PIC_20160903_210721.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/9eb3a674-9182-79e9-ef6c-ab7058a9cb36.jpeg)

・尻尾は電源とOBDアダプタの2本
![PIC_20160903_210819.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/1a3a7fad-5846-5f55-1414-7735341fd877.jpeg)


車両上の様子は以下のような感じです。

・運転の邪魔にならない箇所に設置
![PIC_20160908_163404.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/c4e83b0c-410e-06f6-f8ff-171186215ffe.jpeg)
（滑り止めを下に敷いています）

## その1

わが愛車にPimetryシステム一式を積み込み、受信側のWindowsマシンでアプリとgstreamerを起動させます。受信している様子はデスクトップ録画アプリで録画しました。

[AG-デスクトップレコーダー][AG-デスクトップレコーダーのURL]

スクリーンセーバーもOFF。

以下は録画されたダッシュボードの様子です。

・夜の駐車場内
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">pimetry2：ニトリ駐車場内徐行 <a href="https://t.co/Cqf0jYcASC">pic.twitter.com/Cqf0jYcASC</a></p>&mdash; syasuder (@syasuder0) <a href="https://twitter.com/syasuder0/status/772772133256634368?ref_src=twsrc%5Etfw">2016年9月5日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

しかし録画できたのはこの動画含めて数分間だけでした。考えてみれば仕方がないのですが、デスクトップ録画中に録画しているWindowsマシンの様子が気になってスマホからVPN経由でリモートデスクトップでつないでしまったために、解像度が切り替わるなどしてデスクトップ録画が気絶したようなのです。
リモートデスクトップの画面では正常にストリーミングがおこなわれていたのに、帰宅してからデスクトップ録画ファイルを確認したところ、静止画が1時間以上続いていました。

ラズパイカメラはPi-Noirなのですが、やっぱり夜間は光が不足していました。次のトライは昼間に実行です。

## その2

前回の反省を踏まえて、デスクトップ録画開始したらWindowsマシンには触れずに出発です。

今度はいい感じに撮影できました。日本橋へ買い出しにいくついでの実験で、買い物中はラズパイ電源断でしたがその間もWindowsマシンは問題なく待ってくれていたようです。

動画は大阪の学園坂です。

・「大阪の学園坂」
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">pimetry2：学園坂 <a href="https://t.co/oo4GgjHpk3">pic.twitter.com/oo4GgjHpk3</a></p>&mdash; syasuder (@syasuder0) <a href="https://twitter.com/syasuder0/status/773156393037639680?ref_src=twsrc%5Etfw">2016年9月6日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

上記動画に該当する部分のpimetryのログを切り出してトレンドグラフ化してみたものが以下です。
![20160907210142.png](https://qiita-image-store.s3.amazonaws.com/0/48424/861a6ac8-d0f5-031e-e66c-dc271f086cbc.png)

急な上り坂なので、ググっと踏み込んで負荷が上がっているのが見て取れました。ただ残念なことに更新周期は2秒かかっているようでした。

ストリーミングの画質が悪いのはあきらめているのですが、数秒ごとにプチフリーズ気味なのが気になりました。
どうにも3G回線の問題ではなく、「パケ詰まり」ぽい挙動です。いまどき基地局のハンドオーバーなんて関係ないでしょうし。
動画がカクカクになるだけなら気にしないんですが、そのせいでタコメーターもフリーズ気味になっているようでした。気に入らないので少しだけ掘ってみました。

まず疑ったのは、
> あれ？このDMMモバイルのSIMって、高速通信ありの契約だっけ？
です。

安心してください。1.75GB残っています。
ライトプランではなく1GB/月の契約で、しかもこの1か月ほとんど使っていないので、パケットが余っていました。

![20160906171240.png](https://qiita-image-store.s3.amazonaws.com/0/48424/1f87387e-d9fb-bff4-53d0-268f8add8f8f.png)

しかしいくら高速通信のGB権利があっても、「何か」が邪魔しているわけですね。

そこで、机の上で3G接続してストリーミングの条件をいじってみました。

> ・・・b 250000 -fps 30 ・・・

これは30fpsで250kbpsということですが、宅内・机上でじっくりながめると、最初のうちは滑らかなのですが、ストリーミング開始数十秒後から問題のプチフリーズ現象が発生してしまいました。

そこで、以下のように15fps、150kbpsに変更してみると・・・
> ・・・b 150000 -fps 15 ・・・

プチフリーズ現象は発生しませんでした。再度元の設定に戻すとまたプチフリーズ。思い切って1.5Mbpsに設定すると、プチフリーズどころかフリーズしっぱなしになりました。

以下のいずれかが邪魔しているのでしょう。

- VPNサーバ
- PPPoEルータ
- DMM.com

さらに画像の傾きが気になったのですが、これは以下の写真のようにカメラが傾いて固定されていただけでした。こっそり直して最終トライです。

・見事にナナメに固定されたPiカメラ
![PIC_20160906_174658.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/684995f9-d3a1-400f-2d11-3952e18e5d33.jpeg)


## その3

大阪と言えば阪神高速環状線です。制限速度を守って走行してみました。首都高などと同様に制限60km/hです。じゃんじゃん追い抜いていく皆さんはすべてスピード違反です。
ストリーミングを150kbpsに抑えた効果か、概ね安定しているようでした。

・「阪神高速環状線」その1
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">pimetry2：阪神高速1 <a href="https://t.co/uK59sXKwug">pic.twitter.com/uK59sXKwug</a></p>&mdash; syasuder (@syasuder0) <a href="https://twitter.com/syasuder0/status/773778952670224386?ref_src=twsrc%5Etfw">2016年9月8日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
トラックが右端車線から合流して左端車線へ移動していきます。

・「阪神高速環状線」その2
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">pimetry2：阪神高速2 <a href="https://t.co/JWqxFY33sb">pic.twitter.com/JWqxFY33sb</a></p>&mdash; syasuder (@syasuder0) <a href="https://twitter.com/syasuder0/status/773782460899352577?ref_src=twsrc%5Etfw">2016年9月8日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
長～いカーブ＋合流連続でややうざいポイントです。

pimetryでときおり60km/hを越えた数値が表示されていますが、車速計はプラスマイナス10%程度の許容誤差がありますので、ご容赦いただけるかと思います。正確なタイヤの直径なんて誰にも分かりません。

ビルに挟まれていかにも電波状況が悪そうな箇所を通過するときに、正常→OFFLINE→正常と遷移する様子も撮れました。以下の動画がそれです。

・「阪神高速環状線」その3(OFFLINEから復帰)
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">pimetry2：阪神高速3：OFFLINEから復帰 <a href="https://t.co/gjzf3BkYnj">pic.twitter.com/gjzf3BkYnj</a></p>&mdash; syasuder (@syasuder0) <a href="https://twitter.com/syasuder0/status/773791237488807936?ref_src=twsrc%5Etfw">2016年9月8日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# まとめ

3GPIとOBDIIアダプタを組み合わせることで、簡単なテレメトリシステムを構築することができました。
今後クルマを乗り換えることがあったら、CAN搭載車でより滑らかなPimetryを実装して試してみたいものです。

複数台で同時中継すればもっと面白いと思うので、アマチュアサーキットレーサーの方などにはぜひ試してみたいただきたいです。

走行試験において電波状況が原因と思われるOFFLINE状態は何回か発生しましたが、いずれも自動的に復帰しました。
ラズパイ上のスクリプトでは再接続などは一切考量していませんが、数時間に渡って3G接続＋VPN接続が常時維持されていたようです。
3GPIを使っているとついついこの辺が当たり前のように感じてしまいますが、普通はこうはいかない気がします。

単純な比較はできないかもしれませんが、例えばウチの海外製スマホなどは半日もしないうちに3G/LTEやWifi接続が謎の接続不可状態になって、端末再起動が必要になったりします。
モバイルルータにWifi接続のPCでも、高速移動すると謎の再接続失敗現象に遭遇する気がします。

これまでQiita上で3GPIを使用した3件の実験をおこないましたが、3GPIでは「3Gはつながりっぱなし」かつ「何も心配いらない」という点は正直驚きです。経験上、NetwaorkManager先生が絡めば有線LANですら接続失敗することがあるのになぜ？という感想です。

いずれにしてもラズパイで3G絡みの実験や試作を行うなら、こういう点は結構重要な気がします。

さて、pimetryの発展としては以下が考えられます。

- GPSへの対応(3GPIに入っています)
- 各種センサの追加(6軸センサなど)

最後までお読みいただきありがとうございました。


# 落穂ひろい

## 動画オーバーレイ

このカテゴリが何と呼ばれるのかよくわかりませんが、pimetryのようなリアルタイム中継が不要ならば、つまり撮影した動画およびOBD情報などのログを事後に一つの動画にまとめ上げるツールが存在します。

[DashWare][DashWareのURL]

[VIRB Edit][VIRB EditのURL]

動画SNSにアップロードする動画を作成するには、これらのツールが便利だと思います。

## OBDII電源

Bluetooth版のELM327アダプタでデバッグを行う場合、BT接続の試験などは電源だけ入っていればよいので、OBDメスコネクタで以下のような冶具を作っておくと便利です。

![PIC_20160830_203904.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/da225a53-e7f7-4834-8bf3-fd9677d24ffc.jpeg)

コネクタはマルツなどで入手できますが、ナイロンコネクタなどの棚ではなく、クルマ関係のエーモン製品などの棚で販売されていたので、私はちょっと迷いました。

![PIC_20160830_141027.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/274be7db-65fe-cecb-bda0-9bc52dcff940.jpeg)

[マルツパーツ:OBDコネクタメス][マルツパーツ:OBDコネクタメスのURL]

# リンク
[メカトラックスさんのサイト][メカトラックスさんのURL]
[猫Piカメラの記事][ラズパイと3GPIで猫PiカメラのURL]
[遠隔操作OAタップの記事][遠隔操作OAタップURL]
[OBD Info-san!][Google Play OBD Info-san!のURL]
[Toruqueをアルテッツァで使う][魚拓のURL]
[PL2303ドライバ][PL2303ドライバのURL]
[ELMデバイスのデータシート][ELMデバイスのデータシートのURL]
[V1.5はクローンだという情報][V1.5はクローンだという情報のURL]
[OBDサポートビット一覧][OBDサポートビット一覧のURL]
[Mode01の一覧][Mode01の一覧のURL]
[Python-OBD][Python-OBDのURL]
[コマンドを使ってVPNクライアント接続する方法][コマンドを使ってVPNクライアント接続する方法のURL]
[ラズパイと3GPIで猫Piカメラ][ラズパイと3GPIで猫PiカメラのURL]
[QJson][QJsonのURL]
[AG-デスクトップレコーダー][AG-デスクトップレコーダーのURL]
[マルツパーツ:OBDコネクタメス][マルツパーツ:OBDコネクタメスのURL]
[DashWare][DashWareのURL]
[VIRB Edit][VIRB EditのURL]
[pimetry2のソース][pimetry2のソースのURL]

[メカトラックスさんのURL]:http://www.mechatrax.com/
[ラズパイと3GPIで猫PiカメラのURL]:http://qiita.com/syasuda/items/5def33db425aaf05bf6d
[遠隔操作OAタップURL]:http://qiita.com/syasuda/items/f683a1c0fe9a7e51fb68
[Google Play OBD Info-san!のURL]:https://play.google.com/store/apps/details?id=ph.jpn.ganchi.obdinfosan_trial&hl=ja
[魚拓のURL]:http://megalodon.jp/2015-0913-1203-15/minkara.carview.co.jp/userid/406970/car/331425/3069383/note.aspx
[PL2303ドライバのURL]:http://www.prolific.com.tw/JP/ShowProduct.aspx?p_id=223&pcid=126
[ELMデバイスのデータシートのURL]:https://www.elmelectronics.com/products/dsheets/
[V1.5はクローンだという情報のURL]:http://minkara.carview.co.jp/userid/194408/blog/29663172/
[OBDサポートビット一覧のURL]:https://en.wikipedia.org/wiki/OBD-II_PIDs#Mode_1_PID_00
[Mode01の一覧のURL]:https://en.wikipedia.org/wiki/OBD-II_PIDs#Mode_01
[Python-OBDのURL]:http://python-obd.readthedocs.io/en/latest/
[コマンドを使ってVPNクライアント接続する方法のURL]:http://netbuffalo.doorblog.jp/archives/4056722.html
[ラズパイと3GPIで猫PiカメラのURL]:http://qiita.com/syasuda/items/5def33db425aaf05bf6d
[QJsonのURL]:http://qjson.sourceforge.net/
[AG-デスクトップレコーダーのURL]:http://www.vector.co.jp/magazine/softnews/110608/n1106081.html
[マルツパーツ:OBDコネクタメスのURL]:http://www.marutsu.co.jp/pc/i/236750/
[DashWareのURL]:http://www.dashware.net/
[VIRB EditのURL]:http://www.iiyo.net/u_page/VIRB_Edit.aspx
[pimetry2のソースのURL]:https://github.com/syasuder/pimetry2