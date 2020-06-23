---
name: フォーム
description: 'Preactでどこでも動作する素晴らしいフォームを作成する方法'
---

# フォーム

PreactのフォームとHTMLのフォームは、ほとんど同じように動作します。両方ともコントロールをレンダリングしイベントリスナをセットします。

両者の主な違いは、ほとんどの場合で`value`の管理をDOMノードもしくはコンポーネントのどちらが行うかです。

---

<div><toc></toc></div>

---

## ControlledコンポーネントとUncontrolledコンポーネント

フォームコントロールについての文章で"Controlled"コンポーネントと"Uncontrolled"コンポーネントという単語をよく見ます。
 "Controlled"と"Uncontrolled"はデータフローの扱い方を表しています。
すべてのフォームコントロールはユーザの入力を自身で管理するためにDOMは双方向のデータフローを採用しています。
例として、ユーザがテキスト`input`要素にタイプすると、その値は必ず反映されます。

それとは対象的にPreactのようなフレームワークは一般的に単方向のデータフローを採用しています。
Preactでは、ほとんどの場合、DOM(例えばinputテキスト要素)は値を管理しません、コンポーネント(例えばinputテキスト要素を持つコンポーネント)が管理します。

Controlledコンポーネントはコンポーネントが入力コントロールの値を管理するコンポーネントです。
UnControlledコンポーネントはコンポーネントが入力コントロールの値を管理しないコンポーネントです。

```jsx
// これはControlledコンポーネントです。Preactは入力されたデータを管理します。
<input value={someValue} onInput={myEventHandler} />;

// これはUncontrolledコンポーネントです。Preactはvalueに値をセットしません。DOMが値を管理します。
<input onInput={myEventHandler} />;
```

一般的に常にControlledコンポーネントを使うべきです。
しかし、UncontrolledコンポーネントはスタンドアロンなコンポーネントやサードパーティのUIライブラリをラップする場合に使用すると便利です。

> value属性の値を`undefined`もしくは`null`にセットすると`Uncontrolled`になることに注意してください。

## 簡単なフォームの作成

ToDoを送信する簡単なフォームを作成しましょう。
`<form>`要素を作成します。そして、`<form>`要素のonSubmitにイベントハンドラをセットします。
テキストinputフィールドにも同様のことをしますが、値を`setState()`を使ってTodoFormコンポーネントに保存していることに注目してください。
お察しの通り、ここではControlledコンポーネントを使っています。
この例では、入力した値を別の要素に表示する必要が有るので、Controlledコンポーネントが適しています。

```jsx
class TodoForm extends Component {
  state = { value: '' };

  onSubmit = e => {
    alert("Submitted a todo");
    e.preventDefault();
  }

  onInput = e => {
    const { value } = e.target;
    this.setState({ value })
  }

  render(_, { value }) {
    return (
      <form onSubmit={this.onSubmit}>
        <input type="text" value={value} onInput={this.onInput} />
        <p>You typed this value: {value}</p>
        <button type="submit">Submit</button>
      </form>
    );
  }
}
```

## Select要素

`<select>`要素はもう少し複雑ですが、他のフォームコントロールと同じように動作します。

```jsx
class MySelect extends Component {
  state = { value: '' };

  onChange = e => {
    this.setState({ value: e.target.value });
  }

  onSubmit = e => {
    alert("Submitted " + this.state.value);
    e.preventDefault();
  }

  render(_, { value }) {
    return (
      <form onSubmit={this.onSubmit}>
        <select value={value} onChange={this.onChange}>
          <option value="A">A</option>
          <option value="B">B</option>
          <option value="C">C</option>
        </select>
        <button type="submit">Submit</button>
      </form>
    );
  }
}
```

## CheckboxとRadioボタン

HTMLではブラウザがcheckboxやradioボタンをトグルやチェックします。
Controlled Componentではトグルやチェックを実装する必要があります。
だから、最初にControlled ComponentでFormを作成する際は混乱するかもしれません。

Controlledコンポーネントでcheckboxをチェックしたりチェックを外したりした際に発火する"change"イベントを監視しているとしましょう。
changeイベントハンドラではcheckboxから受け取ったEventオブジェクトにあるcheckedの値をコンポーネントの`state`にセットします。
これによって、コンポーネントの再レンダリングが発動します。その中で先程`state`にセットされたcheckedの値をcheckboxに反映します。

```jsx
class MyForm extends Component {
  toggle = event => {
      let checked = event.target.checked;
      this.setState({ checked });
  };

  render(_, { checked }) {
    return (
      <label>
        <input
          type="checkbox"
          checked={checked}
          onChange={this.toggle}
        />
      </label>
    );
  }
}
```

しかし、Eventオブジェクトにあるcheckedの値を使用する必要はありません。なぜなら、コンポーネントはcheckboxにcheckedの値を提供しているので変更前のcheckedの値を持っているからです。
だから、`change`イベントではなく`click`イベントを監視します。`click`イベントはチェックボックスや_それに関連した`<label>`_をクリックするたび発火します。
checkboxはBooleanである`true`と`false`を切り替えるだけです。checkboxやlabelをクリックするとstateにあるcheckedに該当する値を反転して意図した値にします。
すると再レンダリングが発動しcheckboxに反映されます。

```jsx
class MyForm extends Component {
  toggle = () => {
      let checked = !this.state.checked;
      this.setState({ checked });
  };

  render(_, { checked }) {
    return (
      <label>
        <input
          type="checkbox"
          checked={checked}
          onClick={this.toggle}
        />
      </label>
    );
  }
}
```
