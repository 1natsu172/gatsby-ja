---
title: "Gatsby Eコマースチュートリアル"
---

# 目次

- [目次](#目次)
- [なぜEコマースサイトでGatsbyを使うのか](#なぜEコマースサイトでGatsbyを使うのか)
- [前提条件](#前提条件)
  - [StripeとGatsbyはどのように連携させるのか](#StripeとGatsbyはどのように連携させるのか)
- [Gatsbyサイトの設定](#Gatsbyサイトの設定)
- [StripeJSプラグインのインストール](#StripeJSプラグインのインストール)
  - [ブラウザでホットリロードされることを確認しよう](#ブラウザでホットリロードされることを確認しよう)
  - [StripeJSプラグインはどのように動作するか](#StripeJプラグインはどのように動作するか)
  - [Stripeのテストキーを取得する](#Stripeのテストキーを取得する)
- [例](#例)
  - [簡単:ボタン1つ](#簡単:ボタン1つ)
  - [発展:ソースプラグインを通じてSKUをインポートする](#発展:ソースプラグインを通じてSKUをインポートする)
- [支払いテスト](#支払いテスト)

この発展的ななチュートリアルでは, Gatsbyを用いて支払いができる基本的なeコマースサイトのUIを構築を学ぶことができます。[Stripe](https://stripe.com)は支払い処理のバックエンドとして用います。

## なぜEコマースサイトでGatsbyを使うのか

Gatsbyをe-コマースのサイトに使うメリットは以下の通りです:

- 静的サイト固有のセキュリティ
- ページをReactから静的ファイルに変換する際の超高速パフォーマンス
- 簡単にホストできる

こちらからホストされたデモを見れます: https://gatsby-ecommerce-stripe.netlify.com/

## 前提条件

- これはより高度なチュートリアルなので、以前にGatsbyでサイトを構築したことがある場合は、このチュートリアルの時間が短縮できそうです。([基本チュートリアルはこちら](/tutorial/))
- Stripe アカウント: [アカウント登録はこちら](https://dashboard.stripe.com/register)

### StripeとGatsbyはどのように連携させるのか

Stripeは顧客から情報を安全に収集して処理できる支払い処理サービスです。Stripeを試すにはこちらにアクセスしてください。[Stripeクイックスタートガイド](https://stripe.com/docs/payments/checkout#tryout)

Stripeの代わりとしては、SquareやBraintreeなどがあります。これらの仕組みはStripeと非常に似ています。

Stripeはバックエンドコンポーネントを必要としない[hosted checkout](https://stripe.com/docs/payments/checkout)を提供します。
製品やSKU、サブスクリプションプランの設定を[Stripe Dashboard](https://stripe.com/docs/payments/checkout#configure)で行うことができます。1つの製品やサブスクリプション(電子書籍)を販売している場合、Gatsbyサイトで商品のSKU IDをハードコーディングできます。
複数の商品を販売している場合、[Stripe ソースプラグイン](https://www.gatsbyjs.org/packages/gatsby-source-stripe/)を利用して、ビルド時に全てのSKUを取得できます。
Gatsbyのサイトを自動的に更新する場合は、Stripe webhookイベントを利用して新しい商品やSKUが追加されたときに再デプロイをトリガー[trigger a redeploy](https://www.netlify.com/docs/)できます。

# Gatsbyサイトの設定

ターミナルで`gatsby new`を実行して新しいGatsbyのプロジェクトを作成し、ディレクトリを先程作成した新しいプロジェクトに変更します。

```shell
gatsby new ecommerce-gatsby-tutorial
cd ecommerce-gatsby-tutorial
```

## StripeJSプラグインのインストール

プラグインを利用してこのデフォルトのスタータの機能を拡張できます。そのようなプラグインの1つが `gatsby-plugin-stripe`で、このプロジェクトにインストールします。

```shell
npm install gatsby-plugin-stripe
```

テキストエディタでルートサイトディレクトリを開き、`gatsby-config.js`に移動します。そして、`gatsby-config.js`のpluginsのところにStripeJSプラグインを追加します。`gatsby-config.js`は次のコード例のようになります。

```js:title=gatsby-config.js
module.exports = {
  siteMetadata: {
    title: `Gatsby E-Commerce Starter`,
    description: `Kick off your next, great Gatsby project with this default starter. This barebones starter ships with the main Gatsby configuration files you might need.`,
    author: `@gatsbyjs`,
  },
  plugins: [
    `gatsby-plugin-react-helmet`,
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        name: `images`,
        path: `${__dirname}/src/images`,
      },
    },
    `gatsby-transformer-sharp`,
    `gatsby-plugin-sharp`,
    {
      resolve: `gatsby-plugin-manifest`,
      options: {
        name: `gatsby-starter-default`,
        short_name: `starter`,
        start_url: `/`,
        background_color: `#663399`,
        theme_color: `#663399`,
        display: `minimal-ui`,
        icon: `src/images/gatsby-icon.png`, // This path is relative to the root of the site.
      },
    },
    `gatsby-plugin-stripe`
  ],
}

```

### ブラウザでホットリロードされることを確認しよう

ターミナルで`npm run develop`を実行しましす。これにより、開発サーバが起動し、サイトに加えた変更がリロードされるので、ブラウザで変更がプレビューできます。ブラウザを[localhost:8000](http://localhost:8000/)で開くと、デフォルトのホームページが表示されます。

> **NOTE**:もし、既にGatsbyの開発サーバを`npm run develop`で起動していた場合、コマンドを実行したターミナルで CTRL + C を押してサーバを再起動し、`npm run develop`コマンドを再度実行することで[localhost:8000](http://localhost:8000/)に反映された`gatsby-config.js`の変更を確認するできます。

### StripeJSプラグインはどのように動作するか
StripeはJavaScriptライブラリを提供します。これにより、Stripeがホストする支払いページに顧客を安全にリダイレクトできます。Gatsbyプラグインの`gatsby-plugin-stripe`は全てのページで`<body>`タグの末尾に次のスニペットを追加します：

```html
<script src="https://js.stripe.com/v3/"></script>
```
これにより、Stripeの不正検出[[fraud detection](https://stripe.com/docs/stripe-js/reference#including-stripejs)]が容易になります
もし支払い処理をさらにカスタマイズしたりSrtipeのデータをサイトに引いてくる場合は、[GatsbyのプラグインライブラリでStripeプラグインを確認](https://www.gatsbyjs.org/plugins/?=stripe)してください。

### Stripeのテストキーを取得する

StripeアカウントにログインしてAPI資格情報を表示し、[開発者] > [APIキー]に移動します。

![Stripe public test key location in Stripe account](stripe-public-test-key.png)

テストモードと本番モードの2つのキーがあります:
You have 2 keys in both test mode and production mode:

- 公開可能キー
- シークレットキー

テスト中は _test_.という文字が含まれたキーを使い、本番のコードではライブキーを使う必要があります。名前が示すように、公開可能キーは公開するコード(フロントエンドやGithubなど)に含まれているな愛がありますが、シークレットキーは誰とも共有したり、公開レポジトリにコミットしたりしないでください。このシークレットキーへのアクセスを制限することは重要です。このシークレットキーを持っている人は誰でもStripeアカウントからリクエストを読み取ったり送信したり、請求や購入に関する情報を見たり、顧客に送信したりする可能性があるからです。

### Stripeアカウントの"Checkout クライアント専用組み込み"を有効にします

このチュートリアルではStripeの"Checkout クライアント専用組み込み"を利用します。この組み込みを利用するにはStipeダッシュボードの対応する[チェックアウトの設定](https://dashboard.stripe.com/account/checkout/settings)でアクティブにする必要があります。

![Stripe control to enable the Checkout client-side only integration highlighted](stripe-checkout-clientside-functionality.png)

> 💡 この変更により、製品を管理するためにStripeが提供するインターフェースも変更されます。以前にこのツールを利用したことがある場合は、このことに留意してください。今までに製品管理を利用したことがない場合、心配する必要はありません。

さらに、この組み込みを利用するには、[アカウント設定](https://dashboard.stripe.com/account)でStripeアカウントの名前を設定する必要があります。

この組み込みの詳細については[Stripe docs](https://stripe.com/docs/payments/checkout#configure)を利用してみてください。

## 例
これらの例の実装は[Github](https://github.com/thorsten-stripe/ecommerce-gatsby-tutorial)で見ることができます。

### 簡単:ボタン1つ
例えば電子書籍などのシンプルな製品を売っている場合、Stripeの支払いページへのリダイレクトを実行するボタンを1つ作成できます。

#### 商品とSKUの作成

商品を販売するためにはまず、[Stripe ダッシュボード](https://dashboard.stripe.com/products)または[Stripe API](https://stripe.com/docs/api/products/create)を利用して、Stripeで商品を作成する必要があります。これは、Stripeがフロントエンドからのリクエストが正当であることを検証し、選択された製品/SKUに適切な金額を請求するために必要です。Stripeでは、Stripeの支払いで使用する全てのSKUに名前を付ける必要があります。全てのSKUに必ず1つ追加してください。

StripeダッシュボードでテストSKUと本番SKUの両方を作成する必要があります。「テストデータの表示」に切り替えてから、ローカル開発用の製品を作成してください。

#### StripeJSをロードして支払いコンポーネントを作成する

`src/components/checkout.js`に新しいファイルを作成します。作成した`checkout.js`は以下のようになります:

```jsx:title=src/components/checkout.js
import React from "react"

const buttonStyles = {
  fontSize: "13px",
  textAlign: "center",
  color: "#fff",
  outline: "none",
  padding: "12px 60px",
  boxShadow: "2px 5px 10px rgba(0,0,0,.1)",
  backgroundColor: "rgb(255, 178, 56)",
  borderRadius: "6px",
  letterSpacing: "1.5px",
}

const Checkout = class extends React.Component {
  // Stripe.jsを公開可能キーで初期化します
  // こちらのダッシュボードからキーを確認できます。
  // https://dashboard.stripe.com/account/apikeys
  componentDidMount() {
    this.stripe = window.Stripe("pk_test_jG9s3XMdSjZF9Kdm5g59zlYd")
  }

  async redirectToCheckout(event) {
    event.preventDefault()
    const { error } = await this.stripe.redirectToCheckout({
      items: [{ sku: "sku_DjQJN2HJ1kkvI3", quantity: 1 }],
      successUrl: `http://localhost:8000/page-2/`,
      cancelUrl: `http://localhost:8000/`,
    })

    if (error) {
      console.warn("Error:", error)
    }
  }

  render() {
    return (
      <button
        style={buttonStyles}
        onClick={event => this.redirectToCheckout(event)}
      >
        BUY MY BOOK
      </button>
    )
  }
}

export default Checkout
```
#### あなたは何をしましたか？

Reactをインポートし、いくつかのstyleのボタンを追加し、React関数を導入しました。`componentDidMount()`や、`redirectToCheckout()`といった関数はStripeの機能の中で最も重要です。`componentDidMount()`関数はコンポーネントが最初にDOMにマウントされた時に起動するReactのライフサイクルメソッドであり、Stripe.jsクライアントを初期化するのに適した場所です。コードは以下のようになります。

```jsx:title=src/components/checkout.js
  componentDidMount() {
    this.stripe = window.Stripe('pk_test_jG9s3XMdSjZF9Kdm5g59zlYd')
  }
```
これによってStripeプラットフォームが識別され、製品とセキュリティの設定に対して支払いリクエストが検証され、Stripeアカウントの支払いが処理されます。

```jsx:title=src/components/checkout.js
  async redirectToCheckout(event) {
    event.preventDefault()
    const { error } = await this.stripe.redirectToCheckout({
      items: [{ sku: 'sku_DjQJN2HJ1kkvI3', quantity: 1 }],
      successUrl: `http://localhost:8000/page-2/`,
      cancelUrl: `http://localhost:8000/`,
    })

    if (error) {
      console.warn('Error:', error)
    }
  }
```

`redirectToCheckout()`関数は支払いのリクエストを検証し、Stripeがホストする支払いページにリダイレクトするか、エラーオブジェクトで解決します。`successUrl`と`cancelUrl`を適切なURLに置き換えてください。

```jsx:title=src/components/checkout.js
  render() {
    return (
      <button
        style={buttonStyles}
        onClick={event => this.redirectToCheckout(event)}
      >
        BUY MY BOOK
      </button>
    )
  }
```

`render()`関数はstyleをボタンに適用し、`redirectToCheckout()`関数をボタンのonclickイベントにバインドします。

#### 支払いコンポーネントをホームページにインポートする

`src/pages/index.js`ファイルに移動しましょう。ここがルートURLに表示されるホームページです。他のimportの下にあるファイルに新しい支払いコンポーネントをimportし、`<Layout>`要素内に`<Checkout />`コンポーネントを追加します。`index.js`ファイルは以下のようになります。

```jsx:title=src/pages/index.js
import React from "react"
import { Link } from "gatsby"

import Layout from "../components/layout"
import Image from "../components/image"
import SEO from "../components/seo"

import Checkout from "../components/checkout" // highlight-line

const IndexPage = () => (
  <Layout>
    <SEO title="Home" keywords={[`gatsby`, `application`, `react`]} />
    <h1>Hi people</h1>
    <p>Welcome to your new Gatsby site.</p>
    <p>Now go build something great.</p>
    <Checkout /> {/* highlight-line */}
    <div style={{ maxWidth: `300px`, marginBottom: `1.45rem` }}>
      <Image />
    </div>
    <Link to="/page-2/">Go to page 2</Link>
  </Layout>
)

export default IndexPage
```

ブラウザで[localhost:8000](http://localhost:8000/)に戻り、`npm run develop`を実行している場合は、大きくて魅力的な"BUY MY BOOK"ボタンが表示されます。さあ、クリックしてみましょう！！

### 発展:ソースプラグインを通じてSKUをインポートする

SKUのIDをハードコーディングする代わりに、ビルド時に[gatsby-source-stripe plugin](https://www.gatsbyjs.org/packages/gatsby-source-stripe/)を使用してSKUを取得できます。

#### Stripeソースプラグインの追加
StripeアカウントからSKUを取得するために使える[gatsby-source-stripe plugin](https://www.gatsbyjs.org/packages/gatsby-source-stripe/)プラグインを追加しましょう。

```shell
npm install gatsby-source-stripe
```
これで、`gatsby-config`ファイルにプラグイン設定を追加できます。

```js:title=gatsby-config.js
module.exports = {
  siteMetadata: {
    title: `Gatsby E-Commerce Starter`,
  },
  plugins: [
    `gatsby-plugin-react-helmet`,
    "gatsby-plugin-stripe",
    {
      resolve: `gatsby-source-stripe`,
      options: {
        objects: ["Sku"],
        secretKey: process.env.STRIPE_SECRET_KEY,
        downloadFiles: true,
      },
    },
  ],
}
```

StripeアカウントからSKUを取得するにはシークレットAPIキーを提供する必要があります。シークレットキーは秘密にしておく必要があるので、フロントエンドやGithubで絶対に共有しないでください。したがって、環境変数を設定して秘密鍵を保存する必要があります。[Gatsby docs](/docs/environment-variables/)で環境変数の使用法について詳しく読むことができます。
プロジェクトのルートディレクトリに`.env.development`ファイルを追加してください:

```text:title=.env.development
# Stripe secret API key
STRIPE_SECRET_KEY=sk_test_xxx
```

定義された環境変数を利用するには`gatsby-config.js`または`gatsby-node.js`で以下のように環境変数を要求する必要があります。

```js:title=gatsby-config.js
require("dotenv").config({
  path: `.env.${process.env.NODE_ENV}`,
})
```

最後に、`.gitignore`ファイルで全ての`.env.*`ファイルが除外されていることを確認してください:

```text:title=.gitignore
# dotenv environment variables files
.env
.env.development
.env.production
```
#### SKUをリストするコンポーネントを作成する

componentsフォルダに新しく`Products`フォルダを追加します。このフォルダはStripeのSKUと対話するコンポーネントが含まれます。まず第一に、SKUを照会およびリストするコンポーネントが必要です:

```jsx:title=src/components/Products/Skus.js
import React from "react"
import { graphql, StaticQuery } from "gatsby"

export default props => (
  <StaticQuery
    query={graphql`
      query SkusForProduct {
        skus: allStripeSku {
          edges {
            node {
              id
              currency
              price
              attributes {
                name
              }
            }
          }
        }
      }
    `}
    render={({ skus }) => (
      <div>
        {skus.edges.map(({ node: sku }) => (
          <p key={sku.id}>{sku.attributes.name}</p>
        ))}
      </div>
    )}
  />
)
```
クエリを検証し、GraphQLで返されるデータを確認できます。GraphQLは`npm run develop`を実行した際にhttp://localhost:8000/___graphql で利用できます。

クエリに満足したら、新しく作成したSKUコンポーネントをインポートできる新しいページを作成しましょう:

```jsx:title=src/pages/advanced.js
import React from "react"
import { Link } from "gatsby"

import Layout from "../components/layout"
import SEO from "../components/seo"

import Skus from "../components/Products/Skus" // highlight-line

const AdvancedExamplePage = () => (
  <Layout>
    <SEO title="Advanced Example" />
    <h1>This is the advanced example</h1>
    <Skus /> {/* highlight-line */}
  </Layout>
)

export default AdvancedExamplePage
```
http://localhost:8000/advanced/ に移動するとSKU名を含む段落のリストが確認できます。

#### 単一のSKUを提示するコンポーネントの作成

SKUを魅力的でインタラクティブにするために、`Products`フォルダに`SkuCard`を新たに作成します。

```jsx:title=src/components/Products/SkuCard.js
import React from "react"

const cardStyles = {
  display: "flex",
  flexDirection: "column",
  justifyContent: "space-around",
  alignItems: "flex-start",
  padding: "1rem",
  marginBottom: "1rem",
  boxShadow: "5px 5px 25px 0 rgba(46,61,73,.2)",
  backgroundColor: "#fff",
  borderRadius: "6px",
  maxWidth: "300px",
}
const buttonStyles = {
  fontSize: "13px",
  textAlign: "center",
  color: "#fff",
  outline: "none",
  padding: "12px",
  boxShadow: "2px 5px 10px rgba(0,0,0,.1)",
  backgroundColor: "rgb(255, 178, 56)",
  borderRadius: "6px",
  letterSpacing: "1.5px",
}

const formatPrice = (amount, currency) => {
  let price = (amount / 100).toFixed(2)
  let numberFormat = new Intl.NumberFormat(["en-US"], {
    style: "currency",
    currency: currency,
    currencyDisplay: "symbol",
  })
  return numberFormat.format(price)
}

const SkuCard = class extends React.Component {
  async redirectToCheckout(event, sku, quantity = 1) {
    event.preventDefault()
    const { error } = await this.props.stripe.redirectToCheckout({
      items: [{ sku, quantity }],
      successUrl: `http://localhost:8000/page-2/`,
      cancelUrl: `http://localhost:8000/advanced`,
    })

    if (error) {
      console.warn("Error:", error)
    }
  }

  render() {
    const sku = this.props.sku
    return (
      <div style={cardStyles}>
        <h4>{sku.attributes.name}</h4>
        <p>Price: {formatPrice(sku.price, sku.currency)}</p>
        <button
          style={buttonStyles}
          onClick={event => this.redirectToCheckout(event, sku.id)}
        >
          購入
        </button>
      </div>
    )
  }
}

export default SkuCard
```
このコンポーネントはSKU名、適切にフォーマットされた価格設定、および「購入」ボタンを利用して、個々のSKUごとに適切なカードをレンダリングします。「購入」ボタンは対応するSKU IDで`redirectToCheckout()`関数をトリガーします。
最後に、`Skus`コンポーネントをリファクタリングしてStripe.jsクライアントを初期化し、`props`でStripe.jsクライアントを伝えながら`SkuCards`をレンダリングする必要があります:

```jsx:title=src/components/Products/Skus.js
import React, { Component } from 'react'
import { graphql, StaticQuery } from 'gatsby'
import SkuCard from './SkuCard' // highlight-line

const containerStyles = {
  display: 'flex',
  flexDirection: 'row',
  flexWrap: 'wrap',
  justifyContent: 'space-between',
  padding: '1rem 0 1rem 0',
}

class Skus extends Component {
  // Initialise Stripe.js with your publishable key.
  // You can find your key in the Dashboard:
  // https://dashboard.stripe.com/account/apikeys
  // highlight-start
  state = {
    stripe: null,
  }

  componentDidMount() {
    const stripe = window.Stripe(process.env.GATSBY_STRIPE_PUBLIC_KEY)
    this.setState({ stripe })
  }
  // highlight-end

  render() {
    return (
      <StaticQuery
        query={graphql`
          query SkusForProduct {
            skus: allStripeSku {
              edges {
                node {
                  id
                  currency
                  price
                  attributes {
                    name
                  }
                }
              }
            }
          }
        `}
        render={({ skus }) => (
          <div style={containerStyles}>
            {skus.edges.map(({ node: sku }) => (
              <SkuCard key={sku.id} sku={sku} stripe={this.state.stripe} /> {/* highlight-line */}
            ))}
          </div>
        )}
      />
    )
  }
}

export default Skus
```

#### カートコンポーネントの追加

`redirectToCheckout()`関数を呼び出して、SKUとその量の配列を提供し、複数のアイテムを同時に請求できます。したがって、支払いページにリダイレクトするそれぞれの「購入」ボタンの代わりに、カートコンポーネントの状態を利用する主要な「支払いに進む」ボタンを提供できます。この例に必要な変更は[GitHub](https://github.com/thorsten-stripe/ecommerce-gatsby-tutorial/tree/cart-example)で確認できます。

# 支払いテスト
テストモード(_test_ という文字列を含むAPIキーを利用する場合)では、Stripeは様々な支払いシナリオをテストするための[テストカード](https://stripe.com/docs/testing#cards)を提供します。
