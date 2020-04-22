---
name: コンテキスト
description: 'コンテキストは間にあるコンポーネントを飛ばしてpropsを渡すことができます。このドキュメントは新しいAPIと古いAPIの両方を説明します。'
---

# コンテキスト

コンテキストは深い階層にある子コンポーネントとの間にあるすべてのコンポーネントの`props`を介することなく深い階層にあるコンポーネントに値を渡すことができます。一般的なユースケースはテーマの設定です。端的に言うとコンテキストはPreactでPub-Subスタイルの更新を行う方法だと考えることができます。

コンテキストには2通りの使用方法があります。それは新しい`createContext`APIと古いコンテキストAPIです。<br>
2つの違いは古い方は間にあるコンポーネントが`shouldComponentUpdate`でレンダリングを中止した場合、子コンポーネントを更新しないことです。<br>
これを理由に、常に`createContext`を使うことを強く推奨します。

---

<div><toc></toc></div>

---

## createContext

最初にコンポーネント間を通過することができるコンテキストオブジェクを生成する必要があります。<br>
これは`createContext(initialValue)`関数で行います。<br>
`createContext(initialValue)`はコンテキストの値をセットするために使われる`Provider`コンポーネントとコンテキストから値を受け取る`Comsumer`コンポーネントを返します。

```jsx
const Theme = createContext('light');

function ThemedButton(props) {
  return (
    <Theme.Consumer>
      {theme => {
        return <button {...props} class={'btn ' + theme}>Themed Button</button>;
      }}
    </Theme.Consumer>
  );
}

function App() {
  return (
    <Theme.Provider value="dark">
      <SomeComponent>
        <ThemedButton />
      </SomeComponent>
    </Theme.Provider>
  );
}
```

> [useContext](/guide/v10/hooks#context)フックを使うと簡単にコンテキストを扱うことができます。

## 古いコンテキストAPI

古いコンテキストAPIは主に下位互換性のために存在しています。<br>
古いコンテキストAPIは`createContext`APIに置き換えられました。<br>
古いコンテキストAPIは間にあるコンポーネントが`shouldComponentUpdate`で`false`を返した場合、更新しない既知の問題があります。<br>
それにもかかわらず使う必要があるなら読み続けてください。

コンポーネントがコンテキストを介してデータを子コンポーネントに渡すには、コンポーネントは`getChildContext`メソッドを持つ必要があります。<br>
`getChildContext`メソッドの返す値は既にコンテキストに格納されている値とマージされます。<br>
コンテキストは関数コンポーネントの第2引数もしくはクラスコンポーネントの`this.context`でアクセスすることができます。

```jsx
function ThemedButton(props, context) {
  return (
    <button {...props} class={'btn ' + context.theme}>
      Themed Button
    </button>;
  );
}

class App extends Component {
  getChildContext() {
    return {
      theme: 'light'
    }
  }

  render() {
    return (
      <div>
        <SomeOtherComponent>
          <ThemedButton />
        </SomeOtherComponent>
      </div>
    );
  }
}
```
