[← 目次に戻る](../README.md)

# 付録A. 理解度チェック

全16章を読み終えたら、次の質問に**自分の言葉で**答えられるか確認してください。
答えに詰まった項目は、右端の「復習」列から該当章に戻れます。

## 一問一答チェックリスト

| # | 質問 | 答えのポイント | 復習 |
|---|---|---|---|
| 1 | Job / Queue / Worker の関係は？ | Producerが**Job**を作り**Queue**に預け、**Worker**が取り出して実行する | [1章](../docs/part1-basics/01-big-picture.md) |
| 2 | Consumer と Worker は同じ？ | 厳密には違う。Consumer=受け取る役、Worker=実行する役 | [1章](../docs/part1-basics/01-big-picture.md) |
| 3 | Producer / Consumer はJava固有？ | いいえ。**言語を問わない一般的な設計概念** | [4章](../docs/part1-basics/04-producer-consumer.md) |
| 4 | RabbitMQ・Kafka・SQSは同じもの？ | いいえ。Broker / ストリーミング基盤 / マネージドQueue と役割が違う | [5章](../docs/part2-queue-systems/05-brokers.md) |
| 5 | Queueは専用製品が必要？ | いいえ。**DBのテーブルをQueueとして使う設計も一般的** | [6章](../docs/part2-queue-systems/06-db-as-queue.md) |
| 6 | 複数Workerだと何が起きる？ | 同じJobを取得して**重複実行**する可能性がある | [7章](../docs/part3-concurrency/07-duplicate-execution.md) |
| 7 | 排他制御 = ロック？ | いいえ。**排他制御という考え方の中に、ロック等の実装がある** | [8章](../docs/part3-concurrency/08-exclusive-control.md) |
| 8 | `sync.Mutex` でサーバー2台の排他はできる？ | **できない**。同一プロセス内のみ。DBロックや分散ロックが必要 | [9章](../docs/part3-concurrency/09-go-mutex.md) |
| 9 | トランザクションとは？ | 複数のDB操作を**ひとまとまり**として扱う仕組み。COMMITで確定、ROLLBACKで取消 | [10章](../docs/part4-database/10-transaction.md) |
| 10 | 「コミットレベル」という用語は？ | 正式には**トランザクション分離レベル** | [11章](../docs/part4-database/11-isolation-level.md) |
| 11 | 分離レベルを上げればJob重複は防げる？ | **防げるとは限らない**。ロック（`FOR UPDATE SKIP LOCKED`等）と組み合わせる | [12章](../docs/part4-database/12-isolation-vs-lock.md) |
| 12 | 排他制御があれば十分？ | いいえ。**再配信・リトライで2回実行されうる** → 冪等性が必要 | [13章](../docs/part5-reliability/13-idempotency.md) |
| 13 | 排他制御と冪等性の違いは？ | 排他=**同時に実行させない** / 冪等=**複数回実行されても問題ない** | [13章](../docs/part5-reliability/13-idempotency.md) |
| 14 | Batch は Queue の上位概念？ | **断定できない**。Batchは文脈で意味が変わる言葉 | [15章](../docs/part6-wrapup/15-batch.md) |

> 💡 **目安**
> 14問中 **10問以上**を自分の言葉で説明できれば、実務で設計の議論に参加できるレベルです。
> 特に **6・7・8・11・13** は事故に直結する項目なので、確実に答えられるようにしてください。

## 設計するときの思考順序

```text
1. その処理、ユーザーを待たせる必要ある？
        → ないなら Job にして Queue へ

2. Queue は何を使う？
        → まずDB Queueで足りないか検討 → 足りなければ専用製品

3. Worker を増やす予定はある？
        → あるなら重複取得対策が必須（FOR UPDATE SKIP LOCKED など）

4. Jobの中でDBを複数回更新する？
        → トランザクションで囲む（ただし外部API呼び出しは外へ）

5. その処理、2回実行されたら困る？
        → 困るなら冪等に設計する（処理済み記録 / 条件付きUPDATE）

6. 失敗したらどうする？
        → リトライ回数の上限、失敗Jobの退避先（Dead Letter Queue）を決める
```

---

| 前 | 目次 | 次 |
|:--|:--:|--:|
| [← 16. 最終まとめ](../docs/part6-wrapup/16-summary.md) | [📖 目次](../README.md) | [用語早見表 →](glossary.md) |
