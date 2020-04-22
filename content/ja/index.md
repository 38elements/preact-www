---
layout: home
title: Preact
show_title: false
toc: false
description: 'Preact is a fast 3kB alternative to React with the same modern API'
---

<jumbotron>
    <h1>
        <logo height="1.5em" title="Preact" text inverted>Preact</logo>
    </h1>
    <p class="tagline">Fast 3kB alternative to React with the same modern API.</p>
    <p class="intro-buttons">
        <a href="/guide/v10/getting-started" class="btn primary">はじめに</a>
        <a href="/guide/v10/switching-to-preact" class="btn secondary">Preactへ移行する</a>
    </p>
</jumbotron>

```jsx
function Counter() {
  const [value, setValue] = useState(0);

  return (
    <>
      <div>Counter: {value}</div>
      <button onClick={() => setValue(value + 1)}>Increment</button>
      <button onClick={() => setValue(value - 1)}>Decrement</button>
    </>
  )
}
```

<section class="sponsors">
  <p>誇らしい<a href="https://opencollective.com/preact">スポンサー</a></p>
  <sponsors></sponsors>
</section>

<section class="home-top">
    <h1>Preactの特徴</h1>
</section>

<section class="home-section">
  <img src="/assets/home/metal.svg" alt="metal">

  <div>
    <h2>DOMに近い</h2>
    <p>
      PreactはDOMをベースに可能な限り薄い仮想DOMによる抽象化を提供します。
      安定したプラットフォームの機能をベースに構築され、実際のイベントハンドラを登録します。そして、他のライブラリとうまく連携します。
    </p>
    <p>
      Preactはトランスパイルなしで直接ブラウザで使用することができます。
    </p>
  </div>
</section>

<section class="home-section">
  <img src="/assets/home/size.svg" alt="size">

  <div>
    <h2>サイズが小さい</h2>
    <p>
      ほとんどのUIフレームワークはアプリケーションを構成するJavaScriptのサイズの大部分を占めると言っていいくらいの大きいです。
      Preactは違います。<em>あなたが書いたコード</em>がアプリケーションの大部分を占めると言っていいくらい小さいです。
    </p>
    <p>
      つまり、ダウンロードしてパースして実行するJavaScriptが少ないので、コードを書くことにより多くの時間を費やすことができます。フレームワークを使いこなすことに苦闘することなく機能を開発することができます。
    </p>
  </div>
</section>

<section class="home-section">
  <img src="/assets/home/performance.svg" alt="performance">

  <div>
    <h2>卓越したパフォーマンス</h2>
    <p>
      Preactが高速な理由は単にサイズだけではありません。
      シンプルで堅実な差分機能の実装よって最も高速な仮想DOMライブラリの1つになりました。
    </p>
    <p>
      私達はPreactを改善し続けて限界までチューニングしています。
      私達は可能な限り最大限Preactのパフォーマンスを引き出せるようにブラウザの開発者と親密に協力しています。
    </p>
  </div>
</section>

<section class="home-section">
  <img src="/assets/home/portable.svg" alt="portable">

  <div>
    <h2>応用できる範囲が広い & 組み込みに適している</h2>
    <p>
      Preactの小さいサイズは他のライブラリが進出できないような新しい用途を開拓しました。
    </p>
    <p>
      Preactを使うと複雑なインテグレーションなしでアプリケーションのパーツを実装できます。
      Preactをウィジットに組み込む際はアプリケーション全体を構築する時と同じツールやテクニックを使用できます。
    </p>
  </div>
</section>

<section class="home-section">
  <img src="/assets/home/productive.svg" alt="productive">

  <div>
    <h2>すぐに生産性を上げる</h2>
    <p>
      生産性を犠牲にしなくてよいなら軽量フレームワークはとても楽しいです。Preactはすぐに生産性を上げます。ボーナス機能もあります。
    </p>
    <ul>
      <li><code>props</code>、<code>state</code>、<code>context</code>は<code>render()</code>に渡されます。</li>
      <li><code>class</code>や<code>for</code>のような標準的なHTML属性を使用します。</li>
    </ul>
  </div>
</section>

<section class="home-section">
  <img src="/assets/home/compatible.svg" alt="compatible">

  <div>
    <h2>エコシステムの互換性</h2>
    <p>
      仮想DOMコンポーネントはボタンからデータプロバイダまですべての再利用可能なコンポーネントを共有することを簡単にします。
      PreactはReactのエコシステムにある何千ものコンポーネントをシームレスに利用できるように設計されています。
    </p>
    <p>
      バンドラーに<a href="/guide/v10/switching-to-preact#how-to-alias-preact-compat">preact/compat</a>を加えるちょっとした変更を加えることで
      複雑なReactコンポーネントでもアプリケーションで使用することができる互換レイヤーを提供することができます。
    </p>
  </div>
</section>

<section class="home-top">
    <h1>動くものを見てみましょう。</h1>
</section>

<section class="home-split">
    <div>
        <h2>Todoリスト</h2>
        <pre><code class="lang-jsx">
export default class TodoList extends Component {
    state = { todos: [], text: '' };
    setText = e =&gt; {
        this.setState({ text: e.target.value });
    };
    addTodo = () =&gt; {
        let { todos, text } = this.state;
        todos = todos.concat({ text });
        this.setState({ todos, text: '' });
    };
    render({ }, { todos, text }) {
        return (
            &lt;form onSubmit={this.addTodo} action="javascript:"&gt;
                &lt;label&gt;
                  &lt;span&gt;Add Todo&lt;/span&gt;
                  &lt;input value={text} onInput={this.setText} /&gt;
                &lt;/label&gt;
                &lt;button type="submit"&gt;Add&lt;/button&gt;
                &lt;ul&gt;
                    { todos.map( todo =&gt; (
                        &lt;li&gt;{todo.text}&lt;/li&gt;
                    )) }
                &lt;/ul&gt;
            &lt;/form&gt;
        );
    }
}
        </code></pre>
    </div>
    <div>
        <h2>実行結果</h2>
        <pre repl="false"><code class="lang-jsx">
import TodoList from './todo-list';

render(&lt;TodoList /&gt;, document.body);
        </code></pre>
        <div class="home-demo">
            <todo-list></todo-list>
        </div>
    </div>
</section>

<section class="home-split">
    <div>
        <h2>GitHubのスター数を取得</h2>
        <pre><code class="lang-jsx">
export default class Stars extends Component {
    async componentDidMount() {
        let stars = await githubStars(this.props.repo);
        this.setState({ stars });
    }
    render({ repo }, { stars=0 }) {
        let url = `https://github.com/${repo}`;
        return (
            &lt;a href={url} class="stars"&gt;
                ⭐️ {stars} Stars
            &lt;/a&gt;
        );
    }
}
        </code></pre>
    </div>
    <div>
        <h2>実行結果</h2>
        <pre repl="false"><code class="lang-jsx">
import Stars from './stars';

render(
    &lt;Stars repo="preactjs/preact" /&gt;,
    document.body
);
        </code></pre>
        <div class="home-demo">
            <github-stars simple user="preactjs" repo="preact"></github-stars>
        </div>
    </div>
</section>

<section class="home-top">
    <h1>Preactについてもっと知りたいですか?</h1>
</section>

<section style="text-align:center;">
    <p>
        Reactの経験があるかどうかに応じてガイドを別けています。
        <br>
        最適なガイドを選びましょう。
    </p>
    <p>
        <a href="/guide/v10/getting-started" class="btn primary">はじめに</a>
        <a href="/guide/v10/switching-to-preact" class="btn secondary">Preactへ移行する</a>
    </p>
</section>
