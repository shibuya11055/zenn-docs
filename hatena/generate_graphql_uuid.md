<!--
{"id":"13574176438087897577","title":"GraphQL Rubyで型を生成するとスキーマ上uuidであればちゃんとuuid型を参照しようとする","categories":["tech Rails GraphQL"],"updated":"2022-05-01T01:55:43+09:00","edited":"2022-05-01T01:58:22+09:00","draft":"no"}
-->

RailsでGraphQLを使っているのですが、Typeを自動生成するときに妙な問題に出くわした。

postgresqlは標準でuuid型という型を装備している。
https://www.postgresql.jp/document/9.4/html/datatype-uuid.html

で、マイグレーションファイルに下記のように書くだけでuuidを自動的に設定してくれる。
```ruby
t.uuid :uuid, null: false, default: 'gen_random_uuid()'
```

```ruby
# schemafile.rb
t.uuid "uuid", default: -> { "gen_random_uuid()" }, null: false
```

このとき、GraphQLの型を自動生成するコマンドを叩くと、uuidフィールドは次のようになる。
```
$ bundle exec rails g graphql:object モデル名
```
```ruby
field :uuid, Types::UuidType, null: false
```

おそらくuuid型はプリミティブではないので、uuid_type.rbを参照しにいこうとしている。
だがuuid型などないのでこのままだとエラーとなる。

試行錯誤したものの、結局String型にして落ち着かせた。
```ruby
field :uuid, String, null: false
```

uuid型があるものなら使いたいが、良い解決策があれば誰か教えてほしい...。
