---
name: Reactとの違い
permalink: '/guide/differences-to-react'
description: 'ReactとPreactの違いは何でしょう。このドキュメントはそれらを詳細に解説します。'
---

# Reactとの違い

PreactはReactの再実装を意図したものではありません。両者には違いがあります。ほとんどの違いは取るに足らない物か、[preact/compat]によって吸収されます。[preact/compat]はReactとの互換性を100%達成しようと試みている薄いレイヤーです。

PreactがReactの機能をすべて含まない理由は**小さい**、**集中した**状態を保ちたいからです。そうでないなら、単にReactプロジェクトを最適化する方が合理的です。Reactは既にとても複雑で素晴らしいアーキテクチャのコードベースです。

---

<div><toc></toc></div>

---

## 主な違い

PreactとReactの主な違いはPreactには合成イベント(Synthetic Event)がないことです。
Preactはイベント処理に内部でブラウザネイティブの`addEventlistener`を使用しています。
DOMイベントハンドラの完全はリストは[GlobalEventHandlers]にあります。

ブラウザのイベントシステムはPreactにとって必要なすべての機能を満たしているので合成イベントは意味がありません。
合成イベントのようなカスタムイベントを完全に実装するとメンテナンスのオーバーヘッドが増加しAPIが複雑になります。

Reactの合成イベントとブラウザのネイティブイベントに以下のような違いがあります。

- ブラウザイベントは`<Portal>`コンポーネントをイベントバブリングで通過しません。
- IE11で`<input type="search">`要素の"x"クリアボタンは`input`イベントを発火しません。
- `<input>`要素では`onChange`の代わりに`onInput`を使用してください。 (**`preact/compat`を使用していない場合のみ**)

その他の主な違いはDOMの使用に少しだけ厳密に準拠していることです。
それの一つの例は`className`の代わりに`class`を使うことができることです。

## バージョンの互換性

Preactと[preact/compat]の両方で、バーションの互換性はReactの現在と過去のメジャーリリースを参考にします。
新機能がReactチームによって発表された場合、[Projectの目的]にとって意味がある場合はPreactコアに追加されるかもしれません。
これはとても民主的なプロセスです。Preactはissueやpull requestを通じでオープンに議論や意思決定をして継続的に進化し続けています。

> 従って、ウェブサイトとドキュメントのPreactとReactの互換性や比較はReact`16.x`と`15.x`が反映されています。

## Preact固有の機能

Preactは実際に(P)Reactコミュニティの作業から生まれたアイディアからいくつかの便利な機能を追加しています。

### `Component.render()`の引数

便利なので、クラスコンポーネントの`this.props`と`this.state`を`render()`に渡します。
1つの`props`と1つの`state`を使用する以下のコンポーネントを見てください。

```jsx
// Works in both Preact and React
class Foo extends Component {
  state = { age: 1 };

  render() {
    return <div>Name: {this.props.name}, Age: {this.state.age}</div>;
  }
}
```

Preactではこれを以下のように書くことができます。

```jsx
// Only works in Preact
class Foo extends Component {
  state = { age: 1 };

  render({ name }, { age }) {
    return <div>Name: {name}, Age: {age}</div>;
  }
}
```

両方とも全く同じ物をレンダリングします。どちらのスタイルを選ぶかは単なる好みの問題です。

### 生のHTML属性/プロパティ名

PreactはReactよりすべての主要なブラウザでサポートされているDOMの仕様を厳守しています。
Reactとの主な違いの1つは`className`属性の代わりに標準の`class`属性を使うことができることです。

```jsx
// This:
<div class="foo" />

// ...is the same as:
<div className="foo" />
```

両方ともサポートされていますが、ほとんどのPreact開発者は短く書くことができるので`class`を使うことを好みます。

### `onChange`の代わりに`onInput`を使う

歴史的な理由によって、Reactは基本的に`onInput`を`onChange`に割り当てます。
`onInput`はDOMネイティブです。そしてPreactがサポートされているすべてのブラウザでサポートされています。
`input`イベントはフォームコントロールが更新された際に通知を受けたいほとんどすべての場合で役立つイベントです。

```jsx
// React
<input onChange={e => console.log(e.target.value)} />

// Preact
<input onInput={e => console.log(e.target.value)} />
```

[preact/compat]を使っている場合、Reactのように`onInput`を`onChange`に割り当てます。
これはReactのエコシステムとの互換性を最大限に確保するためです。

### JSXコンストラクタ

このアイディアは元は[hyperscript]と呼ばれていました。これはReactのエコシステムを超える価値があります。
だから、Preactは元のやり方を推奨しています。([詳しくは: why `h()`?](http://jasonformat.com/wtf-is-jsx))
`h()`はトランスパイルされたコードを見ると`React.createElement`より少し読みやすいです。

```js
h(
  'a',
  { href:'/' },
  h('span', null, 'Home')
);

// vs
React.createElement(
  'a',
  { href:'/' },
  React.createElement('span', null, 'Home')
);
```

ほとんどのPreactアプリケーションでは`h()`が使用されています。
しかし、コアでは`h()`と`createElement()`の両方がサポートされています。
だから、どちらを使うかは重要ではありません。

### contextTypesは必要ありません

Reactの古いコンテキストAPIではコンポーネントに`contextTypes`もしくは`childContextTypes`を実装する必要があります。
Preactではこの制限はありません。すべてのコンポーネントは`getChildContext()`から生成されたすべての`context`の値を受け取ります。

## `preact`には無くて`preact/compat`に有る機能

`preact/compat`はReactのコードをPreactに移行するための**互換**レイヤーです。
既存のReactユーザはコードはそのままでバンドラの設定にいくつかのエイリアスをセットするだけで
とても手軽にPreactを試すことができます。

### Children API

Reactの`Children`APIはコンポーネントの`children`を反復処理するためのAPIです。
PreactではこのAPIは必要ありません。代わりにネイティブの配列のメソッドを使います。

```jsx
// React
function App(props) {
  return <div>{Children.count(props.children)}</div>
}

// Preact: Convert children to an array and use standard array methods.
function App(props) {
  return <div>{toChildArray(props.children).length}</div>
}
```

### 固有のコンポーネント

[preact/compat]は以下のような特殊な用途で使用することを目的としたコンポーネントを提供しています。

- [PureComponent](/guide/v10/switching-to-preact#purecomponent): `props`もしくは`state`が変化した場合のみ更新されます。
- [memo](/guide/v10/switching-to-preact#memo): `PureComponent`と用途が似ていますがこちらは比較のための関数を指定することができます。
- [forwardRef](/guide/v10/switching-to-preact#forwardRef): 指定した子コンポーネントに`ref`をセットします。
- [Portals](/guide/v10/switching-to-preact#portals): 指定した仮想DOMツリーを別のDOMコンテナにレンダリングします。
- [Suspense](/guide/v10/switching-to-preact#suspense): **実験的機能** 仮想DOMツリーの準備ができていない間は予備の仮想DOMツリーのレンダリングを可能にします。
- [lazy](/guide/v10/switching-to-preact#suspense): **実験的機能** 非同期のコードを遅延ロードします。ロード完了を通知します。

[Projectの目的]: /about/project-goals
[hyperscript]: https://github.com/dominictarr/hyperscript
[preact/compat]: /guide/v10/switching-to-preact
[GlobalEventHandlers]: https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers
