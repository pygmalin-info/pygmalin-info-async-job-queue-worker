# 非同期処理 / Job・Queue・Worker / 排他制御 入門

> **対象**: Python / Java / Laravel(PHP) / Go のいずれかを学習・実務で触っている初級〜中級エンジニア
> **ゴール**: 「Job・Queue・Worker・Batch・Producer・Consumer・トランザクション・ロック・冪等性」の関係を、**言語やフレームワークを超えた共通概念**として説明できるようになること

この資料は「Celeryの使い方」「Laravel Queueの設定方法」といった**特定FWのマニュアルではありません**。
どの言語でも共通して出てくる**考え方の地図**です。

---

## 📖 この資料は「1本のストーリー」です

用語集ではありません。全16章が、次の1つの物語として並んでいます。

```text
Webアプリで「仕事」が発生した
        ↓
すぐ実行すると遅いので、あとで実行することにした   … Job / 非同期処理     → Part 1
        ↓
その仕事をどこかに預けておく                       … Queue / Broker      → Part 1・2
        ↓
別のプロセスが取り出して実行する                   … Worker / Consumer   → Part 1
        ↓
速くしたいのでWorkerを増やした
        ↓
「同じ仕事を2つのWorkerが取ったらどうなる？」      … 重複実行問題        → Part 3
        ↓
同時に触らせない仕組みが要る                       … 排他制御 / ロック    → Part 3
        ↓
DBの更新も途中で壊れては困る                       … トランザクション     → Part 4
        ↓
同時に動くトランザクションはお互いをどう見る？     … 分離レベル          → Part 4
        ↓
それでも2回実行されることはある                     … 冪等性              → Part 5
```

**途中の章から読むと「なぜそれが必要なのか」が分からなくなります。** Part 1 から順に読んでください。

---

## 📚 目次

### [Part 1. 基礎概念](docs/part1-basics/)

登場人物と、その関係を押さえます。

| 章 | タイトル | ひとことで |
|---|---|---|
| 1 | [まず全体像](docs/part1-basics/01-big-picture.md) | Producer → Queue → Worker → Job の関係 |
| 2 | [Jobとは何か](docs/part1-basics/02-job.md) | 後で実行したい1つの仕事 |
| 3 | [Queueとは何か](docs/part1-basics/03-queue.md) | 仕事を待たせておく場所／同期と非同期 |
| 4 | [Producer / Consumer](docs/part1-basics/04-producer-consumer.md) | Java固有ではない一般的な設計概念 |

### [Part 2. Queueの実体](docs/part2-queue-systems/)

「Queue」と呼ばれるものの中身は、実は同じではありません。

| 章 | タイトル | ひとことで |
|---|---|---|
| 5 | [RabbitMQ / Kafka / SQS](docs/part2-queue-systems/05-brokers.md) | 「全部同じQueue」ではない |
| 6 | [DBをQueueとして使う](docs/part2-queue-systems/06-db-as-queue.md) | 専用製品だけがQueueではない |

### [Part 3. 同時実行と排他制御](docs/part3-concurrency/)

**この資料の山場です。**

| 章 | タイトル | ひとことで |
|---|---|---|
| 7 | [複数Workerと重複実行問題](docs/part3-concurrency/07-duplicate-execution.md) | 同じJobを2回処理してしまう |
| 8 | [排他制御](docs/part3-concurrency/08-exclusive-control.md) | 「排他制御 = ロック」ではない |
| 9 | [Goの排他制御](docs/part3-concurrency/09-go-mutex.md) | `sync.Mutex` はプロセスをまたげない |

### [Part 4. DBを守る仕組み](docs/part4-database/)

混同しやすい3つ（トランザクション・分離レベル・ロック）を切り分けます。

| 章 | タイトル | ひとことで |
|---|---|---|
| 10 | [トランザクション](docs/part4-database/10-transaction.md) | 複数のDB操作をひとまとまりに |
| 11 | [トランザクション分離レベル](docs/part4-database/11-isolation-level.md) | 「コミットレベル」は正式用語ではない |
| 12 | [分離レベルとロックの違い](docs/part4-database/12-isolation-vs-lock.md) | 分離レベルを上げても重複は防げない |

