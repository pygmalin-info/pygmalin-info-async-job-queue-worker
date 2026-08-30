[← 目次に戻る](../../README.md)

# 1. まず全体像

非同期処理の登場人物は、たった5つです。

<img width="662" height="173" alt="スクリーンショット 2026-08-30 20 21 03" src="https://github.com/user-attachments/assets/66ecd449-451f-43a5-a78f-ec02e94dedfd" />

<img width="428" height="240" alt="スクリーンショット 2026-08-30 20 22 06" src="https://github.com/user-attachments/assets/612769c4-cbf7-4ad1-9cf6-63b01a208187" />


## ⚠️ 注意：Consumer と Worker は完全な同義ではない

初学者向けの説明では「Consumer ≒ Worker」で構いませんが、厳密には**役割の切り口が違います**。

* **Consumer** … 「Queue/Brokerから**メッセージを受け取る**」という**通信上の役割**を指す言葉。Brokerから見た相手側。
* **Worker** … 「受け取った仕事を**実際に処理する**」という**実行主体（プロセス / スレッド / goroutine）** を指す言葉。


<img width="472" height="345" alt="スクリーンショット 2026-08-30 20 23 11" src="https://github.com/user-attachments/assets/36e94d94-fadc-46bd-8010-8d3c0dddec7f" />

---

| 前 | 目次 | 次 |
|:--|:--:|--:|
| — | [📖 目次](../../README.md) | [2. Jobとは何か →](02-job.md) |
