エンジニアにとって記事を書くということは日常茶飯事だと思うのですが、みなさまはどのように管理されているでしょうか。  
私はmarkdownエディタ（Obsidian）で下書きし、それをブラウザ上のエディタにコピペして投稿していました。  
特に不便に感じてはいませんでしたが、先日同僚のテックリードと雑談したところ、  
「自分はZenn cliを使っていて、自動で投稿されるようにしています。」  
「Github Actionsでtextlintを走らせて、自動で文章の校正をしています。」
というお話をいただき、なにそれイケてる!!と思いすぐにその環境を導入した話です。
# Zennとはてなブログの使い分け
技術的な話以外のことを書きたいときははてなブログを利用しています。  
そのためどうせならどちらも管理したいな..というのを目指しましたが、結局はてなブログに関してはvscodeの拡張に頼ることにしました。
# ZennをGithubで管理する
この環境でブログを投稿するまでの流れは下記の通りです。
```
1. ブランチを切る

2. コマンドでファイル作成

3. 記述

4. git add して commit して push

5. PRを作成するとtextlintが走り自動で文章チェックが入る

6. 通ればmainブランチにマージ

7. 自動でZennに投稿される
```
## Zenn CLIの導入とGithub連携
Zennには`Zenn CLI`が用意されており、ローカルで作成・編集・プレビューを行なうことができます。
さらにGithubと連携することでmainブランチ（設定可）に差分があると自動で投稿してくれます。  
[Zenn CLI](https://zenn.dev/zenn/articles/install-zenn-cli)  
[Github連携](https://zenn.dev/zenn/articles/connect-to-github)

コマンドでファイル名にハッシュを含む新しい記事を生成。
```
npx zenn new:article
```
Zennでどう見えるのかをローカル環境で確認できます。
```
npx zenn new:article
```

## Github Actionsでlinterを走らせる
次にlinterの設定をしていきます。
`textlint`というツールがあり、走らせることで書いた文章を自動でチェックしてくれます。  
細かく校正のルールを設定できるのですが、SmartHRがプリセットのnpmパッケージを出しており、なんかよさそうだ！と思ったのでこちらを使用しました。  
https://www.npmjs.com/package/textlint-rule-preset-smarthr

またtextlintは`textlint-rule-prh`という表記揺れを防ぐ設定を行なうこともできます。  
こちらもSmartHRで用意されているものを使用しました。  
https://github.com/kufu/textlint-rule-preset-smarthr/tree/main/dict

## reviewdogを使う
textlintの実行結果のログをみてその位置を追うのは辛いので、PRに自動でコメントがつくように`reviewdog`を使用しました。  
便利すぎ。  
https://github.com/reviewdog/reviewdog

## 実際に自動投稿してみる
まずはテストで「。」がない文章を投稿してみます。
すると
![reviewdog](https://cdn-ak.f.st-hatena.com/images/fotolife/s/shibuya01055/20220220/20220220234334.png)
しっかり自動でレビューが入りました。

これを修正してマージすると・・・
![自動投稿](https://cdn-ak.f.st-hatena.com/images/fotolife/s/shibuya01055/20220220/20220220234551.png)
投稿されている！
これは未公開の記事ですが、CLIで生成されるファイルのメタ情報に`published: true`を指定すると公開記事になります。
```
---
title: "github連携テスト"
emoji: "🐈"
type: "tech"
topics: []
published: false # <-これ
---
```


## vscodeの拡張
Zenn Editorという拡張も導入しました。  
書きながらとなりにプレビューを置けるので非常に便利です。
https://marketplace.visualstudio.com/items?itemName=negokaz.zenn-editor

# はてなブログを管理する
次にはてなブログです。  
githubと連携するには`push-to-hatenablog`を使うとよさそうでした。
https://github.com/mm0202/push-to-hatenablog  
ただ`Docker`を利用する必要がありそうで、そこまででもないんだよな〜という感じがあり、使用を断念。
## vscodeの拡張で投稿できる
`hatenablogger`という拡張があり、これを使うことでvscode上から投稿できるとのこと！
設定もお手軽なのでこちらを利用することにしました。
https://uraway.hatenablog.com/entry/2018/12/12/001545
## textlintをpre-commitで実行する
**(こちらの項で設定しているhuskyはreviewdogの良さを消してしまうため使用しないことにしました。もし使ってみたい場合は参考にしてください。)**

はてなブログの場合、Zennと投稿までの流れが変わります。
```
1. ブランチを切る

2. .mdファイル作成

3. 記述

4. git add して commit して push <- しなくてもいい

5. vscodeのコマンドパレットで投稿
```

vscode上で完結するため、「CIが通ったらマージして~」というフローがありません。
なんならgitで管理する意味を少し失いそうです。（笑）

そこでhuskyを使用し、commit時に自動でtextlintを走らせるようにしました。  
投稿するのは必ずcommitしてからという自分ルールを決めて・・・。  

パッケージをインストールします。
```
yarn add -D husky lint-staged
```
.huskyディレクトリ作成。
```
yarn husky install
```

package.json  
`articles/*.md`はzenn、`hatena/*.md`がはてなブログの記事です。
```json
 "scripts": {
   "lint": "textlint 'articles/*.md', 'hatena/*.md'",
   "lint:fix": "textlint --fix 'articles/*.md', 'hatena/*.md'",
   "prepare": "husky install"
 },
 "lint-staged": {
   "*.{md}": [
     "yarn lint"
     "yarn lint:fix"
   ]
 }
```

.husky/pre-commitを作成。
```
yarn husky add .husky/pre-commit "yarn lint-staged"
```

これでcommit時にtextlintが実行されるようになりました。  
ちなみにこの記事を書き終えたときに実行されたものです。  
めちゃくちゃ怒ってくれました。
```
yarn run v1.22.4
$ /Users/shibuya.kyohei/work2/zenn-docs/node_modules/.bin/lint-staged
✔ Preparing lint-staged...
⚠ Running tasks for staged files...
  ❯ package.json — 2 files
    ❯ *.md — 1 file
      ✖ yarn lint [FAILED]
↓ Skipped because of errors from tasks. [SKIPPED]
✔ Reverting to original state because of errors...
✔ Cleaning up temporary files...

✖ yarn lint:
error Command failed with exit code 1.
$ textlint 'articles/*.md', 'hatena/*.md' /Users/shibuya.kyohei/work2/zenn-docs/hatena/management_article_by_github.md

/Users/shibuya.kyohei/work2/zenn-docs/hatena/management_article_by_github.md
    3:38  error    文末が"。"で終わっていません。                                                                              smarthr/ja-no-mixed-period
    9:9   ✓ error  ！！ => !!                                                                                                  smarthr/prh-rules
   12:16  ✓ error  ひらがなで表記したほうが読みやすい形式名詞: 時 => とき                                                      smarthr/ja-keishikimeishi
   33:43  ✓ error  行う => 行なう
「行なう」を使用する                                                                                        smarthr/prh-rules
   38:30  error    文末が"。"で終わっていません。                                                                              smarthr/ja-no-mixed-period
   42:23  ✓ error  【dict2】 "することができます"は冗長な表現です。"することが"を省き簡潔な表現にすると文章が明瞭になります。
解説: https://github.com/textlint-ja/textlint-rule-ja-no-redundant-expression#dict2  smarthr/ja-no-redundant-expression
   53:41  error    【dict5】 "設定を行う"は冗長な表現です。"設定する"など簡潔な表現にすると文章が明瞭になります。
解説: https://github.com/textlint-ja/textlint-rule-ja-no-redundant-expression#dict5              smarthr/ja-no-redundant-expression
   53:44  ✓ error  行う => 行なう
「行なう」を使用する                                                                                        smarthr/prh-rules
   65:18  error    文末が"。"で終わっていません。                                                                              smarthr/ja-no-mixed-period
   94:44  ✓ error  【dict2】 "することができると"は冗長な表現です。"することが"を省き簡潔な表現にすると文章が明瞭になります。
解説: https://github.com/textlint-ja/textlint-rule-ja-no-redundant-expression#dict2  smarthr/ja-no-redundant-expression
  118:11  error    文末が"。"で終わっていません。                                                                              smarthr/ja-no-mixed-period
  122:14  error    文末が"。"で終わっていません。                                                                              smarthr/ja-no-mixed-period
  141:20  error    文末が"。"で終わっていません。                                                                              smarthr/ja-no-mixed-period

✖ 13 problems (13 errors, 0 warnings)
✓ 6 fixable problems.
Try to run: $ textlint --fix [file]

info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
husky - pre-commit hook exited with code 1 (error)
```

修正してcommitが通れば、あとはコマンドパレットから`Hatenablogger: Post or Update`を選択して投稿します。

ちなみにこのエラー文は好きなように変更でき、下記のようにするとさらに指摘箇所が見やすくなります。
```
textlint -f pretty-error 'articles/*.md', 'hatena/*.md'
```

## git管理する意味ある？という懸念
vscodeで完結するのでgit管理する意味がないように一瞬思いました。
ターミナルで`yarn lint`を実行すれば同じです。

ただ私が使用するデバイスが複数あり、同期したい気持ちもありました。
そういった意味だとgithubに置いておけばpullするだけで環境が同期できて便利になります。

# 振り返り
「記事を書く環境がなんかモダンぽい！」という自己満もあるのですが、総じて良かったなと感じたことが２点あります。
## 下書きという概念がなくなった
今まではObsidianで下書き -> WEBエディタにペースト、
チェックして細かい修正があればその場対応、投稿という流れでした。  
そのため下書きと実際の内容に差分が発生しており、あまりObsidianに記事を残す意味が感じられませんでした。  
今回の方法だとvscodeで修正したものが本番のものと同じになります。
Zennもはてなブログも、投稿・更新は自分の環境からになるのです。  
（Obsidianは別用途で今も使い続けています。）

## vscodeで書く理由ができた
markdownの記事はもちろん以前からvscodeで書けましたが、特にその理由はなく、いくつかのmarkdownツールを触ってきました。  
今回`Zenn Editor`、`hatenablogger`の存在がvscodeと他のツールを差別化する起因となり、ここに落ち着くことができました。

---
また改良を重ねるかもしれませんが、いい体験になることが期待できます。  
あとGithubに草も生えるしなんかいいよね。

# 追記
最終的にlinterの実行はhuskyを使わずにすべてreviewdogに任せることにしました。  
自動でレビューされた内容をvscodeの拡張、[GitHub Pull Requests and Issues](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github)を使って修正時に怒られている箇所を特定しやすくしています。
![スクリーンショット 2022-02-21 15.27.59.png](https://cdn-ak.f.st-hatena.com/images/fotolife/s/shibuya01055/20220221/20220221153925.png)