### [Part 5. それでも壊れないために](docs/part5-reliability/)

| 章 | タイトル | ひとことで |
|---|---|---|
| 13 | [冪等性](docs/part5-reliability/13-idempotency.md) | 2回実行されても結果が壊れない設計 |

### [Part 6. 整理とまとめ](docs/part6-wrapup/)

| 章 | タイトル | ひとことで |
|---|---|---|
| 14 | [Python / Java / Laravel / Go 概念対応表](docs/part6-wrapup/14-language-comparison.md) | 名前は違っても構図は同じ |
| 15 | [Batchとの違い](docs/part6-wrapup/15-batch.md) | 文脈で意味が変わる言葉 |
| 16 | [最終まとめ](docs/part6-wrapup/16-summary.md) | 全体像を1枚の図に |

### 付録

| | タイトル | 使い方 |
|---|---|---|
| A | [理解度チェック](appendix/checklist.md) | 読了後の自己確認（一問一答14問＋設計時の思考順序） |
| B | [用語早見表](appendix/glossary.md) | 読んでいる途中に引く辞書 |

---

## 🎯 学習の進め方

**個人で読む場合**

1. Part 1〜2 を通読して、登場人物と全体像をつかむ
2. Part 3 は**手を止めてじっくり**読む（ここが理解の分かれ目）
3. Part 4〜5 まで読み切る
4. [付録A. 理解度チェック](appendix/checklist.md) で答えられない項目を見つけ、該当章に戻る

**研修で使う場合の目安**

| 回 | 範囲 | ねらい |
|---|---|---|
| 1回目 | Part 1・2 | 「非同期って何のため？」が説明できる |
| 2回目 | Part 3 | 重複実行が**なぜ**起きるかを図で説明できる |
| 3回目 | Part 4 | トランザクション・分離レベル・ロックを区別できる |
| 4回目 | Part 5・6＋付録A | 自分の使う言語に引きつけて説明できる |

---

## 📂 ディレクトリ構成

```text
async-job-queue-worker/
├── README.md                      ← いまここ（目次）
├── docs/
│   ├── part1-basics/              基礎概念
│   │   ├── 01-big-picture.md
│   │   ├── 02-job.md
│   │   ├── 03-queue.md
│   │   └── 04-producer-consumer.md
│   ├── part2-queue-systems/       Queueの実体
│   │   ├── 05-brokers.md
│   │   └── 06-db-as-queue.md
│   ├── part3-concurrency/         同時実行と排他制御
│   │   ├── 07-duplicate-execution.md
│   │   ├── 08-exclusive-control.md
│   │   └── 09-go-mutex.md
│   ├── part4-database/            DBを守る仕組み
│   │   ├── 10-transaction.md
│   │   ├── 11-isolation-level.md
│   │   └── 12-isolation-vs-lock.md
│   ├── part5-reliability/         冪等性
│   │   └── 13-idempotency.md
│   └── part6-wrapup/              整理とまとめ
│       ├── 14-language-comparison.md
│       ├── 15-batch.md
│       └── 16-summary.md
└── appendix/
    ├── checklist.md               理解度チェック
    └── glossary.md                用語早見表
```

各 Part フォルダには `README.md`（そのPartの目次）があります。
各章の末尾には**前の章 / 目次 / 次の章**へのリンクがあるので、順に読み進められます。

---

## ✍️ 表記のルール

| 記号 | 意味 |
|---|---|
| 💡 **ポイント** | 押さえてほしい要点 |
| ⚠️ **注意** | 誤解しやすい・事故につながりやすい点 |
| ✅ / ❌ | 正しい理解 / よくある誤解 |

* 図は **Mermaid**（GitHub上で自動描画）と **ASCII図**を併用しています
* コード例は「イメージがつかめる最小限」に絞っています。動作する完全なコードではありません
* 各用語は「**厳密な違い**」と「**初学者向けのざっくりした説明**」を両方示しています
