<!--
title:   Raspbian stretch Liteで初期設定
tags:    RaspberryPi,raspbian
id:      fec9fe2486f6a6536ef2
private: true
-->

# 追記(2022.12.04)

Raspbery Pi Imagerを使えば、この記事のような些末な作業はバイパスすることができます。

https://www.raspberrypi.com/software/


# はじめに

この記事では、raspbianの最低限の設定と、SSHでの接続までの手順をまとめます。(stretch Lite版)

## タイムゾーンなどの設定

やらなくても困りませんが、タイムゾーンがずれているとアラーム設定などがわかりにくいのでここで設定しておきます。
ついでにキーボードの設定などもしておきます。

以下の手順はRaspbianのlite版を使う場合の手順です。

```bash:コンソール
pi@raspberrypi:~ $ sudo raspi-config
```

変更するのは以下の４点です。

- ロケール
- タイムゾーン
- キーボードレイアウト
- WiFiの設定

まずローカライズを選択。

![vlcsnap-2018-03-19-16h32m28s632.png](https://qiita-image-store.s3.amazonaws.com/0/48424/7178fbf9-7a4a-05d1-8abe-12041d077257.png)

ロケールの設定をします。

![vlcsnap-2018-03-19-16h32m33s392.png](https://qiita-image-store.s3.amazonaws.com/0/48424/0e215459-433b-5e57-e0f7-318b2c6a3390.png)

en_US.UTF-8を選んでスペースバーで選択します。

![vlcsnap-2018-03-19-16h33m00s437.png](https://qiita-image-store.s3.amazonaws.com/0/48424/089cae94-7824-017c-ae0a-90543d6675d5.png)

ja_JP.UTF-8を選んでスペースバーで選択します。

![vlcsnap-2018-03-19-16h33m17s224.png](https://qiita-image-store.s3.amazonaws.com/0/48424/6f96379c-6204-9068-3b7c-9d42b520e224.png)

OKを選んでエンターキー。

![vlcsnap-2018-03-19-16h33m53s084.png](https://qiita-image-store.s3.amazonaws.com/0/48424/78758ed9-163c-4742-db57-b3b188aa9edc.png)

つづいてタイムゾーン設定です。

![vlcsnap-2018-03-19-16h34m04s967.png](https://qiita-image-store.s3.amazonaws.com/0/48424/0b9349d6-51d4-7d85-6289-c41dc24549c9.png)

「Asia」アジアを選択。

![vlcsnap-2018-03-19-16h34m24s666.png](https://qiita-image-store.s3.amazonaws.com/0/48424/7f277cb3-6c3f-31eb-5b0c-7643ccb8a6d6.png)

「Tokyo」東京を選択。

![vlcsnap-2018-03-19-16h34m45s349.png](https://qiita-image-store.s3.amazonaws.com/0/48424/98984de9-08c7-7373-f8da-39c5885a1216.png)

少々時間がかかります。

![vlcsnap-2018-03-19-16h35m38s803.png](https://qiita-image-store.s3.amazonaws.com/0/48424/54389830-067b-6ab5-36e0-bc70735d3fb9.png)

つづいてキーボードレイアウトの設定です。これをやらないと大抵のキーボードは英語キーボードと認識されてしまって、記号の入力などで困ります。

![vlcsnap-2018-03-19-16h35m59s055.png](https://qiita-image-store.s3.amazonaws.com/0/48424/bd5032e4-042d-24ab-6bdf-d84d4ff251c7.png)

これも少々時間がかかります。

![vlcsnap-2018-03-19-16h36m06s947.png](https://qiita-image-store.s3.amazonaws.com/0/48424/56458cb0-e5ce-e7b9-592d-4d178395edc1.png)

適当なモデルを選択。

![vlcsnap-2018-03-19-16h36m26s389.png](https://qiita-image-store.s3.amazonaws.com/0/48424/89ef25f6-6931-0000-8db0-aabc8710e2e4.png)

「Other」を選択します。

![vlcsnap-2018-03-19-16h36m36s684.png](https://qiita-image-store.s3.amazonaws.com/0/48424/447d07ba-cb25-2c45-3090-daef30e976a9.png)

「Japanese」を選択します。

![vlcsnap-2018-03-19-16h36m47s725.png](https://qiita-image-store.s3.amazonaws.com/0/48424/9ffef836-2ad7-6859-a123-5610823119dc.png)

「OADG 109」を選択します。この辺は地味にトラブルが多いです。

![vlcsnap-2018-03-19-16h36m54s747.png](https://qiita-image-store.s3.amazonaws.com/0/48424/f3b6d628-b2ae-6052-bcc0-98a2ab83eeb2.png)

以下２件はデフォルトのままで。

![vlcsnap-2018-03-19-16h37m01s058.png](https://qiita-image-store.s3.amazonaws.com/0/48424/49007fb7-dfaf-0c4a-fa41-f62ea741b90e.png)

![vlcsnap-2018-03-19-16h37m07s312.png](https://qiita-image-store.s3.amazonaws.com/0/48424/58907895-5fa2-bcfc-d6c1-f4db3eb53dac.png)

最後にWiFiのチャネル/バンド設定です。各国で規制が異なるためこの設定が必要です。

![vlcsnap-2018-03-19-16h37m25s085.png](https://qiita-image-store.s3.amazonaws.com/0/48424/4295b0c1-78a2-8ddf-1781-c9aabe817efb.png)

「JP Japan」を選択します。

![vlcsnap-2018-03-19-16h37m42s544.png](https://qiita-image-store.s3.amazonaws.com/0/48424/1bded223-121b-55dd-9e9f-6672a3e480c0.png)

![vlcsnap-2018-03-19-16h37m47s029.png](https://qiita-image-store.s3.amazonaws.com/0/48424/34240bd6-9a49-3610-48fe-7720f660a1ae.png)

「Finish」を選択して、「sudo reboot」します。

![vlcsnap-2018-03-19-16h38m36s360.png](https://qiita-image-store.s3.amazonaws.com/0/48424/e02a68e0-a706-e057-f3b0-109c03ccfc7b.png)

再起動します。

![vlcsnap-2018-03-19-16h38m44s406.png](https://qiita-image-store.s3.amazonaws.com/0/48424/d17c35b4-830a-e83f-4956-d2641da77b63.png)

再起動後のレインボーです。

![vlcsnap-2018-03-19-16h38m49s556.png](https://qiita-image-store.s3.amazonaws.com/0/48424/47bc2a5d-9c70-ec36-d059-7fa96d84ada0.png)

## SSHの有効化

SSHサーバを有効にします。まずraspi-configを起動。

```bash:コンソール
pi@raspberrypi:~ $ sudo raspi-config
```

以下のようにカーソルキーとエンターキーで選んで設定していきます。

![vlcsnap-2018-03-19-13h22m34s512.png](https://qiita-image-store.s3.amazonaws.com/0/48424/511b182b-3696-2157-ad38-b1a85bb1ce75.png)

![vlcsnap-2018-03-19-13h22m42s326.png](https://qiita-image-store.s3.amazonaws.com/0/48424/2d70c30a-b137-7f60-2065-b778feea2e74.png)

![vlcsnap-2018-03-19-13h22m48s821.png](https://qiita-image-store.s3.amazonaws.com/0/48424/51e65df2-38d7-51f7-7aba-6db4f35c64c4.png)

最後はFinishです

![vlcsnap-2018-03-19-13h22m52s501.png](https://qiita-image-store.s3.amazonaws.com/0/48424/22708536-bb59-6a54-6a5a-f27767510a8a.png)

以降はUSBキーボードとHDMIディスプレイがなくても、SSHでログインすることができます。

SSHで接続するには、仮想端末あるいは端末エミュレータと呼ばれるものを使用します。ここでは[TeraTerm][TeraTermのURL]
を使用します。

以下、最短で使い方を説明します。

まずPC(ここではWindowsマシンを想定)をラズパイと同じネットワークに接続します。直結でも構いません。

「新しい接続」で「TCP/IP」を選択し、「ホスト」にIPアドレスを設定します。

![20180319145426.png](https://qiita-image-store.s3.amazonaws.com/0/48424/ef3dc886-a09e-cdd1-8fe9-9efcaeb2b586.png)

「プレインパスワードを使う」を選択し、ユーザ名/パスフレーズにpi/raspberryを指定して「OK」ボタンを押します。

![20180319155613.png](https://qiita-image-store.s3.amazonaws.com/0/48424/3c4b7044-3be6-a177-cf0f-1c699e4e7650.png)

これでラズパイの仮想端末にログインできました。

![20180319155733.png](https://qiita-image-store.s3.amazonaws.com/0/48424/6634d328-2890-b2d0-6ff2-e82fa019d268.png)

あとはHDMI/USBキーボードでの操作とおおよそおなじことができます。ただし、LANケーブルが抜けたり、ラズパイが再起動したときにはTeraTermで再度接続する必要があります。HDMI/USBキーボードの場合は、そんなことは不要ですね。
TeraTermでSSH経由でラズパイを操作する最大の利点は、画面のコピペができることです。

例えばこの記事に書かれたコマンドをそのままコピーして、TeraTerm上でペーストすれば、打ち間違いが起こりません。
具体的には、TeraTerm上では、マウスの右クリックでクリップボードの文字列を入力することができます。
TeraTermの使い方については、Google先生に聞くのが近道です。

ここでは最低限の注意点として以下を挙げておきます。

- ラズパイのデフォルトのユーザ名/パスワードは広く知られていることに注意
- SSHでプレインパスワードを使用することに注意

お仕事でラズパイを使う場合、SSHについてはセキュリティがわかる人と相談して運用したほうが良いでしょう。

## バージョンメモ

- 2018-03-13-raspbian-stretch-lite.zip