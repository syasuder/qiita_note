<!--
title:   Node-RedでPiConsole I/F
tags:    3GPI,PiConsole,Python,RaspberryPi,node-red
id:      7b8f6dfa65e782fe4f58
private: true
-->
# はじめに

「PiConsole I/Fでカップ麺タイマー」(TODO:ハイパーリンク)の補足として、Node-RedからPiConsole I/Fを触ってみます。

Node-Redは最近のrasbian(jessie)などには最初から入っています。ここではJhonny-Fiveなどは利用せずに素のrasbianのままのNode-Redを使います。

PiConsole I/FのLピカ(SWあり)を短時間で気軽に試せます。

ただし、前の記事(TODO:)で接続や、端末からのアクセスなどは確立済みであることを前提としております。

（お願い）
わたくし自身はNode-Red初心者なので、トンチンカンなことを書いている部分もあるかと思いますが、コメントでやんわりと指摘していただけると助かります。

# 最初の一歩：Lピカ

ラズパイでNode-Redを利用する場合、

- ラズパイ自体をノードとして使う

というパターンもあるようです。Arduinoなどではこの方面がお盛んなようです。

この記事ではラズパイ上でNode-Redを起動して、ラズパイ自身にDeployします。

まずは起動です。

```shell-session:端末エミュレータ上
pi@raspberrypi:~/.node-red $ sudo node-red-start

Start Node-RED

Once Node-RED has started, point a browser at http://192.168.22.38:1880
On Pi Node-RED works better with the Iceweasel browser

Use   node-red-stop                          to stop Node-RED
Use   node-red-start                         to start Node-RED again
Use   sudo systemctl enable nodered.service  to autostart Node-RED at every boot
Use   sudo systemctl disable nodered.service to disable autostart on boot

To find more nodes and example flows - go to http://flows.nodered.org
You may also need to install and upgrade npm
      sudo apt-get install npm
      sudo npm i -g npm@2.x
（・・・）
```

ブラウザ上で「http://192.168.22.38:1880」にアクセスすると、Node-Redにアクセスできます。

