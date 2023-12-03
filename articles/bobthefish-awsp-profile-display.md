---
title: "fish+oh-my-fish+bobthefish+JetBrains Mono Nerd Fonts+awspでターミナルプロンプトにAWSのプロファイルを表示して事故を回避する"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["fish","aws","terminal"]
published: false
---

## はじめに

札幌市でエンジニアをしています、n13uです。自分が副代表を務める[SocialChangeLab合同会社](https://socialchangelab.jp)にて、システム開発関連事業を取りまとめて（開発もして）います。
実際に案件に取り組んでいる中で、実際に起きた問題と解決方法をまとめたものになります。

今回は普段利用している fish + oh-my-fish + bobthefish 環境での fish\_prompt カスタマイズと、awspを利用した際に現在のAWSプロファイルをターミナル上のプロンプトに表示する方法をまとめました。
利用してるAWSのプロファイルが表示されていれば、アカウント間違いによるインスタンス削除の事故などを軽減できるはず、、、、

## 前提環境

- fish
- oh-my-fish 導入済み
- bobthefish 導入済み
- nodejs + awsp 導入済み
- nerd fonts インストール済み
    - お使いのターミナルでnerd fontsを表示フォントに設定済み

上記それぞれが完了した状態で進めます

## やること

1. awspをfish対応する

awspはbashで実装されているため、fishには対応していません。そのため、公式リポジトリのinstallation通りに進めてもプロファイルがうまく切り替わらないです。
bassの導入とaliasの設定でawspが利用可能になるとのことで、[shigemk2さんの記事「fish-shellでawsp」](https://www.shigemk2.com/entry/2022/02/02/222851)を参考に、bassの導入とaliasの設定を行います。


### bass 導入

今回はoh-my-fish導入済みであることを想定して、oh-my-fishでbassを導入します。
以下のコマンドを実行するだけです

```fish
omf install bass
```

### alias 設定

fishのconfigにalias設定を記述し、awspを有効化します

```fish:~/.config/fish/config.fish
alias awsp="bass source _awsp""
```

これでawspの導入・設定は完了です。

`awsp`コマンド実行後、利用したいプロファイルを選んでEnter押下すれば、そのプロファイルに切り替わります。
プロファイルの切り替わりを検証したい場合は、 `echo $AWS_PROFILE` を実行し、出力結果がプロファイルのとおりになっていることを確認してください。

2. fish\_prompt の設定

今回は、oh-my-fish+bobthefish環境を利用します。
bobthefishを導入していると以下のようなプロンプトが表示されているはずです。

![](https://storage.googleapis.com/zenn-user-upload/3f2e9ed432df-20231203.png)

できることなら、既存のプロンプトを壊すことなく、AWSプロファイルを表示させたいです。
そのため、今回は `fish\_mode\_prompt` 関数を利用してAWSプロファイルを表示させます。

:::message alert

fish\_mode\_prompt関数は、bobthefish上のviモード表示を許可している場合にのみプロンプトの表示を行う関数です。
筆者環境では利用していないため、上書きしていますが利用している場合は既存のfish\_mode\_promt の末尾に追記するなどしてください。

:::

### 実際のプログラム

以下のfishプログラムを `~/.config/fish/functions/fish_mode_prompt.fish` に記述します。

```fish:~/.config/fish/functions/fish_mode_prompt.fish

function fish_mode_prompt
 
 set -l aws_glyph ''

 [ -z "$AWS_PROFILE" ]
 and return

 __bobthefish_start_segment "ff9900" "000000" 
 echo -ns $aws_glyph ' ' $AWS_PROFILE ' '
 set_color normal
end
```

#### プログラム解説

関数を宣言します。`fish\_mode\_prompt` でプロンプトを表示してくれます。

```fish
function fish_mode_prompt
```

先にaws用のアイコン（グリフ）を設定します。
set で変数宣言ができるため、`aws_glyph`変数にアイコンを格納します。
アイコンは、NerdFont環境下でのみ表示されます。

```fish
 set -l aws_glyph ''
```

awspを利用し、プロファイルを設定した場合には、`AWS_PROFILE`変数にプロファイル名が格納されますが、利用していない場合は表示させたくないので、空の場合は早期リターンで何も表示しません。

```fish
 [ -z "$AWS_PROFILE" ]
 and return
```

`bobthefish_start_segment` 関数を利用して背景と文字を設定します。第一引数が背景色、第二引数は文字色です。
次の行で `echo` を利用して `aws_glyph`と`aws_profile`を表示します。
`set_color normal`で色を元に戻して終了です。

```fish
 __bobthefish_start_segment "ff9900" "000000" 
 echo -ns $aws_glyph ' ' $AWS_PROFILE ' '
 set_color normal
```

## 完成形

以下のように表示されていればOKです！（念のためプロファイル情報などはモザイクを掛けています）

![](https://storage.googleapis.com/zenn-user-upload/6517629c3aac-20231203.png)

## 最後に

AWSプロファイル切り替えていると何を利用しているか分からなくなりますが、これで少しでも利便性が上がると嬉しいなという思いでの記事公開でした。
ただ文字数が長いとターミナルの半分がプロンプトになってしまうのはどうにかしたい、、、改善点です。

## 宣伝

札幌市では、現在札幌にエンジニアを集め、成長を促す取り組みとして、Sapporo Engineeer Base という活動に取り組んでいます。
よければTwitter（X）のフォローやウェブサイトのぞきに来ていただけると嬉しいです！

Twitter: [https://twitter.com/seb_sapporo](https://twitter.com/seb_sapporo)
Website: [https://sapporo-engineer-base.dev](https://sapporo-engineer-base.dev)
