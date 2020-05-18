---
name: Preactアプリケーションのデバッグ
description: '問題が起きたときにPreactアプリケーションをデバッグする方法'
---

# Preactアプリケーションのデバッグ

Preactにはデバックを容易にするツールが付属しています。それらは`preact/debug`にパッケージングされています。そして、`preact/debug`をインポートすることで使うことができます。

`preact/debug`をインポートすることによってChromeやFirefoxやEdgeのブラウザ拡張である[Preact Devtools]と接続します。

`<table>`要素のネストが間違っている等の間違いを見つけると警告やエラーを出力します。

---

<div><toc></toc></div>

---

## インストール

[Preact Devtools]はブラウザのウェブストアでインストールすることができます。

- [For Chrome](https://chrome.google.com/webstore/detail/preact-developer-tools/ilcajpmogmhpliinlbcdebhbcanbghmd)
- [For Firefox](https://addons.mozilla.org/en-US/firefox/addon/preact-devtools/)
- [For Edge](https://microsoftedge.microsoft.com/addons/detail/hdkhobcafnfejjieimdkmjaiihkjpmhk)

インストールした後、`preact/debug`をインポートして拡張との接続を初期化する必要があります。
このインポートがアプリケーション全体で**最初に**インポートされていることを確認してください。

> `preact-cli`は`preact/debug`を自動的に導入します。`preact-cli`を使っている場合、次のステップをスキップして大丈夫です。

アプリケーションのメインエントリーファイルに以下のように`preact/debug`をインポートします。

```jsx
// Must be the first import
import "preact/debug";
import { render } from 'preact';
import App from './components/App';

render(<App />, document.getElementById('root'));
```

### productionからdevtoolsを削除する

ほとんどのバンドラーでは必ず使われない`if`文を見つけた場合、その分岐を削除します。
これを利用してdevelopment時のみ`preact/debug`を含めることができます。そして、production時には貴重なバイト数を節約することができます。

```jsx
// Must be the first import
if (process.env.NODE_ENV==='development') {
  // Must use require here as import statements are only allowed
  // to exist at the top of a file.
  require("preact/debug");
}

import { render } from 'preact';
import App from './components/App';

render(<App />, document.getElementById('root'));
```

ビルドツールで`NODE_ENV`変数が正しくセットされているか確認してください。

## デバッグ時の警告とエラー

Preactが無効なコードを発見すると警告やエラーを表示することがあります。
それらを修正してアプリケーションを完璧にしましょう。

### `render()`に渡された親要素が`undefined`

これはコードがアプリケーションをDOMノードではなく何も存在しないところにレンダリングしようとしていることを意味します。
両者の比較です。

```jsx
// What Preact received
render(<App />, undefined);

// vs what it expected
render(<App />, actualDomNode);
```

このエラーが発生する主な理由は`render()`関数が実行される際にDOMが存在していないからです。
存在することを確認して下さい。

### `createElement()`に`undefined`がコンポーネントとして渡される

Preactはコンポーネントの代わりに`undefined`を渡すとエラーをスローします。
このエラーのよくある原因は`default`エキスポートと`named`エクスポートを取り違えていることです。

```jsx
// app.js
export default function App() {
  return <div>Hello World</div>;
}

// index.js: Wrong, because `app.js` doesn't have a named export
import { App } from './app';
render(<App />, dom);
```

逆の場合も同じエラーがスローされます。
それは`named`エキスポートと宣言して`default`エキスポートを使おうとする場合です。
これを手早く確かめる方法は(エディタがまだそれを実行していない場合)インポートしたものを単に表示することです。

```jsx
// app.js
export function App() {
  return <div>Hello World</div>;
}

// index.js
import App from './app';

console.log(App);
// Logs: { default: [Function] } instead of the component
```

### 渡されたJSXリテラルがJSXとして2度評価される

JSXリテラルもしくはコンポーネントをJSXに再度渡すことは無効です。それはエラーを引き起こします。

```jsx
const Foo = <div>foo</div>;
// Invalid: Foo already contains a JSX-Element
render(<Foo />, dom);
```

単に変数を直接渡すだけで修正することができます。

```jsx
const Foo = <div>foo</div>;
render(Foo, dom);
```

### テーブルの不適切なネストを見つける

HTMLにはテーブルの構造に対して非常に明確な方針があります。
それから外れるとデバッグすることが非常に難しいレンダリングエラーが発生します。
Preactはこれを見つけてエラーを出力します。
テーブルの構造について詳しく知りたい場合は[MDNのドキュメント](https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Basics)を読んでください。

### 無効な`ref`プロパティ

`ref`プロパティに不適切な値が格納されていた場合、このエラーがスローされます。
これには少し前に非推奨になった文字列ベースの`ref`プロパティも含まれます。

```jsx
// valid
<div ref={e => {/* ... */)}} />

// valid
const ref = createRef();
<div ref={ref} />

// Invalid
<div ref="ref" />
```

### 無効なイベントハンドラ

うっかり間違った値をイベントハンドラに渡すことがあるかもしれません。
イベントハンドラに渡す値は常に`function`もしくは`null`でなければなりません。
それ以外は無効です。

```jsx
// valid
<div onClick={() => console.log("click")} />

// invalid
<div onClick={console.log("click")} />
```

### フックはrender()メソッドのみで実行することができる

このエラーはコンポーネントの外でフックを使用したときに発生します。
フックは関数コンポーネントの内側でのみサポートされています。

```jsx
// Invalid, must be used inside a component
const [value, setValue] = useState(0);

// valid
function Foo() {
  const [value, setValue] = useState(0);
  return <button onClick={() => setValue(value + 1)}>{value}</button>;
}
```

### 非推奨の`vnode.[property]`を使用する

Preact Xは内部の`vnode`のプロパティ名に互換性を破壊する変更を加えました。

| Preact 8.x         | Preact 10.x            |
| ------------------ | ---------------------- |
| `vnode.nodeName`   | `vnode.type`           |
| `vnode.attributes` | `vnode.props`          |
| `vnode.children`   | `vnode.props.children` |

### 子要素のkey属性が重複している

仮想DOMベースのライブラリのユニークな点の1つは子要素が移動した場合、それを見つける必要があることです。
だから、どの子要素がどれであるかを知るためにフラグを立てる必要があります。
_これは動的に子要素を生成する場合にのみ必要です。_

```jsx
// Both children will have the same key "A"
<div>
  {['A', 'A'].map(char => <p key={char}>{char}</p>)}
</div>
```

それを行う正しい方法はユニークなキーを付与することです。
ほとんどの場合、反復処理を行うデータは何らかの形で`id`を持っています。

```jsx
const persons = [
  { name: 'John', age: 22 },
  { name: 'Sarah', age: 24}
];

// Somewhere later in your component
<div>
  {persons.map(({ name, age }) => {
    return <p key={name}>{name}, Age: {age}</p>;
  })}
</div>
```

[Preact Devtools]: https://preactjs.github.io/preact-devtools/
