---
title: '【Day.js】React+Day.jsで簡単にカレンダーコンポーネントができた話（Day.jsの基本メソッドの紹介もあるよ）'
emoji: '☀️'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['react', 'dayjs', 'フロントエンド', 'javascript', 'typescript']
published: false
publication_name: 'spacemarket'
---

こんにちは！
[スペースマーケット](https://www.spacemarket.com/)でフロントエンドエンジニアをしているwharaguchiです。

今回は、Day.jsを使ってカレンダーコンポーネントを作成したところ、とても簡単に作ることができたので、そちらを紹介したいと思います。

## 今回のゴール

今回のゴールは以下の仕様を満たしたカレンダーコンポーネントです。

- 前月・次月の移動ができる
- 表示している月に応じてカレンダーの表示内容を変更する
- 日曜日始まりのカレンダーを作成し、不足している情報を前月、次月から補完する

実際のコードと画面も用意しているので、そちらも併せてご確認ください。

https://github.com/wh2626/blog-calendar-component

https://blog-calendar-component.vercel.app/

## この記事で学べるDay.jsの基本メソッド

| 役割                                   | メソッド（例）                   |
| -------------------------------------- | -------------------------------- |
| 現在の日付情報を取得する               | `dayjs()`                        |
| dayjsオブジェクトのフォーマットを行う  | `dayjs().format('YYYY年M月D日')` |
| dayjsオブジェクトの日付情報を加算する  | `dayjs().add(1, 'month')`        |
| dayjsオブジェクトの日付情報を減算する  | `dayjs().subtract(1, 'month')`   |
| 指定された月の日数を取得する           | `dayjs().daysInMonth()`          |
| 指定された単位の始まりの情報を取得する | `dayjs().startOf('month')`       |
| 指定された単位の終わりの情報を取得する | `dayjs().endOf('month')`         |
| 曜日の情報を取得する                   | `dayjs().day()`                  |

## やっていくこと

以下の順序でカレンダーコンポーネントを作成していきたいと思います。

- 選択した月の表示
- 月の移動
- 表示している月に応じたカレンダーデータの作成
- 前月、次月のカレンダーデータを取得
- 週ごとに配列を作成する

### 選択した月の表示

選択した月の表示を行います。
初期表示時点では、このコンポーネントを表示した際の月の情報を表示したいので、Day.jsで現在の日付情報を取得するための`dayjs()`を使用します。

おいおいユーザーの操作によって選択した月の情報を管理しておきたいため、useStateの初期値に`dayjs()`を設定しておきます。

```ts:useCalendar.ts
const [selectedMonth, setSelectedMonth] = useState<dayjs.Dayjs>(dayjs())
```

https://day.js.org/docs/en/parse/now

コンポーネント側で表示をする際は、Day.jsのformatメソッド使って表示します。

```tsx:Calendar.tsx
<span>
  {selectedMonth.format('YYYY年M月')}
</span>
```

https://day.js.org/docs/en/display/format

### 月の移動

選択している月の前月、または次月の切り替えを行います。
useStateで管理した`selectedMonth`に対し、前月なら`selectedMonth`を-1し、次月なら+1をする処理を作成します。

-1する場合は、`selectedMonth`に対し`subtract`メソッドを、+1する場合は、`add`メソッドを使用します。

```ts:useCalendar.ts
const handlePrevMonth = useCallback(() => {
  setSelectedMonth(selectedMonth.subtract(1, 'month'))
}, [selectedMonth])

const handleNextMonth = useCallback(() => {
  setSelectedMonth(selectedMonth.add(1, 'month'))
}, [selectedMonth])
```

```tsx:Calendar.tsx
<div className="flex justify-between items-center mb-4">
  <button onClick={handlePrevMonth} className="cursor-pointer font-bold">
    ＜
  </button>
  <span className="text-center font-bold text-lg">
    {selectedMonth.format('YYYY年M月')}
  </span>
  <button onClick={handleNextMonth} className="cursor-pointer font-bold">
    ＞
  </button>
</div>
```

第一引数で数、第二引数で日、週、月、年などの単位を指定することができます。
今回はひと月単位で移動させたいので、どちらも引数は、1とmonthを指定します。

https://day.js.org/docs/en/manipulate/add
https://day.js.org/docs/en/manipulate/subtract

これで、選択している月と、月の移動部分の実装は完了です🙌
矢印をポチポチすると、表示が切り替わることが確認できると思います。

### 選択されている月に応じたカレンダーデータの作成

次に選択されている月のカレンダー情報を作成します。

まずは選択されている月の日数を取得し、その数だけ配列を回すことで作成できます。
選択されている月の日数を取得するには、`daysInMonth`メソッドを使用することで取得ができます。

https://day.js.org/docs/en/display/days-in-month

```ts:useCalendar.ts
const daysInMonth = selectedMonth.daysInMonth() // （2025年5月の場合）31
const selectedMonthDateList = Array.from({ length: daysInMonth }, (_, i) =>
  selectedMonth.startOf('month').add(i, 'day')
)
```

`startOf`メソッドは第一引数で指定した単位の始まりの情報を生成するためのメソッドです。
言葉で説明するのが難しいのですが、例えば今が2025年5月8日13時だとしたら、以下のようになります。

| 第一引数                   | 結果               |
| -------------------------- | ------------------ |
| `dayjs().startOf('year')`  | 2025年1月1日 00:00 |
| `dayjs().startOf('month')` | 2025年5月1日 00:00 |
| `dayjs().startOf('day')`   | 2025年5月8日 00:00 |

※実際にはdayjsオブジェクトで返却されますが、ここではわかりやすくテキストで書いています。

https://day.js.org/docs/en/manipulate/start-of

### 前月、次月のカレンダーデータを生成

今回作成するカレンダーは日曜日始まり土曜日終わりで作成したいので、選択された月の初日と最終日が何曜日かによって、カレンダー上で不足している日を前月と次月から取得する必要があります。

例えば2025年4月を選択している場合、4月1日は火曜日なので前月の日曜日と月曜日の情報を3月の最終週から取得します。（3月30日、3月31日）
同様に、4月30日は水曜日なので、次月の木曜日〜土曜日の情報を5月の初週から取得します。（5月1日〜5月3日）
そしてそれぞれ取得した情報を4月の情報に追加します。

上で説明した`add`メソッドと`subtract`メソッドを使用することで、前月、次月の情報も生成することができます。
まずは3月の情報を生成してみましょう。

Day.jsでは`day()`メソッドを使用することで曜日の情報（ナンバー）を取得することができます。
4月1日の曜日を取得したいので、`startOf('month')`と`day()`メソッドを使用して曜日情報を取得します。

https://day.js.org/docs/en/get-set/day

4月1日は火曜日なので、`selectedMonthStartDay`には2という値が返されます。

```ts:useCalendar.ts
const selectedMonthStartDay = selectedMonth.startOf('month').day() // 2
```

この値を使って、`subtract`メソッドで3月30、31日の情報を配列に入れてあげます。
このままだと配列の0番目に31日の情報が入るため、最後にreverseして30日、31日という形にしてあげます。

```ts:useCalendar.ts
const prevMonthDateList = Array.from(
  { length: selectedMonthStartDay },
  (_, i) => selectedMonth.startOf('month').subtract(i + 1, 'day')
).reverse()
```

これで3月の情報は生成できました。

次に5月の情報ですが、こちらも基本的には同じようなことをしてあげればOKです。
まず`endOf`メソッドで4月30日の曜日を取得し、土曜日を表す6から4月30日の曜日（水曜日なので3）を引いた数を今度は`add`メソッドで配列を生成します。

```ts:useCalendar.ts
const selectedMonthEndDay = selectedMonth.endOf('month').day()
const LAST_DAY_OF_WEEK_INDEX = 6
const nextMonthDateList = Array.from(
  {
    length: LAST_DAY_OF_WEEK_INDEX - selectedMonthEndDay, // 6 - 3なので3日分5月の情報を生成する
  },
  (_, i) => selectedMonth.endOf('month').add(i + 1, 'day')
)
```

https://day.js.org/docs/en/manipulate/end-of

3月と5月の情報の配列ができあがったので、4月の配列にガッチャンコします。

```ts:useCalendar.ts
const calendarData = [
  ...prevMonthDateList, // 3月
  ...selectedMonthDateList, // 4月
  ...nextMonthDateList, // 5月
]
```

これでカレンダーの情報を生成することができました！やったね！！

`selectedMonth`の情報を基準にカレンダー情報を生成しているので、月の操作によって選択された月が変わっても、自動でカレンダーの情報を更新してくれるようになっています。

### 週ごとに配列を作成する

カレンダーの情報はできたので、今度は行ごとの配列を作りたいと思います。

ここでは、特に大したことはやっていないのですが、簡単に説明すると、reduceを使って先ほど作成したカレンダー情報をぶんぶんぶん回し、`weekIndex`が一致する場合は同じ配列に情報を追加し、一致しなければ次の配列に情報を追加していく、ということをしています。

```ts:useCalendar.ts
const calendarData = useMemo(() => {
  const weeklyCalendarData = Object.values(
    [
      ...prevMonthDateList,
      ...selectedMonthDateList,
      ...nextMonthDateList,
    ].reduce((acc, date, index) => {
      const weekIndex = Math.floor(index / 7)

      return {
        ...acc,
        [weekIndex]: [...(acc[weekIndex] ?? []), date],
      }
    }, [] as dayjs.Dayjs[][])
  )

  return weeklyCalendarData
}, [selectedMonthDateList])
```

reduceについてもう少し詳しく書いた記事があるので、気になる方はそちらも併せてご覧ください🙌

https://zenn.dev/spacemarket/articles/cec64a21e2545a

あとは完成した`calendarData`をmapで回してあげ、スタイリングをすれば完成です！！！
おつかれさまでした！

## さいごに

いかがだったでしょうか？
個人的にカレンダーコンポーネントを作るのはもっと難しいものだと考えていましたが、Day.jsを使用することで大分楽にカレンダーコンポーネントを作れると感じました！
是非作ってみてください！

最後に宣伝です！
スペースマーケットでは現在エンジニアを募集しております！
ちょっと話を聞いてみたいといったようなカジュアルな面談でも構いませんので、ご興味のある方は是非ご応募お待ちしております！
