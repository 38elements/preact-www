---
name: オプションフック
description: 'Preactにはいつくかのオプションフックがあります。それらを使うと差分処理の各段階で実行されるコールバック関数をセットすることができます。'
---

# オプションフック

レンダーオプションはPreactのレンダリングを変更させるプラグインのためのコールバック機構です。

Preactはレンダリングプロセスの各段階での状態を観察または変更するためのコールバック関数を設定することができます。
それらのコールバック関数は一般的にオプションフックと呼ばれます([フック](https://preactjs.com/guide/v10/hooks)と混同しないよう注意してください。)。
それらはPreactの機能を拡張したりPreact向けのテストツールを作成することに使われます。
`preact/hooks`、`preact/compat`、`devtools`はオプションフックを使っています。

このAPIはPreact向けのツールやPreactを拡張するライブラリの開発者向けです。

---

<div><toc></toc></div>

---

## バージョン管理とサポート

オプションフックは`preact`パッケージに含まれています。そして、それはセマンティックバージョンで管理されています
しかし、オプションフックはセマンティックバージョンとは関係なく変更される可能性があります。
これは`VNode`オブジェクトのようにオプションフックで扱う内部APIの構造にも当てはまります。

## オプションフックを設定する

オプションフックはエキスポートされた`options`オブジェクトを変更して設定します。

オプションフックを設定する場合は必ず既存のオプションフックを新しいオプションフック内で実行するように設定してください。
これをしないとコールチェーンが壊れます。そして既存のオプションフックに依存しているコードは正常に動作しません。その結果、`preact/hooks`やDevToolsのようなアドオンは正常に動作しなくなります。
変更する特別な理由がない限り既存のオプションフックにも同じ引数を渡してください。

```js
import { options } from 'preact';

// 既存のvnodeオプションフックを保存する
const oldHook = options.vnode;

// vnodeオプションフックを設定する
options.vnode = vnode => {
  console.log("Hey I'm a vnode", vnode);

  // 既存のオプションフックが存在している場合、それを実行する
  if (oldHook) {
    oldHook(vnode);
  }
}
```

eventオプションフック以外のオプションフックは戻り値を返しません。それらは戻り値を気にする必要はありません。

## 利用可能なオプションフック

#### `options.vnode`

**API:** `(vnode: VNode) => void`

最も一般的なオプションフックです。
`vnode`オプションフックはVNodeオブジェクトが生成されるたびに実行されます。
VNodeはPreactの仮想DOM要素を表します。それはJSX要素と呼ばれることもあります。

#### `options.unmount`

**API:** `(vnode: VNode) => void`

vnodeがアンマウントされる直前(DOMが存在している時)に実行されます。

#### `options.diffed`

**API:** `(vnode: VNode) => void`

vnodeがレンダリングされた直後に実行されます。

#### `options.event`

**API:** `(event: Event) => any`

DOMイベントが関連する仮想DOMのイベントリスナに渡される直前に実行されます。
`options.event`が設定されている場合、イベントリスナの引数の`event`は`options.event`の戻り値に置き換えられます。

#### `options.requestAnimationFrame`

**API:** `(callback: () => void) => void`

`preact/hooks`内の副作用と副作用ベースの処理のスケジューリングを行う関数を設定します。

#### `options.debounceRendering`

**API:** `(callback: () => void) => void`

コンポーネントがレンダリングされる処理を遅延させる処理を行う関数を設定します。

`options.debounceRendering`はデフォルトでは`Promise.resolve()`を遅延処理に使います。Promiseが使えない場合は`setTimeout`を使います。

#### `options.useDebugValue`

**API:** `(value: string | number) => void`

`useDebugValue`フックが実行された時に実行されます。
