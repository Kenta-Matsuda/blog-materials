---
title: Strands Agents の会話履歴は window_size を超えると消える。ConversationManager の3つのモードで挙動を比べた
tags: AWS StrandsAgents AIエージェント Python 生成AI
---

<!--
書く前のメモ（投稿前に消す）
記事の核（1文）：Strands Agents の既定の SlidingWindowConversationManager は、会話履歴が window_size 件を超えると古いメッセージを消すので、消えた内容をエージェントは答えられない。NullConversationManager や SummarizingConversationManager に変えると、消さずに残す、または要約して残す挙動に変わる。
核を支える一次ソース：本文の「参考」の公式ドキュメント3ページと、GitHub のソースコード2ファイル
読者：Strands Agents でエージェントを作り始めた人。会話履歴の設定項目を見たが、何が起きるかを確かめていない人

検証で確かめること（著者が実行してキャプチャを貼る）：
  1. 本文の「実行結果」のコードブロックは、期待される出力として書いた。messages と removed_message_count の数は、ソースコードの計算どおりなら本文の値になる。モデルの返答の文言は実行のたびに変わるので、本文では文言を書かずに内容だけを書いた
  2. 検証した strands-agents のバージョンを、本文の「検証の環境」に書く（下書きでは空けてある）
  3. SummarizingConversationManager の要約メッセージの role。ソースコードでは、要約用のエージェントを使う経路だけ user の role で包む処理が見えた。既定の経路の role は確かめられていないので、本文では role を書いていない
  4. summary_ratio=0.5、preserve_recent_messages=2 で6件の履歴を要約すると、先頭の3件（1往復目と、2往復目のユーザーの発言）が要約される計算になる。実際にこの区切りになるかを、キャプチャで確かめたい
  5. pin_first と proactive_compression は GitHub の main のソースコードにある引数である。PyPI で配布中のバージョンに入っているかは確かめていない。本文では proactive_compression だけを、公式ドキュメントの例に載っている範囲で触れた
  6. アドベントカレンダーの記事にするか、単独の記事にするか
-->

Strands Agents の `Agent` には、会話履歴の扱いを決める `conversation_manager` という引数があります。既定の設定では、履歴が決まった件数を超えると古いメッセージから消えていきます。

この記事では、件数を超えた古い履歴が本当に残らないことを、エージェントに名前を覚えさせて確かめます。そのうえで、ConversationManager を別の種類に変えると挙動がどう変わるかを比べます。

# ConversationManager の3つの種類

Strands Agents のエージェントは、会話履歴を `agent.messages` というリストに持っています。エージェントを呼ぶたびに、このリストの中身がまとめてモデルに送られます。そのため、会話が長くなるほど、モデルに送る量が増えていきます。

