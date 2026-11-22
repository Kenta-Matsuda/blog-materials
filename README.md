# blog-materials

ブログ記事の原稿をまとめたリポジトリです。記事は Qiita などに投稿します。

登壇資料は [presentation-materials](https://github.com/Kenta-Matsuda/presentation-materials) に、記事を書くときのルールとテンプレートは [blog-template](https://github.com/Kenta-Matsuda/blog-template) にあります。

## 記事一覧

| 年月 | タイトル | 状態 | パス |
| --- | --- | --- | --- |
| 2026.11 | 技術書典21で、AWSのニッチなサービスを解説する「AWS Deep Cuts」を９種類頒布します！ | 公開済み（https://qiita.com/kenken38/items/8f6059b0658317681589 ） | [`2026_11/techbookfest21-aws-deep-cuts/`](2026_11/techbookfest21-aws-deep-cuts/) |
| 2026.12 | Amazon Nova 2 Sonic の応答テキストは2種類ある：SPECULATIVE と FINAL の違いと使い分け | 下書き | [`2026_12/nova-sonic-speculative-final/`](2026_12/nova-sonic-speculative-final/) |

状態は「下書き」「著者確認中」「公開済み」のどれかを書きます。公開済みの記事には、投稿先の URL を足します。

## 進行中の企画

- [Qiita Advent Calendar 2026「AWS関連で、今年学んだこと」の計画](2026_12/README.md)

## 構成

```
README.md                         # 入り口（記事一覧）
YYYY_MM/README.md                 # その月の企画があるときの計画（無い月は置かない）
YYYY_MM/<記事名>/article.md        # 記事の原稿（Qiita の記法で書いた Markdown）
YYYY_MM/<記事名>/assets/           # 記事が参照する画像（あるときだけ）
```

`YYYY_MM` は、記事を公開する年月です。`<記事名>` は、英小文字とハイフンで付けます。

## 記事の追加手順

1. [blog-template](https://github.com/Kenta-Matsuda/blog-template) の `templates/article.md` をコピーして、`YYYY_MM/<記事名>/article.md` に置く。
2. 原稿の先頭にある「書く前のメモ」に、読者と、著者にしか書けない材料を書いてから本文を書く。
3. blog-template の点検スクリプトで、言い回しと装飾の量を確かめる。

   ```bash
   node ../blog-template/tools/check.cjs YYYY_MM/<記事名>/article.md
   ```

4. このファイルの「記事一覧」に、新しい順で行を足す。
5. 著者が手で直して投稿したら、状態を「公開済み」にして URL を足す。「書く前のメモ」は投稿前に消す。

## 原稿を書くときの決まり

- 書く前に、記事の核を1文で決める。技術ブログの核は、たいてい技術的な事実である。本文は、核を伝えるために必要なものだけで肉付けする。
- 読者に関係のない経緯（いつ気づいたか、どの順で直したか、著者しか知らないアプリの事情）は書かない。著者の体験は、核の理解を助ける分だけ入れる。
- 核の部分は、一次ソース（公式ドキュメントなど）を本文に示して、丁寧に書く。確かめられなかったことは断定せず、「書く前のメモ」に残して著者に確かめる。
- 誤解を招く言い換えやたとえを使わない。公式の用語をそのまま使う。
- 「読者がすべきこと」のような、読者に行動を求める節は作らない。読者は、タイトルを見て技術的に気になって来ただけかもしれない。
- タイトルは、その技術を知らない人にも内容が伝わる、分かりやすいものにする。
- 載せるコードは、動かして確かめる。
- 著者の気持ちや動機を、執筆を手伝うエージェントが作らない。
- IP アドレス、ドメイン名、AWS アカウント ID のような、環境を特定できる値は伏せる。
- 投稿は著者が判断する。
