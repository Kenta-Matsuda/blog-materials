---
title: Amazon Nova 2 Sonic の応答テキストは2種類ある：SPECULATIVE と FINAL の違いと使い分け
tags: AWS bedrock NovaSonic 生成AI TypeScript
---

<!--
書く前のメモ（投稿前に消す）
記事の核（1文）：Nova 2 Sonic は AI の応答のテキストを、話す前の予定（SPECULATIVE）と、話した後の書き起こし（FINAL）の2種類で返す。contentStart の generationStage で見分け、画面の表示には SPECULATIVE、会話履歴には FINAL を使う。
核を支える一次ソース：本文の「参考」の4ページ（AWS 公式ドキュメント）
著者の体験として残したこと：2種類を区別せずに表示すると同じ内容が二重に出る、という1点だけ。日付や経緯は省いた

確認できていないこと（著者に確かめたい）：
  1. 割り込みの通知（{ "interrupted" : true }）は、Nova Sonic（初代）の公式サンプルで確認した。Nova 2 Sonic のドキュメントでの記載は確認できていない。アプリの実機ログで、Nova 2 Sonic でも同じ形で届いているかを確かめたい
  2. 1回の応答で、SPECULATIVE と FINAL のブロックが1対1で届くのか、FINAL が文ごとに分かれて届くのか。公式は FINAL を「文の単位の書き起こし」としか書いていない。本文では個数を断定していない
  3. 実機のイベントのログを1往復ぶん載せられると、順序の説明の根拠が増える
  4. アドベントカレンダーの何日目の記事か
-->

:::note info
**この記事の要点**

- Amazon Nova 2 Sonic は、AI の応答のテキストを SPECULATIVE と FINAL の2種類で返します
- SPECULATIVE は音声の前に届く「これから話す内容」、FINAL は音声の後に届く「実際に話した内容の書き起こし」です
- 画面の表示には SPECULATIVE、会話履歴の保存には FINAL を使います
:::

# 2種類のテキストの違い

Amazon Nova 2 Sonic は、音声を聞き取って音声で答えるモデルです。応答の音声と一緒に、応答の内容をテキストでも返します。このテキストには次の2種類があり、区別せずに画面へ出すと、同じ内容が二重に表示されます。

| | SPECULATIVE | FINAL |
|---|---|---|
| 届く順序 | 音声の前 | 音声の後 |
| 中身 | これから話す予定の内容 | 実際に話した内容を、文の単位で書き起こしたもの |
| 向いている用途 | 字幕などの画面の表示 | 会話履歴の保存 |

以降では、この表の根拠と、2種類の見分け方を順に書いていきます。

# テキストが届く順序

Nova 2 Sonic との通信には、双方向ストリーミングの API（`InvokeModelWithBidirectionalStream`）を使います。モデルの出力は、`contentStart`、中身のイベント、`contentEnd` の3つで1つのブロックになります。

AI の応答では、ブロックは次の順序で届きます。

1. テキスト（SPECULATIVE）
2. 音声
3. テキスト（FINAL）

この順序は、公式ドキュメントの [Event flow sequence](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-event-flow.html) に載っています。また [Handling output events with the bidirectional API](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-output-events.html) のページは、SPECULATIVE を「話す予定の内容の予告」、FINAL を「音声の生成が終わった後に届く、実際に話した内容の文単位の書き起こし」と説明しています。

なお、ユーザーの発話を音声認識したテキストも、同じ形のブロックで届きます。こちらは `role` が `USER` で、`generationStage` は FINAL です。

# 見分け方

2種類のテキストは、ブロックの先頭に届く `contentStart` で見分けます。次の JSON は、公式ドキュメントの形式から、説明に使う項目だけを抜き出したものです。

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

見分けるときに間違えやすい点が2つあります。

1つ目は、`additionalModelFields` の値が JSON のオブジェクトではなく、JSON を文字列にしたものだという点です。そのため、`generationStage` を読む前に、この文字列を JSON として読み直します。

2つ目は、`additionalModelFields` が付くのは `type` が `TEXT` の `contentStart` だけだという点です。公式ドキュメントに載っている `AUDIO` と `TOOL` の `contentStart` の形式には、この項目がありません。そのため、`type` を確かめてから `generationStage` を読みます。

