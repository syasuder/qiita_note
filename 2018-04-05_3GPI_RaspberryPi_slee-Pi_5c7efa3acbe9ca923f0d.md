<!--
title:   外部入力端子付slee-Pi 2（slee-Pi 2 Plus）の動作
tags:    3GPI,RaspberryPi,slee-Pi
id:      5c7efa3acbe9ca923f0d
private: true
-->
# はじめに

[前回の記事][前回の記事のURL]では、[メカトラックス][メカトラックスさんのURL]さんの[slee-Pi 2][sleepi2のURL]を使った間欠動作（タイマー動作）と死活監視を試しましたが、この記事では[slee-Pi 2 Plus][sleepi2のURL]の外部入力端子の基本的な使い方を確認します。

[slee-Pi 2 Plus][sleepi2のURL]と[slee-Pi 2][sleepi2のURL]の違いはメカトラックスさんのサイトに説明があります。

[slee-Pi 2 Plus][sleepi2のURL]から引用
> slee-Pi 2 Plus：slee-Pi 2 Plus は slee-Pi 2 に外部入出力回路を追加したものです。設定すれば、外部入力による起動と強制電源断が可能です。

ちなみにですが、[slee-Pi 2][sleepi2のURL]にコネクタだけ実装しても[slee-Pi 2 Plus][sleepi2のURL]にはなりません。


## 使用機材

この記事では以下の機材を用います。

- ラズパイ3B(含マイクロSDカード 4GB以上)
- [slee-Pi 2 Plus][sleepi2のURL]
- 12V DCアダプター([3GPi Ver.2][3GPIのURL]付属のものなど)

# 準備

組み立ててから簡単な配線をおこないます。

## 組み立て

40ピンヘッダを向きを合わせて刺すだけです。
以下の写真の通り、下がラズパイ、上に[slee-Pi 2 Plus][sleepi2のURL]が来るように接続します。

![20180405154726.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/81a297f8-8804-2713-cae3-2169a8ef8c25.jpeg)

[slee-Pi 2][sleepi2のURL]との違いは、以下の写真の部分のコネクタやICが実装されていることです。

![20180405154215.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/0bff2479-ec76-6f45-7ca1-aad450d76649.jpeg)

## 外部入力端子への配線

JSTのEHコネクターが適合します。2極なので品名はEHR-2です。

公式ドキュメントに書かれています。

