[← 目次に戻る](../../README.md)

# 4. Producer / Consumer

> ⚠️ **重要な誤解の訂正**
> Producer / Consumer は **Java固有の用語ではありません**。
> Python・Java・Laravel・Go・Node.js など、**言語を問わず使われる一般的な設計上の概念（デザインパターン）** です。
> 「Producer-Consumerパターン」として、OS・分散システム・メッセージング全般で共通して登場します。

## Producer（生産者）

> 仕事やイベントを作り、Queue / Brokerへ送る側

WebアプリのコントローラーやAPIハンドラが Producer になることが多いです。

## Consumer（消費者）

> Queue / Brokerからメッセージを受け取り、処理する側

常駐しているWorkerプロセスが Consumer になることが多いです。

```text
┌────────────┐
│ Producer   │
│ 仕事を作る │
└─────┬──────┘
      │
      │ Publish / Send
      ↓
┌────────────────┐
│ Queue / Broker │
│ 仕事を預かる   │
└──────┬─────────┘
       │
       │ Consume / Receive
       ↓
┌────────────┐
│ Consumer   │
│ 仕事を受ける│
└─────┬──────┘
      ↓
   Jobを実行
```

## 💡 用語は製品によって呼び方が違うだけ

**同じことをしているのに、名前が違う**——ここが初学者の混乱ポイントです。

| やること | RabbitMQ | Kafka | SQS | Redis | DB |
|---|---|---|---|---|---|
| **入れる** | Publish | Produce / Publish | SendMessage | RPUSH / LPUSH | INSERT |
| **取り出す** | Consume / Basic.Get | Poll / Consume | ReceiveMessage | BRPOP / LPOP | SELECT |
| **完了を伝える** | Ack | Commit Offset | DeleteMessage | （削除済み） | UPDATE status |

> 💡 **ポイント**
> 「Publish」「Send」「Enqueue」「Push」は、だいたい**同じこと**を指しています。
> 「Consume」「Receive」「Poll」「Dequeue」「Pop」も同様です。
> **言葉の違いに惑わされず、"入れる / 取り出す / 完了を伝える" の3つで捉える**と、どの製品も同じ形に見えてきます。

## Pull型 と Push型

Consumerがメッセージを受け取る方法にも2種類あります。

```text
Pull型（取りに行く）             Push型（届けてもらう）
Consumer → 「ある？」→ Queue     Queue → 「はいどうぞ」→ Consumer
（SQS, Kafka）                   （RabbitMQ の basic.consume）
```

どちらでも「Producer → Queue → Consumer」という構図は変わりません。

---

| 前 | 目次 | 次 |
|:--|:--:|--:|
| [← 3. Queueとは何か](03-queue.md) | [📖 目次](../../README.md) | [5. RabbitMQ / Kafka / SQS →](../part2-queue-systems/05-brokers.md) |
