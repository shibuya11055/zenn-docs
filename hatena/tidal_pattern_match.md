<!--
{"id":"13574176438072323536","title":"【Tidal Cycles・Haskell】パターンマッチの記法に気づけなかった","categories":["tech"," Haskell"," Tidal Cycles"],"updated":"2022-03-13T01:29:06+09:00","edited":"2022-03-13T01:29:48+09:00","draft":"yes"}
-->

最近[こちらの書籍](https://www.amazon.co.jp/%E6%BC%94%E5%A5%8F%E3%81%99%E3%82%8B%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E3%80%81%E3%83%A9%E3%82%A4%E3%83%96%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E3%81%AE%E6%80%9D%E6%83%B3%E3%81%A8%E5%AE%9F%E8%B7%B5-%E2%80%95Show-Us-Your-Screens-ebook/dp/B07L8ZJMNS/ref=sr_1_5?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=MF940PHV6T4K&keywords=%E3%82%AF%E3%83%AA%E3%82%A8%E3%82%A4%E3%83%86%E3%82%A3%E3%83%96%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0&qid=1647100683&sprefix=%E3%82%AF%E3%83%AA%E3%82%A8%E3%82%A4%E3%83%86%E3%82%A3%E3%83%96%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%2Caps%2C281&sr=8-5)を読みながらTidal Cyclesで遊んでいます。

その中で謎に思えた記法があったのでメモ。

# Tidal Cyclesとは
> TidalCyclesは、音楽の即興演奏と作曲のために設計されたライブコーディング環境です。特に、Haskellに埋め込まれたドメイン固有言語であり、可聴パターンまたは視覚パターンの生成と操作に重点を置いています。 
> （ウィキペディアより）

# 理解できなかった記法
Tidal CyclesはHaskellを利用するのですが、下記のコードが不思議でした。
```haskell
do
  let 
    inverse 1 = 0
    inverse 0 = 1
    pat1 = "{1 0 0 1 0 1 0 1/2 1 0 0 1/3}%8"
  d1 
    $ gain pat1 # s "bd"
  d2
    $ gain (inverse <$> pat1) 
    # sound "cp"
```

`inverse 0` と `inverse 1`という変数を定義して〜（ここから間違ってる）。  
d1ではキックをならし、d2でクラップを鳴らす。  
`$ gain (inverse <$> pat1)`で、pat1の値を渡して。。  
????

```haskell
let
  inverse 1 = 0
  inverse 0 = 1
・
・
・
$ gain (inverse <$> pat1)
```
この挙動がよく分かりませんでした。
本の中では「patが0のときは１を、１のときは0を返すのでbdが鳴っていないときにcpが鳴る」的なことが書かれていたのですが、理屈が分からず困惑しました。

そもそも変数のような書き方じゃないぞこれは。
```haskell
let
  inverse 1 = 0
  inverse 0 = 1
```

# fmap
調べると`<$>`はfmapと呼ばれ、ここではpat1を回して前述の`inverse`に引数を渡すような動きになる。
```haskell
$ gain (inverse <$> pat1)
```

ということはinverseは変数ではない、関数か。  
（後から本を読み返したらちゃんと「関数」と書いてありました・・。）  

# パターンマッチだった
https://www.tohoho-web.com/ex/haskell.html#pattern_match

答えはパターンマッチ！ということでした。  
分からなかった！  
```haskell
let
  -- 1を受け取ったら0を返す
  inverse 1 = 0
  -- 0を受け取ったら1を返す
  inverse 0 = 1
```

にしてもRubyの場合３系からパターンマッチが導入されたりと触れる機会が少ないと感じていたのですが、どうやら関数型言語だとよくあるそうです。  
普段はオブジェクト指向の言語を触っているせいか、関数型言語は難しく感じるな〜〜〜。  

おわり。