この量をどう抑えるかを決めるのが ConversationManager です。公式ドキュメントの [Conversation Management](https://strandsagents.com/docs/user-guide/concepts/agents/conversation-management/) によると、SDK には次の3つが入っています。

| 種類 | 履歴の扱い |
| --- | --- |
| `SlidingWindowConversationManager` | 新しいほうから決まった件数だけ残し、古いメッセージを消す。`Agent` の既定 |
| `NullConversationManager` | 履歴に手を加えない |
| `SummarizingConversationManager` | 古いメッセージを消す代わりに、要約に置き換える |

`SlidingWindowConversationManager` の残す件数は、`window_size` という引数で決めます。GitHub のソースコード（[sliding_window_conversation_manager.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/agent/conversation_manager/sliding_window_conversation_manager.py)）では、`window_size` の既定値は40です。つまり、何も設定しなければ、直近の40件より古いメッセージは消えます。

# 検証のコード

確かめ方は単純で、1往復目でエージェントに名前を伝え、4往復目で名前を聞き返します。途中で名前とは関係のない話を2往復はさむので、`window_size` を小さくしておけば、名前を伝えた1往復目が履歴から押し出されるはずです。

次のコードは、コマンドライン引数で ConversationManager の種類を切り替えて、4往復の会話をします。1回呼ぶごとに `agent.messages` の件数を表示し、最後に履歴の中身と、消したメッセージの数（`removed_message_count`）を表示します。

```python:history_demo.py
import sys

from strands import Agent
from strands.agent.conversation_manager import (
    NullConversationManager,
    SlidingWindowConversationManager,
    SummarizingConversationManager,
)

PROMPTS = [
    "私の名前はけんけんです。",
    "好きなAWSサービスはLambdaです。",
    "今日は雨が降っています。",
    "私の名前は何でしたか？",
]


def print_history(agent: Agent) -> None:
    for i, message in enumerate(agent.messages):
        text = "".join(block.get("text", "") for block in message["content"])
        print(f"  [{i}] {message['role']}: {text[:30]!r}")


def main(mode: str) -> None:
    if mode == "sliding":
        manager = SlidingWindowConversationManager(window_size=4)
    elif mode == "null":
        manager = NullConversationManager()
    elif mode == "summarizing":
        manager = SummarizingConversationManager(
            summary_ratio=0.5,
            preserve_recent_messages=2,
        )
    else:
        raise ValueError(f"unknown mode: {mode}")

    agent = Agent(
        system_prompt="短く日本語で答えてください。",
        conversation_manager=manager,
        callback_handler=None,
    )

    for turn, prompt in enumerate(PROMPTS, start=1):
        if mode == "summarizing" and turn == len(PROMPTS):
            # 要約は履歴があふれたときに動くので、検証のために手で呼ぶ
            manager.reduce_context(agent)
            print("--- reduce_context を呼んだ後 ---")
            print_history(agent)

        result = agent(prompt)
        print(f"[{turn}] {prompt}")
        print(f"  -> {str(result).strip()}")
        print(f"  messages: {len(agent.messages)}")

    print("--- 最後の履歴 ---")
    print_history(agent)
    print(f"removed_message_count: {manager.removed_message_count}")


if __name__ == "__main__":
    main(sys.argv[1])
```

`callback_handler=None` は、返答を標準出力にストリーミングで出す既定の動作を止める指定です。出力が混ざらないように止めています。モデルは指定していないので、既定の Amazon Bedrock のモデルが使われます。

## 検証の環境

| 項目 | 値 |
| --- | --- |
| strands-agents | <!-- 検証したバージョンを書く --> |
| Python | <!-- 検証したバージョンを書く --> |
| モデル | Amazon Bedrock の既定のモデル |

# SlidingWindowConversationManager は件数を超えた履歴を消す

まずは既定の `SlidingWindowConversationManager` を、`window_size=4` で動かします。

```bash
python history_demo.py sliding
```

<!-- キャプチャ：python history_demo.py sliding の実行結果 -->

```text:実行結果（件数の部分）
[1] 私の名前はけんけんです。
  messages: 2
[2] 好きなAWSサービスはLambdaです。
  messages: 4
[3] 今日は雨が降っています。
  messages: 4
[4] 私の名前は何でしたか？
  messages: 4
--- 最後の履歴 ---
  [0] user: '今日は雨が降っています。'
  [1] assistant: '...'
  [2] user: '私の名前は何でしたか？'
  [3] assistant: '...'
removed_message_count: 4
```

4往復目の質問に、エージェントは名前を答えられませんでした。返答は「名前はまだ伺っていない」という内容で、1往復目に伝えた「けんけん」は出てきません。

件数を見ると、何が起きたかが分かります。1回の呼び出しで、ユーザーの発言とエージェントの返答の2件が履歴に増えます。そのため3往復目が終わった時点で履歴は6件になり、`window_size` の4件を超えた分として、1往復目の2件が消えました。4往復目のモデルには、2往復目と3往復目の4件と、新しい質問だけが送られています。なので、モデルから見ると、名前は一度も聞いていない状態です。

最後に表示した履歴でも、残っているのは3往復目と4往復目だけで、`removed_message_count` は消えた件数の合計の4になっています。消えたメッセージは、要約などの形でも残っていません。

## window_size は往復の数ではなくメッセージの数

この結果で分かるとおり、`window_size=4` で残るのは4往復ではなく、4件のメッセージ（2往復）です。

ここは公式ドキュメントの中でも書き方が分かれています。[Conversation Management](https://strandsagents.com/docs/user-guide/concepts/agents/conversation-management/) のページは `window_size` を「残すメッセージの最大数」と書いていますが、[State Management](https://strandsagents.com/docs/user-guide/concepts/agents/state/) のページのコード例のコメントは「message pairs」（メッセージの組）になっています。ソースコードの `apply_management` は `agent.messages` の件数と `window_size` を比べているので、数えているのはメッセージの件数です。往復の数で考えて `window_size=10` にすると、残るのは5往復だけになります。

## 削るのは呼び出しが終わった後

もう1つ押さえておきたいのが、削るタイミングです。4往復目のモデルに送られた履歴は、残っていた4件に新しい質問を足した5件で、`window_size` の4件を一時的に超えています。

これは、`SlidingWindowConversationManager` が既定では、エージェントの1回の呼び出しが終わった後に履歴を削るためです。公式ドキュメントでは、Python 版の `per_turn` という引数でこのタイミングを変えられると説明しています。

| `per_turn` の値 | 削るタイミング |
| --- | --- |
| `False`（既定） | エージェントの呼び出しが終わった後だけ |
| `True` | モデルを呼ぶ前に毎回 |
| 整数の `N` | モデルを `N` 回呼ぶごと |

今回のようにツールを使わない会話では、1回の呼び出しでモデルを呼ぶのは1回なので、違いはほとんどありません。一方で、1回の呼び出しの中でツールを何度も使うエージェントでは、呼び出しの途中で履歴が `window_size` を大きく超えることがあります。途中でも削りたいときに `per_turn` を使います。

# NullConversationManager に変えると履歴は全部残る

次に、同じ会話を `NullConversationManager` で動かします。

```bash
python history_demo.py null
```

<!-- キャプチャ：python history_demo.py null の実行結果 -->

```text:実行結果（件数の部分）
[1] 私の名前はけんけんです。
  messages: 2
[2] 好きなAWSサービスはLambdaです。
  messages: 4
[3] 今日は雨が降っています。
  messages: 6
[4] 私の名前は何でしたか？
  messages: 8
--- 最後の履歴 ---
  [0] user: '私の名前はけんけんです。'
  [1] assistant: '...'
  [2] user: '好きなAWSサービスはLambdaです。'
  [3] assistant: '...'
  [4] user: '今日は雨が降っています。'
  [5] assistant: '...'
  [6] user: '私の名前は何でしたか？'
  [7] assistant: '...'
removed_message_count: 0
```

今度は、エージェントが「けんけん」と答えました。履歴は呼び出しのたびに2件ずつ増えて8件になり、1件も消えていません。

ただし、`NullConversationManager` は、履歴がモデルのコンテキストウィンドウ（一度に受け取れる入力の量）を超えたときも何も削りません。そのため、長い会話ではエラーで止まります。公式ドキュメントも、短い会話やデバッグ、履歴を自分で管理する場合に向いた種類として紹介しています。

# SummarizingConversationManager に変えると要約が残る

最後に `SummarizingConversationManager` です。こちらは、古いメッセージを消す代わりに要約へ置き換えます。

## 件数では要約されない

まず気をつけたいのは、`SummarizingConversationManager` には `window_size` のような件数の上限が無いことです。ソースコード（[summarizing_conversation_manager.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/agent/conversation_manager/summarizing_conversation_manager.py)）では、呼び出しの後に動く `apply_management` は履歴を変えません。要約するのは `reduce_context` で、これはモデルのコンテキストウィンドウを超えたときに呼ばれます。

そのため、`reduce_context` を呼ぶ行を消して同じ会話をすると、結果は `NullConversationManager` と同じになります。

<!-- キャプチャ：reduce_context を呼ぶ行を消して、python history_demo.py summarizing を実行した結果（messages が 2, 4, 6, 8 と増える） -->

4往復程度の会話でコンテキストウィンドウを超えることは無いので、検証のコードでは4往復目の前に `reduce_context` を手で呼んでいます。

## 要約された後の履歴

`reduce_context` を呼ぶと、古いほうから決まった割合のメッセージが要約されます。割合は `summary_ratio`（既定は0.3）で、直近の `preserve_recent_messages` 件（既定は10）は要約せずに残します。既定のままだと10件より短い履歴は要約できないので、検証では `summary_ratio=0.5`、`preserve_recent_messages=2` にしました。この設定では、6件の履歴のうち古いほうの3件が要約されます。

```bash
python history_demo.py summarizing
```

<!-- キャプチャ：python history_demo.py summarizing の実行結果 -->

```text:実行結果（件数の部分）
[1] 私の名前はけんけんです。
  messages: 2
[2] 好きなAWSサービスはLambdaです。
  messages: 4
[3] 今日は雨が降っています。
  messages: 6
--- reduce_context を呼んだ後 ---
  [0] user: '...'
  [1] assistant: '...'
  [2] user: '今日は雨が降っています。'
  [3] assistant: '...'
[4] 私の名前は何でしたか？
  -> ...
  messages: 6
--- 最後の履歴 ---
  [0] user: '...'
  [1] assistant: '...'
  [2] user: '今日は雨が降っています。'
  [3] assistant: '...'
  [4] user: '私の名前は何でしたか？'
  [5] assistant: '...'
removed_message_count: 3
```

`reduce_context` を呼んだ後、履歴の先頭の3件が1件の要約に置き換わり、履歴は4件になりました。要約の中には、ユーザーの名前が「けんけん」であることと、好きなAWSサービスが Lambda であることが入っています。

<!-- キャプチャ：要約メッセージ（履歴の [0]）の全文 -->

そして4往復目では、エージェントは「けんけん」と答えました。元の1往復目のメッセージは履歴から消えていますが、要約が残っているので、モデルは名前を知ることができます。`removed_message_count` は、要約に置き換えた3件です。

なお、公式ドキュメントの API リファレンスには、コンテキストウィンドウを超える前に先回りして要約する `proactive_compression` という引数の例も載っています。`proactive_compression=True` にすると、ソースコードではコンテキストウィンドウの70%に達したときに要約します。

# SlidingWindowConversationManager で件数がずれる場面

ここまでの検証はツールを使わない会話なので、履歴はきれいに `window_size` の件数になりました。ツールを使うエージェントでは、次の2つの理由で、残る件数が `window_size` と一致しないことがあります。

1つ目は、ツールの呼び出しとその結果を切り離さないことです。ソースコードでは、履歴を削る位置を、ツールの結果（`toolResult`）だけが先頭に残らないように後ろへずらしています。そのため、削った後の履歴は `window_size` より少なくなることがあります。

2つ目は、コンテキストウィンドウを超えたときの動きです。`should_truncate_results`（既定は `True`）が有効だと、`SlidingWindowConversationManager` は古いメッセージを消す前に、まず大きなツールの結果を短く切り詰めます。この場合は、件数は変わらずにメッセージの中身が短くなります。

# 参考

- [Conversation Management](https://strandsagents.com/docs/user-guide/concepts/agents/conversation-management/)（Strands Agents 公式ドキュメント）：3つの ConversationManager の説明、`window_size`、`should_truncate_results`、`per_turn`、`summary_ratio`、`preserve_recent_messages`
- [State Management](https://strandsagents.com/docs/user-guide/concepts/agents/state/)（Strands Agents 公式ドキュメント）：`agent.messages` と、既定の ConversationManager
- [strands.agent.conversation_manager.conversation_manager](https://strandsagents.com/docs/api/python/strands.agent.conversation_manager.conversation_manager/)（API リファレンス）：`proactive_compression` の例
- [sliding_window_conversation_manager.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/agent/conversation_manager/sliding_window_conversation_manager.py)（GitHub）：`window_size` の既定値、削る位置の決め方、ツールの結果の切り詰め
- [summarizing_conversation_manager.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/agent/conversation_manager/summarizing_conversation_manager.py)（GitHub）：`apply_management` と `reduce_context` の動き、要約する件数の計算
