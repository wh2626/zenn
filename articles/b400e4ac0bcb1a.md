---
title: '【Day.js】React+Day.jsで作成するレンジ版カレンダーコンポーネント'
emoji: '🍊'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['react', 'dayjs', 'フロントエンド', 'javascript', 'typescript']
published: true
publication_name: 'spacemarket'
---

こんにちは！
[スペースマーケット](https://www.spacemarket.com/)でフロントエンドエンジニアをしているwharaguchiです。

前回カレンダーコンポーネント単体の作り方を紹介したところ、社内で「開始日と終了日を選択できるレンジ版はどう作るの？」と質問をいただいたので、今回はレンジ版のカレンダーコンポーネントを作成してみました！

前回の記事は以下です。
https://zenn.dev/spacemarket/articles/caee5ddd8a8937

今回の記事も前回の記事と同じ構成で進めていきます。

## 今回のゴール

まず、今回のゴールは、前回作成したカレンダーコンポーネントを元に、以下の仕様を追加したものを作成するところをゴールとしています。

- 開始日と終了日を選択できる
- 選択範囲内の日付に対しスタイルをあてる

今回も実際のコードと画面も用意しているので、よろしければご覧ください。

https://github.com/wh2626/blog-calendar-component

https://blog-calendar-component.vercel.app/

## やっていくこと

以下の順序で作成していきたいと思います。

- 開始日と終了日を管理するstateを作成
- stateに応じて日付選択時に格納するデータを更新
- 選択範囲内の日付にスタイルを適用

### 開始日と終了日を管理するstateを作成

前提として、上述のとおり前回作成したコンポーネントを元に作成するので、元の実装を見たいという方は前回のブログをご覧ください。

まず開始日と終了日を管理するstateを作成します。

今回は開始日と終了日と管理する日付が2つあるので、配列で管理します。
また今回は初期値は空の状態にします。

```ts:useRangeCalendar.ts
const [selectedDates, setSelectedDates] = useState<dayjs.Dayjs[]>([])
```

### stateに応じて日付選択時に格納するデータを更新

次に日付をクリックした際のロジックを変更します。

クリックをしたタイミングで`selectedDates`に入っている状態が1つであれば、つまり基準日のみを選択されている状態であれば、2つ目の日をセットします。

逆に1つ以外、つまり何も選択されていない状態か、すでに日が2つ選択されている状態であれば、1つ目の基準日をセットします。

ここで1つ目の日付を「開始日」ではなく、「基準日」と表現しているのは、1つ目に選択した日付が必ずしも開始日になるわけではないためです。

ユーザーの行動として終了日から選択することもあるので、クリックされた1つ目の日を開始日、2つ目の日を終了日とするのではなく、2つ目の日が選択されたら、常に若い日が配列の0番目になるようにソートをしています。

```ts:useRangeCalendar.ts
const handleSelectDate = useCallback(
  (date: dayjs.Dayjs) => {
    if (selectedDates.length === 1) {
      setSelectedDates((prev) => {
        const newDates = [...prev, dayjs(date).startOf('day')].sort(
          (a, b) => a.valueOf() - b.valueOf()
        )
        return newDates
      })
    } else {
      setSelectedDates([dayjs(date).startOf('day')])
    }
    // ...略
  },
  [selectedDates, selectedMonth]
)

```

### 選択範囲内の日付にスタイルを適用

最後に選択範囲内の日についても見た目の変更をします。
必要なスタイルは、以下3つになります。

- 選択時
- 選択範囲内
- それ以外

各スタイルは、Day.jsのメソッドを使用して作ることができます。

まずは選択時のスタイルです。
選択時は、`selectedDates`で管理されている日のいずれかに該当しているかをチェックする必要があります。

そのため、JavaScriptの`some`メソッドと、Day.jsの`isSame`メソッドを組み合わせてチェックします。

```ts:RangeCalendar.tsx
const isSelected = selectedDates.some((d) => d.isSame(date, 'day'))
```

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/some

https://day.js.org/docs/en/query/is-same

次に選択範囲内のスタイルです。

選択範囲内かどうかはまず、stateで管理されている日が2つないと成立しないため、`selectedDates`のlengthをチェックしています。

また、`selectedDates`の1つ目の日以降かつ、2つ目の日以前である必要があるため、Day.jsのメソッドである`isBefore`と`isAfter`をメソッドを使用します。

https://day.js.org/docs/en/query/is-before

https://day.js.org/docs/en/query/is-after

```ts:RangeCalendar.tsx
const isInRange =
  selectedDates.length === 2 &&
  !isSelected &&
  date.isAfter(selectedDates[0], 'day') &&
  date.isBefore(selectedDates[1], 'day')
```

上記で作成した各変数を使用してスタイルを出し分けて、コンポーネント側でそれぞれを適用します。

```ts:RangeCalendar.tsx
const CalendarCell: FC<{
  date: dayjs.Dayjs
  handleSelectDate: (date: dayjs.Dayjs) => void
  selectedDates: dayjs.Dayjs[]
}> = ({ date, handleSelectDate, selectedDates }) => {
  const isSelected = selectedDates.some((d) => d.isSame(date, 'day'))

  const isInRange =
    selectedDates.length === 2 &&
    !isSelected &&
    date.isAfter(selectedDates[0], 'day') &&
    date.isBefore(selectedDates[1], 'day')

  const { bgColor, fontWeight } = isSelected
    ? { bgColor: '#3eae6b', fontWeight: 'bold' }
    : isInRange
    ? { bgColor: '#abdfc0', fontWeight: 'bold' }
    : {
        bgColor: 'transparent',
        fontWeight: 'normal',
      }

  return (
    <td>
      <button
        onClick={() => handleSelectDate(date)}
        className="text-center w-full cursor-pointer p-2"
        style={{
          backgroundColor: bgColor,
          fontWeight: fontWeight,
          color: '#333',
        }}
      >
        {date.date()}
      </button>
    </td>
  )
}
```

以上で完成です！

最後までご覧いただきありがとうございました！
