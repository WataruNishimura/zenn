---
title: "metadataBaseを使うとNext.js OG画像のURLを変えられる件について"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

こんにちは,現在この記事を北海道は釧路より書いています.
札幌市在住のWebフロントエンドエンジニアのn13uです.
本当は会社ブログで書こうと思ったんですが,一旦雑記まで.

![](https://storage.googleapis.com/zenn-user-upload/9d83840e7e87-20241020.jpeg)

本業はVercelのパートナーをやってるのパートナーをやってる最近SuperXというCMSの開発をしているちょっと株式会社という会社です. 今回は会社の話はしません.

---

# 前置き（飛ばしても良い）

とあるサービス開発開発の話です. Next.js 14.x App Routerを利用していてOG画像を生成したいねという話になりました. 

:::message
**OG画像とは？**
SNS等で記事のURLを共有した際に,出てくるサムネイルの画像のことを指します. よくOGP画像と言われたりもしますが,OGPのPはProtocolのPでありプロトコルの画像ってよくわからんなというのでOG画像と呼んでいます.
:::

OG画像の表示は画像へのURLをmetaタグで書くことで行うのですが,そのとき絶対URLで指定する必要があります. つまり,`/image.jpeg`や`/hogehoge/fugafuga.png`みたいなURLは利用できず, `https://tsurui-yoitoko.jp/shitsugen.jpg`プロトコルやドメインも含んだ完全なURLである必要があります.

そういったこともあり,一般的にOG画像は画像を１枚１枚丹精を込めて作り,metaタグに１ページ１ページ仕込むといったことが必要になります.

でもって,ブログサービスのようなコンテンツが常に生成され短期の間でタイトルや文章がコロコロ変わるようなサービスではちまちま画像を作るのは大変です.

そこで出てくるのがOG画像（ないしは他のOG情報）をいい感じに動的に生成したいという願いです

> PHPとかRubyとかサーバーサイドでhtmlを作って返すタイプのアプリケーションだとそこまで辛くないかもですが, 画像の作成とレスポンスってめんどいんですよね（体感）

# Next.jsでOG画像を動的に生成する

公式ドキュメントを見る限り,OG画像を動的に生成する場合は`page.tsx`と同じようにページフォルダの中に`opengraph-image.tsx`を作成し,その中で`ImageResponse`のオブジェクトを返すことで指定した画像が生成されます.（画像の作成にはJSXを利用でき,大変使いやすいです.）

サンプルだとこんな感じです.

```typescript
import { ImageResponse } from 'next/og'
 
// Image metadata
export const alt = 'About Acme'
export const size = {
  width: 1200,
  height: 630,
}
 
export const contentType = 'image/png'
 
// Image generation
export default async function Image() {
  // Font
  const interSemiBold = fetch(
    new URL('./Inter-SemiBold.ttf', import.meta.url)
  ).then((res) => res.arrayBuffer())
 
  return new ImageResponse(
    (
      // ImageResponse JSX element
      <div
        style={{
          fontSize: 128,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        About Acme
      </div>
    ),
    // ImageResponse options
    {
      // For convenience, we can re-use the exported opengraph-image
      // size config to also set the ImageResponse's width and height.
      ...size,
      fonts: [
        {
          name: 'Inter',
          data: await interSemiBold,
          style: 'normal',
          weight: 400,
        },
      ],
    }
  )
}
```

上記のコードを記述しておくと同じ階層ページのHTML,`<head>`タグ内に以下のようなmetaタグが生成されます.

```html
<meta property="og:image" content="<generated>" />
<meta property="og:image:alt" content="About Acme" />
<meta property="og:image:type" content="image/png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
```

引用元：https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image


ここで**お,便利で使いやすいやんけ〜使ったろ〜**と思うと落とし穴にはまります.

# OG画像のホスト名がlocalhostになる問題

ローカル開発だと気にならないんですが,何も設定せずに`npm run build && npm run start`を実行した後,いざ公開してみると以下のようになってしまいます. 
```
<meta property="og:image" content="http://localhost:3000/opengraph-image.jpg" />
```
これはまずいですね...

## 解決策その①と多分動かない例

じゃあ,HOSTNAME指定してあげればええやんという話ですね. `npm run start -- -H $HOSTNAME`とかでいい感じに指定してあげればいいんですが,これ多分DockerDockerとかを使っていると名前解決が失敗したりでそもそもサーバーが立ち上がらんみたいなことが起こりうると考えました

:::message
（検証するの面倒なので検証してません）
:::

# 本題, metadataBaseの出番です（前置き長い〜）

`metadataBase` とは何かというと, Next.jsにてmetadataを生成する際に, それらで使うURLのベースを指定できる設定項目です.

使い方は簡単. 

```typescript
export const metadata = {
  metadataBase: new URL("https://acme.com") // URL | string | undefined
}
```

これを書くだけでOG画像のURLが切り替わります（もちろん他のog:url）とかもこのURLになります.

:::message
`generateMetadata`を使っても同じ結果が得られます
:::

ドキュメント：https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadatabase

:::message
ちなみに... Vercelを使っているとおおよそこの問題には遭遇しません. というのもNext.jsが勝手に忖度して環境変数`VERCEL_URL`の値がmetadataBaseのデフォルト値になるらしいので！
:::

# 最後に注意点

:::message alert
`metadataBase`ですが, 開発環境だと動きません.
`build & start`された環境じゃないと動かないのでお気をつけて.
:::

# 以上

--- 

# 参考 
https://github.com/vercel/next.js/discussions/53566