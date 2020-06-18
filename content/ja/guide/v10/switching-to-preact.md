---
name: ReactからPreactへの移行
permalink: '/guide/switching-to-preact'
description: 'ReactからPreactへの移行するために必要なことをすべて説明します。'
---

# (Reactから)Preactへの移行

`preact/compat`はReactのエコシステムに存在する多くのライブラリをPreactで使用することができるようにする互換レイヤです。既存のReactアプリケーションをPreactに移行する場合はこれを使うことをお勧めします。

`preact/compat`を使うことによってワークフローやコードベースを変更することなく既存のReact/ReactDOMで作られたコードを成長させていくことができます。
`preact/compat`はバンドルサイズを2kb増加させますが、npmにあるほとんどのReact用のモジュールを利用できるようになるという利点があります。
`preact/compat`パッケージはPreactコアに`react`と`react-dom`と同じように動作するような調整を加えたものです。

---

<div><toc></toc></div>

---

## compatの設定

`preact/compat`を設定するには`react`と`react-dom`を`preact/compat`にエイリアスする必要があります。
バンドラでエイリアスする方法を詳しく知りたい場合は[はじめに](/guide/v10/getting-started#aliasing-react-to-preact)を読んでください。

## PureComponent

`PureComponent`クラスは`Component`クラスと似た動作をします。
違いは新しい`props`が古い`props`と等しい場合、`PureComponent`はレンダリングをスキップする点です。
`PureComponent`はこれを行うために浅い(shallow)比較で古い`props`と新しい`props`を比較して`props`が参照的に等しいか確認します。
これによって不要な再レンダリングを避けることができます。これでアプリケーションは大幅にスピードアップします。

```jsx
import { render } from 'preact';
import { PureComponent } from 'preact/compat';

class Foo extends PureComponent {
  render(props) {
    console.log("render")
    return <div />
  }
}

const dom = document.getElementById('root');
render(<Foo value="3" />, dom);
// Logs: "render"

// 2回目のレンダリングではログを出力しません。
render(<Foo value="3" />, dom);
```

> `PureComponent`はレンダリングコストが高い場合のみ有効です。レンダリングコストが低い場合、`props`を比較するオーバーヘッドよりレンダリングだけした方が早いことがあります。

## memo

`memo`は関数コンポーネントのクラスにおける`PureComponent`に相当します。
デフォルトでは`PureComponent`と同じ比較をしますが、個別に比較を行う関数を設定することもできます。

```jsx
import { memo } from 'preact/compat';

function MyComponent(props) {
  return <div>Hello {props.name}</div>
}

// デフォルトの比較を使用する
const Memoed = memo(MyComponent);

// 比較を行う関数を設定する
const Memoed2 = memo(MyComponent, (prevProps, nextProps) => {
  // `name`が変わった場合のみ再レンダリングする
  return prevProps.name === nextProps.name;
})
```

> `memo`に渡す比較関数は再レンダリングをスキップしたい場合は`true`を返します。`shouldComponentUpdate`は再レンダリングをスキップしたい場合は`false`を返します。両者の戻り値が逆なことに注意してください。

## forwardRef

`forwardRef`を使うとコンポーネントの外部からコンポーネント内部の要素を参照することができます。

```jsx
import { createRef, render } from 'preact';
import { forwardRef } from 'preact/compat';

const MyComponent = forwardRef((props, ref) => {
  return <div ref={ref}>Hello world</div>;
})

// refはMyComponentではなくMyComponent内部の`div`の参照を持ちます。
const ref = createRef();
render(<MyComponent ref={ref} />, dom)
```

これはライブラリの開発にとても役立ちます。

## Portals

`createPortal`を使うとレンダリング時に別のDOM Nodeの下にレンダリングしたものを加えることができます。
ターゲットとなるDOM Nodeはレンダリング時よりも前に**存在している必要があります**。

```html
<html>
  <body>
    <!-- App is rendered here -->
    <div id="app"></div>
    <!-- Modals should be rendered here -->
    <div id="modals"></div>
  </body>
</html>
```

```jsx
import { createPortal } from 'preact/compat';
import MyModal from './MyModal';

function App() {
  const container = document.getElementById('modals');
  return (
    <div>
      I'm app
      {createPortal(<MyModal />, container)}
    </div>
  )
}
```

> Preactはブラウザのイベントシステムを使用しているのでイベントがPortalコンテナを通じて他のツリーにバブルアップしないことを忘れないでください。

## Suspense (実験的な機能)

`Suspense`を使うとSuspenseの下に存在する子コンポーネントがロード中の場合はプレイスフォルダを表示することができます。
一般的なユースケースとして、レンダリングする前にネットワークからコンポーネントをロードする必要があるcode-splittingに使われます。

```jsx
import { Suspense, lazy } from `preact/compat`;

const SomeComponent = lazy(() => import('./SomeComponent'));

// Usage
<Suspense fallback={<div>loading...</div>}>
  <Foo>
    <SomeComponent />
  </Foo>
</Suspense>
```

この例ではUIは`loading...`というテキストを`SomeComponent`コンポーネントがロードされてPromiseがresolveするまで表示します。

> この機能は実験的な機能でバグがあるかもしれません。テストの可視性を高めるためにプレビュー版に含めました。Production環境では使用しないことをお勧めします。
