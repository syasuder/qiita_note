<!--
title:   3GPIで遠隔操作OAタップ～関連法規も調べ、問合せもしてみた
tags:    3GPI,RaspberryPi,raspbian
id:      f683a1c0fe9a7e51fb68
private: false
-->
# はじめに

[猫Piカメラ][猫PiカメラURL]につづき、[メカトラックスさん][メカトラックスさんのURL]にお借りしている3GPIを使って遠隔操作可能なOAタップを作ってみます。
インターネット経由でどこからでも電源切り入りが可能です。

全体構成は以下の様な感じです。

3GPI付きラズパイA+のGPIO(1点)でSSRを制御し、AC100Vを入り切りするOAタップ：
![20160702234638.png](https://qiita-image-store.s3.amazonaws.com/0/48424/4647716a-4881-7635-a2f4-49745f375cbb.png)


## 重要なお願い

この記事の内容はあくまで実験/試作であり、実運用や製品開発では安全に配慮して関連法規に従ってください。

本記事では実験的にOAタップを自作し、遠隔操作する手順を示します。
OAタップの自作には遠隔操作の有無にかかわらず感電や火災の危険が伴います。その危険性を理解できる方が、自己責任の下で参考にされるようにお願いします。

自作した機器を業務でお使いになる場合には設備を管理する部署などと相談して設置してください。自宅でコンセントがこげても笑い話ですが、事業所の場合は笑い話で済まない場合があります。

個人的に経験がないので具体的なアドバイスはできませんが、遠隔OAタップを製造/販売することを検討される向きは、本記事の末尾の「遠隔操作の規制について」を参照して、専門業者や検査機関などに相談していただきたいです。

# 準備

長々と前置き失礼致しました。それでは本題に入ります。まずはハードとソフトです。

## ハードウェア

今回使用した主な部品は以下のようなものです。

![20160702225334.png](https://qiita-image-store.s3.amazonaws.com/0/48424/5ce4eae1-b7d8-3b94-8ac1-4449f0865e6c.png)

- SSR
- トランジスタ(2SC1815-Y)
- ベース抵抗(2.2kΩ)x2個
- AC100Vの差し込み

写真に写っていないものとして、
- ラズパイ本体
- ブレッドボード
- 差し込み(コンセント・配線付き)
- Y端子
などを使用しました。

GPIOで制御する「100V開閉リレー」として、タンスの中に眠っていたSSRを使いました。今は亡き松下電器製です。ゼロクロス対応です。底面がすべて放熱板になっており10Aまで流せます。
![20160701230903.png](https://qiita-image-store.s3.amazonaws.com/0/48424/cae3be1f-88b9-1376-7645-c1794835bf14.png)

SSRのINPUT側に4V以上の電圧をかければ負荷側がONしますが、ラズパイのGPIO直結では微妙に電圧が足りなさそうなので、2SC1815をかませす。

回路図は以下のような感じです。

SSR制御回路図：
![PI-SSR.PNG](https://qiita-image-store.s3.amazonaws.com/0/48424/856aedc4-f64f-0562-9f33-2a32f76ef99d.png)

SSRにはリレーのシンボルを使っています。
AC側の接点は1式なので単相100Vの片切りになります。SSRがオフでも100Vの配線の片方はつながったままということです。
SSRのINPUT側はデータシートによると端子間1.5kΩとのことなので、R2,R3は10kΩくらいで充分と思います。手持ちがなかったので2.2kΩにしています。
1815も手持ちのものですが小信号用ならなんでも構いません。抵抗分圧だけで済ませても実験なら問題ないでしょうし、ロジック変換ボードなどを使う方が安心な向きもあるかもしれません。

小信号部分はブレッドボード上に組みました。

1815のベース側に3.3Vを印加してONすることを確認：
![20160701231145.png](https://qiita-image-store.s3.amazonaws.com/0/48424/742f8050-6d57-74da-7775-cf06c51bee1e.png)

3.3Vと5Vはラズパイから引っ張り出しています。1815のベースに3.3Vを印加してSSRがONしたということは、とりあえずスイッチとして動作しているということで進めます。

100V配線部分はY端子で処理してみました。電線は0.75sqくらいのものです。

![20160701231438.png](https://qiita-image-store.s3.amazonaws.com/0/48424/39c01ff3-7621-2377-f7f0-09d27e1107b0.png)

## ソフトウェア

wiringPiでGPIOをONします。GPIO21を使うことにして、以下のようなコマンドで出力に設定します。

```shell-session:ラズパイ上
pi@raspberrypi:~ $ gpio -g mode 21 out
```

1を出力してSSRのパイロットランプが点灯することを確認しておきます。

```shell-session:ラズパイ上
pi@raspberrypi:~ $ gpio -g write 21 1
```

## NAT越え

インターネット上にサーバーを用意できるなら、ラズパイからポーリングさせるのも悪くないです。そのような仕組みのサービスもIoT向けと銘打って今どきはよりどりみどりのようです。

ここでは今後の3GPIのデバッグにも役立ちそうなポートフォワーディングで済ませます。参考にさせていただいたのは以下の記事です。

[Qiita:異なるprivateネットワーク内の端末をsshで繋ぐ][ポートフォワードの記事URL]

イメージとしては、3GPIでインターネットに接続したラズパイに対して、インターネット経由でssh接続できる感じで大変便利です。

さて、ラズパイの3G接続サービスがグローバルアドレスを払い出す場合話は簡単です。
sshでもVNCでもインターネット側からグローバルアドレスを指定して接続すればよいのです。その場合ポートフォワードは不要です。

遠隔操作する側のマシン(遠隔コントローラ)がグローバルアドレスで待ち受けできる場合は、
以下のようにラズパイからフォワーディングしておき、localhostに接続すればラズパイにつながります。

```shell-session:ラズパイ上
pi@raspberrypi:~ $ ssh -f -N -R 10022:localhost:22 user@(遠隔コントローラ)
```

```shell-session:(遠隔コントローラ)上
user@(遠隔コントローラ):~ $ ssh -p 10022 pi@localhost
pi@localhost's password:
pi@raspberrypi:~ $
```

遠隔操作する側のマシン(遠隔コントローラ)がNATの中にあって、どうにもならない場合でもインターネット上に踏み台サーバを用意できればなんとかなります。

ちょっと整理しておきます。
・遠隔操作する側のマシン(遠隔コントローラ)
・中継サーバ(踏み台サーバ)→10022で待ち受け
・遠隔操作されるラズパイ(遠隔コントローリ)

ラズパイで実行するsshコマンドは先ほどの場合と同じで、ホストを踏み台サーバに変更するだけです。

```shell-session:ラズパイ上
pi@raspberrypi:~ $ ssh -f -N -R 10022:localhost:22 user@jumphost
pi@raspberrypi:~ $
```

次に遠隔操作する側のマシン(遠隔コントローラ)から踏み台サーバにsshで接続します。

```shell-session:(遠隔コントローラ)上
user@(遠隔コントローラ):~$ ssh user@jumphost
```

踏み台サーバからラズパイに接続します。

```shell-session:(遠隔コントローラ)から接続している踏み台サーバ上
user@(踏み台サーバ):~ $ ssh -p 10022 pi@localhost
pi@localhost's password:
pi@raspberrypi:~ $
```

このようにNAT内のラズパイに対して別のNAT内のマシンからsshで接続することができます。
スイッチやルータを遠隔操作したいような方々の方が、この辺りに詳しいような気もしますので釈迦に説法になっているかもしれません。

どうしてVPNを使わないのか、という指摘もあるかと思いますがVPNではインタフェースが追加されるのでどうしてもルーティング設定の問題が出てきます。適材適所で使い分ければよいと思います。

# 実験

遠隔操作OAタップの先に"アンマネージド"なスイッチングハブを接続し、3G経由でラズパイからON/OFFしました。
この手のハブはどういうわけかときどき再起動してやらないと気絶しますので、すぐにでも実運用したいくらいです。（笑

実験台のスイッチングハブの上にブレッドボード：
![20160702185346.png](https://qiita-image-store.s3.amazonaws.com/0/48424/f58bb124-e446-22aa-bff9-e4ecf3c822bd.png)

まずOFF→ONの場合です。
SSRのパイロットランプ(赤)が点灯します。
![20160702185419.png](https://qiita-image-store.s3.amazonaws.com/0/48424/9b527966-964f-9609-fbc0-2b14bacee69c.png)

一瞬遅れてスイッチのPowerランプ(緑)が点灯します。
![20160702185440.png](https://qiita-image-store.s3.amazonaws.com/0/48424/82bd8ad6-40b4-702f-8c20-fb5d70a610e6.png)

次にON→OFFです。
やはり一瞬早くパイロットランプが消灯します。
![20160702185503.png](https://qiita-image-store.s3.amazonaws.com/0/48424/3c0c8353-2eed-7ce2-d77b-14d839880c15.png)

遅れてスイッチのランプが消灯します。
![20160702185527.png](https://qiita-image-store.s3.amazonaws.com/0/48424/d677a3fa-5207-5f2f-d87e-6c4be81d0e54.png)

# 改良すべき点

今回試作した回路では、ラズパイのGPIO出力がONのときしか電源が入らないため、ラズパイが落ちるとAC電源も落ちてしまいます。用途によってはSSRの制御接点をb接点制御(通常閉)とするように回路を変更したほうが良いかもしれません。

正常にAC100Vが供給されているかどうかをモニタする機能も欲しいところです。おおもとのコンセントが抜けたり死んでる場合、誰かが現地へ行かないといけませんので(その場合ラズパイも動けないかもしれませんが)。100V駆動のメカリレーの接点をGPIOに入れれば済みますが、手持ちがなかったので今回は見送りました。

また、AC100Vの負荷側にフューズやサーキットプロテクタ(CP)を入れるべきです。CPは小さいブレーカのようなものです。常時運用するならさらに、きちんとした箱にシステム全体を格納するべきでしょう。

# 今後の発展

マネージドなネットワーク機器の場合、管理用のシリアルポートが付いていたりしますので、ラズパイのUARTと接続すれば便利かもしれません。3G経由でルータやサーバのシリアルコンソールにログインしたい人は結構いると思います。

# 落穂ひろい

## 遠隔操作の規制について

数年前、宅外から遠隔操作可能な家庭用エアコンの販売について規制当局から物言いがついたことがありました。大手メーカー製だったので大きく報道され、ご記憶の方もおられると思います。

その後、規制の一部が改正されたようです。

[HEMS、この1年――家電の遠隔操作を阻む法規制が改正に][遠隔操作法規制改正のURL]
> 電安法の技術基準には「危険が生ずるおそれのないもの」であれば遠隔操作が許されるという項目がある。従来の基準では、「一部の機器を対象とした音声による遠隔操作」のみがこれに該当した。今回の改正ではこの除外項目の範囲が拡大され、一定の条件を満たせば「危険が生ずるおそれのないもの」として遠隔操作機能を搭載してよいことになった。

"一定の条件を満たせば"という手綱は残っていることに注意が必要です。

さて、その後も徐々に緩和方向へ改正が続けられていたようですが、今年の3月からはいわゆる遠隔操作OAタップ(マルチタップ)について緩和されたようです。

[経産省-電安法-遠隔操作可能な配線器具の範囲拡大について][遠隔操作範囲拡大URL]
>現行の電気用品安全法の技術基準の運用では、配線器具の遠隔操作について、負荷機器が特定できる場合に限り認められており、エンドユーザーが負荷機器を自由に選択できる配線器具は、火災等のリスクを伴う機器がつながれるおそれがあり認められていませんでした。

データセンターのネットワークスイッチやルータをいちいち人の手で電源バチ切りしているとしたら「IoTバンザイ」と皮肉を言いたくなりますね。

この記事を書いている途中で知ったのですが、この遠隔操作OAタップの規制緩和はcerevoさんのおかげかもしれません。以下のニュースリリースを見る限り時期も一致しているような気がします。

[Cerevo、遠隔操作電源タップ「OTTO」の販売再開][Cerevoさん遠隔操作電源タップ販売再開ニュースのURL]
> 同社では2015年4月、電気用品安全法上適法な製品であると判断しOTTOの販売を開始した。その後、経済産業省電気用品安全課より同製品について「電気用品安全法第8条1項のリスク評価を行い、通信回線による遠隔操作の安全性が確認されない限り、違反となる可能性がある」との指摘を受け、同年8月から国内販売を自粛していた。

平成26年(2014年)の報告書を読み違えてしまったのかと推測しますが、果たして・・・

> 参考）「解釈別表第四に係わる遠隔操作」に関する報告書の追加検討報告書

上記お知らせで参考に挙げられている追加報告書が以下で公開されています。

[「解釈別表第四に係わる遠隔操作」に関する報告書の追加検討報告書][追加報告書URL]
> ＜検討対象＞
> 「コンセント」、「延長コードセット」、「コードリール」、「マルチタップ」、「コードコ
> ネクターボデイ」及び「アダプター」に区分されるもの。

重要なのは上記の「対象品名」です。以下の表から、遠隔OAタップが「特定電気用品」として扱えばよいことがわかるからです。紛らわしいので「遠隔マルチタップ」と呼称すべきかもしれません。

[特定電気用品の一覧があるページ][JET-電気用品安全法の概要URL]

さて、「解釈別表第四」の「解釈」とは電気安全法(PSEマーク)の平成25年以降の「新解釈」のことです。この新解釈については日本電気協会がまとめています。

[電気用品の技術基準の解説−][電気用品の技術基準の解説のURL]

これは以下の様にかなり分厚い本です。

水色の第14版が最新：
![20160701132356.png](https://qiita-image-store.s3.amazonaws.com/0/48424/bec36635-ab15-74fa-691f-76ca56a32bd4.png)

1302ページあるので自立する：
![20160701132414.png](https://qiita-image-store.s3.amazonaws.com/0/48424/7c65da66-bd62-5660-3c29-2a65f8c344fe.png)

この中の「別表第四」が以下の部分です。

解釈別表第四に係わる遠隔操作に関する報告書:
![20160701143159.png](https://qiita-image-store.s3.amazonaws.com/0/48424/536f02e8-5727-6f2f-73ee-5be738d84ec1.png)
> 平成26年3月12日

追加報告書の日付は以下のようになっていました。
> 平成28 年3 月22 日

2年かけて緩和されたようです。
元の報告書だけを読むと、「別表第八と同様に緩和」という字面ですが、よく読むと「できない理由の羅列」となっています。
一方追加報告書では「緩和に必要な条件」がまとめられています。緩和ありきで書かれたのでしょうか。
ちなみに別表第八の事例にエアコンの遠隔操作が書かれています。

まとめますと、遠隔操作の緩和部分について概略を知るには、以下の2点セットが必要ということになります。

・[電気用品の技術基準の解説−][電気用品の技術基準の解説のURL]を9,000円弱で購入し、「解釈別表第四に係わる遠隔操作に関する報告書」に目を通す。

・[「解釈別表第四に係わる遠隔操作」に関する報告書の追加検討報告書][追加報告書URL]に目を通す

深堀するならそれぞれが参照する項目を追いかける必要がありますが、電話で経産省の担当者が親切にも教えてくれたのは、「解釈別表第四に係わる遠隔操作に関する報告書」から引用されている以下のハンドブックです。

[リスクアセスメントのハンドブック][ハンドブックのURL]

実務としては上記で公開されているハンドブックに従えばよいということのようです。
そうかといって私のようなPSE素人が勝手に検査しても後で困るでしょうけど。

PSE慣れしている業者に丸投げして必要な書類を作ってもらうのが近道なのかもしれません。
JETにも電話してみましたが、「イチゲンさんお断り」の印象でした。具体的には「品名が正確に分からないと、どうにもなりません。」の一点張りでした。つまり新製品開発をした場合は、門前払いを食らいそうだということです。いまどきはクルマのユーザー車検でも車検場の人は官でも民でももう少し丁寧に応対してくれる気がします。もう少しなんとかならないものかという感想です。

一方、最初に電話した経産省の担当者はフランクに教えてくれました。この場をお借りしてお礼申し上げます。

とにかく遠隔操作可能なOAタップについては規制が緩和されたのですから、今後「スマホで操作可能」という類の製品がじゃんじゃん開発されることを期待しています。

この記事がそういう開発のきっかけになればなおうれしいです。

# リンク
[メカトラックスさんのサイト][メカトラックスさんのURL]

[ラズパイと3GPIで猫Piカメラ][猫PiカメラURL]

[Cerevo、遠隔操作電源タップ「OTTO」の販売再開][Cerevoさん遠隔操作電源タップ販売再開ニュースのURL]

[経産省-電安法-遠隔操作可能な配線器具の範囲拡大について][遠隔操作範囲拡大URL]

[「解釈別表第四に係わる遠隔操作」に関する報告書の追加検討報告書][追加報告書URL]

[JET-電気用品安全法の概要URL][JET-電気用品安全法の概要URL]

[リスクアセスメントのハンドブック][ハンドブックのURL]

[Qiita:異なるprivateネットワーク内の端末をsshで繋ぐ][ポートフォワードの記事URL]

[HEMS、この1年――家電の遠隔操作を阻む法規制が改正に][遠隔操作法規制改正のURL]

[電気用品の技術基準の解説−][電気用品の技術基準の解説のURL]

[Cerevoさん遠隔操作電源タップ販売再開ニュースのURL]:https://fabcross.jp/news/2016/20160628_cerevo_otto.html

[メカトラックスさんのURL]:http://www.mechatrax.com/

[猫PiカメラURL]:http://qiita.com/syasuda/items/5def33db425aaf05bf6d

[遠隔操作範囲拡大URL]:http://www.meti.go.jp/policy/consumer/seian/denan/topics.html#8 "経産省-電安法-遠隔操作可能な配線器具の範囲拡大について"

[追加報告書URL]:http://www.eam-rc.jp/pdf/result/remote_control_4_2.pdf "「解釈別表第四に係わる遠隔操作」に関する報告書の追加検討報告書"

[電気用品の技術基準の解説のURL]:http://www.denki.or.jp/pub/cat/new/y0220.html "電気用品の技術基準の解説−"

[JET-電気用品安全法の概要URL]:http://www.jet.or.jp/law/pse/summary.html

[ハンドブックのURL]:http://www.meti.go.jp/product_safety/recall/risk_assessment.html

[遠隔操作法規制改正のURL]:http://techon.nikkeibp.co.jp/article/FEATURE/20131227/325301/?rt=nocnt "HEMS、この1年――家電の遠隔操作を阻む法規制が改正に"

[ポートフォワードの記事URL]:http://qiita.com/FGtatsuro/items/e2767fa041c96a2bae1f