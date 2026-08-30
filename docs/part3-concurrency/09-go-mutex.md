[← 目次に戻る](../../README.md)

# 9. Goの排他制御（sync.Mutex とその限界）

Goでは、同一プロセス内の排他に `sync.Mutex` を使います。

```go
var mu sync.Mutex

func process() {
    mu.Lock()
    defer mu.Unlock()

    // 排他したい処理
}
```

* `mu.Lock()` … 「今から入ります。他の人は待ってください」
* `defer mu.Unlock()` … 関数を抜けるときに必ず解放する（`defer` を使うのがGoの定石。パニックしても解放される）

```text
goroutine 1 ── Lock() ──▶ [ 処理中 ] ──▶ Unlock()
goroutine 2 ── Lock() ──▶ ...待機... ──▶ [ 処理中 ] ──▶ Unlock()
```

## ⚠️ 最重要の注意：sync.Mutex の守備範囲

> **`sync.Mutex` は同一プロセス内の排他に使うものであり、複数サーバー・複数プロセス間の排他をそのまま解決するものではありません。**

```text
sync.Mutex が守れる範囲
┌──────────────────────────────┐
│ Goプロセス（1つ）             │
│   goroutine A ─┐             │
│   goroutine B ─┼─ Mutexで排他 │  ✅
│   goroutine C ─┘             │
└──────────────────────────────┘

守れない範囲
┌────────────┐        ┌────────────┐
│ Goプロセス1 │  ？？  │ Goプロセス2 │  ❌ Mutexは無関係
└────────────┘        └────────────┘
  サーバー1              サーバー2
```

Goのプロセスを2つ起動した時点で、**それぞれが別々のMutexを持つ**ため、互いを一切ブロックしません。

## では、プロセスをまたぐ排他はどうするか

| 手段 | 具体例 | 向いている場面 |
|---|---|---|
| **DBロック** | `SELECT ... FOR UPDATE SKIP LOCKED` | すでにDBを使っている / Job Queueの取得 |
| **分散ロック** | Redis の `SET lock:job:123 <token> NX EX 30` | DBを経由したくない / 短時間の排他 |
| **Queue側の仕組み** | SQSのVisibility Timeout、RabbitMQのAck | Queue製品を使っている場合はこれが最も簡単 |

Redis分散ロックの最小イメージ：

```go
// NX = キーが存在しない時だけセット, EX = 30秒で自動失効
ok, err := rdb.SetNX(ctx, "lock:job:123", token, 30*time.Second).Result()
if !ok {
    return // 誰かが処理中なのでスキップ
}
defer rdb.Del(ctx, "lock:job:123") // ※実運用ではtoken照合してから削除する
```

> ⚠️ **注意**
> 分散ロックには**必ず有効期限（TTL）を付けます**。
> Workerがクラッシュしたときにロックが永久に残ると、そのJobは二度と処理されなくなります。
> 一方でTTLが短すぎると、処理中に期限切れになって別Workerが入ってきます。
> **「分散ロックは思ったより難しい」** ——だからこそ、可能ならDBロックやQueueの仕組みに任せるほうが安全です。

> 💡 **他言語での対応**
> * Java … `synchronized` / `ReentrantLock` → **同一JVM内**のみ
> * Python … `threading.Lock` → **同一プロセス内**のみ（さらにマルチプロセスでは `multiprocessing.Lock`）
> * PHP/Laravel … リクエストごとにプロセスが独立するため、そもそもプロセス内Mutexの出番が少ない。`Cache::lock()`（アトミックロック）を使う
>
> **どの言語でも「プロセス内ロックは、プロセスをまたげない」** という点は共通です。

---

| 前 | 目次 | 次 |
|:--|:--:|--:|
| [← 8. 排他制御](08-exclusive-control.md) | [📖 目次](../../README.md) | [10. トランザクション →](../part4-database/10-transaction.md) |
