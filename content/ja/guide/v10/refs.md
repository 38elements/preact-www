---
name: リファレンス(Ref) 
description: 'Preactがレンダリングした生のDOM Nodeにアクセスするにはリファレンス(Ref)を使います。'
---

# リファレンス(Ref)

PreactがレンダリングしたDOM要素やコンポーネントのDOM Nodeを使う必要がある時がよくあります。その時はリファレンス(Ref)を使います。

典型的なユースケースはDOM Nodeの実際の大きさを測るケースです。

コンポーネントに`ref`属性を付与してコンポーネントインスタンスの参照を取得することは可能ですが、これは一般的に推奨されません。
これは親と子の間に強い結合関係が生じます。これはコンポーネントネントモデルのコンポーサビリティを損ないます。
ほとんどの場合でクラスコンポーネントのメソッドを直接実行するよりもコールバック関数をpropsにセットした方が違和感がありません。

---

<div><toc></toc></div>

---

## `createRef`

`currentRef`関数は`current`プロパティがセットされる空のオブジェクトを返します。
以下のように`currentRef`関数の戻り値を`ref`属性にセットすると、`render`メソッドが実行される度、DOM Nodeもしくはコンポーネントインスタンスを`current`プロパティに代入します。

```jsx
class Foo extends Component {
  ref = createRef();

  componentDidMount() {
    console.log(this.ref.current);
    // Logs: [HTMLDivElement]
  }
  
  render() {
    return <div ref={this.ref}>foo</div>
  }
}
```

## リファレンスコールバック

もう1つの要素の参照を取得する方法は以下のようにコールバック関数を渡すことです。
少し長くなりますが`createRef`と似たような動作をします。

```jsx
class Foo extends Component {
  ref = null;
  setRef = (dom) => this.ref = dom;

  componentDidMount() {
    console.log(this.ref);
    // Logs: [HTMLDivElement]
  }
  
  render() {
    return <div ref={this.setRef}>foo</div>
  }
}
```

> ref属性へ渡すコールバック関数がインライン関数で定義されている場合、コールバック関数は2回実行されます。1回目は`null`、2回目は実際の参照が渡されます。これはエラーの原因になることがよくあります。`createRef`APIの場合は`ref.current`が定義されているか確認します。

## まとめ

DOM Nodeの横幅と高さを測るためにDOM Nodeのリファレンスを取得する必要があるケースがあるとします。
プレイスフォルダを実際に測った値に置き換えるだけのシンプルなコンポーネントがあります。

```jsx
class Foo extends Component {
  // ここではDOM Nodeの実際の横幅と高さを使用したいと思います
  state = {
    width: 0,
    height: 0,
  };

  render(_, { width, height }) {
    return <div>Width: {width}, Height: {height}</div>;
  }
}
```

計測が意味を持つのは`render`メソッドが実行されてコンポーネントがDOMにマウントされた後です。
それより前はDOM Nodeが存在していないので、測定しても意味がありません。

```jsx
class Foo extends Component {
  state = {
    width: 0,
    height: 0,
  };

  ref = createRef();

  componentDidMount() {
    // 安全のために: リファレンスがあるかチェックする
    if (this.ref.current) {
      const dimensions = this.ref.current.getBoundingClientRect();
      this.setState({
        width: dimensions.width,
        height: dimensions.height,
      });
    }
  }

  render(_, { width, height }) {
    return (
      <div ref={this.ref}>
        Width: {width}, Height: {height}
      </div>
    );
  }
}
```

これでコンポーネントは常にマウントされた時に横幅と高さを表示します。
