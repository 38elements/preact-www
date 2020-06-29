---
name: はじめに
description: "初歩的なPreactの使い方の説明をします。ここではツールの設定やアプリケーションを書く方法を説明します。"
---

# はじめに

このガイドはPreact初心者向けの内容です。

ここでは主要な3つの方法を紹介します。あなたがPreact初心者なら、[Preact CLI](#preact-cliを使ったお勧めの方法)を選択することを勧めます。

---

<div><toc></toc></div>

---

## ビルドツールを使わない方法

Preactはビルドやツール無しでブラウザで直に使うためのパッケージを提供しています。

```html
<script type="module">
  import { h, Component, render } from 'https://unpkg.com/preact?module';

  // Create your app
  const app = h('h1', null, 'Hello World!');

  render(app, document.body);
</script>
```

[🔨 Glitch上で編集する](https://glitch.com/~preact-no-build-tools)

この方法の主な利点はJSXを使わないのでビルドステップが必要ないことです。
しかし、直に`h`や`createElement`を呼び出しを書くことは書きにくいかもしれません。
JSXはHTMLと見た目が似ているので多くの開発者にとって理解することが容易であるという利点があります。
JSXはビルドステップを必要とします。だから、それの代わりに[HTM][htm]を使うことを強く勧めます。

### HTM

[HTM][htm]は標準的なJavaScriptで動作するJSXに似た構文で記述します。
HTMはビルドステップの代わりにJavaScriptの[Tagged Templates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_templates)を使います。
`Tagged Templates`はES2015で追加され、すべての[モダンブラウザ](https://caniuse.com/#feat=template-literals)でサポートされています。
HTMは今までのフロントエンドのビルドツールよりシンプルで学習コストが低いのでPreactアプリケーションを実装する方法として採用が拡大中です。

```html
<script type="module">
  import { h, Component, render } from 'https://unpkg.com/preact?module';
  import htm from 'https://unpkg.com/htm?module';

  // Initialize htm with Preact
  const html = htm.bind(h);

  const app = html`<h1>Hello World!</h1>`;
  render(app, document.body);
</script>
```

[🔨 Glitch上で編集する](https://glitch.com/~preact-with-htm)

HTMについて詳しく知りたい場合はHTMの[ドキュメント][htm]をチェックしてください。

## Preact CLIを使ったお勧めの方法

[Preact CLI]は最新のウェブ開発に最適化されたPreactアプリケーションのビルドをするためのすぐに使うことができるソリューションです。
それはwebpack、BabelそしてPostCSSのような標準的なツールを基にしてできています。
Preact CLIを使い始めるのに何らかの設定や事前の知識は必要ありません。このシンプルさからPreactを使用する最も一般的な方法になっています。
その名前の通り、Preact CLIは **c**ommand-**li**ne のツールでターミナルで動作します。
インストールは以下のようにしてグローバルにインストールします。

```bash
npm install -g preact-cli
```

その後、ターミナルに`preact`コマンドが新しく加えられます。
これで`preact create`コマンドを使って新しいアプリケーションを作成することができます。

```bash
preact create default my-project
```

上記のコマンドは[default template](https://github.com/preactjs-templates/default)をベースに新しいアプリケーションを作成します。
コマンドを入力して表示された選択肢の中からプロジェクトの要件にあった物を選択します。
すると指定したディレクトリ(この場合`my-project`)にアプリケーションが生成されます。

> **Tip:** テンプレートフォルダーがあるGithubレポジトリはカスタムテンプレートとして利用することができます。
>
> `preact create <username>/<repository> <project-name>`

### 開発の準備

これで、アプリケーションを開始する準備ができました。
生成したプロジェクトフォルダ(上記の`my-project`)で以下のコマンドを実行して開発サーバを起動します。

```bash
# 生成したプロジェクトフォルダに移動
cd my-project

# 開発サーバを起動
npm run dev
```

サーバが起動するとブラウザで開いて確認するための開発用のローカルのURLが表示されます。
これで、アプリケーションを開発する準備ができました。

### Production用のビルド

CLIは高度に最適化されたプロダクションビルドを行う`build`コマンドを用意しています。

```bash
npm run build
```

ビルドが完了すると直接サーバにデプロイできる`build/`フォルダが生成されます。

> Preact CLIのコマンドについて詳しく知りたい場合は[Preact CLI Documentation](https://github.com/preactjs/preact-cli#cli-options)を確認してください。

## 既存のビルドシステムとの統合

既にビルドシステムが存在いる場合、バンドラが含まれている可能性が高いです。
大多数の人はバンドラに[webpack](https://webpack.js.org/)、 [rollup](https://rollupjs.org)もしくは[parcel](https://parceljs.org/)を使っています。
Preactはそれらをすぐに使うことができます。変更は必要ありません。

### JSXの設定

JSXをJavaScriptに変換するためにBabelプラグインを使用する必要があります。
一般的には[@babel/plugin-transform-react-jsx](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx)が使われています。
インストール後、必要なJSXの機能を設定します。

```json
{
  "plugins": [
    ["@babel/plugin-transform-react-jsx", {
      "pragma": "h",
      "pragmaFrag": "Fragment",
    }]
  ]
}
```

> Babelの素晴らしいドキュメントが[こちら](https://babeljs.io/)にあります。Babelに関する疑問や設定方法についてはこちらを確認することをお勧めします。

### ReactをPreactにエイリアスする

Reactの広大なエコシステムを利用したいと思うことがあるでしょう。
元々React用に書かれているライブラリやコンポーネントでも互換レイヤとシームレスに連携します。
これを利用するにはすべての`react`と`react-dom`のインポートをPreactへ向ける必要があります。
このことを`エイリアスする`と言います。

#### webpackでエイリアスする

webpackでエイリアスするには`resolve.alias`セクションを設定に追加する必要があります。
使用している設定では既にこのセクションが有るかもしれませんがPreact用のエイリアスがありません。

```js
const config = { 
   //...snip
  "resolve": { 
    "alias": { 
      "react": "preact/compat",
      "react-dom/test-utils": "preact/test-utils",
      "react-dom": "preact/compat",
     // Must be below test-utils
    },
  }
}
```

#### Parcelでエイリアスする

Percelでは`package.json`に以下のような`alias`キーを追加します。

```json
{
  "alias": {
    "react": "preact/compat",
    "react-dom/test-utils": "preact/test-utils",
    "react-dom": "preact/compat"
  },
}
```

#### Jestでエイリアスする

バンドラと同様に[Jest](https://jestjs.io/)もモジュールのパスを書き換えることができます。
シンタックスが正規表現に基づいているためwebpackとは少し違います。
以下をJestの設定に加えてください。

```json
{
  "moduleNameMapper": {
    "^react$": "preact/compat",
    "^react-dom/test-utils$": "preact/test-utils",
    "^react-dom$": "preact/compat"
  }
}
```

[htm]: https://github.com/developit/htm
[Preact CLI]: https://github.com/preactjs/preact-cli
