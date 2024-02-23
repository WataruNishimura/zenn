---
title: "Postmanを使ってGraphQL経由でGitHub APIからGitHubを操作してみる"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github","postman","graphql"]
published: true
---

## はじめに

札幌でフロントエンドエンジニアをやりつつ、札幌市のエンジニア育成事業の運営をしている [n13u/NishimuraWataru](https://x.com/_n13u_) です。

GitHub CLIのextension開発をしている際にGitHubにはリポジトリやissue情報を取得するためのAPIがあり、それらがGraphQLでアクセス可能なことを知りました。
extension開発をするうえでAPIの利用が必須になってくるので、いざAPIを触ってみよおうと思ったところGitHub CLIの`gh api graphql`を利用した記事が多くCLIからのアクセスはやや面倒となり、Postmanを使ってみようと至った話です。

### GitHub CLI extension

GitHub CLI extensionはGitHub CLIの拡張機能としてユーザー自身が独自に機能追加し、開発できる外部ライブラリです。scaffoldのツールも充実しており、Golang以外での開発もできる代物です（GitHub CLI本体はGolangで開発されています）

GitHub CLI の開発については以下記事を参考に取り組んでいます。

[GitHub CLI 拡張機能の作成 - GitHub Docs](GitHub CLI 拡張機能の作成 - GitHub Docs)
[GitHub Projects(ProjectV2)にIssueやPullRequestを追加するGitHub CLIの拡張を作った](https://swfz.hatenablog.com/entry/2022/12/13/045749)

## Postman について

公式の説明によると、API（この場合は特にWeb APIを指す）開発・利用のためのAPIプラットフォームとのことです。もともとはデスクトップ上でHTTPでのAPIをテストできるGUI curl的なものだった記憶ですが、現在はPostman flowsといったAPI間の接続をローコードで記述し自動化できるサービスや、Postman自体のMQTT/GraphQL対応によりプラットフォームとして進化してきているようです。

2023年12月に開催されていたPostman Conferenceでも似たような話がされていました（参加レポ書いていない...）

[What is Postman](https://www.postman.com/)

## GitHub API を試してみる

Postmanのインストールや設定は割愛します。

### Collectionを作成する

作成したQueryの保存やGraphQL Schemeの管理のためにも、PostmanでCollectionを作りましょう。
Postman Desktop 画面右上のCollectionの横にある`「+」`からCollectionを作成します。

![](https://storage.googleapis.com/zenn-user-upload/6f8a04ad2993-20240223.png)

モーダル一番下の青字、`View more templates`をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/06f2b76dc58e-20240223.png)

そうするとテンプレートが色々出てくるので左上の検索窓に`「GraphQL」`と入力すと、`GraphQL Basics`と書かれたテンプレートが出てくるのでこれをクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/3e4a0163538b-20240223.png)

更に開くと、右上にオレンジ色の`Use Template`のボタンがあるのでこちらをクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/a02f95496d83-20240223.png)

画面上でGraphQLアイコンのQueries, Mutations... などができていれば問題ないです。Queriesを選択すると画面右側にリクエストが書けるようになっています。

### Personal Acccess Token を取得する

GitHub API にアクセスするためには、`Personal Access Token`が必要です。

GitHub上で右上のユーザーアイコンをクリックし、出てくるナビゲーションの以下の「Settings」をクリックしてください。
![](https://storage.googleapis.com/zenn-user-upload/8a61ebb7a380-20240223.png)

いろいろな設定が出てきますが、左側サイドバーの一番下にある「Developer Settings」をクリックします。
![](https://storage.googleapis.com/zenn-user-upload/8032fd447644-20240223.png)

そうすると画面左側のサイドバーが急に数が減るのでこれも一番下の「Personal access tokens」をクリック。するとメニューが開くので「Fine-grained tokens」のほうをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/686c9af1101e-20240223.png)

右上の「Generate new Token」をクリックし、赤色のアスタリスクで示された必要事項を入力します。
![](https://storage.googleapis.com/zenn-user-upload/7d69cb46d3b5-20240223.png)

「Repository access」にてOnly selected Repositoriesを選択し、読み書きしたいリポジトリを選択します。今回は`WataruNishimura/ferraille`を選択します。

![](https://storage.googleapis.com/zenn-user-upload/a58c1cb49803-20240223.png)

そうすると画面下Repository Permissionsが出てくるので、「Contents」「Issues」「Pull requests」あたりのAccessを`Read-only`に変更します。

:::message alert
リポジトリの設定を変更したい、issueをAPI経由で書き換えたい場合のみ、AccessをRead and writeにしてください。それ以外の場合はRead-onlyにしておくことを推奨します。
:::

![](https://storage.googleapis.com/zenn-user-upload/feb04343cafd-20240223.png)

最後に画面一番下のOverviewにてPermissionsの一覧が見えるので間違いが無いか確認して、Generate token をクリック、でてきたトークンをコピーしてください

![](https://storage.googleapis.com/zenn-user-upload/6764296c434f-20240223.png)

### Postmanにトークンを設定

上記でコピーしたトークンを設定します。
Postman、Queriesの画面内にある「Authorizaton」タブをクリック。
Typeを`Bearer Token`に設定し、token欄にコピーしたトークンを貼り付けてください。
貼付け後画面右上側にある「Save」をクリックしておきましょう。

![](https://storage.googleapis.com/zenn-user-upload/99c16c62f8e6-20240223.png)

### Queryを送ってみる

まずは適当にQueryを送ってみましょう。
その前にPostmanでQueriesを選択し、`{{base_url}}`と書かれた部分を`https://api.github.com`に置き換えましょう。

![](https://storage.googleapis.com/zenn-user-upload/b276e199a5f9-20240223.png)

以下は私のパブリックリポジトリにアクセスして作成日を取得するQueryです。
この場合`owner`がユーザー名、`name`がリポジトリ名になります。
`WataruNishimura/ferraille`→`{name}/{owner}`という仕組みです。
```graphql
query {
    repository(name: "ferraille", owner: "WataruNishimura") {
        createdAt
    }
}
```

このクエリをPostmanの画面右側リクエストにコピーして貼り付け、「Query」をクリックして実行します。

![](https://storage.googleapis.com/zenn-user-upload/cc6dfda4705c-20240223.png)

以下のようなリクエストが返ってくれば成功です。
```json
{
    "data": {
        "repository": {
            "createdAt": "2024-02-11T11:05:08Z"
        }
    }
}
```

### schema情報を読み取り補完させる

Postmanでは、graphqlのschemaを読み取りQueryやMutationの作成時にスキーマをエディタ上で補完してくれる機能を持ちます。（イントロスペクションと呼ばれる仕組みです）

同じく、PostmanでQueriesを開き「Schema」タブをクリックして開きます。「Using GraphQL introspection.」をクリックして完了すればOKです。

![](https://storage.googleapis.com/zenn-user-upload/fe5396ce409f-20240223.png)

エディタ上で以下画像のようにQueryの引数やフィールドを選択できるようになっているはずです。
![](https://storage.googleapis.com/zenn-user-upload/8f81fd798bb1-20240223.png)


## 最後に

上記設定を終わらせれば、Mutationsを利用しissueの情報更新などもPostman経由で簡単に行うことができるようになります。良きGitHubライフを！


## 参考記事

[既存のGraphQLサービスからSchemeを取得する](https://zenn.dev/nekoshita/articles/7c454e8e552c0d))
