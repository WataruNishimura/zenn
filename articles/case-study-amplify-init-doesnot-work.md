---
title: "【備忘録】AWS Amplifyで amplify init で既存環境が出てこない問題の解決"
emoji: "🔧"
type: "tech"  
topics: ["aws","amplify"]
published: true
---

## はじめに

札幌市でエンジニアをしています、n13uです。自分が副代表を務める[SocialChangeLab合同会社](https://socialchangelab.jp)にて、システム開発関連事業を取りまとめて（開発もして）います。
実際に案件に取り組んでいる中で、実際に起きた問題と解決方法をまとめたものになります。
amplifyの開発リポジトリをGitでローカルにクローンし、amplify init をしたときに既存の環境が出てこず、新規のプロジェクト作成をプロンプト上で促される場合の問題解決方法をまとめました。


## 起こっていること

AWS Organizatios と AWS IAM Identitiy Center を利用し、マルチアカウント構成をしています。
とあるAWSアカウントAのAccess権限にCLIからアクセスできるよう、`aws sso`のサブコマンドでssoセッションとプロファイルの作成を完了しています。

とあるAmplifyプロジェクトSample-Amplifyがあり、そのAppIdが20020731、GitHubリポジトリ名がSampleAmplifyですでにAmplify環境はできている状態です。
ここで別環境（例えば開発者AがつくったAmplify開発環境を開発者Bも利用したい場合など）にGitHubリポジトリをクローンし、`amplify init`をした際に、
```
? Do you want to use an existing environment?
```
のダイアログが出ずに、新規の環境作成を促されます。

### 前提環境

| パッケージ・ライブラリ | バージョン |
| ---- | ---- |
| aws-amplify@cli | v11.1.0 |
| nodejs | v18 |

## 原因

AWS Profileの切り替えが上手く行っておらず、かつ `.aws`配下の `.aws/credentials`上に`aws_access_key_id`と`aws_secret_access_key`が無いことが問題でした。

`aws sso login`もしくは`aws configure`でCLI上からIAMユーザー等にログインし、プロファイルを作成してもcredentialsが正常に生成されてなかったり、取得されていなかったりする場合に、AWSアカウント上のAmplify環境を取得できずに、新規の環境作成を促されるとのことでした。

## 対策

AWS IAM Identity Center を利用している場合は、ログインコンソールに移動後、`Commad line and prorgramatic access`のボタン押下後に出てくる`Get Access for ... `というタイトルのモーダルに記述の認証情報を以下の形式で記述します。
その後、`amplify init --profile {プロファイル名}`を実行すると正常に動くはずです。

![](https://storage.googleapis.com/zenn-user-upload/3974af80e15c-20231203.png)
![](https://storage.googleapis.com/zenn-user-upload/d27496a13896-20231203.png)

### 認証情報を以下のように記述

```:~/.aws/credetials
[{プロファイル名}]
aws_access_key_id={aws_access_key_id}
aws_secret_access_key={aws_secret_access_key}
aws_session_token={aws_session_token}
```

ちなみにこれは appId を指定して pull した際も、同様の問題が置きます。pull の場合は、指定したappIdが見つからないよと言われます。

一般的なIAMユーザーの場合は、`aws_access_key_id`と`aws_secret_access_key`の記述のみで大丈夫です。

## 最後に

めちゃくちゃ初歩的なミス？でしたが参考になればというところで、、、
記事に間違いや訂正などあればぜひ、コメントいただけますと幸いです。。。

## 宣伝

札幌市では、現在札幌にエンジニアを集め、成長を促す取り組みとして、Sapporo Engineeer Base という活動に取り組んでいます。
よけえばTwitter（X）のフォローやウェブサイトのぞきに来ていただけると嬉しいです！

Twitter: [https://twitter.com/seb_sapporo](https://twitter.com/seb_sapporo)
Website: [https://sapporo-engineer-base.dev](https://sapporo-engineer-base.dev)
