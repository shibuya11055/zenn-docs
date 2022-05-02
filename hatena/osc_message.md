<!--
{"id":"13574176438088621422","title":"OSCメッセージを利用したオーディオリアクティブを作ってみたい","categories":["tech","   SuperCollider","   TidalCycles","   Processing"],"updated":"2022-05-03T01:59:34+09:00","edited":"2022-05-03T02:14:20+09:00","draft":"no"}
-->

私は今まで音量のみでオーディオリアクティブのような映像を作成していました。  
最近だとこういうの。

[https://twitter.com/TENTEN11055/status/1515545356477480960:embed]

ですが映像の幅に頭を悩ませており、というのも音量の情報だけでは表現に限界があります。

で、barbe_generative_diaryさんがツイートされていたご本人の記事で  
**SuperColliderが吐くOSCメッセージをProcessingに渡してあげる手順**が細かく記載されており、「これだ〜！」と思いました。  
[https://twitter.com/b_generative_d/status/1518748884109266945:embed]
この記事で世の中のオーディオリアクティブクリエイターはOSCを利用していたのか〜（多分）と一歩前身した気分に。

## いざ
もろもろのセットアップを済ませ、SuperColliderから音を出し、いざProcessingを実行！
```
### [2022/5/3 1:36:14] PROCESS @ OscP5 stopped.
### [2022/5/3 1:36:14] PROCESS @ UdpClient.openSocket udp socket initialized.
### [2022/5/3 1:36:15] ERROR @ UdpServer.start()  IOException, couldnt create new DatagramSocket @ port 2020 java.net.BindException: Address already in use
### [2022/5/3 1:36:16] INFO @ OscP5 is running. you (127.0.0.1) are listening @ port 2020
```
2020番がすでに使用されている？  
試しにOSC_Data_Monitorから2020番を外してみたところ正常にOSCメッセージの内容が表示されました！  
他ツールで使用していると競合してしまうのでしょうか。
```
-OscMessage----------
received from	/127.0.0.1:57120
addrpattern	/dirt/play
typetag	sssfsfsfsfsfsiss
[0] _id_
[1] 3
[2] cps
[3] 0.5625
[4] cutoff
[5] 800.0
[6] cycle
[7] 1232.0
[8] delta
[9] 1.7777777
[10] n
[11] 16.0
[12] orbit
[13] 2
[14] s
[15] jungbass

---------------------

```

これで１サンプルごとの細かい情報を得ることに成功！  
情報がたくさんでやれることもたくさんなので、まず音名（[15]）での分岐を試してみたいと思います。