# 中身が食い違う場合

SPECULATIVE と FINAL は、ふつうは同じ内容です。ただし SPECULATIVE は話す前の予定で、FINAL は話した後の書き起こしなので、2つが食い違う場合があります。

その1つが割り込みです。ユーザーが AI の発話の途中で話し始めると、AI の音声は途中で止まります。先ほどの Handling output events のページは、音声が割り込みで止まった場合も、FINAL には実際に話した内容が入ると書いています。一方で SPECULATIVE は音声の前に届く予定の内容なので、話されなかった部分を含むことがあります。

# 使い分け

## 画面の表示には SPECULATIVE

SPECULATIVE は音声の前に届くので、字幕を音声に遅れずに出せます。公式のサンプルコード（[Getting started with speech-to-speech](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-getting-started.html)）も、AI のテキストは SPECULATIVE のときだけ表示しています。

FINAL を表示に使うと、テキストが音声の後に届くので、字幕が音声より遅れます。

## 会話履歴の保存には FINAL

FINAL には実際に話した内容が入るので、会話履歴やログに向いています。公式ドキュメントの [Managing chat history](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-chat-history.html) も、会話履歴を保存するときは `generationStage` を見るように書いていて、SPECULATIVE を履歴には使わない予告として扱っています。

私は英会話の練習アプリで、最初は2種類を区別せずに会話ログへ足していました。その結果、AI の発話が会話ログに2回ずつ記録されました。

# 判定のコード

次のコードは、届いたイベントを2種類に振り分ける例です。私のアプリの処理を、説明のために短くしています。`showCaption` と `saveHistory` は、表示と保存を行う関数だと考えてください。

```typescript
type Stage = 'SPECULATIVE' | 'FINAL' | undefined;

let role: string | undefined;
let stage: Stage;

// event は、モデルから届いた JSON の event の中身
function onEvent(event: any) {
  if (event.contentStart) {
    role = event.contentStart.role;
    stage = undefined;
    if (event.contentStart.type === 'TEXT') {
      // additionalModelFields は JSON を文字列にしたもの
      const fields = JSON.parse(event.contentStart.additionalModelFields ?? '{}');
      stage = fields.generationStage;
    }
  } else if (event.textOutput) {
    const text: string = event.textOutput.content;
    // 割り込みの通知は、表示にも保存にも使わない
    if (/"interrupted"\s*:\s*true/.test(text)) return;
    if (role === 'ASSISTANT' && stage === 'SPECULATIVE') showCaption(text);
    if (stage === 'FINAL') saveHistory(role, text);
  }
}
```

コードの中の割り込みの通知とは、`{ "interrupted" : true }` という内容のテキストです。私のアプリでは、割り込みが起きると、これが `textOutput` として届きます。Nova Sonic（初代）の公式サンプルコード（[Using the Bidirectional Streaming API](https://docs.aws.amazon.com/nova/latest/userguide/speech-bidirection.html)）も、この文字列で割り込みを検出しています。

# まとめ

- Nova 2 Sonic の応答のテキストは、音声の前に届く SPECULATIVE と、音声の後に届く FINAL の2種類です
- 2種類は、`type` が `TEXT` の `contentStart` にある `generationStage` で見分けます
- 画面の表示には SPECULATIVE を、会話履歴の保存には FINAL を使います

# 参考

すべて AWS の公式ドキュメントです（2026年11月に確認）。

- [Handling output events with the bidirectional API](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-output-events.html)：SPECULATIVE と FINAL の説明、`contentStart` の形式
- [Event flow sequence](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-event-flow.html)：イベントが届く順序
- [Managing chat history](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-chat-history.html)：会話履歴を保存するときの `generationStage` の扱い
- [Getting started with speech-to-speech](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-getting-started.html)：SPECULATIVE を表示に使うサンプルコード
- [Using the Bidirectional Streaming API](https://docs.aws.amazon.com/nova/latest/userguide/speech-bidirection.html)：Nova Sonic（初代）のサンプルコード。割り込みの通知の検出
