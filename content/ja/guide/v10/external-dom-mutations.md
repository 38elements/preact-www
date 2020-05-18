---
name: 外部のDOMとの連携
permalink: '/guide/external-dom-mutations'
description: 'PreactとjQueryやその他の直接DOMを操作するJavaScriptのスニペットと連携する方法'
---

# 外部のDOMとの連携

DOMを自由に操作したり、状態を保持したり、コンポーネントに組み込まれること想定しているサードパーティライブラリと連携する必要がある場合があります。連携可能な素晴らしいUIツールキットや再利用可能な要素が多く存在します。

Preact(Reactも同様)ではこのタイプのライブラリと連携するために仮想DOMのrendering/diffingアルゴリズムに指定したコンポーネント(もしくはそのDOM要素)が外部のDOMの予期しない変更を戻さないようにする必要があります。

---

<div><toc></toc></div>

---

## テクニック

これはコンポーネントに`false`を返す`shouldComponentUpdate()`メソッドを定義するだけで簡単にできます。

```jsx
class Block extends Component {
  shouldComponentUpdate() {
    return false;
  }
}
```

短く書くと以下のようになります。

```jsx
class Block extends Component {
  shouldComponentUpdate = () => false;
}
```

このライフサイクルフックを追加することで、仮想DOMツリーが変更された際に再レンダリングしません。
これによって、コンポーネントは削除されるまで静的なコンポーネントのルートDOM要素への参照を保持します。
コンポーネントのルートDOM要素への参照は`this.base`です。それは`render()`が返したルートJSX要素に対応するDOM要素です。

---

## 概要

以下はコンポーネントの再レンダリングを"オフ"にする例です。
`render()`は最初のDOM構造を生成するためにコンポーネントを生成し配置する過程で実行されることに注意してください。

```jsx
class Example extends Component {
  shouldComponentUpdate() {
    // 差分による再レンダリングをしません。
    return false;
  }

  componentWillReceiveProps(nextProps) {
    // 必要に応じてpropsが来たときの処理を書きます。
  }

  componentDidMount() {
    // 配置されました。自由にDOMを操作することができます。
    let thing = document.createElement('maybe-a-custom-element');
    this.base.appendChild(thing);
  }

  componentWillUnmount() {
    // コンポーネントがDOMから削除されます。DOMのクリーンアップ処理をここに書きます。
  }

  render() {
    return <div class="example" />;
  }
}
```


## デモ

[![デモ](https://i.gyazo.com/a63622edbeefb2e86d6c0d9c8d66e582.gif)](http://www.webpackbin.com/V1hyNQbpe)

[**Webpackbinで動くデモを見る**](https://www.webpackbin.com/bins/-KflCmJ5bvKsRF8WDkzb)


## 実際の例

または、[preact-token-input](https://github.com/developit/preact-token-input/blob/master/src/index.js)で、このテクニックの使い方を確認してください。
`preact-token-input`はDOM内の足場としてコンポーネントを使用します。
`preact-token-input`はコンポーネントの更新を無効にしてinput要素の処理を[tags-input](https://github.com/developit/tags-input)に引き継がせます。
より複雑な例として[preact-richtextarea](https://github.com/developit/preact-richtextarea)があります。これは編集可能な`<iframe>`の再レンダリングを避けるテクニックを使っています。
