# Vuexによるデータフローの設計・状態管理

書籍「Vue.js入門 基礎から実践アプリケーション開発まで」７章まとめ。  
[Vuex公式はこちら](https://vuex.vuejs.org/ja/)

このフォルダ内で以下のコマンドを順に実行すると動作を確認できます。

1. `npm install`
2. `npm run serve`

## 導入

データフロー：アプリケーションの状態がどこから、どのように読み書きされるかを表す.  
　→データの保管場所・データ読み込みや更新の方法を指す事が多い。

特に大規模な開発ではデータフローの良し悪しで実装の難易度が変わってしまう。  

本書7章のサンプルアプリを動かしてみましょう。  

* 保持すべきデータがたくさんあり、これらの保存・復元機能がある
* 各項目の値が変更されるたびに状態を変更
* 必要なタイミングで保存処理を行う

これらの処理を一から実装するのは大変。  
大きなアプリだといずれ破綻するかも。。  

## Vuexとは

[公式サイト](https://vuex.vuejs.org/ja/)によると、、
```
Vuex は Vue.js アプリケーションのための 状態管理パターン + ライブラリです。
```

つまり、、データフローをいい感じにてくれるツールです。  
データの保管・データ読込・更新の手段を非常に使いやすいインターフェースで提供してくれてます。

## 7.2 データフローの設計

* 単方向データフロー   
<img src="https://vuex.vuejs.org/flow.png" width="400px">

以下のサンプルをみてみましょう。  
Viewが直接stateの値を変更しているのではなく、Actionを介してstateの値を変更しているのがわかります。

```
const store = {
  //状態
  state: {
    count: 0
  },

  // 更新処理(Action)
  increment() {
    this.state.count += 1
  }
}

new Vue({
  template: 
  <div>
    <p>{{ count }}</p>
    <button v-on:click="increment">+</button>
  </div>
  ,

  data: store.state,

  methods: {
    increment () {
      store.increment() // 良い例(単方向データフロー)。Actionを介してstateの値を変更している。
      // store.state.count += 1 //悪い例(双方向データフロー)。Viewが直接stateの値を変更している。
    }
  }
})
```

悪い例(双方向データフロー)の場合、以下の問題が発生します。
* 同じような更新を別のコンポーネントから行いたい場合、再利用が難しい。
* 更新処理に変更を加えたい場合、修正すべき箇所が増える。
* データの取得と更新が同時になり、デバッグが難しくなる。

より良いデータフローの設計のためには、以下の原則を守りましょう。

* 信頼できる唯一の情報源
* 「状態の取得・更新」のカプセル化
* 単方向データフロー

### 7.2.1 信頼できる唯一の情報源

管理する対象のデータを一箇所に集約すること

メリット
* どのコンポーネントも同一のデータを参照するため、データ・表示の不整合が起こりにくい
* 複数のデータを組み合わせやすい

### 7.2.2 「状態の取得・更新」のカプセル化

更新・取得処理をstore内に記述し、コンポーネントからは具体的な実装が隠されている。

メリット
* 情報の取得・更新ロジックの再利用
* デバッグ時に確認する箇所がへる
* 処理変更時の修正箇所が少なくなる

### 7.2.3単方向データフロー

データの取得と更新の窓口が異なるようにし、ビューが直接stateの値を変更できなくする。

メリット
* データ取得・更新のメソッドが限られるので、理解しやすい。
* データ更新と取得が同時にできないので、デバッグが用意.

## 7.3 vuexによる状態管理

Vuexはvuejsアプリ向けの状態管理ライブラリ。  
以下の3つのデータフロー設計要件を満たしているため、変更に強く、管理しやすい。

* 信頼できる唯一の情報源  
　→アプリの状態とそれに付随するロジックが一箇所にまとまっている
* 「状態の取得・更新」のカプセル化
　→状態の更新はミューテーション、取得はゲッターを仕様し、詳細な実装は隠蔽できる。そのため、「情報の取得・更新のカプセル化」が実現されている。
* 単方向データフロー  
　→情報の取得と変更の窓口が異なっているため、実装が単一方向データフローになる。

さらにVueDevToolsで状態の変化をログで見ることができる。  
　→実際にやって見よう。

### 7.3.1 Vuexのインストール

```
$ npm install vuex
```

インストール後はimport 文で読み込む(store.js を参考に)
```
import Vue from 'vue'
import Vuex from 'vuex'

// vueにvuexを登録
Vue.use(Vuex)

// ストアの定義
const store = new Vuex.Store({
  ...
})
```

## 7.4 Vuexのコンセプト

Vuexはライブラリとしての機能だけでなく、実装パターンも含めてVuexである。

### 7.4.1 ストア

ストア： アプリケーションの状態を保持する。他にも状態管理に関する機能を盛り込んであり、Vuexの根幹。

```
// ストアの作成と代入
const store = new Vuex.Store({ /* オプション */ })
```

Vuexは「信頼できる唯一の情報源」であることを前提に実装されている。  

```アプリケーション内でストアが１つのみ存在するようにしましょう。```

ストアの構成概念は以下の4つ

* アプリケーションのステート
* ステートの一部や、ステートから計算された値を返すゲッター(Getter)
* ステートを更新するミューテーション(Mutation)
* Ajaxのような非同期処理や、LocalStorageへの読み書きのような外部APIとのやりとりを行うアクション(Action)

<img src="https://vuex.vuejs.org/vuex.png" width="500px">

#### ストアの作成

ステートとしてcountを増減させる例

```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

// 7.4.1 ストアの定義
const store = new Vuex.Store({
  // 7.4.2 ステート
  state: {
    count: 0
  },
  // 7.4.3 ゲッター
  getters: {
    squared: (state) => state.count * state.count
  },
  // 7.4.4 ミューテーション
  mutations: {
    increment (satate, payload) {
      // ペイロード内の値を使ってステートを更新
      state.count += payload.amount
    }
  },
  // 7.4.5 アクション
  actions: {
    incrementAction (ctx) {
      /*
       * この辺でajax呼んだりする。
       */

      // `increment`ミューテーションを実行する
      ctx.commit('increment', { amount: 5 })
    }
  }
})

// ミューテーション呼出。第二引数をペイロードという。
store.commit('increment', { amount: 5 })
```

### 7.4.2 ステート

ステートとはアプリケーション全体の状態を保持するオブジェクト。
データ保持の手法の使い分けには以下のような指標がある。

```アプリケーション全体で保持すべき・複数ーのコンポーネントで仕様するデータはストア内で持つべき```  
```特定のコンポーネントでしか使わないデータはコンポーネント内のdataオブジェクトで持つべき```

```
(例)
◆ストア内で持つべきデータ
　・サーバーからデータを取得中かどうかを表すフラグ
　・ログイン中ーのユーザー情報など。アプリ全体で仕様するデータ
　・ECサイトにおける商品データなど、複数の場所で使用されるデータ

◆コンポーネントで持つべきデータ(ストアで持つべきでない)
　・マウスポインタがある要素の上に存在するかどうか
　・ドラッグ中の要素の座標
　・入力中のフォームの値
```

実装例は上記参照。  
ステートの特徴として重要な点は、  
```ステートの変更は常に追跡されている。ステートに何らかの変更が加わると、その変更は自動的にコンポーネントの算出プロパティやテンプレートに反映される。(vuedevtool参照)```

### 7.4.3 ゲッター

ゲッターはステートから別の値を算出するのに使用。  
gettersオプションに関数を持つオブジェクトを使用する。

実装例は上記参照。  
ステートの特徴として重要な点は、  

```ここでは計算した値を返す以外の処理は行うべきでない```  

### 7.4.4 ミューテーション

実装例は上記参照。 
ミューテーションはステートを更新するために用いる。  
```Vuexではミューテーション以外がステートを更新することを禁止している。```  
```ミューテーションは直接呼出せない。store.commitを使用。```  
```ミューテーション内で行う処理は全て同期的に行う必要がある。```  

ちなみにvuedevtoolを使用すると、ステートの変化と、変化を引き起こしたミューテーションが時系列順で確認できる。  
また、非同期処理を行いたい場合は次に紹介するアクションという機能を代わりに使用すること。

### 7.4.5 アクション

アクションは非同期処理や外部APIとの通信を行い、最終的にミューテーションを呼び出すために用いる。  
サンプルコードだと画面(ビューから呼び出されている？)  
　→保存・復元ボタン押下時に呼び出されているようです。

実装例は上記参照。 
アクションの定義は```第一引数にコンテキストと呼ばれるオブジェクトが渡される```  

コンテキストには以下が含まれる。
* state : 現在のステート
* getters : 定義されているゲッター
* dispatch : 他のアクションを実行するメソッド
* commit : ミューテーションを実行するメソッド

上の例では「ctx」オブジェクトに[state getters dispatch commit]全て内包されているが、
```第一引数のコンテキストを分割代入も可能```。サンプルコードの「store.js」を参照。

```
// ctxオブジェクト使用例
actions: {
  xxxAction (ctx){
    ctx.commit('xxxxxxMutetion')
  },
  ...

// 分割代入例
actions: {
  xxxAction ({ commit}, payload) {
    commit('xxxxxxMutetion', {
      somedata: payload.somedata
    })
  }
}

// 非同期処理を呼び出す例
actions: {
  xxxAsyncAction ({ commit }, payload) {
    return someAsyncMethod(payload.somedata)
      
      .then(data => {
        // 非同期処理の戻り値をミューテーションのペイロードとして渡す
        commit('xxxxxxMutetion', {
          somedata: data.somedata
        })
      })
  }
}
```

## 7.5 タスク管理アプリケーションの状態管理

ではサンプルアプリを起動して少し触ってみよう。

```
`npm install`
`npm run serve`
```
### 7.5.1 アプリケーションの使用と準備

使用は実際に触って把握しましょう。

[Vuex公式サンプルも参考になります](https://github.com/vuejs/vuex/tree/dev/examples/shopping-cart)

* store.js : アプリケーションのストアに使用
* main.js : アプリケーションのエントリポイント
* App.vue : 実際に表示されるアプリケーション

実際にソースを確認してみましょう。

### 7.5.2 タスクの一覧表示

```
// store.js

const store = new Vuex.store({
  state: {
    tasks: {
      ......
    }
  }
})
```

```
// App.vue
  <h2>タスク一覧... のあたり

...

<script>
computed: {
  tasks (){
    ......
  }
}
```

### 7.5.3 タスクの新規作成と完了

タスクの追加とタスクの完了はステートを変更する処理なのでミューテーションを使用。  
store.js 、、「addTask」「toggleTaskStatus」として定義。  

次に付与するタスクはnextTaskIdとして保持。

### 7.5.4 ラベル機能の実装

ミューテーションを使用して実装。7.5.3と似たような感じ。

### 7.5.5 ラベルのフィルタリング

現在のタスクの一覧からラベルによるフィルタリングをして一部を表示する機能、、  
　→ゲッターとして実装(store.js)  

store.js に 「filteredTasks」として実装。

フィルターが変更されたとき「changeFilter」ミューテーションが呼ばれている。

```
getters: {
    // フィルター後のタスクを返す
    filteredTasks (state) {
      ......

```

### 7.5.6 ローカルストレージへの保存と復元

保存・復元ボタン押された時に「save」「restore」アクションが呼び出される。

App.Vue : 保存・復元ボタンと「save」「restore」アクション呼びだし。
store.js : 「save」「restore」アクション定義。ローカルストレージ保存。

```
  actions: {
    // ローカルストレージにステートを保存する
    save ({ state }) {
      const data = {
        tasks: state.tasks,
        labels: state.labels,
        nextTaskId: state.nextTaskId,
        nextLabelId: state.nextLabelId
      }
      localStorage.setItem('task-app-data', JSON.stringify(data))
    },

    // ローカルストレージからステートを復元する
    restore ({ commit }) {
      const data = localStorage.getItem('task-app-data')
      if (data) {
        commit('restore', JSON.parse(data))
      }
    }
  }
```

### 7.5.7 Vuexによるアプリケーションの考察

#### ここまでのまとめ

* ストア：アプリケーションの状態をステートとして表し、```基本的にはゲッターとミューテーションで操作```  
　→```ゲッターとミューテーションでカバーできない処理をアクションとして実装```

* store.js App.js：```アプリケーションのロジックはstore.js。``` ```App.vueではほぼ呼び出しのみ。```   
　→ロジックをVuexに集中させることによって異なるコンポーネントから同一の処理を呼べる。処理を置いやすく、管理も楽。

* App.vueにも状態保持してる(newTaskName 等)けど...？：特定のフォームの値を一時的に保存するためだけに使用。```App.vue以外、特定のコンポーネントからしか使われない処理なので、ストアに持たずにデータとして保持。```
