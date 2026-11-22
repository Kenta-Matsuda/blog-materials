---
title: 技術書典21で、AWSのニッチなサービスを解説する「AWS Deep Cuts」を９種類頒布します！
tags: AWS 技術書典 ハンズオン
---

<!--
書く前のメモ（本文は 2026-11-22 に著者の公開版に差し替えた）
読者：AWS を使うエンジニア全般（著者に確認済み）。公開版では、マニアックな内容なので「AWS ヲタク」に絞った（著者の判断）
読者が読む前に困っていること：新しい・ニッチな AWS サービスを、前提から手を動かして理解できる日本語の資料が少ない
読者が読んだ後にすること：気になる巻の頒布ページを見る。技術書典21（オンライン 12/6 まで、オフライン 11/23）で手に取る
著者にしか書けない材料：各巻の題材とハンズオンの中身（技術書典に登録した概要と、各巻の原稿から取った）。動機は、本として出すことでボリュームのある解説ができること（著者の補足。最初の回答「紙面で読める形にしたい」をエージェントが字義どおりに書き、ニュアンスが違った）
省く話：本づくりの裏話（実機検証、PDF のフォントの修正）、NCP-GENL の本（著者の回答により省く）
画像：assets/ の 18 枚（各巻の表紙と試し読み）。著者が Qiita にアップロードして、src を差し替える
Qiita：2026-11-21 に Qiita CLI で限定共有として投稿（記事 ID 8f6059b0658317681589）。著者が手直しして公開した：https://qiita.com/kenken38/items/8f6059b0658317681589
-->

:::note info
**この記事は、AWS ヲタクに向けて書いています。**

- AWS のあまり知られていないサービスや最新サービスを扱うシリーズ「AWS Deep Cuts」を、技術書典21 で 電子書籍として頒布します！
- 「AWS Deep Cuts」で扱う AWS サービスは、基本的に**マニアックすぎて業務で役立ちません！**
:::

# はじめに

技術書典21 で、AWS Deep Cuts を 9 冊の電子書籍として頒布します。オンラインマーケットでの頒布は 2026/11/21 から 12/6 まで、オフライン会場は 11/23 です。

