[← 目次に戻る](../../README.md)

# 14. Python / Java / Laravel / Go 概念対応表

> ⚠️ **この表の読み方**
> これは**あくまで概念の対応表**です。
> **「完全に同じ仕組みが存在する」という意味ではありません。**
> 同じ行に並んでいても、実装・保証・使い方は各フレームワークで大きく異なります。
> 「だいたいこの位置にあるもの」という**地図**として使ってください。

| 概念 | Python | Java | Laravel | Go |
|---|---|---|---|---|
| **Job** | taskなど | Job / Taskなど | Job | struct / taskなど |
| **Queue** | Celeryなど | 各種Queue | Laravel Queue | 各種Queue / 自作 |
| **Worker** | Celery Workerなど | Worker / Batchなど | `queue:work` | goroutine / Worker |
| **Mutex** | Lock | synchronized / Lock | PHPの実行環境による | `sync.Mutex` |
| **DB Transaction** | DB API | `@Transactional`など | `DB::transaction` | `database/sql` |
| **DB Queue** | 可能 | 可能 | 可能 | 可能 |

## 補足：それぞれの事情

**Python**
* Celery が事実上の定番。BrokerとしてRedis / RabbitMQを使う。
* `threading.Lock` はスレッド間、`multiprocessing.Lock` はプロセス間。GIL の影響でCPU処理の並列性には制約がある。

**Java**
* 「Queue」と言ったとき、**JVM内のデータ構造**（`BlockingQueue`）を指す場合と、**外部のメッセージング**（JMS / Kafka）を指す場合があり、文脈で意味が変わる。
* `synchronized` / `ReentrantLock` は **同一JVM内**のみ有効。
* Spring Batch など「Batch」を名前に持つ仕組みが充実している（→ 第15章）。

**Laravel / PHP**
* `Job` クラス、`dispatch()`、`queue:work` と、**フレームワークが一式を用意**しているのが特徴。
* ドライバを設定で切り替えられる（`sync` / `database` / `redis` / `sqs` …）。
* PHPは基本的に**リクエストごとにプロセスが独立**するため、言語レベルのMutexという発想が薄い。プロセス間の排他には `Cache::lock()`（アトミックロック）を使う。

**Go**
* 言語標準に `goroutine` と `channel` があり、**プロセス内のQueue的なもの（channel）を非常に簡単に作れる**のが特徴。
* ただし channel は**プロセスが死ねば中身も消える**。永続化が必要なJobには、DBや外部Queueが必要。
* `sync.Mutex` は同一プロセス内のみ（→ 第9章）。

> ⚠️ **注意（よくある誤解）**
> * 「Go の channel = Queue」→ **半分正しく、半分違います**。channelはプロセス内の受け渡し機構であり、**永続性も再配信もありません**。
> * 「Laravel Queue は Celery と同じ」→ **概念は近いが、保証も設定も別物**です。
> * 「Java の BlockingQueue はメッセージキュー」→ **JVM内のデータ構造**であり、RabbitMQ等とはレイヤーが違います。

> 💡 **ポイント**
> **重要なのは「どの言語にも Producer / Queue / Worker という構図がある」ということ**です。
> 名前と実装は違っても、考えるべきこと（重複・排他・トランザクション・冪等性）は**全部同じ**です。

---

| 前 | 目次 | 次 |
|:--|:--:|--:|
| [← 13. 冪等性](../part5-reliability/13-idempotency.md) | [📖 目次](../../README.md) | [15. Batchとの違い →](15-batch.md) |