・Node-redの画面例
![20161006141250.png](https://qiita-image-store.s3.amazonaws.com/0/48424/93e485c5-13ef-436e-ab71-9aeb202fddd1.png)

基本的な使い方としては、左側のパレットに並んでいる「ノード」をマウスのドラッグで真ん中のシートに配置して、角丸四角のポッチ同士をマウスドラッグでつないでやることで「フロー」を作成することができます。それから右上の「Deploy」ボタンを押せば、反映されます。

とはいいつつ、ラズパイではGPIO番号の混乱がありますので、とりま以下のフローをImportしていただくのが早いと思います。

Node-Redの右上の「≡」をクリックして「Import」＞「Clipboard」とたどり、以下のJSONをコピペして「OK」を押してみてください。

・Lピカサンプルフロー
> [{"id":"c94a4386.f4a82","type":"rpi-gpio in","z":"56ab294c.f317f8","name":"SW1 WHITE","pin":"35","intype":"in","read":false,"x":224,"y":92,"wires":[["193ced88.b7f2a2"]]},{"id":"193ced88.b7f2a2","type":"rpi-gpio out","z":"56ab294c.f317f8","name":"LED1 RED","pin":"38","set":"","level":"0","out":"out","x":563,"y":96,"wires":[]},{"id":"89fdc1ed.3864a","type":"rpi-gpio out","z":"56ab294c.f317f8","name":"LED2 YELLOW","pin":"40","set":"","level":"0","out":"out","x":546,"y":197,"wires":[]},{"id":"b792df36.d84958","type":"rpi-gpio in","z":"56ab294c.f317f8","name":"SW2 BLACK","pin":"36","intype":"in","read":false,"x":221,"y":196,"wires":[["71211442.0079cc"]]},{"id":"b2aada29.8df27","type":"rpi-gpio out","z":"56ab294c.f317f8","name":"BUZZER","pin":"22","set":"","level":"0","out":"out","x":543,"y":319,"wires":[]},{"id":"71211442.0079cc","type":"function","z":"56ab294c.f317f8","name":"反転","func":"msg.payload = ~msg.payload & 1;\nreturn msg;","outputs":1,"noerr":0,"x":387,"y":254,"wires":[["89fdc1ed.3864a","b2aada29.8df27"]]},{"id":"521341c2.1aa97","type":"comment","z":"56ab294c.f317f8","name":"Lピカ","info":"","x":133,"y":35,"wires":[]}]


するとマウスにフローがひっついて来ますので、適当な場所で左クリックでシートに貼り付けされます。以下のような画面になったらImport成功です。

・サンプルフローインポート例
![20161006142103.png](https://qiita-image-store.s3.amazonaws.com/0/48424/809f5db5-6c99-05d2-c8b0-1e243fbdacc0.png)

この状態で右上の「Deploy」をクリックしてから、SW1またはSW2を指で押してみてください。LEDがピカったり、ブザーが鳴ったりするかと思います。たまにDeployに失敗しますが、もう一度クリックすれば大抵は成功します。

ノードの設定についてさわりだけ説明しておきます。

・SW1 WHITEノードの設定
![20161006142315.png](https://qiita-image-store.s3.amazonaws.com/0/48424/7bd409ff-f829-894f-0fc7-c7b37c025be8.png)

このバージョンのNode-RedではBCMの番号で指定します。ごちゃごちゃ設定がありますが、とりあえずこのPinだけ設定すれば使えます。

以上がわたくしが考えた「たぶん最短のLピカ(SWあり)」です。

以下ではラズパイ上のNode-RedでI2Cデバイスを気軽に使う方法を試しますがこちらは最短でも最適でもありません。

# Node-RedからラズパイのI2Cデバイスを使う

詳細は落穂ひろいに書きますが、NodeまたはNode-RedからラズパイのI2Cデバイスを利用する方法には複数のルートがあります。

ここでは「PiConsole I/Fでカップ麺タイマー」(TODO:ハイパーリンク)で作成したPythonスクリプトを流用します。これならnpmの森に迷い込まなくても済みます。

・I2Cデバイスサンプルフロー
> [{"id":"83f7e056.6e7818","type":"exec","z":"56ab294c.f317f8","command":"/home/pi/sandbox/node_st7032i.py","addpay":true,"append":"","useSpawn":"","name":"ST7032py","x":388,"y":466,"wires":[["a391b4b7.46be1"],["ff4bbc1c.af4d08"],["13f87b40.c296e5"]]},{"id":"fb4f135.45e99f","type":"inject","z":"56ab294c.f317f8","name":"LCDテスト","topic":"","payload":"HOGE 4 1","payloadType":"string","repeat":"","crontab":"","once":false,"x":156,"y":463.5,"wires":[["83f7e056.6e7818"]]},{"id":"a391b4b7.46be1","type":"debug","z":"56ab294c.f317f8","name":"","active":true,"console":"false","complete":"false","x":635,"y":451,"wires":[]},{"id":"ff4bbc1c.af4d08","type":"debug","z":"56ab294c.f317f8","name":"","active":true,"console":"false","complete":"false","x":632,"y":514,"wires":[]},{"id":"13f87b40.c296e5","type":"debug","z":"56ab294c.f317f8","name":"","active":true,"console":"false","complete":"false","x":649,"y":581,"wires":[]},{"id":"2792295a.46a356","type":"exec","z":"56ab294c.f317f8","command":"/home/pi/sandbox/node_adt7410.py","addpay":false,"append":"","useSpawn":"","name":"ADT7410py","x":386,"y":617.5,"wires":[["34e77b98.ea2204"],["ea4002a2.03b1d8"],["9d979ce3.b9e38"]]},{"id":"b399f9d0.736c2","type":"inject","z":"56ab294c.f317f8","name":"温度取得","topic":"","payload":"","payloadType":"none","repeat":"","crontab":"","once":false,"x":160,"y":603.5,"wires":[["2792295a.46a356"]]},{"id":"34e77b98.ea2204","type":"debug","z":"56ab294c.f317f8","name":"","active":true,"console":"false","complete":"false","x":649,"y":660.5,"wires":[]},{"id":"ea4002a2.03b1d8","type":"debug","z":"56ab294c.f317f8","name":"","active":true,"console":"false","complete":"false","x":656,"y":726.5,"wires":[]},{"id":"9d979ce3.b9e38","type":"debug","z":"56ab294c.f317f8","name":"","active":true,"console":"false","complete":"false","x":673,"y":793.5,"wires":[]}]

・I2Cデバイスサンプルフローをインポートした画面例
![20161006143347.png](https://qiita-image-store.s3.amazonaws.com/0/48424/39370bee-c082-8f1f-c702-460e1488b0d7.png)

温度センサモジュールを使用しない場合は下半分は削除してかまいません。

Pythonスクリプトの呼び出しには、Node-RedのExecノードを利用しています。上記フローでは、スクリプトのパスを以下のように指定していますので、環境に合わせて変更しておきます。LCDモジュールは以下のように設定しています。

> /home/pi/sandbox/node_st7032i.py

・Execノードのパス設定
![20161006143420.png](https://qiita-image-store.s3.amazonaws.com/0/48424/9d29756f-a5b2-1d24-9e47-0e42ddbd319c.png)

温度センサモジュールは以下の設定です。

> /home/pi/sandbox/node_adt7410.py

肝心のスクリプトですが、前の記事で作成したものの薄いラッパになっています。

```py3:node_st7032i.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import sys

from st7032i import St7032iLCD as LCD
I2C_ADDR_LCD = 0x3e

if __name__ == '__main__':
    if len(sys.argv) < 4:
        print("Usage:", sys.argv[0], "<string> <x> <y>")
        sys.exit()

    msg = sys.argv[1]
    x = int(sys.argv[2])
    y = int(sys.argv[3])

    lcd = LCD(I2C_ADDR_LCD)

    # show characters in 2 lines
    lcd.set_cursor(x, y)
    lcd.print(msg)
```

```py3:node_adt7410.py
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

from adt7410 import ADT7410 as THERMO
I2C_ADDR_THERMO = 0x48

if __name__ == '__main__':
    thermo = THERMO(I2C_ADDR_THERMO)
    temp = thermo.read_temp() / 128.0
    print("{0:6.2f}".format(temp), end='')
```

先ほどのパス設定に合わせてこれらのスクリプトと(前の記事で作成したst7032i.pyとadt7410.py)を配置して、chmodで実行権限をつけておきます。

以上で準備が整いましたので、Node-Red上でDeployボタンをクリックしてから、Node-Red画面上で「LCDテスト」ノードの左側の四角をクリックします。

LCDモジュールに以下のように表示されます。

・LCDテスト表示
![PIC_20161006_144508.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/faea89c9-cd7d-8e3f-e331-a3664dc16a1b.jpeg)


温度取得の場合は、Debugペインに温度が表示されます。

・温度がDebugに表示された
![20161006144755.png](https://qiita-image-store.s3.amazonaws.com/0/48424/fc1844a1-a79c-dc72-a313-94960028cd25.png)


# ボタンを押すと液晶に現在の温度を表示するサンプル

前の記事のサンプル4をNode-Red上で構成してみます。フローは以下の通りです。

> [{"id":"a28607c2.f1e4e","type":"exec","z":"56ab294c.f317f8","command":"/home/pi/sandbox/node_st7032i.py","addpay":true,"append":"","useSpawn":"","name":"ST7032py","x":391,"y":1074.5,"wires":[["ec5e9054.bf77f"],[],[]]},{"id":"73dcbe7e.eeee98","type":"exec","z":"56ab294c.f317f8","command":"/home/pi/sandbox/node_adt7410.py","addpay":false,"append":"","useSpawn":"","name":"ADT7410py","x":395,"y":969,"wires":[["3c0d5920.f23ae6"],[],[]]},{"id":"3c0d5920.f23ae6","type":"function","z":"56ab294c.f317f8","name":"引数追加","func":"msg.payload = msg.payload + \" 0 0\";\nreturn msg;","outputs":1,"noerr":0,"x":237,"y":1070.5,"wires":[["a28607c2.f1e4e"]]},{"id":"ec5e9054.bf77f","type":"debug","z":"56ab294c.f317f8","name":"","active":true,"console":"false","complete":"false","x":662,"y":1075,"wires":[]},{"id":"1e482485.b5ec9b","type":"rpi-gpio in","z":"56ab294c.f317f8","name":"SW2 BLACK","pin":"36","intype":"in","read":false,"x":178,"y":967,"wires":[["73dcbe7e.eeee98"]]},{"id":"484f3b9c.00723c","type":"comment","z":"56ab294c.f317f8","name":"温度取得してLCDへ","info":"","x":195,"y":860,"wires":[]}]

・インポート済み
![20161006144658.png](https://qiita-image-store.s3.amazonaws.com/0/48424/ae5c9e86-0638-4a6e-e287-23b5604c3d62.png)

DeployしてSw2(黒)押下でLCDに温度が表示されます。

・比較的暑い部屋
![PIC_20161006_145206.jpg](https://qiita-image-store.s3.amazonaws.com/0/48424/d813e25f-0d17-406e-439c-9aa288bc5033.jpeg)


# 落穂ひろい

そもそもNode-Redを使ってみようと考えたのは、以下の記事(およびそれに触発されたであろう記事など)を見かけたからです。

[ラズパイのNode-RedでJhonny-Fiveを使う記事][ラズパイのNode-RedでJhonny-Fiveを使う記事のURL]

[Jhonny-Five][Jhonny-FiveのURL]は初見だったのでGoogle先生に少し訊いてみたところ、それなりに盛り上がっているプロジェクトのようです。

いろいろ試行錯誤した結果、この界隈を俯瞰すると以下のようになっているようです。

- Jhonny-FiveはNode系でありNode-Red前提ではない
- Jhonny-FiveをNode-Redに持ってきた人がいる([node-red-contrib-gpio][node-red-contrib-gpioのURL])
- Jhonny-Fiveのプラグインとして[raspi-io][raspi-ioのURL]がある

わたくしにはrasbianの素のNode-RedではI2Cが使えないように見えたので、以下を試しました。

- [node-red-contrib-gpio][node-red-contrib-gpioのURL]のnodebotでI2C
- Jhonny-FiveをNode-RedではなくNode環境で使ってI2C

どちらも動作しましたが現時点ではイマイチだったので、Node-RedからPythonスクリプトを呼び出すことにしました。イマイチな点として以下を挙げておきます。

- nodebotのノードとJhonny-Fiveのノードが共存できない
-- Jhonny-Fiveノードだけで押し切るならNode-Redを使う意味があまりない
- 素のNode環境ならPythonの方が「電池入り」なので直感的

Node-RedおよびJhonny-FiveのチュートリアルではLチカまでしか説明されていないものが非常に多く、上記に気付きにくいので注意が必要かもしれません。

イマイチさを確認したバージョンはおよそ以下の通りです。

```json:package.json
{
    "name": "raspi-io-piconsole",
    "version": "0.0.1",
    "private": true,
    "dependencies": {
	"johnny-five":"0.9.58",
	"raspi-io": "6.1.0",
	"sleep" : "4.0.0",
    },
    "scripts": {"start": "node app.js"}
}
```

```shell-session:端末エミュレータ上
3 Oct 22:08:31 - [info] Node-RED version: v0.14.6
3 Oct 22:08:31 - [info] Node.js  version: v4.6.0
```

ちなみにNode-Redでオリジナルノードを作成することも考えましたが、普通のGPIOと共存できない気がしたので試していません。

最後に所感です。JS/ESに免疫があるならNode環境でJhonny-Fiveを使うのが良い気がします。Python-Yellow的なものがあればもっとよい気がします。あるなら知りたいです。

# リンク
[メカトラックスさんのサイト][メカトラックスさんのURL]
[3GPI][3GPIのURL]
[ラズパイのNode-RedでJhonny-Fiveを使う記事][ラズパイのNode-RedでJhonny-Fiveを使う記事のURL]
[Jhonny-Five][Jhonny-FiveのURL]
[node-red-contrib-gpio][node-red-contrib-gpioのURL]
[raspi-io][raspi-ioのURL]

[メカトラックスさんのURL]:http://www.mechatrax.com/
[3GPIのURL]:http://www.mechatrax.com/products/3gpi
[ラズパイのNode-RedでJhonny-Fiveを使う記事のURL]:https://timchooblog.wordpress.com/2016/06/14/configuring-the-johnny-five-robotics-library-to-work-in-node-red/
[Jhonny-FiveのURL]:http://johnny-five.io/
[node-red-contrib-gpioのURL]:https://github.com/monteslu/node-red-contrib-gpio
[raspi-ioのURL]:https://github.com/nebrius/raspi-io