[mechatrax/slee-pi2 ハードウェア 3. インターフェース](https://github.com/mechatrax/slee-pi2/wiki/%E3%83%8F%E3%83%BC%E3%83%89%E3%82%A6%E3%82%A7%E3%82%A2#3-%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%95%E3%82%A7%E3%83%BC%E3%82%B9)

私はマルツパーツ店頭で買いました。

[EHコネクター 2.5mmピッチ ハウジング 2極(10個入り)【EHR-2*10】](https://www.marutsu.co.jp/pc/i/46582/)

![20180405154949.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/8558d5db-7a6d-4260-e939-cb9328cedc49.jpeg)

↓↓↓コンタクトピンはこれです。

[EHコネクター コンタクト(10個入)【SEH-001T-P0.6*10】](https://www.marutsu.co.jp/pc/i/59209/)

![20180405155041.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/0444bdcb-935c-c98a-01e2-73c3ca0ac195.jpeg)

動作確認程度なら、わざわざケーブルを作るまでもありません。
しかし、しっかり配線しておけばとしばらく間が空いた時などに再セットアップが素早くできる気がします。

以下では、ケーブルを作る場合と、作らない場合の2通りを示します。

### きちんとケーブルを作る場合

まずケーブル加工には専用圧着工具が必要です。お高いです。

めったに試作しないなら、プロのハード屋さんに製作を依頼するのが良いですが、取引がなければ例えばミスミさんが利用できます。（法人または個人事業主）

[EHコネクタ　圧着済みコンタクトケーブル【50個入り】](https://jp.misumi-ec.com/vona2/detail/110400380420/)

私は、[ハーフピッチ対応精密圧着ペンチ【PA09】](https://www.marutsu.co.jp/pc/i/35601/)で、自作しました。これは試作に限っては問題ないですが、納品物などでは専用工具を使うか、専用工具を使う業者に依頼するのが良いでしょう。

![20180405155323.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/8a5af2dc-6246-0b18-e993-3ab5d91ac207.jpeg)

私はこのケーブルを適当なブレッドボードに配線してタクトスイッチを押すと短絡（ショート）するようにしました。

![20180405155424.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/b8f0f4b7-1d26-49ab-b439-4af3aa994955.jpeg)

ブレッドボード側は被覆だけ剥いて、手持ちのサトーパーツのねじ無し端子台で済ませています。

[貫通型スクリューレス端子台 2極【ML-950-2P】](https://www.marutsu.co.jp/pc/i/20385/)

ブレッドボードを使わず、タクトスイッチにはんだ付けしてしまっても良いでしょう。

### ケーブルを作りたくない場合

以下のようなジャンプワイヤを用います。

[テストワイヤー メス-オス(5色、10本入り)【TTW-201】](https://www.marutsu.co.jp/pc/i/1003749/)

オス-メスのジャンプワイヤ：
![20180405155513.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/d614223c-ac7e-1ce5-aad6-a5469010c17e.jpeg)

かなり無理やりですが、EHR-2のピッチは2.5mmなのでわりとピッタリ入ります。

ジャンプワイヤでコネクタ代用：
![20180405155645.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/0454f0c6-24d0-8deb-41db-0b7bd21bd2bd.jpeg)

この際、コネクタの形状が同じで、CN3とCN4を間違えやすいので注意します。

配線は以上です。


上記で外部入力に接続したタクトスイッチをSW2(a.k.a. クローンの攻撃)と呼ぶことにします。

以下、raspbianの導入など準備は[前回の記事][前回の記事のURL]を参照ください。


# 本題

以上で配線した外部入力を用いて機能を試していきます。

## パワーマネジメントモジュールによる外部入力端子の監視を試す

SW1と同様に起動とシャットダウンが外部入力で可能です。

### 外部入力で起動

sudo poweroffでラズパイをシャットダウンしてから、SW2を押すとラズパイが起動します。

これを利用すれば、外部装置からラズパイの電源を投入できます。

### 外部入力でシャットダウン

ラズパイが起動した状態で以下のコマンドを実行します。

```bash:コンソール
pi@raspberrypi:~ $ sleepi2ctl -s extin-powerdown 1
pi@raspberrypi:~ $ sleepi2ctl -g extin-powerdown
1
```

この状態でSW2を10秒間長押しするとラズパイがシャットダウンします。

SW1またはSW2を押してラズパイを起動した後で、以下のコマンドで確認するとゼロに戻っています。

```bash:コンソール
pi@raspberrypi:~ $ sleepi2ctl -g extin-powerdown
0
```

これを永続化するには、以下のファイルを編集します。

```bash:/etc/default/sleepi2
・・・
#
# Forced shutdown by external input
#
# EXTIN_FORCED_SHUTDOWN=0 : disabled
# EXTIN_FORCED_SHUTDOWN=1 : external input has been detected 10s and shutdown
#
# EXTIN_FORCED_SHUTDOWN=0
EXTIN_FORCED_SHUTDOWN=1
・・・
```

上記ファイルの説明は、以下の公式ドキュメントにあります。

[githubのmechatrax/sleepi2-utils](https://github.com/mechatrax/sleepi2-utils)

sudo rebootしてから以下のコマンドで確認します。

```bash:コンソール
pi@raspberrypi:~ $ sleepi2ctl -g extin-powerdown
1
```

以上で、外部入力によって[slee-Pi 2 Plus][sleepi2のURL]経由でラズパイの電源を操作できることが確認できました。

## sleepi2monによる外部入力端子の監視を試す

メカトラックスさんが提供しているパッケージには監視デーモンが含まれていますのでこれも試してみます。
監視デーモンにより、外部入力をトリガとして任意のコマンドを実行させることができます。

公式ドキュメントは[github](https://github.com/mechatrax/sleepi2-monitor)にあります。

### 外部入力でシャットダウン

デフォルトではコマンド実行は無効となっていますので、以下のように設定ファイルを変更してからラズパイを再起動して設定を反映します。

```bash:/etc/sleepi2-monitor.conf
・・・
[extin]
command = "shutdown -h now"
oneshot = true
#threshold = 0      # 0:無効
threshold = 3       # 3秒
・・・
```

```bash:コンソール
$ sudo reboot
```

ラズパイが起動してからSW2を3秒ほど長押しすると、[slee-Pi 2 Plus][sleepi2のURL]の赤LEDが点滅してから、ラズパイがシャットダウンします。

### 外部入力で再起動

次に、SW2長押し時の動作を変更します。

今はシャットダウンした状態のはずですので、[slee-Pi 2 Plus][sleepi2のURL]のSW1を押してラズパイを起動します。設定ファイルを以下のように変更してからラズパイを再起動して設定を反映します。

```bash:/etc/sleepi2-monitor.conf
・・・
[extin]
#command = "shutdown -h now"
command = "shutdown -r now"
oneshot = true
#threshold = 0      # 0:無効
threshold = 3       # 3秒
・・・
```

```bash:コンソール
$ sudo reboot
```

SW2を長押しすると、ラズパイが再起動します。

![20180405161043.png](https://qiita-image-store.s3.amazonaws.com/0/48424/d57daedd-24d9-5a9f-3bda-4c692fba64f0.png)

### 外部入力長押しで任意コマンド

ここまでは外部入力をトリガとして、shutdownコマンドを実行しました。
ここでは任意のスクリプトを実行させてみます。

まずシェルスクリプトを書きます。実行されたことがわかるようにdateコマンドを実行してテキストファイルに追記するようにします。

```bash:/home/pi/sleepi/cmd.sh
#!/bin/sh
echo `date` >> /home/pi/sleepi/sleepi.log
```

chmod 775 cmd.shのようなコマンドで実行フラグを付けておきます。

```bash:コンソール
pi@raspberrypi:~/sleepi $ ls -al cmd.sh
-rwxrwxr-x 1 pi pi 52 Apr  4 23:37 cmd.sh
```

ユーザpiのまま実行してみます。

```bash:コンソール
pi@raspberrypi:~/sleepi $ ./cmd.sh
pi@raspberrypi:~/sleepi $ cat sleepi.log
Wed 4 Apr 23:25:49 JST 2018
```

続いて、設定ファイルを以下のように変更します。

```bash:/etc/sleepi2-monitor.conf
・・・
[extin]
command = "/home/pi/sleepi/cmd.sh"
oneshot = false
threshold = 3
・・・
```

oneshotをfalseにすることで、外部入力がONの間、コマンドが繰り返し実行されるはずです。

sudo rebootでラズパイを再起動した後で、sleepi.logを監視しながら、ブレッドボードのSW2を長押しします。

```bash:コンソール
pi@raspberrypi:~/sleepi $ tail -f sleepi.log
・・・
Wed 4 Apr 23:38:04 JST 2018
Wed 4 Apr 23:38:05 JST 2018
Wed 4 Apr 23:38:06 JST 2018
・・・（ここでいったん指を離して再度押し続ける）
Wed 4 Apr 23:38:10 JST 2018
Wed 4 Apr 23:38:11 JST 2018
・・・
```

スレッショルドは3秒に設定していますので最初のログが出力されるまで3秒かかりますが、連続動作は1秒ごとに行われるようです。

外部装置からラズパイに動作をさせる指令をおこなうのに使えそうです。

# まとめ

[前回の記事][前回の記事のURL]から続いて、[slee-Pi 2 Plus][sleepi2のURL]のチュートリアル的な記事を書かせていただきました。

メカトラックスさん提供のパッケージを使えば、[slee-Pi 2 Plus][sleepi2のURL]の機能を簡単に利用することができます。

次の記事では、slee-Pi 2 Plusの応用編を書いてみようと思います。

まとめは以上です。この記事が、システム開発・試作のお役に立てれば幸いです。

最後までお読みいただきありがとうございました。

# 落穂ひろい

## 外部入力の制御回路

この記事では簡易的にタクトスイッチ（押しボタン）を使いました。

実際のシステムでは外部に回路があって、マイコンなどで制御する仕様もあると思われます。

その場合、絶縁が必要ならフォトカプラ、絶縁不要ならアナログスイッチでよいと思われます。

念のためON抵抗などが問題ないかはメカトラックスさんに確認したほうが良いでしょう。


# リンク

[メカトラックスさんのサイト][メカトラックスさんのURL]
[前回の記事][前回の記事のURL]

[slee-Pi][sleepi2のURL]
[3GPi Ver.2][3GPIのURL]

[slee-Piのセットアップ][slee-PiのセットアップのURL]
[sleepi2-monitorのマニュアル][sleepi2-monitorのURL]
[sleepi2-utilsのマニュアル][sleepi2-utilsのURL]

[メカトラックスさんのURL]:https://mechatrax.com/
[前回の記事のURL]:https://qiita.com/syasuda/items/dd1564896e75b03b2b7d
[3GPIのURL]:https://mechatrax.com/products/3gpi/
[sleepi2のURL]:https://mechatrax.com/products/slee-pi/
[slee-PiのセットアップのURL]:https://github.com/mechatrax/slee-pi2/wiki/%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2#12-%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB
[sleepi2-monitorのURL]:https://github.com/mechatrax/sleepi2-monitor
[sleepi2-utilsのURL]:https://github.com/mechatrax/sleepi2-utils