---
name: APIリファレンス
description: 'Preactモジュールでエクスポートされているすべての関数について詳しく学びましょう。'
---

# APIリファレンス

このページはエクスポートされているすべての関数の概要を記載します。

---

<div><toc></toc></div>

---

## Component

`Component`はステートフルなPreactコンポーネントを作成するために拡張できるベースクラスです。<br>
コンポーネントは直接インスタンス化するのではなく、レンダラーによって管理され必要に応じて生成されます。

```js
import { Component } from 'preact';

class MyComponent extends Component {
  // (see below)
}
```

### Component.render(props, state)

すべてのコンポーネントは`render()`関数を持つ必要があります。`render()`はコンポーネントの現在の`porps`と`state`が渡されます。そして、仮想DOM要素、配列、`null`を返す必要があります。

```jsx
import { Component } from 'preact';

class MyComponent extends Component {
	render(props, state) {
		// props is the same as this.props
		// state is the same as this.state

		return <h1>Hello, {props.name}!</h1>;
	}
}
```

コンポーネントとその使い方を詳しく知りたい場合は[コンポーネントのドキュメント](/guide/v10/components)をチェックしてください。

## render()

`render(virtualDom, containerNode, [replaceNode])`

仮想DOM要素を親DOM要素である`containerNode`内にレンダリングします。戻り値はありません。

```jsx
// DOM tree before render:
// <div id="container"></div>

import { render } from 'preact';

const Foo = () => <div>foo</div>;

render(<Foo />, document.getElementById('container'));

// After render:
// <div id="container">
//  <div>foo</div>
// </div>
```

オプションの`replaceNode`パラメータは`containerNode`の子要素でなければなりません。<br>
レンダリングの開始地点を推測する代わりに、Preactは差分アルゴリズムを使用して渡された要素を更新または置換します。

```jsx
// DOM tree before render:
// <div id="container">
//   <div>bar</div>
//   <div id="target">foo</div>
// </div>

import { render } from 'preact';

const Foo = () => <div id="target">BAR</div>;

render(
  <Foo />,
  document.getElementById('container'),
  document.getElementById('target')
);

// After render:
// <div id="container">
//   <div>bar</div>
//   <div id="target">BAR</div>
// </div>
```

第1引数はコンポーネントもしくは要素を表す有効な仮想DOMでなければなりません。<br>
必ずコンポーネントを渡す際は直接コンポーネント化するのではなくPreactにコンポーネント化を任せてください。直接コンポーネント化すると予期せず途中で止まります。

```jsx
const App = () => <div>foo</div>;

// DON'T: Invoking components directly breaks hooks and update ordering:
render(App(), rootElement); // ERROR
render(App, rootElement); // ERROR

// DO: Passing components using h() or JSX allows Preact to render correctly:
render(h(App), rootElement); // success
render(<App />, rootElement); // success
```

## hydrate()

既にプリレンダリングもしくはサーバーサイドレンダリングによってアプリケーションをHTMLに出力している場合、ブラウザでのロード時にレンダリング処理をバイパスします。<br>
これは`render()`を`hydrate()`に置き換えることで有効になります。これはイベントリスナを取り付けてコンポーネントツリーをセットしている処理中の差分処理をスキップします。<br>
これは[プリレンダリング](/cli/pre-rendering)もしくは[サーバーサイドレンダリング](/guide/v10/server-side-rendering)と連携した場合のみ動作します。


```jsx
import { hydrate } from 'preact';

const Foo = () => <div>foo</div>;
hydrate(<Foo />, document.getElementById('container'));
```

## h() / createElement()

`h(type, props, ...children)`

与えられた`props`を持つ仮想DOM要素を返します。<br>
仮想DOM要素はアプリケーションのUIの階層構造に所属するノードを表す軽量なデータです。<br>
基本的に`{ type, props }`という形式のオブジェクトです。


`type`と`props`の後の残りのパラメーターは配列である`children`に格納されます。<br>
`children`は次のどれかです。

- スカラー値 (string、number、boolean、null、undefined等)
- ネストされた仮想DOM要素
- 上記の無限にネストされた配列

```js
import { h } from 'preact';

h('div', { id: 'foo' }, 'Hello!');
// <div id="foo">Hello!</div>

h('div', { id: 'foo' }, 'Hello', null, ['Preact!']);
// <div id="foo">Hello Preact!</div>

h(
	'div',
	{ id: 'foo' },
	h('span', null, 'Hello!')
);
// <div id="foo"><span>Hello!</span></div>
```

## toChildArray

このヘルパー関数は`props.children`の値の構造やネストに関係なくそれをフラット化します。<br>
`props.children`が既に配列の場合、コピーを返します。<br>
この関数は`props.children`が配列でない場合に役に立ちます。それはJSXの静的な処理と動的な処理の組み合わせで発生します。

仮想DOM要素が子要素を1つ持つ場合、`props.children`は子要素を参照します。<br>
複数の子要素を持つ場合、`props.children`は必ず配列です。<br>
`toChildArray`を使用するとすべてのケースを一貫して処理することができます。

```jsx
import { toChildArray } from 'preact';

function Foo(props) {
  const count = toChildArray(props.children).length;
  return <div>I have {count} children</div>;
}

// props.children is "bar"
render(
  <Foo>bar</Foo>,
  container
);

// props.children is [<p>A</p>, <p>B</p>]
render(
  <Foo>
    <p>A</p>
    <p>B</p>
  </Foo>,
  container
);
```

## cloneElement

`cloneElement(virtualElement, props, ...children)`

この関数は仮想DOM要素の浅い(shallow)コピーを作成します。<br>
一般的に要素の`props`を追加することや上書きすることに使用します。

```jsx
function Linkout(props) {
  // add target="_blank" to the link:
  return cloneElement(props.children, { target: '_blank' });
}
render(<Linkout><a href="/">home</a></Linkout>);
// <a href="/" target="_blank">home</a>
```

## createContext

[Context documentation](/guide/v10/context#createcontext)のcreateContextの項目を見てください。

## createRef

`current`プロパティが`ref`が最後にセットされた値を参照する`ref`オブジェクトを生成します。

詳細は[リファレンスのドキュメント](/guide/v10/refs#createref)(/guide/v10/context#createcontext)を見てください。

## Fragment

子要素を持つことができるが、DOM要素としてレンダリングされない特殊なコンポーネントです。<br>
フラグメントはDOMコンテナーのラップなして複数の兄弟関係にある子要素を返すことを可能にします。

```jsx
import { Fragment, render } from 'preact';

render(
  <Fragment>
    <div>A</div>
    <div>B</div>
    <div>C</div>
  </Fragment>,
  document.getElementById('container')
);
// Renders:
// <div id="container>
//   <div>A</div>
//   <div>B</div>
//   <div>C</div>
// </div>
```
