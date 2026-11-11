# blog-materials

ブログ記事の原稿をまとめたリポジトリです。記事は Qiita などに投稿します。

登壇資料は [presentation-materials](https://github.com/Kenta-Matsuda/presentation-materials) に、記事を書くときのルールとテンプレートは [blog-template](https://github.com/Kenta-Matsuda/blog-template) にあります。

## 記事一覧

| 年月 | タイトル | 状態 | パス |
| --- | --- | --- | --- |
| 2026.12 | Nova 2 Sonic は AI の発話テキストを2回送ってくる（SPECULATIVE と FINAL の使い分け） | 下書き | [`2026_12/nova-sonic-speculative-final/`](2026_12/nova-sonic-speculative-final/) |

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

- 記事の中心は、著者の体験と言葉にする。著者の気持ちや動機を、執筆を手伝うエージェントが作らない。
- 制度、料金、仕様、手順は、公式の情報か著者の情報で確かめてから書く。
- IP アドレス、ドメイン名、AWS アカウント ID のような、環境を特定できる値は伏せる。
- 投稿は著者が判断する。
