---
name: Enzymeを使って単体テストを行う
permalink: '/guide/unit-testing-with-enzyme'
description: 'Enzymeを使ってPreactアプリケーションの単体テストを行う。'
---

# Enzymeを使って単体テストを行う

Airbnbの[Enzyme](https://airbnb.io/enzyme/)はReactのコンポーネントをテストするためのライブラリです。
Enzymeはアダプターを使用することで異なるバージョンのReactとそれに類するライブラリをサポートします。
PreactチームがメンテナンスしているPreact用のアダプターがあるのでEnzymeを使ってPreactのコンポーネントのテストを行うことができます。

Enzymeは[Karma](http://karma-runner.github.io/latest/index.html)を使った通常のブラウザやヘッドレスブラウザでのテストや
NodeJSで仮想的なブラウザAPIの実装である[jsdom](https://github.com/jsdom/jsdom)を使ったテストをサポートします。

Enzymeの詳しい使い方とAPIリファレンスは[Enzymeのドキュメント](https://airbnb.io/enzyme/)を見てください。
このガイドの残りの部分はEnzymeをPreactで設定する方法とPreactと一緒に使う場合とReactと一緒に使う場合の違いについて説明します。

---

<div><toc></toc></div>

---

## インストール

以下のようにEnzymeとPreactアダプターをインストールします。

```sh
npm install --save-dev enzyme enzyme-adapter-preact-pure
```

## 設定

Preactのアダプターを使うためにテストのセットアップコードでEnzymeの設定を行う必要があります。

```js
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-preact-pure';

configure({ adapter: new Adapter() });
```

各テストランナーでEnzymeを使う方法についてはEnzymeのドキュメントの[Guides](https://airbnb.io/enzyme/docs/guides.html)セクションを見てください。

## 例

以下のような初期値を表示し、それを更新するボタンがあるシンプルな`Counter`コンポーネントがあるとします。

```jsx
import { h } from 'preact';
import { useState } from 'preact/hooks';

export default function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);
  const increment = () => setCount(count + 1);

  return (
    <div>
      Current value: {count}
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

以下のようにMochaやJest等のテストランナーを使って、期待通りに動作するかチェックするテストを書くことができます。

```jsx
import { expect } from 'chai';
import { h } from 'preact';
import { mount } from 'enzyme';

import Counter from '../src/Counter';

describe('Counter', () => {
  it('should display initial count', () => {
    const wrapper = mount(<Counter initialCount={5}/>);
    expect(wrapper.text()).to.include('Current value: 5');
  });

  it('should increment after "Increment" button is clicked', () => {
    const wrapper = mount(<Counter initialCount={5}/>);

    wrapper.find('button').simulate('click');

    expect(wrapper.text()).to.include('Current value: 6');
  });
});
```

サポートしているバージョンや他の例はPreact用アダプターのレポジトリの[examples/](https://github.com/preactjs/enzyme-adapter-preact-pure/blob/master/README.md#example-projects)ディレクトリを見てください。

## Enzymeの仕組み

Enzymeはコンポーネントとその子コンポーネントをレンダリングするために設定されたアダプターライブラリを使います。
それから、アダプターはアプトプットを標準化された内部表現("React Standard Tree")に変換します。
それから、Enzymeはアウトプットを検索し更新をトリガするメソッドを持つオブジェクトをラップします。
ラッパオブジェクトのAPIはCSSに似た[selectors](https://airbnb.io/enzyme/docs/api/selector.html)を使って該当部分を見つけます。

## フルレンダリング、浅い(shallow)レンダリング、文字列レンダリング

Enzymeは3つのレンダリングモードがあります。

```jsx
import { mount, shallow, render } from 'enzyme';

// コンポーネントツリー全体をレンダリングします。
const wrapper = mount(<MyComponent prop="value"/>);

// MyComponentのDOMノードのみレンダリングします。(子コンポーネントはモックされてプレイスフォルダでレンダリングします。)
const wrapper = shallow(<MyComponent prop="value"/>);

// コンポーネントツリーをHTMLでレンダリングしてパースしたものを返します。
const wrapper = render(<MyComponent prop="value"/>);
```

 - `mount`関数はコンポーネント全体をブラウザでレンダリングするのと同じ方法でレンダリングします。

 - `shallow`関数はコンポーネント内のDOMノードのみレンダリングします。
   子コンポーネントはそれを表すプレイスフォルダに置き換えられます。

   このモードの利点は子コンポーネントに依存しないのでその依存関係を解決することなくテストを書くことができます。

   浅い(shallow)レンダリングモードは内部の動作がPreact用のアダプターとReact用のアダプターでは異なります。詳しくは以下の「Reactとの違い」のセクションを見てください。

 - `render`関数(Preactの`render`関数と混同しないでください)はコンポーネントをHTML文字列にレンダリングします。
   これはサーバー上で出力をテストすることやエフェクトのトリガ無しでコンポーネントをレンダリングすることに役立ちます。

## `act`でステートの更新とその効果をトリガする

前の例では、`.simulate('click')`を使ってボタンをクリックしました。

Enzymeは`simulate`によってステート(state)が更新されそれの効果がトリガされることを知っています。だから、Enzymeは`simulate`関数が終了する直前でステートを更新して効果をトリガします。
Enzymeは最初に`mount`や`shallow`を使ってレンダリングする時や`setProps`を使ってコンポーネントを更新する時も同じ処理を行います。

しかし、Enzymeのメソッド以外でイベントが発生した場合、例えばイベントハンドラ(ボタンの`onClick` prop)を直接実行した場合、Enzymeは変化に気づきません。
この場合、テストはステートの更新とその効果のトリガを実行する必要があります。そして、Enzymeにアウトプットであるビューを再生成させる必要があります。

- ステートの更新とその効果を同期的に実現するために、`preact/test-utils`の`act`関数を更新をトリガするコードをラップして使います。
- `update`メソッドでレンダリングされたアウトプットのEnzymeのビューを更新します。

例えば、以下にカウンターを加算するテストの別バージョンを示します。
`simulate`メソッドを経由する代わりにボタンのonClick propを実行するように変更しました。

```js
import { act } from 'preact/test-utils';
```

```jsx
it('should increment after "Increment" button is clicked', () => {
    const wrapper = mount(<Counter initialCount={5}/>);
    const onClick = wrapper.find('button').props().onClick;

    act(() => {
      // ボタンのクリックハンドラを実行しますが、今回はEnzyme APIを経由する代わりに直接実行します。
      onClick();
    });
    // Enzymeのアウトプットのビューを再生成します。
    wrapper.update();

    expect(wrapper.text()).to.include('Current value: 6');
});
```

## Reactとの違い

理想はReact + Enzymeで書かれたテストがEnzyme + Preactでもその逆でも簡単に動作させることができることです。
これによって、ReactとPreactを切り替える際にテストを書き直さなくてよくなります。

しかし、Preact用のアダプターとReact用のアダプターには注意すべき違いがあります。

- 内部で浅い(shallow)レンダリングは異なる動作をします。
  1階層だけレンダリングすることは両者とも同じですが、React用とは異なり、Preact用のアダプターの方は実際のDOMノードを作成ます。そして通常のライフサイクルフックと副作用を実行します。
- `simulate`メソッドはPreat用のアダプターでは実際のDOMイベントをディスパッチします。React用のアダプターでは`on<EventName>` propを呼ぶだけです。
- Preactではステート(state)の更新(`setState`を実行した後)はまとめて行われ、非同期で適用されます。
  Reactではステートの更新はコンテキストに応じて直接もしくはまとめて適用されます。
  テストを書きやすくするために、Preact用のアダプターは`setState`もしくは`setProps`によるレンダリングとステートの更新の後にステートの更新とその効果を全部反映します。
  ステートの更新またはその効果がそれ以外の方法で更新された場合、テストコードで`preact/test-utils`パッケージの`act`をステートの更新とその効果を手動でトリガしてまとめて実行する必要があります。

より詳しい説明は[Preact用のアダプターのREADME](https://github.com/preactjs/enzyme-adapter-preact-pure#differences-compared-to-enzyme--react)を見てください。
