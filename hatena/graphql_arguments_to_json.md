<!--
{"id":"13574176438098123879","title":"GraphQL Mutationのテストにて、整数な引数が型エラーで怒られた","categories":["rails","   tech","   GraphQL"],"updated":"2022-06-02T01:17:13+09:00","edited":"2022-06-02T01:21:20+09:00","draft":"no"}
-->

RspecでGraphQLのmutationを叩いたときに問題が発生したのでメモ。

## mutationが謎に落ちた
下記のようなユーザー登録のmutationのspecを実行しようとした。  
（graphql-deviseのmutationをテスト）

```ruby
let!(:query) do
  <<~GRAPHQL
    mutation userRegister(
      $email: String!
      $password: String!
      $passwordConfirmation: String!
      $gender: Int!
    ) {
      userRegister (
        email: $email,
        password: $password,
        passwordConfirmation: $passwordConfirmation,
        gender: $gender,
      ) {
        credentials {
          accessToken
        }
        user {
          email
          gender
        }
      }
    }
  GRAPHQL
end

let!(:path) { '/graphql_auth' }
let!(:variables) { 
  { 
    email: 'test@example.com', 
    password: 'password1234!', 
    passwordConfirmation: 'password1234!',
    gender: 0,
  } 
}
let!(:params) { { query: query, variables: variables } }
let(:post_request) { post path, params: params }

before do
  post_request
end

it 'ユーザーが作成されている' do
  expect(User.count).to eq 1
end
```

だがテストが失敗する。。
```
Failure/Error: expect(Coach.count).to eq 1

  expected: 1
      got: 0

  (compared using ==)
# ./spec/requests/mutations/coach/register.rb:95:in `block (3 levels) in <top (required)>'

Finished in 40.95 seconds (files took 7.89 seconds to load)
1 example, 1 failure
```

エラーの詳細も出てこず、何が原因かよくわからなかった。

## gemの中を見にいく
同僚に相談したところ、gemの中でbinding.pryする方法を教えていただいた。  
docekrコンテナでvimを使えるようにした後`bundle open graphql_devise`で  
graphql-deviseのファイルを開き、  
`graphql_devise/graphql_controller.rb`に`binding.pry`を差し込む。

再びテストを実行すると止まってくれた。
```
    9: def auth
    10:   binding.pry
    11:   result = if params[:_json]
    12:     GraphqlDevise::Schema.multiplex(
    13:       params[:_json].map do |param|
    14:         { query: param[:query] }.merge(execute_params(param))
    15:       end
    16:     )
    17:   else
 => 18:     GraphqlDevise::Schema.execute(params[:query], **execute_params(params))
    19:   end
    20:
    21:   render json: result unless performed?
    22: end
```

ここで  
` GraphqlDevise::Schema.execute(params[:query], **execute_params(params))`  
を実行してみると次のようなエラーが出た。
```
[1] pry(#<GraphqlDevise::GraphqlController>)> GraphqlDevise::Schema.execute(params[:query], **execute_params(params)).to_h
=> {"errors"=>
  [{"message"=>"Variable $gender of type Int! was provided invalid value",
    "locations"=>[{"line"=>9, "column"=>3}],
    "extensions"=>
     {"value"=>"0", "problems"=>[{"path"=>[], "explanation"=>"Could not coerce value \"0\" to Int"}]}}]}
```

`$gender`は整数を期待しているが`"0"`という文字列が渡っているらしい。
困惑・・。

## パラメータをjsonにして解決した
調べていると[こちらのissue](https://github.com/rmosolgo/graphql-ruby/issues/2897)を発見。  
[コメント](https://github.com/rmosolgo/graphql-ruby/issues/2897#issuecomment-617838748)を読むとRailsが文字列としか認識できていないとのこと。  
またそれを回避するにはjsonにしてあげる必要があるようです。

```ruby
let!(:params) { { query: query, variables: variables.to_json } } # to_jsonを加えた
let(:post_request) { post path, params: params }
```

このようにしてmutationが正常に実行され、無事テストも通すことができました。