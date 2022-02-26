<!--
{"id":"13574176438067499143","title":"【Rails】deviseで用意したログインAPIを叩いた際、見知らぬパラメータが渡っていた","categories":["tech"," rails"],"updated":"2022-02-27T01:37:01+09:00","edited":"2022-02-27T01:37:21+09:00","draft":"no"}
-->

RailsにてGem deviseで認証機能を実装した際、ログイン時の`post`で謎のパラメータが渡る問題に直面。  
その解決法をメモします。

# 謎のパラメータが渡る
ログインするためのAPIを叩いたところ、エラーがかえってきました。
```
Processing by DeviseTokenAuth::SessionsController#create as HTML
  Parameters: {"email"=>"test1@example.com", "password"=>"[FILTERED]", "session"=>{"email"=>"test1@example.com", "password"=>"[FILTERED]"}}
Unpermitted parameter: :session
```
原因は`Unpermitted parameter: :session`にあると。  
しかしAPIを叩く際に`sessison`という名のパラメータは渡していません・・・。

# Railsの仕様だった
こちらの記事に答えが書いてありました。
[paramsとwrap_parameters](https://qiita.com/kazutosato/items/fbaa2fc0443611c627fc)


> コントローラのparamsには、"user"というキーが自動的に付きます（値がラップされます）。これが、ActionController::ParamsWrapperの機能です。

ひえーなるほど。  
つまり今回は`SessionsController`なので、`session`でラップされたパラメータも自動で用意されていたというわけです。  
脱線しますがストロングパラメータを下記のように書けるのはこの機能のおかげだったのですね。
```ruby
def user_params
  params.require(:user).permit(:name, :email)
end
```

# 解決方法
## 独自のSessionsControllerを用意する
deviseのSessionsControllerを継承した独自のコントローラーを用意。  
その際に`wrap_parameters format: []`を差し込むことで、パラメータに`session`が入らなくなります。
```ruby
class Api::V1::Auth::SessionsController < DeviseTokenAuth::SessionsController
  wrap_parameters format: []
end
```

## ルーティングの修正
先ほど作成したコントローラーが呼ばれるように、`routes.rb`を修正。
```ruby
Rails.application.routes.draw do
  mount_devise_token_auth_for 'User', at: 'api/v1/auth', controllers: {
    registrations: 'api/v1/auth/registrations',
    sessions: 'api/v1/auth/sessions' #追加
  }
end
```

これで`session`というパラメータが入ることなく、`email`と`password`でログインできました。
