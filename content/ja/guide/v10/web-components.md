---
name: Web Components
description: 'Web ComponentsとPreactを連携させる方法'
---

# Web Components

Preactは軽量なのでWeb Componentsを作成することによく使われます。
具体的には、Preactを使ってWeb Componentsにラップされるコンポーネントを作成します。
これによって、全く異なるフレームワークを使っている他のプロジェクトでもPreactで作成したコンポーネントを簡単に再利用できます。
PreactでWeb Componentsを作成する方法は[preact-custom-element](https://github.com/preactjs/preact-custom-element)を見てください。

Web ComponentsとPreactのゴールが異なるので両者は競合関係にありません。

---

<div><toc></toc></div>

---

## Web Componentsをレンダリングする

Preactから見るとWeb Componentsは単なるスタンダードなDOM要素にすぎません。
PreactはWeb Componentsが登録されたタグ名を使ってレンダリングします。

```jsx
function Foo() {
  return <x-foo />;
}
```

## インスタンスメソッドにアクセスする

`ref`を使うとWeb Componentsのインスタンスにアクセスすることができます。

```jsx
function Foo() {
  const myRef = useRef(null);

  useEffect(() => {
    if (myRef.current) {
      myRef.current.doSomething();
    }
  }, []);

  return <x-foo ref={myRef} />;
}
```

## カスタムイベントをトリガする

PreactはWeb標準のビルトインのDOMイベントに関しては次のような変換処理を行います。
例えば、`onChange`propが`<div>`に渡されると、propの先頭の`on`を除いて小文字に変換した`"change"`をaddEventListenerに渡します。
カスタムイベントの場合は小文字への変換を行わず、先頭の`on`を除いた文字列をそのままaddEventListenerに渡します。

```jsx
// 以下は標準のDOMイベントです。"click"イベントとして登録します。
<div onClick={() => console.log('click')} />

// 以下はカスタムイベントです。
// "IonChange"イベントとして登録します。
<my-foo onIonChange={() => console.log('IonChange')} />
// "ionChange"イベントとして登録します。(iが小文字であることに注目)
<my-foo onionChange={() => console.log('ionChange')} />
```
