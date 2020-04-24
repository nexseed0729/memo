# サーバサイドプログラミング入門
Source: [サーバサイドプログラミング入門](https://www.nnn.ed.nico/courses/497/chapters/6890)

## Node.js
この学習資料での Node.js のインストール手順  
- ubuntu の Docker で実施
- nvm という Node.js のバージョンマネージャー導入
    - `curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash`
    - `source ~/.bashrc`
    - `nvm` と打ち込んで、使い方が表示されればインストール完了
- nvm を使って Node.js をインストール
    - `nvm install v10.14.2`
    - `nvm use v10.14.2`
        - `Now using node v10.14.2 (npm v6.4.1)` と表示されればインストール完了
- Node.js を使ってみる
    - `node` とだけ入力すると `>` が表示され、コンソールが入力を受け付ける状態になる。これは **REPL** (レプル；Read-Eval-Print Loop) というもの。
- プロジェクトを作って、そこに JavaScript ファイルを作成すれば `node app.js <some_arg>` というように実行できる。

## アルゴリズムの改善
実行時間を測定するには time コマンドを使う。

```bash
time node app.js
```
これを実行すると、以下のように結果が表示される。

```
real	0m2.029s
user	0m2.040s
sys     0m0.023s
```
これは実行にかかった、実際の時間（real）、今の実行ユーザーとしてかかった時間（user）、システムが別のことに使った時間（sys）を表示している。

フィボナッチ数列（fib(n) = fib(n-1) + fib(n-2) ただし fib(0) = 0, fib(1) = 1）の計算は、以下のように増加していく。
- n が２の時の fib(2) は、fib(1) + fib(0)、つまり、0 + 0 + 1で 1回
- n が３の時の fib(3) は、fib(2) + fib(1)、これは、1 + 0 + 1 で 2回
- n が４の時の fib(4) は、fib(3) + fib(2)、これは、2 + 1 + 1 で 4回
- n が５の時の fib(5) は、fib(4) + fib(3)、これは、4 + 2 + 1 で 7回
- n が６の時の fib(6) は、fib(5) + fib(4)、これは、7 + 4 + 1 で 12回
- n が７の時の fib(7) は、fib(6) + fib(5)、これは、12 + 7 + 1 で 20回
- n が８の時の fib(8) は、fib(7) + fib(6)、これは、20 + 12 + 1 で 33回
- n が50の時の fib(50) は、fib(49) + fib(48)、これは、12,586,269,024 + 7,778,742,048 + 1 で 20,365,011,073回
- n が100の時の fib(100) は、fib(99) + fib(98)、これは、354,224,848,179,261,915,074 + 218,922,995,834,555,169,025 + 1 で 573,147,844,013,817,084,100回

このように、回を重ねると後に倍々に計算回数が増えるような処理の増え方を **指数オーダー** といい、 *O(2^n)* と表す。  
  
### 👍プロファイルツールの使い方
処理に時間がかかっている様子やどれくらいメモリを使っているのかを調べる方法に **プロファイル** と呼ばれる方法がある。

```bash
node --prof app.js
```

以上を実行することで、`isolate-0x395f810-v8.log` というような名前のログファイルにプロファイルの様子が出力される。このログファイルは、次のコマンドでわかりやすく表示させることができる。

```bash
node --prof-process isolate-0x395f810-v8.log 
```

こうすると、プロファイル結果が表示される（かなり長い）。この中の `[Summary]` という項目に、プログラムの処理全体の概要が示されている。

```
 [Summary]:
   ticks  total  nonlib   name
   1575   89.6%   94.1%  JavaScript
      0    0.0%    0.0%  C++
      3    0.2%    0.2%  GC
     84    4.8%          Shared libraries
     98    5.6%          Unaccounted
```

また、`[Bottom up (heavy) profile]` という項目に、時間のかかった処理が並んでいる。

```
 [Bottom up (heavy) profile]:
  Note: percentage shows a share of a particular caller in the total
  amount of its parent calls.
  Callers occupying less than 1.0% are not shown.

   ticks parent  name
   1573   89.5%  LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%    LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%      LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%        LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%          LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%            LazyCompile: *fib /workspace/fibonacci/app.js:3:13
```

一番左の ticks という列は、各処理にかかったイベントループ数を示している。これらをみると、fib という関数の呼び出しにほとんどの時間がかかっていることがわかる。

### メモ化を使ったアルゴリズムの改善
Map を用いて以下のようにフィボナッチ関数を実装する。

```js
'use strict';

const memo = new Map();
memo.set(0, 0);
memo.set(1, 1);

const fib = n => {
  if (memo.has(n)) {
    return memo.get(n);
  } 
  const value = fib(n-1) + fib(n-2);
  memo.set(n, value);
  return value;
}

const length = 40;
for (let i=0; i <= length; i++) {
  console.log(fib(i));
} 
```

`time node app.js` を実行すると、

```
real	0m0.079s
user	0m0.058s
sys     0m0.021s
```

このように大幅に速くなったことがわかる。プロファイルも実行してみる。（`node --prof app.js`→`node --prof-process isolate-xxx.log`）

出力されたプロファイルの `[Summary]` を見てみると以下のようになり、total の数値をみると、JavaScript の処理にかかる時間が大幅に減ったことがわかる。

```
 [Summary]:
   ticks  total  nonlib   name
      2    2.2%  100.0%  JavaScript
      0    0.0%    0.0%  C++
      4    4.5%  200.0%  GC
     87   97.8%          Shared libraries
```

`[Bottom up (heavy) profile]` の項目をみると、最も時間のかかっている処理がもはや fib 関数の実行ではないことがわかる。

```
 [Bottom up (heavy) profile]:
  Note: percentage shows a share of a particular caller in the total
  amount of its parent calls.
  Callers occupying less than 1.0% are not shown.

   ticks parent  name
     36   40.4%  /root/.nvm/versions/node/v10.14.2/bin/node
     34   94.4%    /root/.nvm/versions/node/v10.14.2/bin/node
     20   58.8%      LazyCompile: ~NativeModule.compile internal/bootstrap/loaders.js:302:44
     20  100.0%        LazyCompile: ~NativeModule.require internal/bootstrap/loaders.js:148:34
      4   20.0%          Script: ~<anonymous> util.js:1:11
```

このメモ化を使ったアルゴリズムの改善がされたこちらの fib 関数のオーダーは、*O(n)* となる。n に対して n 倍の時間をかければ問題を解くことができるというこのようなオーダーを線形オーダーという。

## 集計処理を行うプログラム

**ここで用いる csv ファイルは [こちら](https://github.com/progedu/adding-up/blob/master/popu-pref.csv)**

```js
const fs = require('fs');
const readline = require('readline')
```

この部分は、Node.js に用意された **モジュール** を呼び出している。

`fs` は FileSystem の略で、ファイルを扱うためのモジュール。
`readline` は、ファイルを一行ずつ読み込むためのモジュー売る。

```js
const rs = fs.createReadStream('./popu-pref.csv');
const rl = readline.createInterface({ 'input': rs, 'output' : {} });
```

以上の部分は、csv ファイルかrあ、ファイルの読み込みを行う Stream を作成し、さらにそれを `readline` オブジェクトの input として設定し、`rl` オブジェクトを作成している。

さて、ここで突然出てきた Stream とはなんなのか？

### Stream
Node.js では、入出力が発生する処理をほとんど **Stream** という形で扱う。Stream とは、非同期で情報を取り扱うための概念で、情報自体ではなく情報の流れに注目する。

Node.js で Stream を扱う際は、Stream に対してイベントを監視し、イベントが発生したときに呼び出される関数を設定することによって、情報を利用する。

このように、あらかじめイベントが発生したときに実行される関数を設定しておいて、起こったイベントに応じて処理を行うことを **イベント駆動型プログラミング** と呼ぶ。

```js
const rl = readline.createInterface({ 'input': rs, 'output': {} })
```

ここで作成された `rl` というオブジェクトも Stream のインターフェースを持っている。利用する際には

```js
rl.on('line', (lineString) => {
  console.log(lineString);
});
```

のようなコードになる。このコードは、`rl` オブジェクトで `line` というイベントが発生したらこの無名関数を呼んでください、という意味。なお、`lineString` には、読み込んだ 1 行の文字列が入っている。

### ファイルからデータを抜き出す
2010 年と 2015 年のデータから「集計年」「都道府県」「15〜19歳の人口」を抜き出す、という処理を実装したい。

```js
rl.on('line', (lineString) => {
  const columns = lineString.split(',');
  const year = parseInt(columns[0]);
  const prefecture = columns[1];
  const popu = parseInt(columns[3]);

  if (year === 2010 | year === 2015) {
    console.log(year);
    console.log(prefecture);
    console.log(popu);
  }
});
```

### グルーピングを行う

```js
'use strict';

const fs = require('fs');
const readline = require('readline');
const rs = fs.createReadStream('./popu-pref.csv');
const rl = readline.createInterface({ 'input' : rs, 'output': {} });
const prefectureDataMap = new Map(); // key: 都道府県 value: 集計データのオブジェクト
rl.on('line', (lineString) => {
  const columns = lineString.split(',');
  const year = parseInt(columns[0]);
  const prefecture = columns[1];
  const popu = parseInt(columns[3]);
  if (year === 2010 | year === 2015) {
    // 連想配列 `prefectureDataMap` からデータを取得する。
    // value の値が falsy の場合に、value に初期値となるオブジェクトを代入する。その県のデータを処理するのがはじめてであれば、value の値は undefined となるのでこの条件を満たし、value に値が代入される
    let value = prefectureDataMap.get(prefecture);
    if (!value) {
      value = {
        popu10: 0,
        popu15: 0,
        change: null,
      };
    }
    
    if (year === 2010) {
      value.popu10 = popu;
    }
    
    if (value === 2015) {
      value.popu15 = popu;
    }
    
    prefectureDataMap.set(prefecture, value);
  }
});

// close イベントは、全ての行を読み込み終わったあとに呼び出される。
// また、人口の変化率（change）の計算は、その県のデータが揃ったあとでしか正しく行えないので、close イベントの中へ実装する
rl.on('close', () => {
  for (let [key, value] of prefectureDataMap) {
    value.change = value.popu15 / value.popu10;
  }
  console.log(prefectureDataMap);
});
```

**`for-of` 構文について**
Map や Array の中身を `of` の前に与えられた変数に代入して for ループと同じことができる。配列に含まれる要素を使いたいだけで、添字は不要な場合に便利。

また、Map に for-of を使うと、キーと値で要素が 2 つある配列が前に与えられた変数に代入される。ここでは **分割代入** という方法を使っている。

### データの並び替え
得られた結果を、変化率ごとに並べ替えてみる。

```js
rl.on('close', () => {
  for (let [key, value] of prefectureDataMap) { 
    value.change = value.popu15 / value.popu10;
  }
  const rankingArray = Array.from(prefectureDataMap).sort((pair1, pair2) => {
    return pair2[1].change - pair1[1].change;
  });
  console.log(rankingArray);
});
```

まず、`Array.from(prefectureDataMap)` の部分で、連想配列を普通の配列に変換する処理を行っている。さらに、Array の `sort` 関数を呼んで、無名関数を渡している。`sort` に対して渡すこの関数は *比較関数* と言い、これによって並び替えのルールを決めることができる。

比較関数は 2 つの引数を受け取って、前者の引数を pair1 を後者の引数 pair2 より前にしたいときは負の整数を、pair2 を pair1 より前にしたいときは正の整数を、pair1 と pair2 の並びをそのままにしたいときは 0 を返す必要がある。

ここでは変化率の降順に並び替え行いたいので、pair2 が pair1 より大きかった場合、pair2 を pair1 より前にする必要がある。

つまり、pair2 が pair1 より大きいときに正の整数を返すような処理を書けばよいので、ここでは pair2 の変化率のプロパティから pair1 の変化率のプロパティを引き算した値を返している。

詳細は [MDN の sort 関数のドキュメント](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)。

（ちなみに、ここでの pair1, pair2 は（`Array.from(prefectureDataMap)` を行ったものであり、） `['hoge 県', { popu10: ..., popu15: ..., change: ...}]` という形をしているので、`pair1[1].change`  などと指定している）

> `Array.from()` について：Array.from() メソッドを用いれば、配列に似た形のもの（ここでは Map）を普通の配列に変換することができる。ここでは、`prefectureDataMap` は、各都道府県名が key で、各集計データオブジェクトが value の Map であったから、`Array.from(prefectureDataMap)` とすると `[['hoge 県', { popu10: ..., popu15: ..., change: ...}],[...],...,[...]]`という形の配列になる。

最後に、`map` 関数を使って、きれいに整形して出力してみる。

```js
rl.on('close', () => {
  for (let [key, value] of prefectureDataMap) {
    value.change = value.popu15 / value.popu10;
  }
  const rankingArray = Array.from(prefectureDataMap).sort((pair1, pair2) => {
    return pair2[1].change - pair1[1].change;
  });
  const rankingStrings = rankingArray.map(([key, value]) => {
    return key + ': ' + value.popu10 + '=>' + value.popu15 + ' 変化率： ' + value.change;
  });
  console.log(rankingStrings);
});
```

## ライブラリ
### 環境変数
環境変数は特別なシェル変数。通常のシェル変数は現在実行しているシェルからしか参照できないが、環境変数はシェルで実行したプロセスからも参照できる。Ubuntu では、デフォルトでユーザー名が代入されている `USER` や、ユーザーのホームディレクトリの場所が代入されている `HOME` などがある。

Ubuntu で、現在のシェルで定義されている環境変数一覧をみるには `printenv` コマンドを使う。

個々の環境変数の値を表示させるときは `echo` コマンドを使う。

```bash
$ echo $HOME
/root
```

`PATH` という環境変数は、実行したいプログラムがある場所を列挙しておくもの。`PATH` に指定された場所にあるプログラムは、そのファイル名を入力するだけで実行できるようになり、絶対パスや相対パスで直接ファイルを指定する必要はない。

インストールが完了したら、

```bash
source ~/.bashrc
```

と入力することで、設定を再読込できる。

### npm パッケージを作ってみる
たとえば、整数を足しあわせたりする sum というライブラリを作成してみる。

```bash
cd ~/workspace
mkdir sum
cd sum
yarn init
```

`yarn init` というコマンドで、npm パッケージを作成するためのチュートリアルを起動する。インストラクションにしたがって Enter を押していく。`entry point` というのは、ライブラリとして読み込まれる JavaScript ファイル名のこと（`index.js` でよい）。

`ISC` ライセンスとは、ソフトウェアを使用、コピー、改変、配布する許可を与えるライセンス。
`private` の設定をすると、公開はしないことになる。

この状態で、index.js を作成する。

```js
`use strict`
function add(numbers) {
  let result = 0;
  for (let num of numbers) {
    result = result + num;
  }
  return result;
}
module.exports = {
  add // add: add と書くのと同じ
};
```

**`module.exports = { ... }` という記法について**  
特定の関数をモジュールとして公開する場合に、`module.exports` オブジェクトのプロパティとして関数を登録する。こうすることで、この場合ではこの sum パッケージに add メソッドが追加される。

### 作成したパッケージを使用する
今回は npm に公開はせず、ローカルの別の場所でこのパッケージを使う。まずは、このパッケージを使うアプリケーションを作成してみる。

```bash
cd ~/workspace
mkdir sum-app
cd sum-app
```

先ほどと同様に `yarn init` と入力してパッケージ作成のチュートリアルを行う。そして、先ほど開発した sum パッケージをインストールする。

```bash
yarn add ../sum
```

次に、sum-app フォルダを開いて、app.js を以下のように編集する。

```js
'use strict'
const s = require('sum');
console.log(s.add([1, 2, 3, 4]));
```

## モジュール化された処理
CRUD を考え、以下の機能を持つモジュールを作成する。

- todo: タスクの作成
- done: タスクの状態の更新（「完了」状態にする）
- list: タスクの状態が未完了のものの読み込み
- donelist: タスクの状態が完了のものの読み込み
- del: タスクの削除

今回はテストコードも書く。`package.json` に以下を追記。

```json
"scripts": {
  "test": "node test.js"
}, ...
```

今回のすべてのデータは、連想配列 Map の key にタスクの文字列を、value に完了しているかどうかの真偽値を入れることによって表現する。

index.js
```js
'use strict';

// key: タスクの文字列 value: 完了しているかどうかの真偽値
const tasks = new Map();

/**
 * TODO を追加する
 * @param {string} task
 */
const todo = task => {
    tasks.set(task, false);
}

/**
 * タスクと完了したかどうかが含まれる配列を受け取り、完了したかを返す
 * @param {array} taskAndIsDonePair
 * @return {boolean} 完了したかどうか
 */
const isDone = taskAndIsDonePair => {
    return taskAndIsDonePair[1];
}

/**
 * タスクと完了したかどうかが含まれる配列を受け取り、完了していないか返す
 * @param {array} taskAndIsDonePair
 * @return {boolean} 完了していないかどうか
 */
const isNotDone = taskAndIsDonePair => {
    return !isDone(taskAndIsDonePair);
}

/**
 * TODO の一覧の配列を取得する
 * @return {array}
 */
const list = () => {
    return Array.from(tasks) // Map を、キーと値で構成される要素 2 つの配列に変換する
        .filter(isNotDone)
        .map(t => t[0]);
}

/**
 * TODO を完了状態にする
 * @param {string} task
 */
const done = task => {
    if (tasks.has(task)) {
        tasks.set(task, true);
    }
}

/**
 * 完了済みのタスクの一覧の配列を取得する
 * @return {array}
 */
const donelist = () => {
    return Array.from(tasks)
                .filter(isDone)
                .map(t => t[0]);
}

/**
 * 項目を削除する
 * @param {string} task
 */
const del = task => {
    tasks.delete(task);
}

// todo 関数 と list 関数をモジュールの関数として追加する
module.exports = { todo, list, done, donelist, del };
```

上のコードにおいて、`Array.from` 関数は、連想配列の Map をキーと値で構成される要素 2 つの配列に変換する。`[['鉛筆を買う', false], ['りんごを買う', true], ...]` のような感じになる。

`list` 関数内部で行われていることを例示すると以下のようなイメージ。

```js
Array.from(Map { '鉛筆を買う' => false, 'ノートを買う' => false, '勉強をする' => true })
// [ [ '鉛筆を買う', false ], [ 'ノートを買う', false ], [ '勉強をする', true ] ]
[ [ '鉛筆を買う', false ], [ 'ノートを買う', false ], [ '勉強をする', true ] ].filter(isNotDone)
// [ [ '鉛筆を買う', false ], [ 'ノートを買う', false ] ]
[ [ '鉛筆を買う', false ], [ 'ノートを買う', false ] ].map(t => t[0])
// [ '鉛筆を買う', 'ノートを買う' ]
```

テストコードは以下のようになる。

```js
'use strict';

const todo = require('./index.js');
const assert = require('assert');

// todo と list のテスト
todo.todo('ノートを買う');
todo.todo('鉛筆を買う');
assert.deepEqual(todo.list(), ['ノートを買う', '鉛筆を買う']);

// done と donelist のテスト
todo.done('鉛筆を買う');
assert.deepEqual(todo.list(), ['ノートを買う']);
assert.deepEqual(todo.donelist(), ['鉛筆を買う']);

// del のテスト
todo.del('ノートを買う');
todo.del('鉛筆を買う');
assert.deepEqual(todo.list(), []);
assert.deepEqual(todo.donelist(), []);

console.log('🎉テストが正常に完了しました😙');
```

これで `yarn test` を実行して、「テストが正常に完了しました」と表示されれば成功。

```js
const assert = require('assert');
```
上記は、Node.js でテストをするためのモジュール `assert` を呼び出している。

```js
assert.deepEqual(todo.list(), ['ノートを買う', '鉛筆を買う']);
```

`assert.deepEqual` は、与えられたオブジェクトや配列の中身まで比較してくれる `equal` 関数。もう少し詳しくは以下を参照。

```js
assert.equal(1, 1); // ✅
assert.equal([1], [1]) // 🛑 AssertionError
// ↑は、JavaScript では配列やオブジェクトを == 演算子で比較した場合、
// 同じオブジェクト自身でないと false になるため

assert.deepEqual([1], [1]); // ✅
assert.deepEqual({p: 1}, {p: 1}); // ✅
```