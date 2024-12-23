---
title: "【JavaScript】Object.groupByを使って日毎の配列を作成する"
emoji: "🎄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript"]
published: false
publication_name: "spacemarket"
---

こんにちは！
毎年妻にクリスマスプレゼントを用意しているのに、自分は用意されていないwharaguchiです。

今年はどうなるのでしょうか。
（ちなみに今年のプレゼントは[マリオのルームソックス](https://store-jp.nintendo.com/item/goods/VM_NSJ_8_CLAAK)にしました🧦かわいい）

さて、今回は、先日あげた以下の記事に `Object.groupBy`についてのコメントをいただいたため、そちらを試した記事になります。
https://zenn.dev/spacemarket/articles/cec64a21e2545a

## Object.groupBy とは

`Object.groupBy`はES2024で新たに追加された静的メソッドで、特定のキーや条件に基づいてグループ化をするためのメソッドです。
先日の記事では、日をキーとした日毎の配列を作成したので確かに`Object.groupBy`でも実現できそうです。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/groupBy

前回と同じデータ、同じゴールを`Object.groupBy`で実装してみたいと思います。

### 元となるデータ

```ts
const baseData = [
  {
    startedAt: "2024-10-30T10:00:00+09:00",
    rooms: {
      id: "1",
    },
  },
  {
    startedAt: "2024-10-30T11:00:00+09:00",
    rooms: {
      id: "2",
    },
  },
  {
    startedAt: "2024-10-31T10:00:00+09:00",
    rooms: {
      id: "3",
    },
  },
  {
    startedAt: "2024-11-01T10:00:00+09:00",
    rooms: {
      id: "4",
    },
  },
  {
    startedAt: "2024-11-01T11:00:00+09:00",
    rooms: {
      id: "5",
    },
  },
];
```

### ゴール

```ts
const expectData = [
  [
    { startedAt: "2024-10-30T10:00:00+09:00", rooms: { id: "1" } },
    { startedAt: "2024-10-30T11:00:00+09:00", rooms: { id: "2" } },
  ],
  [{ startedAt: "2024-10-31T10:00:00+09:00", rooms: { id: "3" } }],
  [
    { startedAt: "2024-11-01T10:00:00+09:00", rooms: { id: "4" } },
    { startedAt: "2024-11-01T11:00:00+09:00", rooms: { id: "5" } },
  ],
];
```

## できたコードと解説

というわけで早速最終的なコードです。

```ts
const result = Object.values(
  Object.groupBy(baseData, (element) => element.startedAt.split("T")[0])
);
```

（前のコードと比べると、だいぶさっぱりしました）

以下各コードの解説です。

### 構文

まずは構文を見ていきましょう。

```js
Object.groupBy(items, callbackFn);
```

`Object.groupBy`では第一引数に配列などの反復可能な要素が指定できます。
今回は、`baseData`の配列を反復させてグループを作成したので`baseData`を指定します。

第二引数はコールバック関数です。
第一引数で指定した反復可能な各要素に対して実行される関数で、引数は、第一引数が現在処理中の要素で、第二引数がindexとなっています。

### 1. Object.groupByでグルーピングを行う

今回もまず日をキーとしてまずは各配列を作成します。
第一引数に`baseData`を指定し、コールバック関数の第一引数で`element`を取得します。
年月日の情報をキーとしたいため、`element.startedAt.split("T")[0]`で日の情報を取り出しグルーピングを行います。

```ts
const result = Object.groupBy(
  baseData,
  (element) => element.startedAt.split("T")[0]
);
```

すると、以下のように年月日の情報をキーとしたオブジェクトができあがります。

```js
{
    '2024-10-30': [
        {
            'startedAt': '2024-10-30T10:00:00+09:00',
            'rooms': {
                'id': '1'
            }
        },
        {
            'startedAt': '2024-10-30T11:00:00+09:00',
            'rooms': {
                'id': '2'
            }
        }
    ],
    '2024-10-31': [
        {
            'startedAt': '2024-10-31T10:00:00+09:00',
            'rooms': {
                'id': '3'
            }
        }
    ],
    '2024-11-01': [
        {
            'startedAt': '2024-11-01T10:00:00+09:00',
            'rooms': {
                'id': '4'
            }
        },
        {
            'startedAt': '2024-11-01T11:00:00+09:00',
            'rooms': {
                'id': '5'
            }
        }
    ]
}
```

### 2. Object.values

最後に前回と同様で`Object.values`で配列に変換してあげればOKです！

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/values

```ts
const result = Object.values(
  Object.groupBy(baseData, (element) => element.startedAt.split("T")[0])
);
```

```ts
[
  [
    {
      startedAt: "2024-10-30T10:00:00+09:00",
      rooms: {
        id: "1",
      },
    },
    {
      startedAt: "2024-10-30T11:00:00+09:00",
      rooms: {
        id: "2",
      },
    },
  ],
  [
    {
      startedAt: "2024-10-31T10:00:00+09:00",
      rooms: {
        id: "3",
      },
    },
  ],
  [
    {
      startedAt: "2024-11-01T10:00:00+09:00",
      rooms: {
        id: "4",
      },
    },
    {
      startedAt: "2024-11-01T11:00:00+09:00",
      rooms: {
        id: "5",
      },
    },
  ],
];
```

完成です！🥳

## 注意点

前回のコードと比べると記述量も減り、コードもわかりやすくなり最高！と言いたいところですが、ちょっと注意点があります。

というのも`Object.groupBy`はES2024で追加された新しいメソッドのため、例えばNode.jsでは[バージョン21以上の環境である](https://nodejs.org/en/blog/announcements/v21-release-announce#v8-118)必要があり、環境によっては使用できなかったりします。

そのため使用する際は、ご自身の開発環境を確認してから使用してください。

## さいごに

いかがだったでしょうか？
まだ使用できる環境は限られていますが、便利なメソッドなのでどんどん使っていきたいですね！

最後に宣伝です！
スペースマーケットでは現在エンジニアを募集しております！
ちょっと話を聞いてみたいといったようなカジュアルな面談でも構いませんので、ご興味のある方は是非ご応募お待ちしております！
