---
title: Nova 2 Sonic は AI の発話テキストを2回送ってくる（SPECULATIVE と FINAL の使い分け）
tags: AWS bedrock NovaSonic 生成AI TypeScript
---

<!--
書く前のメモ（投稿前に消す）
読者：Amazon Nova 2 Sonic の双方向ストリーミング API で、AI の発話を字幕や会話ログに出そうとしている人
読者が読む前に困っていること：AI の発話が画面やログに2回出る。または、字幕が音声より遅れて出る
読者が読んだ後にすること：contentStart の generationStage を見て、画面に出すテキストには SPECULATIVE、保存するテキストには FINAL を使う
著者にしか書けない材料（数字、失敗とその理由、気づき）：
  - 2026年7月25日に、会話ログの重複を直した（アプリのリポジトリの修正の記録より）
  - 2026年10月18日に、捨てていた SPECULATIVE を字幕の先出しに使うようにした（同上）
  - それまでの討論モードの字幕は「再生済みの秒数 × 16文字/秒」で後追いに出していた（同上）
  - 2026年9月の登壇資料で、2つのテキストを「下書き」と「清書」と書いた
省く話：討論モードのターンテイキング、翻訳の字幕

著者に確かめたいこと（答えをもらってから本文を直す）：
  1. 読者は上の1文でよいか
  2. 重複に最初に気づいたのはどの画面か（会話ログ、字幕、フィードバック）。原因が分かるまでにどのくらいかかったか
  3. 「起きたこと」の例は、ホテルのステージの最初の台詞を使った例である。実際のログがあれば差し替える
  4. 字幕の先出しは実機で確かめたか。確かめていれば、FINAL が音声の終わりから何ミリ秒後に届いたかを書きたい
  5. 気持ちを入れたい箇所があれば、著者の言葉で足す（エージェントは気持ちを書いていない）
  6. アドベントカレンダーの何日目の記事か（決まったら「はじめに」の先頭に書く）
事実の出典：本文中のリンク先の AWS 公式ドキュメント（2026年11月12日に確認）
-->

:::note info
**この記事は、Amazon Nova 2 Sonic の双方向ストリーミング API で、AI の発話を字幕や会話ログに出そうとしている人に向けて書いています。**

- Nova 2 Sonic が AI の発話テキストを2回送ってくる理由と、2つのテキストの違いを書きます
- 字幕には SPECULATIVE、会話ログには FINAL を使うという使い分けを持ち帰れます
:::

# はじめに

私は Amazon Nova 2 Sonic を使って、AI と音声で英会話を練習するアプリを作っています。Nova 2 Sonic は、音声を聞き取って音声で答える AI モデルで、音声と一緒に、話した内容のテキストも返してきます。

私はこのテキストを、会話ログと字幕に使いました。ところが2026年7月に、AI の発話が会話ログに2回ずつ記録される不具合が出ました。原因を調べると、Nova 2 Sonic は AI の発話1回につき、テキストを2回送ってくる仕様でした。

この記事では、2回届くテキストの違いと、私がアプリを2回直した経緯を書いていきます。

# 起きたこと：AI の発話が会話ログに2回ずつ記録された

会話ログには、たとえば次のように、同じ文が2行ずつ並びます。ホテルのフロント係を相手にチェックインを練習する場面の、AI の最初の台詞を使った例です。

```text
Good evening! Welcome to the Grand Pacific Hotel. Do you have a reservation?
Good evening! Welcome to the Grand Pacific Hotel. Do you have a reservation?
```

なお会話ログは、会話の後に AI が英語を評価するときの入力にも使っています。

# 原因：テキストは音声の前と後に1回ずつ届く

Nova 2 Sonic の出力は、`contentStart`、中身、`contentEnd` の3つのイベントで1つのブロックになっています。AI が1回話すと、ブロックは次の順で届きます（公式ドキュメントの [Event flow sequence](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-event-flow.html)）。

1. テキスト（SPECULATIVE）
2. 音声
3. テキスト（FINAL）

このように、テキストのブロックは音声の前と後に1つずつあり、それぞれの `contentStart` に `generationStage` という値が付いています。公式ドキュメントの [Handling output events with the bidirectional API](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-output-events.html) は、2つを次のように説明しています。

| generationStage | 届く時点 | 中身 |
|---|---|---|
| SPECULATIVE | 音声の前 | これから話す予定の内容 |
| FINAL | 音声の生成が終わった後 | 実際に話した内容を、文の単位で書き起こしたもの |