AWS Deep Cuts は、AWS のあまり知られていないサービスや、発表されたばかりのサービスを 1 つずつ取り上げ、前提知識から解説して、ハンズオンで動きを確かめる技術シリーズです。これまで、GitHub の [AWS-Deep-Cuts](https://github.com/AWS-Deep-Cuts) にあるハンズオンのリポジトリと、Web の記事で公開してきました。

今回は新たに 9 つの題材を選び、完全新作の本（PDF）として公開しました。これまでの Web 記事からパワーアップし、100ページを超えるボリューミーな解説を加えています。

この記事では、9 冊それぞれが扱う内容について紹介します。

# 頒布情報

| 項目 | 内容 |
|---|---|
| イベント | [技術書典21](https://techbookfest.org/event/tbf21) |
| サークル | [松田研究所](https://techbookfest.org/organization/uifF8PZ4iUULwqyiTynePb)（オフライン会場：さ10） |
| オンライン | 2026/11/21（土）〜 12/6（日） |
| オフライン | 2026/11/23（月・祝）11:00〜17:00<br>池袋・サンシャインシティ 展示ホールD（文化会館ビル2F） |
| 形式 | 電子版のみ（PDF、A5・フルカラー） |

オフライン会場でもオンラインでも、価格は 1冊 500 円です。

オフライン会場で買う場合は、技術書典の[イベントページ](https://techbookfest.org/event/tbf21)の案内に従って、入場券と、技術書典の「かんたん後払い」アプリが必要ですのでご注意ください。

# 書籍紹介

9 冊とも、大枠の構成は同じです。

| 章 | 内容 |
|---|---|
| 第1章 | そのサービスが解決する問題と、理解に必要な前提知識 |
| 第2章 | サービスのしくみ、料金、制約、使いどころ |
| 第3章 | 自分の AWS アカウントで動かして確かめるハンズオン |
| 付録 | 用語集 |

それでは 1 冊ずつ、計 9 冊を紹介していきます。

# vol.1 IAM Access Troubleshooter

![vol1-cover.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/2aa25042-a8bd-49db-967c-1b1bf22ca964.png)

AccessDenied の原因を、IAM の評価の記録から特定する本です。許可ポリシー、アクセス許可の境界、セッションポリシー、SCP が何重にも重なると、どのポリシーで拒否されたのかは、エラーメッセージだけでは分からないことがあります。

IAM Access Troubleshooter（2026 年 8 月からプレビュー）は、拒否された 1 つのリクエストについて、IAM がどのポリシーをどう評価したかを「評価の詳細」として表示するサービスです。本書はこのサービスを、IAM がポリシーを評価する順序という前提知識から解説します。

ハンズオンでは、わざと拒否されるように作ったロールを、4 つのステージで 1 つずつ直します。拒否されるたびに、エラーメッセージに含まれる authorization ID で評価の詳細を開き、原因を特定して設定を直します。

- 102 ページ
- [技術書典の頒布ページ](https://techbookfest.org/product/8z43FQDwkrb5rJTFBdSNnn)

<details><summary>試し読み</summary>

![vol1-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/c95bcbfc-0ad9-4998-a73e-e21fe63e93ab.png)

</details>

# vol.2 IAM Access Analyzer

![vol2-cover.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/4d14e380-6adb-4889-9261-8d23fd7cc75f.png)

ポリシーを実際に試して分かるのは、試したリクエストの結果だけです。オブジェクトの削除が拒否されても、試していない `s3:PutBucketPolicy` が許可されていないかは分かりません。

vol.2 は、ポリシーが何を許可しうるかを、リクエストを 1 つも送らずに確かめる本です。IAM Access Analyzer のカスタムポリシーチェックは、「このポリシーは、問いに当てはまるリクエストを 1 つでも許可しうるか」に、自動推論で答えます。本書はこのしくみを、自動推論の考え方から解説します。

ハンズオンでは、ポリシーの検証と 3 つのチェック（指定したアクセス、新しいアクセス、パブリックアクセス）を順に試します。そのあと、AWS が公開している CloudFormation 用の検証ツール `cfn-policy-validator` で、デプロイ前に CI で行うのと同じ検査をします。おまけとして、4 クイーン問題をポリシーに置き換えて Access Analyzer に解かせる手順も収録しています。

- 128 ページ
- [技術書典の頒布ページ](https://techbookfest.org/product/5weZWAHUGN3mML72VTzGiM)

<details><summary>試し読み</summary>

![vol2-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/62d81de6-7069-420a-ae90-da534247fdb2.png)

</details>

# vol.3 AWS Nitro Enclaves

![vol3-cover.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/8400e9b1-f1bc-416f-8ffc-9467cb7b50f1.png)

ディスクも通信も暗号化できますが、計算するときには、データはメモリの上で復号されます。そのため、サーバーの root を持つ人には読めてしまいます。

vol.3 は、使用中のデータを、ホストの管理者からも隔離して守る本です。AWS Nitro Enclaves は、EC2 インスタンスの vCPU とメモリを切り出して、root からも隔離された VM（Enclave）を作ります。本書は、Enclave が自分の中身を証明するアテステーション文書と、KMS のキーポリシーを組み合わせて、Enclave の中だけで秘密を復号するしくみまでを解説します。

ハンズオンの題材は、運営者にも中身を見せずに、シークレットサンタ（プレゼント交換）の組み合わせを決めるサービスです。東京リージョンで Enclave を起動し、運営者が参加者の希望をのぞき見ようとする 4 つの方法を自分で試して、どれも失敗することを確かめます。

- 160 ページ
- [技術書典の頒布ページ](https://techbookfest.org/product/jhXpNQE3nGG44jcm8Gq9yN)

<details><summary>試し読み</summary>

![vol3-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/1b224d4a-0010-4b92-a531-7609b58bfcf3.png)

</details>

# vol.4 AWS Lambda MicroVMs

![vol4-cover.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/2fa010ed-9e06-47f2-bfe1-11db409213c8.png)

AI エージェントがその場で生成したコードは信頼できないため、隔離した環境で実行する必要があります。そのうえで、会話の途中で作ったファイルを次の依頼でも使えること、利用者を待たせないこと、利用者が次の依頼を考えている間の費用を抑えることも求められます。コンテナ、Lambda 関数、EC2 は、それぞれこれらの条件のどれかを満たしにくい選択肢です。

vol.4 は、AI エージェントが生成したコードを、セッションごとの microVM に閉じ込める本です。2026 年 6 月に一般提供が始まった AWS Lambda MicroVMs は、セッションごとの専用の microVM を API で起動し、メモリとディスクを保ったままサスペンドできるサービスです。本書はこのサービスを、サンドボックスに求められる上の 4 つの条件から解説します。

ハンズオンでは、MicroVM イメージの作成、起動、認証付きエンドポイントへのアクセス、サスペンドとレジュームを順に確かめます。最後に、Amazon Bedrock に生成させた Python のコードを MicroVM の中で実行します。

- 126 ページ
- [技術書典の頒布ページ](https://techbookfest.org/product/gfnUSb5EGsRiyW8yVT5SQ5)

<details><summary>試し読み</summary>

![vol4-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/7156579b-8b4d-401f-a4b9-80ee0f2a3ed6.png)

</details>

# vol.5 AWS Continuum

![vol5-cover.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/edbc840b-26f8-4d96-9987-000b10c19749.png)

脆弱性の指摘は、AI コーディングツールに聞けば手に入ります。しかし、どの指摘が本当に悪用できるのかを確かめる検証には手間がかかり、指摘が増えるほど負担は重くなります。

vol.5 は、脆弱性の検証を AI エージェントに任せる本です。2026 年 6 月に発表された AWS Continuum は、セキュリティの作業を行う AWS の AI エージェントの機能の総称です。本書は、その一部である AWS Security Agent を中心に解説します。Security Agent はコードを AWS の中で分析し、確信度と検証の状態を持つ Finding として記録します。Claude Code からプラグイン経由で依頼することもできます。

ハンズオンでは、意図的に脆弱性を仕込んだレコード店のアプリを使い、自分の予想、AI コーディングツールだけで行うレビュー、Security Agent のフルスキャンの 3 つを採点表で比べます。そのあと、修正と差分スキャン、脅威モデリングまで進めます。

- 166 ページ
- [技術書典の頒布ページ](https://techbookfest.org/product/mhW7Z72yAqSWN3jFKEbvXg)

<details><summary>試し読み</summary>

![vol5-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/78c120ce-db87-4a33-9963-b7b803e772bb.png)

</details>

# vol.6 Regional NAT Gateway

![vol6-cover.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/65ba151b-c337-4c82-a63d-0bc2ef99d557.png)

NAT ゲートウェイは、AZ ごとに置いて可用性をとるか、1 つにまとめて NAT ゲートウェイとパブリックサブネットの数と料金を減らすか、という二択で設計されてきました。

2025 年 11 月に、NAT ゲートウェイの availability mode に regional を指定できるようになりました。regional にすると、1 つの NAT ゲートウェイが VPC の複数の AZ にまたがって動き、パブリックサブネットも不要になります。vol.6 は、この Regional NAT Gateway のしくみと使いどころを解説する本です。公式ドキュメントのページによって説明が異なる点や、説明が見つからない点は、自分のアカウントで観測して確かめます。

ハンズオンでは、パブリックサブネットの無い VPC に regional NAT ゲートウェイを作成し、2 つの AZ から外へ出る通信の送信元 IP を観測します。新しい AZ に拡張されるまでの時間を測り、送信元 IP を自分で指定する手動モードで、送信元 IP を自分の EIP に変えます。

- 134 ページ
- [技術書典の頒布ページ](https://techbookfest.org/product/21nN3VccpXc4rLeujyAqtb)

<details><summary>試し読み</summary>

![vol6-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/bdcb7233-c820-4a74-9274-0976503078ad.png)

</details>

# vol.7 Amazon S3 Files

![vol7-cover.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/edd044f0-4328-46b3-b1e9-64677e87a079.png)

S3 の CSV を EC2 で集計するたびに、`aws s3 cp` でダウンロードして書き戻していないでしょうか。ファイルが数百個になると、どれが最新かを管理するだけで手間がかかります。

vol.7 は、S3 バケットを、複数のサーバーからマウントできる共有ファイルシステムにする本です。Amazon S3 Files は、S3 の汎用バケットを、NFS（Network File System）のファイルシステムとして提供します。ただし、ファイルとオブジェクトの違いが消えるわけではありません。本書では、書いた内容が S3 に届くまでの時間、名前の変更にかかる時間と費用、両側から書き換えたときにどちらの版が残るかを、しくみとハンズオンの両方から確かめます。

ハンズオンでは、CloudShell と EC2 の 2 つの画面で、ファイルシステムを作成してマウントし、同期の動きを計測します。応用として、git と SQLite がこのファイルシステムの上で動くかも試します。

- 150 ページ
- [技術書典の頒布ページ](https://techbookfest.org/product/av72Bqu5PkL3r3LWeq6Ls6)

<details><summary>試し読み</summary>

![vol7-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/de4fea40-7c52-4a44-95fb-0dac28e067ba.png)

</details>

# vol.8 AWS IoT Core Device Location

![vol8-cover.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/58b8af4e-2e98-4d0f-971f-42c489cf9e9d.png)

地図アプリの現在地に表示される薄い円は、何を表しているのでしょうか。GPS を持たない端末は、周囲の電波を測って位置を推し量っています。しかし Wi-Fi の測定値は、アクセスポイントの MAC アドレスと電波の強さからなり、アクセスポイントがどこにあるかは含まれません。そのため、アクセスポイントの場所を知っている仕組みに、計算を任せる必要があります。

vol.8 は、Wi-Fi などの測定値から、GPS を使わずに位置を推定する本です。AWS IoT Core Device Location は、Wi-Fi、携帯電話の基地局、IP アドレス、GNSS（GPS のような衛星による測位の総称）の測定値から位置を推定して、GeoJSON で返すサービスです。本書はこのサービスを、位置の推定の考え方から解説します。

ハンズオンでは、自分の PC で測った Wi-Fi の値を AWS に送り、GPS なしで自分の居場所を推定してもらいます。推定された位置を実際の位置と答え合わせし、信頼度やアクセスポイントの数を変えたときの違いも確かめます。AWS のリソースは作成せず、費用は最大で 0.010 USD 程度です。

- 106 ページ
- [技術書典の頒布ページ](https://techbookfest.org/product/mSLHjXasSFb274JMRn517y)

<details><summary>試し読み</summary>

![vol8-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/199eacc2-5c39-42f3-9300-2fe887e020e0.png)

</details>

# vol.9 Amazon GameLift Servers FlexMatch

![vol9-cover.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/166ece06-1c40-47b4-aed5-21a1be676e69.png)

研修やハッカソンのチーム分けでは、人数をそろえ、経験者を偏らせず、得意分野も散らし、「この 2 人は同じチームに」という頼みにも応える必要があります。条件が増えるほど、1 つ直すと別の条件が崩れます。

同じ問題を毎日大量に解いているのが、対戦ゲームのマッチメイキングです。vol.9 は、チームの条件を JSON のルールセットに書くだけで組み合わせを返す Amazon GameLift Servers FlexMatch を解説する本です。スタンドアロンモードなら、ゲームサーバーなしで、チーム分けの結果だけを受け取れます。

ハンズオンでは、架空の勉強会の参加者 24 人を、6 回のラウンドで 6 人ずつ 4 チームに分けます。幹事の要望を 1 つずつルールに足して結果がどう変わるか、満たせないルールのときに、申し込みの記録（チケット）がどう残るかを確かめます。ゲーム開発の知識は不要です。

- 146 ページ
- [技術書典の頒布ページ](https://techbookfest.org/product/7b4Mr4bG5tWBj1EWfuGi15)

<details><summary>試し読み</summary>

![vol9-sample.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3120542/66e1e424-39d8-4e30-86ec-3662d9371b7f.png)

</details>

# おわりに

2026/11/23 のオフライン会場では、さ10 で同じ電子版を頒布します。もし興味があればお立ち寄りいただけると嬉しいです！

ハンズオンの手順とコードは、本とは別に、GitHub の [AWS-Deep-Cuts](https://github.com/AWS-Deep-Cuts) のリポジトリでも公開しています。

なお、AWS Deep Cuts は Amazon Web Services の公式資料ではありません。サービスの仕様、料金、提供リージョンは変わることがあるため、ハンズオンを実施する前に公式ドキュメントで最新の情報を確かめてください。

それでは、よい AWS ライフを！

