---
title: "Node.js各パッケージマネージャーのworkspace認識基準"
emoji: "🔭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nodejs","npm"]
published: false
---

# はじめに

札幌のフロントエンドエンジニア**[n13u](https://x.com/_n13u_)**です。

皆さんはNode.jsパッケージマネージャーは何を使われていますか？
npm, yarn, pnpm, deno, bun etc...

それぞれに**モノレポ管理用のworkspaces機能**がありますが、それぞれ設定が若干異なり挙動が異なるためパッケージマネージャーを複数併用していると時々わからなくなります。

特にNode.js環境向けCLIツール開発でパッケージマネージャーに依存環境をインストールさせる際に同じワークスペース認識方法を使わないと正しく機能しません。
今回は`ESLint`のConfig作成CLIツール`eslint/create-config`にコントリビュートしている際に気づいたこの問題？についてまとめました。

# workspaceに対応しているパッケージマネージャー（類）

- npm
- yarn(classic, berry以降で別の仕組みです)
- pnpm
- bun

まあほぼ全部ですね。(そもそもDenoにはパッケージの概念がない)


