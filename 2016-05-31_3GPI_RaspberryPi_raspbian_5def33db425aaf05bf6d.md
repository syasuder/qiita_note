<!--
title:   ラズパイと3GPIで猫Piカメラ
tags:    3GPI,RaspberryPi,raspbian
id:      5def33db425aaf05bf6d
private: false
-->
# はじめに

メカトラックスさんから3GPIをお借りすることができましたので、ラズパイと組み合わせて近所をうろつく野良猫を激写して通知してくれる猫Piカメラシステムを組んでみました。

![20160602105224.png](https://qiita-image-store.s3.amazonaws.com/0/48424/ef6595c2-0601-b347-ad7c-59e1099264f7.png)


コンセプト検証なのでできるだけコードは書かないようにします。

## 猫Piカメラとは

"人感センサ"とも呼ばれるPIRセンサは、センサの前の「生暖かい」生き物を検出することができます。
人間が近づくと明かりが灯るセンサライトなどでよく見かけるセンサです。

使用したPIRセンサモジュール
![PIC_20160529_191441.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/35fc0f6e-c8fe-5f6e-22d1-b0676a674687.jpeg)

このセンサで猫を検出して撮影するセンサカメラシステムを構築してみます。

もちろん撮影した画像は3GPI経由で即時通知します。
3GPIのおかげで電源さえあればどこでも猫の観察ができるというわけです。

システム構成は以下のような感じです。
![20160531182246 .png](https://qiita-image-store.s3.amazonaws.com/0/48424/ccecf89f-8870-44f8-b0d8-57f81353edfe.png)

# 材料

- ラズパイ(3GPIが使えるものなら何でも。今回はA+)
- 3GPI
- Pi-NoIRカメラ

その他立ち上げには以下を使いました。

- キーボード
- HDMIありのディスプレイ
- USB-LANアダプタ

# 下ごしらえ

## USBハブ経由で3GPI

A+はUSBポートが1つしかないので、立ち上げのためのキーボードやUSB-LANアダプタを接続するにはUSBハブ経由で接続する必要があります。
ハブ経由だとUSBのトラブルが起こりがちなので、この点心配していましたが問題なく動作しました。

以下はディスプレイなどを接続した状態の写真です。

全体写真：
![PIC_20160529_192227.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/ff64f863-c114-2118-266a-1f6ebc97f3a7.jpeg)
(Pi-NoIRカメラモジュール表面の茶色いのはポリイミドテープで、ポカヨケ絶縁用です。)

## PIRセンサのGPIO

たまたま[こちらの記事][PIRの記事のURL]と同じPIRセンサが手持ちにありましたので接続してテストしておきます。
幸い記事のまんまで3GPIが使用するGPIOと被ることもありませんでした。

以下にポートの割り付けを引用しておきます。

表1:3GPIが使用するGPIO

|GPIO名|PIN番号|設定|機能|
|:-----|:-----|:----|:---|
|GPIO17 | 11 | 出力 | 電源制御 |
|GPIO27 | 13 | 出力 | リセット |
|GPIO22 | 15 | 入力 | 電源監視 |

([3GPIのハードウェア][3GPIハードのURL]から引用)

表2:PIRセンサで使用するGPIO

|GPIO名|PIN番号|設定|機能|
|:-----|:-----|:----|:---|
|GPIO18 | 12 | 入力 | PIRセンサ |

([PIRの参考記事][PIRの記事のURL]を参考)

## Pi-NoIRカメラ

赤外線カメラは期間限定で安売りしているときに買ったものを使います。
赤外線の光源があれば暗闇でも撮影できます。

## DMM.comのSIMで接続テスト

手持ちのDMM.comのMVNOのSIMをスマホから抜いて3GPIに挿入です。
micro-SIMなので手持ちのアダプタで標準SIM(mini-SIM)化して使用しています。
この手のアダプタを使うとスライド式のスロットで引っかかりがちですが、3GPIはその心配がありません。
代わりに厚みに注意が必要そうです。
![PIC_20160529_205526a.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/04506db7-cd75-1a33-f694-470c68adce56.jpeg)

3GPI付属SDカードのjessieなら、以下のコマンドだけで接続できました。

```shell-session:ラズパイ上
dmmpi@raspberrypi:~ $ sudo nmcli con add type gsm ifname "*" con-name DMM apn dmm.com user dmm@dmm.com password dmm
```
(何年も前からDMMモバイルを使っている人なら、apnはdmm.comではなくvmobile.jpです。)

pppのインタフェースが生成されることをifconfigコマンドで確認しておきます。
ここまで3GPI専用のコマンドやスクリプトは一切叩いていません。

```shell-session:ラズパイ上
pi@raspberrypi:~ $ ifconfig
lo        Link encap:Local Loopback
          ～中略～
ppp0      Link encap:Point-to-Point Protocol
          inet addr:100.92.193.xxx  P-t-P:10.64.64.64  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
          RX packets:31 errors:0 dropped:0 overruns:0 frame:0
          TX packets:46 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3
          RX bytes:2415 (2.3 KiB)  TX bytes:2829 (2.7 KiB)
```

ここまでお膳立てされていると、本来やりたいことだけに集中できます。

最後にインターネット上の通信の確認です。これは自前のウェブサーバのログで行うことにしました。
存在しないURLへのGETリクエストを投げます。

```shell-session:ラズパイ上
pi@raspberrypi:~ $ wget http://example.com/iam3gpi
--2016-05-22 22:12:05--  http://example.com/iam3gpi
Resolving example.com (example.com)... xxx.171.xxx.xxx
Connecting to syasuda.com (example.com)|xxx.171.xxx.xxx|:80... connected.
HTTP request sent, awaiting response... 404 Not Found
2016-05-22 22:12:08 ERROR 404: Not Found.
```

ウェブサーバ上でログを確認します。

```shell-session:ウェブサーバ上
syasuda@Ubuntu12:~$ cat /var/log/nginx/access.log.* | grep iam3gpi
49.239.xxx.xxx - - [22/May/2016:22:12:08 +0900] "GET /iam3gpi HTTP/1.1" 404 205 "-" "Wget/1.16 (linux-gnueabihf)"
```

ラズパイのPPP0のIPアドレスと、サーバに残るIPアドレスが異なっていることも確認できました。
DMM.comから払い出されるIPアドレスはプライベートアドレスだということです。

あとは接続状態に注意すれば、スクリプトやアプリから見ればイーサネットやWiFi接続のLANと変わりがありません。


## カメラ部分のテスト

Piカメラはラズパイに直結してraspi-configで有効化するだけで使えます。
ちょっと古いですが、以下のページが分かりやすいです。

[Piカメラのセットアップ][PIカメラのセットアップのURL]

Pi-NoIRカメラでも同じです。

撮影用のコマンドはraspbianに最初から入っています。
以下のコマンドを実行すると、HDMI出力にカメラの画像が表示され、撮影が行われます。Piカメラの赤LEDも点灯します。
`shell-session:ラズパイ上
pi@raspberrypi:~/images $ raspistill -o selfy.jpg
`

Pi-NoIRカメラによるラズパイの自撮り画像が以下です。
![selfy.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/07d747b8-feaf-873a-e004-cb661d4e2a38.jpeg)
ピンボケなのはフォーカスが初期設定のままのためです。おそらく数十cmに設定されているかと思います。今回の用途ではフォーカス調整は不要ですが、Pi/Pi-NoIRカメラのフォーカスの調整方法などは[こちら][Piカメラの仕様のURL]に書かれているようです。


## 画像アップロードのテスト(scp)

ラズパイからインターネット上のサーバに画像をアップロードするテストをしておきます。
IPでつながるのですから、テストというよりも動作確認です。
使いたいプロトコルがMVNO事業者によって遮断されていたりするとあとで大変困ります。

インターネット上のサーバにscpで手っ取り早く送信して確認しました。
先ほどのセルフィを送信します。

使用するサーバはパスワード認証(ユーザ名とパスワード)を許可していないので、ラズパイからサーバへのssh接続のためには多少の準備が必要です。
具体的には以下の手順です。

- ラズパイでキーペアを作る(ssh-keygen)
- ラズパイの公開鍵をサーバに登録する(authorized_keys)


まずラズパイでキーペアを作ります。

```shell-session:ラズパイ上
pi@raspberrypi:~ $ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/pi/.ssh/id_rsa):
Created directory '/home/pi/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/pi/.ssh/id_rsa.
Your public key has been saved in /home/pi/.ssh/id_rsa.pub.
The key fingerprint is:
b1:68:17:65:0f:ac:fb:f4:f1:xx:xx:xx:xx:37:b0:23 pi@raspberrypi
The key's randomart image is:
+---[RSA 4096]----+
～～中略～～
|             .o++|
+-----------------+
pi@raspberrypi:~ $ ls .ssh
id_rsa  id_rsa.pub
```

次に公開鍵(id_rsa.pub)の内容をサーバに登録します。
ラズパイ上で作成したid_rsa.pubを分かりやすいようにrpi_id_rsa.pubにリネームしてサーバにアップロードします。
サーバにssh接続できるマシンやツールからアップロードします。

```shell-session:サーバ上
syasuda@xxxxx:~/.ssh$ ls
authorized_keys  rpi_id_rsa.pub
syasuda@xxxxx:~/.ssh$ cat rpi_id_rsa.pub >> authorized_keys
```

これでラズパイからサーバへssh接続できます。

```shell-session:ラズパイ上
pi@raspberrypi:~ $ ssh -l <ログインユーザ名> -p <ポート番号> <サーバホスト名>
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.19.0-58-generic x86_64)
～～中略～～
Last login: Mon May 23 19:47:03 2016 from xxx.com
syasuda@server:~$
```

ポート番号などを覚えるのが面倒なのでconfigに書いておきます。

```shell-session:ラズパイ上
pi@raspberrypi:~ $ cat ~/.ssh/config
Host nekopi-server
    HostName        <サーバホスト名>
    Port            <ポート番号>
    IdentityFile    ~/.ssh/id_rsa
    User            <ユーザ名>
```

こうしておけば以下のようにすっきり書けます。

```shell-session:ラズパイ上
pi@raspberrypi:~/images $ scp selfy.jpg nekopi-server:
selfy.jpg                                                                                                                                                                                         100% 2598KB  35.1KB/s   01:14
```

サーバ側で受信を確認します。

```shell-session:サーバ上
syasuda@xxxx:~$ ls -al
-rw-r--r-- 1 syasuda syasuda 2660475 May 23 21:18 selfy.jpg
```

## LINE BOT APIによる通知

サーバにアップロードされた画像をスマホに通知します。

Push通知のインフラはいろいろあると思いますが、何をするにも面倒なのでLINE BOT APIで通知することにします。

LINE BOT APIについては[こちら][LINEBOTAPIの記事のURL]の記事を参考に立ち上げておきます。

[Qiita:LINE BOT API Trialでできる全ての事を試してみた][LINEBOTAPIの記事のURL]

上記の記事には受信＋送信のスクリプトサンプルが含まれます。
送信だけならコールバックの設置は不要です。ただし、どこかに設置しないと通知相手のMIDを調べるのが大変かもしれません。

そこでサンプルから送信部分だけを切り出したものが以下です。

```php:sendmsg.php
<?php
error_log("sendmsg start.");

// アカウント情報設定
$channel_id = "xxxxxxxxxxxx";
$channel_secret = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
$mid = "uxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";

// リソースURL設定
$original_content_url_for_image = "http://xxxxx.com/selfy.jpg";
$preview_image_url_for_image = "http://xxxxx.com/selfy.jpg";

// メッセージ送信先
$to = "uxxxxxxxxxxxxxxxxxxxxxxxxxx";

// メッセージコンテンツ生成
$image_content = <<< EOM
        "contentType":2,
        "originalContentUrl":"{$original_content_url_for_image}",
        "previewImageUrl":"{$preview_image_url_for_image}"
EOM;

// 受信メッセージに応じて返すメッセージを変更
$event_type = "138311608800106203";

$content = <<< EOM
        "contentType":1,
        "text":"こちら猫Piカメラです。何かを検出しました!"
EOM;
$post = <<< EOM
{
    "to":["{$to}"],
    "toChannel":1383378250,
    "eventType":"{$event_type}",
    "content":{
        "toType":1,
        {$content}
    }
}
EOM;
error_log($post);

api_post_request("/v1/events", $post);

$content = $image_content;
$post = <<< EOM
{
    "to":["{$to}"],
    "toChannel":1383378250,
    "eventType":"{$event_type}",
    "content":{
        "toType":1,
        {$content}
    }
}
EOM;

error_log($post);
api_post_request("/v1/events", $post);

error_log("sendmsg end.");

function api_post_request($path, $post) {
    $url = "https://trialbot-api.line.me{$path}";
    $headers = array(
        "Content-Type: application/json",
        "X-Line-ChannelID: {$GLOBALS['channel_id']}",
        "X-Line-ChannelSecret: {$GLOBALS['channel_secret']}",
        "X-Line-Trusted-User-With-ACL: {$GLOBALS['mid']}"
    );

    $curl = curl_init($url);
    curl_setopt($curl, CURLOPT_POST, true);
    curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($curl, CURLOPT_POSTFIELDS, $post);
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
    $output = curl_exec($curl);
    error_log($output);
}
```

テキストだけならラズパイ単体のみで送信できますが、
画像はインターネット上でアクセス可能な場所へアップロードする必要があります。
サーバーにアップロード済みのラズパイの自撮り画像をLINEに通知してみました。

LINEクライアントに届いたメッセージのスクリーンショット:
![Screenshot_2016-06-01-21-00-23.png](https://qiita-image-store.s3.amazonaws.com/0/48424/6fb141fa-cea7-b0e4-72f6-0d9edba26fb9.png)


## PIRセンサ接続

あらかじめ加工しておいたコネクタをヘッダピンに刺すだけです。

PIR接続部の写真:
![PIC_20160529_193233.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/d9a875e3-e47b-63f4-6521-34d476e1be91.jpeg)

[こちらの記事][PIRの記事のURL]と同じようにスクリプトからGPIOを監視します。

上記の記事のスクリプから切り出した以下のようなスクリプトで、PIRセンサの出力が0/1で変化することを確認しておきます。

```bash:test.sh
#!/bin/sh
#定数
gpio=18       #GPIO18番を使用
duration=1    #ポーリング間隔（秒）

# gpio がまだ初期化されていなければ初期化
if [ ! -d /sys/class/gpio/gpio${gpio} ]
then
    echo $gpio > /sys/class/gpio/export
fi
value=/sys/class/gpio/gpio${gpio}/value #gpio の値

while :
do
    current_value=`cat $value`
    echo ${current_value}

    sleep ${duration}s
done
```

センサの前に手をかざしたりします。

```shell-session:ラズパイ上
pi@raspberrypi:~/sandbox/pir $ ./test.sh
1
0
0
1
1
～～以下略～～
```

0/1が変化すればOKです。

## 3Gのオンデマンド通信

画像のアップロードのときだけ3G通信をする仕掛けを確認します。
3GPI用のraspbianに入っているユーティリティを使って3GPIの電源を制御して3G通信のON/OFFを切り替えます。

```shell-session:ラズパイ上
pi@raspberrypi:~/3gpi-utils $ 3gpictl
Usage: 3gpictl [OPTION]

Options:
  --poweron        turn on 3g module
  --poweroff       turn off 3g module
～～以下略～～
```
同ユーティリティは、メカトラックスさんが[こちら][3gpi-utilsのURL]で公開しておられます。

このツールで`--poweron`すれば自動的にダイヤルアップ～ppp接続確立までNetworkManagerが自動的にやってくれます。きわめて簡単です。そのように設定済みのraspbianだからこそです。

ここで、猫検出から画像アップロードまでの流れを確認しておきます。
初期状態は3GPIの電源OFFとします。

流れは以下のようになるでしょう。

1. 猫検出まで待つ
1. 検出したら撮影
1. 3GPIの電源ON
1. 画像アップロード
1. 3GPIの電源OFF
1. 1.に戻る

3の直後に4をやろうとするとおそらく失敗します。PPPの接続確立までには時間がかかるからです。
最悪値を決め撃ちしてsleepコマンドで待っても動きそうですが、ここではpppの確立時にスクリプトを叩いてもらうようにします。

現在の構成では、if-up時のみに実行したいので、以下にスクリプトを配置します。

/etc/network/if-up.d/

ここではopenssh-serverのものを参考に以下の様なスクリプトとしました。

```bash:/etc/network/if-up.d/999nekocam
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

echo "`date`: ppp0 if-up" >> /home/pi/nekocam.log
# 以降に必要な処理を書く
```
ifの分岐が並んでいるのは無関係なinterface、例えばloなどがupしたときに呼ばれた場合のガードです。
`IFACE`だけで`ppp0`以外を撥ねると2回呼ばれてしまうので、`DEVICE_IFACE`も確認しています。
この辺りはどうしても環境依存になってしまうように思います。汎用化すると見通しの悪いスクリプトになりそうです。

というわけで先ほどの流れを以下のように変更します。

メインスクリプトのフロー：

1. 猫検出まで待つ
1. 検出したら撮影
1. 待ち合わせフラグを立てる
1. 3GPIの電源ON
1. 待ち合わせフラグが下りるのを待つ
1. 画像アップロード
1. 3GPIの電源OFF
1. 1.へ戻る

if-upスクリプトのフロー：

1. 待ち合わせフラグを下ろす
1. 終了

待ち合わせフラグというのはここではただのファイルです。
いろいろ問題があるので、まじめに運用するならpython辺りで書いた方が良いかもしれません。
ちなみにですが、if-upに処理を直接書くのはやめておきました。
処理が止まるとNetworkManager様のご機嫌が悪くなるので。

# 調理

下ごしらえが済んだのでメインのスクリプトを書きます。
ここまでに動作確認してきたスクリプトを組み合わせていきます。

書いたスクリプトが以下です。

```bash:/etc/network/if-up.d/999nekocam
#!/bin/sh
#set -xv

#定数
GPIO=18       #GPIO18番を使用
DURATION=1    #ポーリング間隔（秒）
WIDTH=480     #撮影写真幅
HEIGHT=360    #撮影写真高さ
ROLL=image
SAVEDIR=/home/pi/sandbox/images

WORKDIR=/home/pi/sandbox/nekopi/
LOCKFILE=uploading.lock
THREEGPICTL=/path/to/3gpictl
DESTDIR=nekopi-server:www/nekopi/
BASEURL=http://XXXXX.com/nekopi/

#
cd $WORKDIR
# gpio がまだ初期化されていなければ初期化
if [ ! -d /sys/class/gpio/gpio${GPIO} ]; then
    echo $GPIO > /sys/class/gpio/export
fi
value=/sys/class/gpio/gpio${GPIO}/value #gpio の値は正論理(1:検出)
prev_value=`cat $value`

echo 'waiting'
while :
do
    current_value=`cat $value`

    if [ ${prev_value} -eq 0 ]; then
	# 立ち上がりエッジ検出
        if [ $current_value -eq 1 ]; then
	    echo 'detected'
	    # ファイル名
            filename=$ROLL-$(date +"%d%m%Y_%H%M%S").jpg
	    # 検出したら撮影
	    raspistill -o $SAVEDIR/$filename -t 1000 -w $WIDTH -h $HEIGHT
	    # 待ち合わせフラグを立てる
	    echo $SAVEDIR/$filename > $LOCKFILE

	    # ここでonの場合はいったんoffにする
            power_status=`$THREEGPICTL --status`
            if [ $power_status = "on" ]; then
	        $THREEGPICTL --poweroff
		sleep 5s # おまじない
            fi
	    # 3GPIの電源ON
	    $THREEGPICTL --poweron
	    # 待ち合わせフラグが下りるのを待つ
	    count=0 # タイムアウトカウンタ
	    while :
	    do
  	        if [ ! -e $LOCKFILE ]; then
		    break
		fi
                sleep ${DURATION}s
		count=$(( count + 1 ))
  	        if [ $count -gt 100 ]; then
		    # 100秒過ぎたらタイムアウト
		    echo "timeout"
		    break
		fi
	    done

	    # 画像アップロード
	    echo "uploading $SAVEDIR/$filename"
	    scp -F/home/pi/.ssh/config $SAVEDIR/$filename $DESTDIR

	    # LINE通知
	    echo 'notifying'
	    php sendimg.php -u $BASEURL/$filename

	    # 3GPIの電源OFF
	    $THREEGPICTL --poweroff
	    # ppp0を削除
	    nmcli c delete ppp0
	    echo 'waiting'
        fi
    fi
    prev_value=${current_value}
    sleep ${DURATION}s
done
```

上記スクリプトでは、sendmsg.phpを引数付に変更したsendimg.phpを呼び出しています。

# 味見

## 自動起動

テストするのにいちいちコマンド叩いてスクリプトを叩くのは面倒なので、[この記事][自動起動の記事のURL]を参考にデーモン化しておきます。電源断はバチ切りになりますが、ときどきfsckしてごまかすことにします。

[Qiita:Raspbian jessieでSystemdを使った自動起動][自動起動の記事のURL]

```bash:/etc/systemd/system/nekopi.service
[Unit]
Description=Neko Pi Script Service
After=multi-user.target

[Service]
Type=idle
ExecStart=/home/pi/sandbox/nekopi/nekopir.sh

[Install]
WantedBy=multi-user.target
```

systemctlのstart/stopでテストしてから、以下でデーモン化します。

```bash:/etc/systemd/system/nekopi.service
pi@raspberrypi:~/sandbox/nekopi $ sudo systemctl enable nekopi
```

以降、電源入れるだけでスクリプトが実行されます。
注意点としては、実行するユーザが変わることです。メインスクリプトのscpでは-Fオプションでconfigファイルを指定しています。

## 最終テスト

センサとカメラの前に生暖かいもの(お湯入りのマグカップ)を近づけて反応を見ます。

マグカップをクイックルワイパーでリモート操作：
![PIC_20160531_165510.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/8efe9483-8624-a45a-c1dc-d55e2e7e646a.jpeg)

LINEクライアントに通知が来ました。
![Screenshot_2016-06-01-21-00-50.png](https://qiita-image-store.s3.amazonaws.com/0/48424/216b2a47-fc47-4453-79b7-fb820119e040.png)

(補足：正直、マグカップのお湯に反応したかどうかはよくわかりません。PIRセンサが後ろにいる人間に反応しているだけかもしれません。)

持ち運びとセッチングが面倒なのでとりあえず百均のタップボックスに入れています。
スリットが両側にあるタイプなので、猫Piカメラにぴったりです。

![PIC_20160531_171544.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/f8db5251-146f-11b2-e7c9-f5502afe2c81.jpeg)

![PIC_20160531_162314.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/3749bde9-a0cc-0e86-5e98-e030caaed8a5.jpeg)

テストの結果、PIRセンサはスリットで一部を隠した方がいい感じでした。Piカメラの画角に対して、PIRセンサの反応する範囲が広すぎました。

まさにこのためにあるようなスリット：
![PIC_20160531_165853.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/7cfb54fd-d941-b47a-6e7e-62ced29b28ea.jpeg)


## 赤外線光源が強力すぎる

実戦投入前に部屋を暗くしてテストしておきます。
赤外線光源は以下のようなものを使っています。おもちゃレベルのものです。
![PIC_20160529_193400.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/e80d8b4e-8b16-2968-71cc-c0e01510e1a4.jpeg)

しかしカメラの真横に光源を置いてテストしてみると、以下のような結果になりました。
上段が光源なし、下段が光源ありです。
![Screenshot_2016-06-01-21-01-05.png](https://qiita-image-store.s3.amazonaws.com/0/48424/a9ece398-bb93-b90b-806b-e1e0970ffe7e.png)
白く飛んじゃいましたので光源だけ倍くらい遠くに置いてみました。
下段が遠い光源です。
![Screenshot_2016-06-01-21-01-12.png](https://qiita-image-store.s3.amazonaws.com/0/48424/aac68164-2213-8ac6-e630-3a18904bfb82.png)

# 実食

## 味見でおなかいっぱい

味見でおなかがいっぱいになってしまいました。
猫と猫Piカメラの実戦は次の機会とさせていただきます。


# まとめ

最後に簡単にまとめておきます。

## 猫Piカメラで確認できたこと

### 3GPIの立ち上げが簡単すぎること

市販のUSB-LAN並みに簡単です。CLIが使える人間にとってはAndroidスマホでMVNOのSIMを使うよりも簡単です。
常時接続の用途なら本当に何もしないで使えると思います。

### 3GPIはラズパイA+で使えること

何気にレアな組み合わせだったかもしれませんが、固定用スペーサも使えましたし、全く問題ありませんでした。

### 3GPIはGPIOを3本しか占有しないこと

ラズパイのヘッダピンがそのまま出ているので、既存のシステムと組み合わせるのが簡単そうです。

## アレンジ

発展システムとして以下の様なアレンジが考えられます。

- PIRセンサで検出後、画像から"猫かどうか"を確認してから通知する
- アップロード中も猫検出できるようにする
- LINE BOT経由で猫Piカメラを操作できるようにする


## 嵌りポイント

深堀せずに回避した問題をアトランダムに書いておきます。

### Packet Corrupt

Azure上の仮想マシンにscpで画像をアップロードしようとするとstallが多発してPacket corruptで失敗しました。
帯域制限か何かにひっかかったぽかったです。
rsyncで帯域制限するとなんとかアップロードできましたが、とてつもなく遅くなるので自前のサーバーを使っています。それでも3G網経由ではscpが遅いです。

### --poweroff --poweron繰り返しでppp0増殖

ppp切断せずにいきなりpoweroffするのが乱暴なのだと思います。スクリプトではなんとなく回避しています。
NetworkManager様が絡むものは極力近づかないようにしておきます。

### LINE BOTの送信制限

開発者サイトでIPアドレスによる制限を設定していると3GPIのようなIPアドレス非固定環境から送信できなくなる場合があります。
いったんサーバーで引き受けて云々をやりだすと面倒なので制限解除して臨んでいます。そもそもPush通知の面倒を避けたいがためにLINE BOTを利用しましたので。


# リンク

[Qiita:人感センサ A500BP (DSUN-PIR, SB00412A-1も) が安いだけでなく Raspberry Pi との相性もバッチリだったので、人感カメラが10分で出来てしまった話。][PIRの記事のURL]
[メカトラックスさんのサイト][メカトラックスさんのURL]
[3GPIのハードウェア][3GPIハードのURL]
[Piカメラのセットアップ][PIカメラのセットアップのURL]
[Piカメラのフォーカス設定][Piカメラの仕様のURL]
[Qiita:LINE BOT API Trialでできる全ての事を試してみた][LINEBOTAPIの記事のURL]
[3gpi-utils][3gpi-utilsのURL]
[Qiita:Raspbian jessieでSystemdを使った自動起動][自動起動の記事のURL]

[PIRの記事のURL]:http://qiita.com/UedaTakeyuki/items/dd4365cefdb7293f79f6 "Qiita:人感センサ A500BP (DSUN-PIR, SB00412A-1も) が安いだけでなく Raspberry Pi との相性もバッチリだったので、人感カメラが10分で出来てしまった話。"

[メカトラックスさんのURL]:http://www.mechatrax.com/ "IoT/M2M機器/Raspberry Pi用周辺機器-メカトラックス株式会社"
[3GPIハードのURL]:https://github.com/mechatrax/3gpi/wiki/%E3%83%8F%E3%83%BC%E3%83%89%E3%82%A6%E3%82%A7%E3%82%A2#14-%E4%BD%BF%E7%94%A8%E3%83%9D%E3%83%BC%E3%83%88 "3GPIのハードウェア情報"

[PIカメラのセットアップのURL]:https://www.rs-online.com/designspark/raspberry-pi-camera "Raspberry Piカメラのセットアップ方法"

[Piカメラの仕様のURL]:http://elinux.org/Rpi_Camera_Module#Adjusting_Lens_Focus "Rpi Camera Module Adjusting Lens Focus"

[LINEBOTAPIの記事のURL]:http://qiita.com/betchi/items/8e5417dbf20a62f2239d "LINE BOT API Trialでできる全ての事を試してみた"

[3gpi-utilsのURL]:https://github.com/mechatrax/3gpi-utils "3gpi-utils"

[自動起動の記事のURL]:http://qiita.com/yosi-q/items/55d6d3d6834c778ae2ea "Qiita:Raspbian jessieでSystemdを使った自動起動"