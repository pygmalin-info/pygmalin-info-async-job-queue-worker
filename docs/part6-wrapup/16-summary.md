[← 目次に戻る](../../README.md)

# 16. 最終まとめ

## 全体像

```text
                    非同期処理
                        │
                        ↓
                  ┌──────────┐
                  │ Producer │
                  │ 仕事を作る│
                  └────┬─────┘
                       ↓
                ┌──────────────┐
                │ Queue/Broker │
                │ 仕事を預かる │
                └──────┬───────┘
                       ↓
              ┌────────────────┐
              │ Consumer/Worker│
              │ 仕事を実行する │
              └───────┬────────┘
                      ↓
                    Job
                      │
                      ↓
             ┌─────────────────┐
             │ DB / 外部API等  │
             └─────────────────┘

        複数Workerで同時実行
                  ↓
              排他制御
                  ↓
             DBロック等

        DB操作を安全にまとめる
                  ↓
             Transaction
                  ↓
        分離レベル / COMMIT
                  ↓
             データ整合性

        それでも再実行される可能性
                  ↓
               冪等性
```

## Mermaidで見る全体像

```mermaid
flowchart TD
    ASYNC["非同期処理"] --> P["Producer<br/>仕事を作る"]
    P --> Q["Queue / Broker<br/>仕事を預かる"]
    Q --> C["Consumer / Worker<br/>仕事を実行する"]
    C --> J["Job"]
    J --> R["DB / 外部API 等"]

    C -.->|"Workerを増やすと"| MULTI["複数Workerで同時実行"]
    MULTI --> EX["排他制御<br/>同時に触らせない"]
    EX --> LOCK["DBロック / 分散ロック<br/>FOR UPDATE SKIP LOCKED"]

    R -.->|"DB操作を安全にまとめる"| TX["Transaction<br/>BEGIN → COMMIT / ROLLBACK"]
    TX --> ISO["分離レベル<br/>お互いをどう見せるか"]
    ISO --> CONS["データ整合性"]

    LOCK -.->|"それでも再実行されうる"| IDEM["冪等性<br/>2回実行されても壊れない"]
    CONS -.-> IDEM
```

## 用語の関係を1枚で

```mermaid
flowchart TD
    subgraph L1["① 仕事の流れ"]
        A1["Producer"] --> A2["Queue"] --> A3["Worker / Consumer"] --> A4["Job"]
    end
    subgraph L2["② 同時実行の問題と対策"]
        B1["同時実行制御"] --> B2["分離レベル<br/>どう見えるか"]
        B1 --> B3["ロック<br/>触らせない"]
        B3 --> B4["排他制御の実装"]
    end
    subgraph L3["③ 壊れないための保険"]
        C1["Transaction<br/>まとまりで扱う"]
        C2["冪等性<br/>再実行に耐える"]
    end
    A3 --> B1
    A4 --> C1
    A4 --> C2
```


> この資料は「用語を覚える」ためではなく、
> **「Webアプリから仕事が発生して、Queueに入り、Workerが処理する。複数Workerが動いたらどうなる？DBはどう守る？失敗したらどうする？」**
> という**1本のストーリー**として理解するためのものです。
>
> 使っている言語が Python でも Java でも Laravel でも Go でも、
> **考えるべきことは同じ**です。

---

| 前 | 目次 | 次 |
|:--|:--:|--:|
| [← 15. Batchとの違い](15-batch.md) | [📖 目次](../../README.md) | [付録：用語早見表 →](../../appendix/glossary.md) |