私は登壇資料で、この2つを「下書き」と「清書」と書きました。当時の私のアプリは、下書きと清書を区別せずに、届いたテキストをすべて会話ログに足していたので、同じ文が2回ずつ記録されていました。

なお、ユーザーの発話を音声認識したテキストも同じ形のブロックで届きますが、こちらの `generationStage` は FINAL です。

# 1回目の修正：FINAL だけを会話ログに残す

そこで2026年7月25日に、FINAL のテキストだけを会話ログに足すように直しました。直すときに気を付けた点は2つあります。

1つ目は、`generationStage` の入っている `additionalModelFields` が、JSON のオブジェクトではなく文字列で届くことです。次の例は、説明に使わない項目を省略しています。

```json
{
  "event": {
    "contentStart": {
      "type": "TEXT",
      "role": "ASSISTANT",
      "additionalModelFields": "{\"generationStage\":\"SPECULATIVE\"}"
    }
  }
}
```

2つ目は、音声のブロックの `contentStart` には `generationStage` が無いことです。私のコードは、`generationStage` が無いテキストを確定したものとして扱います。そのため `type` を見ずに判定すると、音声のブロックまで確定したテキストとして扱ってしまいます。そこで、`type` が `TEXT` のときだけ判定するようにしました。

次のコードは、ブラウザと Nova 2 Sonic の間で音声を受け渡す私のアプリの中継サーバーから、この判定の部分を抜き出して短くしたものです。

```typescript
let currentTextIsFinal = false;

function onOutputEvent(json: any) {
  const start = json.event?.contentStart;
  if (start?.type === 'TEXT') {
    // additionalModelFields は JSON の文字列で届く
    const raw = start.additionalModelFields;
    const fields = typeof raw === 'string' ? JSON.parse(raw) : raw;
    // generationStage が無いテキストは、確定したものとして扱う
    currentTextIsFinal = fields ? fields.generationStage === 'FINAL' : true;
  }
  // type が AUDIO の contentStart では、currentTextIsFinal を変えない

  const output = json.event?.textOutput;
  if (output && currentTextIsFinal) {
    appendToTranscript(output.role, output.content); // FINAL だけを会話ログに足す
  }
}
```

# FINAL だけにして残った問題：字幕が音声より先に出せない

この修正で会話ログの重複は直りましたが、字幕には別の問題が残りました。FINAL は音声の生成が終わった後に届くので、FINAL だけを使うと、その文の字幕を音声より先に出せません。

AI 2人とユーザーが討論するモードでは、10月まで、再生済みの秒数に1秒あたり16文字を掛けた分だけ字幕を出し、音声の生成が終わると残りを一度に出していました。そのため、字幕は音声に合っていませんでした。

そこで2026年10月18日に、捨てていた SPECULATIVE を字幕の先出しに使うことにしました。SPECULATIVE は音声の前に届くので、字幕を音声より先に用意できます。今は字幕の各文を、その文の音声が始まる見込みの0.35秒前に、1文ずつ出しています。一方で会話ログには、これまでどおり FINAL を使っています。

公式のサンプルコードも、画面に出す AI のテキストには SPECULATIVE を使っています（[Getting started with speech-to-speech](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-getting-started.html)）。

# 読者がすべきこと

Nova 2 Sonic のテキストは、使い道に合わせて選びましょう。

| 使い道 | 使うテキスト | 理由 |
|---|---|---|
| 字幕のように、すぐに画面へ出す表示 | SPECULATIVE | 音声の前に届く |
| 会話履歴、ログ、会話の後の評価 | FINAL | 実際に話した内容が入る |

公式ドキュメントの [Managing chat history](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-chat-history.html) も、会話履歴には FINAL を使い、SPECULATIVE は履歴に使わないように書いています。また、ユーザーが AI の発話の途中で話し始めて AI の音声が止まった場合も、FINAL には実際に話した内容が入ると、先ほどの Handling output events のページに書かれています。

実装するときは、次の2点も確かめましょう。

- `contentStart` の `type` が `TEXT` のときだけ、`generationStage` を読む
- `additionalModelFields` は文字列で届くので、JSON として読み直す

# おわりに

Nova 2 Sonic のテキストを扱う方は、次のことを試してみてください。

- `contentStart` の `generationStage` をログに出して、SPECULATIVE と FINAL の両方が届いていることを確かめる
- 画面に出すテキストと保存するテキストを分けて、それぞれにどちらを使うかを決める
- 公式ドキュメントの Event flow sequence で、ブロックが届く順序を確かめる
