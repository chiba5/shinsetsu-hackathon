# shinsetsu-hackathon — すれ違い通信「変な文章生成マシーン」

Scrapbox: https://scrapbox.io/shinsetsu-hackathon/

M5Stack AtomS3R / Tab5 の BLE ですれ違うと 5W1H の断片を受け取り、
揃うと変な文章ができる。元ネタは「いつどこでだれがなにをしたゲーム」
（源流は 19 世紀の Consequences → 1925 年の Exquisite Corpse）。

## ここにあるもの

| ファイル | 役割 |
|---|---|
| `words.json` | 語句表。スロット 6 つ × 20 語。**チームで自由に増やしてよい** |
| `sim.py` | BLE 無しで交換ルールを PC 上で試すシミュレータ |
| `imagegen.py` | 完成文から画像を 1 枚作る（発展目標）。既定は API を呼ばない dry |
| `docs/origin.html` | 発表用: 元ネタの系譜を 1 枚に |

## 動かす

```bash
python sim.py                      # 対話モード（meet A B / show A / auto 20）
python sim.py --auto 30 --seed 1   # 一気に 30 回すれ違わせて結果を見る
python imagegen.py "暇だったので　真夜中に　パリで　従順な犬が　全力で　ラーメンを食べた"
```

## 交換ルール（ファームに移すときの正）

- 各デバイスは **自分の一式**（全スロットに 1 語ずつ、起動時にランダム）と
  **組み立て中の文章**（最初は空）を持つ
- すれ違ったら、**受け取る側**が「自分の組み立て中の文章に無いスロット」を
  1 つランダムに選び、相手の一式からその語を受け取る。両方向に同時に行う
- 全スロットが埋まったら完成。以降は受け取らない
- Scrapbox の「B が A の足りない部分を選んで渡す」と結果は同じ。
  受け取る側が選ぶ形にすると **名札（advertise）だけで成立し、接続が要らない**

### BLE に載せるとき

- 名札（manufacturer data）に「自分の一式」を **語句番号 6 バイト** で入れる
  （語句表はファームに焼く。文字列は載せない。26 バイト枠に収まる）
- 受け取る側はスキャンで名札を読み、RSSI がしきい値以上なら上のルールで 1 語取る
- 同じ相手からは 1 回だけ（MAC で重複排除）。完成後は無視

## 広場サーバ（文章 → 画像 → 表示）

```
AtomS3R ──POST /submit──▶ PC: server.py（OpenAI で画像生成）◀──GET /latest.json, /image/N.jpg── Tab5
                                    └── ブラウザ /gallery（プロジェクタ・保険）
```

```bash
python server.py --dry     # API を呼ばずに動作確認（Pillow があれば文字入りの代替画像）
python server.py           # 本番。OPENAI_API_KEY を環境変数に入れておく
python test_server.py      # 口を一通り叩く自動テスト（dry）
```

### 接点（ここだけ合わせれば合体できる）

| 口 | 使う側 | 内容 |
|---|---|---|
| `POST /submit` | AtomS3R | `{"device":"A","sentence":"暇だったので　真夜中に　…"}` または `{"device":"A","words":["…",…]}`。即 `{"id":12,"status":"queued"}` |
| `GET /latest.json` | Tab5 / ブラウザ | `{"latest_id":12,"items":[{id,device,sentence,status,image},…]}`（新しい順 20 件）|
| `GET /image/12.jpg` | Tab5 / ブラウザ | 生成画像 JPEG 1024×1024（100〜200 KB）|
| `GET /` | ブラウザ | 広場ページ。3 秒ごと自動更新 |

- AtomS3R 側のコードは `firmware/atom_post.ino`（`postSentence()` を完成時に 1 回呼ぶだけ）
- 同じデバイスから同じ文が 60 秒以内に来たら同じ id を返す（再送しても二重生成しない）
- 生成は 10〜30 秒。`status` が `queued → working → done`（失敗は `error`）

### 当日の合体手順

1. スマホのテザリングを ON。PC・AtomS3R・Tab5 を全部それにつなぐ
2. PC で `python server.py`。`ipconfig` で IPv4 アドレスを確認（例 `192.168.43.12`）
3. スマホのブラウザで `http://<IP>:8000/` が開くか確認（開かなければ Windows ファイアウォールで Python を許可）
4. `firmware/atom_post.ino` の SSID / PASS / SERVER を書き換えてファームに混ぜる
5. AtomS3R で文章を完成 → サーバの画面に `[queue] #1 A: …` が出る → 数十秒後 `[done]`
6. ブラウザ / Tab5 に出る

## 語句表の書き方

`words.json` の `slots` は並び順のまま文章になる。`key` は固定（ファーム側と一致させる）、
`label` と `words` は自由。1 語 20 字以内。組み合わせたときにギャップが出る語
（時代・場所・固有名詞のズレ）が面白い。
