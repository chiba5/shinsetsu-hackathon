# shinsetsu-hackathon — すれ違い通信「変な文章生成マシーン」

## 状況（2026-09-05）

- ハッカソン当日中。チーム 3 人（千葉・Akiba・Miyamoto）。共有メモは https://scrapbox.io/shinsetsu-hackathon/
- **Scrapbox など共有先への書き込みは、本文を提示して OK をもらってから**（本人指示）
- リポジトリ: https://github.com/chiba5/shinsetsu-hackathon （公開）
- 元ネタ: いつどこでだれがなにをしたゲーム（源流 Consequences / Exquisite Corpse）。`docs/origin.html`

## 分担

- 千葉（本人）: **PC 側の広場サーバ + 画像生成**。低レイヤーの知識は無いので BLE の話は噛み砕く
- 他 2 人: AtomS3R の BLE ファーム（文字情報の交換まで動いている）

## 仕組み

各デバイス（M5Stack AtomS3R）が 5W1H の一式を持ち、すれ違うと相手の一式から自分に無いスロットを 1 つ受け取る。
6 スロット揃ったら完成 → PC のサーバへ POST → OpenAI で画像 → Tab5 / ブラウザの「広場」に表示。
詳細と接点の仕様は `README.md`。

## PC 側の状態

- `server.py` **完成・本番動作確認済み**（gpt-image-1, quality low, JPEG, 10〜13 秒/枚）
- 動かす: `python server.py`（本番）/ `--dry`（課金なし）。手投げは `python submit.py A --random`
- この PC の Wi-Fi IP は `ipconfig` で確認（Wi-Fi を変えると変わる）。サーバは `0.0.0.0:8000`
- `OPENAI_API_KEY` は環境変数に既にある。**実際の API 呼び出し（課金）は本人に確認してから**

## 未解決（次にやること）

1. チームに確認: **AtomS3R は Wi-Fi を使っているか** / **Tab5 は誰が触るか**
2. Wi-Fi あり → `firmware/atom_post.ino` を組み込んでもらう。無し → 「PC が BLE で直接拾う」スクリプト（bleak）を書く
3. `firmware/tab5_hiroba.ino` は**実機未検証**。Tab5 の BLE は Arduino で落ちる既知問題があるので Wi-Fi のみ使う
4. Tab5 が動かなければ広場はブラウザ + プロジェクタで確定

## 環境の注意

- 本人のターミナルは **Windows PowerShell 5.1**。`&&` 不可、`curl` は別物。コマンドは 1 行ずつ、投稿は `submit.py`
- デバイス: AtomS3R = ESP32-S3、電池なし（モバイルバッテリー要）、カメラなし。Tab5 = ESP32-P4 + C6（Wi-Fi は C6 経由）
