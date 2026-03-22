# LUNA Lua ガイド

**対象ファームウェア**: AiBridgeMCP v3.9.1 LUNA / v5.9.1 LUNA Observatory
**ドキュメントリビジョン**: r1.8
**最終更新**: 2026-03-21
**作者**: Nishioka Sadahiko
**対象**: Claude などの AI と ESP32 を通じて外部機器を Lua スクリプトで制御したいユーザー

---

> **【重要事項】** 本ソフトウェアは MITライセンス（無償・商用利用可）に基づき、**バイナリ形式**で配布されます。ソースコードは非公開です。

---

## 改訂履歴

| Rev | 日付 | 内容 |
|-----|------|------|
| r1.0 | 2026-03-17 | 初版（Phase 0〜5 完了・全 MCP ツール実機確認済み） |
| r1.1 | 2026-03-18 | log.read / log.save / log.load 追加（Phase 6/6b）・スクリプト間データ受け渡し |
| r1.2 | 2026-03-18 | log モジュール「2つの役割」解説強化・log.load() 動作仕様明記・fs_read 確認方法追記 |
| r1.3 | 2026-03-18 | LUNA システム ブランド名確定・冒頭説明追加・ガイドバージョン管理導入 |
| r1.4 | 2026-03-18 | serial 通信2経路の解説追加・競合ルール・Claude 直接制御の方法追記 |
| r1.5 | 2026-03-18 | luna モジュール追加（v3.9.1）・スクリプトチェーン実行・セクション 2-5 新設 |
| r1.6 | 2026-03-20 | notify.set() メッセージ引数対応（バグ修正）・全コード例を引数付きに更新・ポーリング確認方法追記（Desktop / Claude Code） |
| r1.8 | 2026-03-21 | v5.9.1 対応：http モジュール追加（http.get / http.put）・AiBridge/Alpaca 連携解説・セクション 2-6 新設・既存 2-6〜2-9 を 2-7〜2-10 に繰り下げ |
| r1.9 | 2026-03-22 | 3-8 新設：MCP Resources リソーススクリプトの書き方（v5.9.1） |
| r1.7 | 2026-03-20 | lua_load → lua_save にツール名変更（全コード例・APIリファレンス更新）・目次内リンクをプレーンテキスト化 |

---

## 目次

- LUNA システムとは
- はじめに
- 第1章　クイックスタート
  - 1-1. 必要なもの
  - 1-2. 最初のスクリプトを実行する
  - 1-3. 基本的な流れ
  - 1-4. スクリプトを保存して再利用する
- 第2章　Lua API リファレンス
  - 2-1. serial モジュール — シリアル通信
  - 2-2. log モジュール — ログ記録・スクリプト間データ受け渡し
  - 2-3. hw モジュール — ハードウェア制御
  - 2-4. notify モジュール — 完了通知
  - 2-5. luna モジュール — スクリプトチェーン実行（v3.9.1）
  - 2-6. http モジュール — HTTP通信・Alpaca 連携（v5.9.1）
  - 2-7. 標準 Lua 5.1 ライブラリ
  - 2-8. 数値型・16進数・ビット演算（実機確認済み）
  - 2-9. 使用できない機能
  - 2-10. クイックリファレンス一覧
- 第3章　実用スクリプト集
  - 3-1. 機器の動作確認
  - 3-2. 定期測定（複数回）
  - 3-3. 条件分岐制御
  - 3-4. エラーハンドリング付き測定
  - 3-5. LED を使った動作インジケーター
  - 3-6. 複数コマンドの順次実行
  - 3-7. コルーチンによる疑似並行処理
  - 3-8. MCP Resources — リソーススクリプトの書き方（v5.9.1）
- 第4章　使い方のコツと注意事項
  - 4-1. Claude との連携の基本
  - 4-2. log.write の使い方のコツ
  - 4-3. タイムアウト値の目安
  - 4-4. よくあるトラブルと対処法
  - 4-5. スクリプト名のルール
  - 4-6. 同時実行について
- 第5章　Claude と一緒に使うインタラクティブプログラミング
  - 5-1. これは何か
  - 5-2. 基本的な進め方
  - 5-3. マニュアルを渡してスクリプトを作らせる
  - 5-4. うまく進めるためのコツ
  - 5-5. 従来のプログラミングとの違い
  - 5-6. LUNA システムでできることのまとめ
- 付録A　MCP ツール一覧（Claude が使うツール）
- 付録B　Lua 言語 最小リファレンス
- 付録C　Lua から Claude へのデータ受け渡し方法（実機確認済み）
- 付録D　実機サンプル：高橋製作所 Temma2 天体望遠鏡制御

---

> ⚠️ **【暫定公開版】**
> この資料は現在作成中です。内容は予告なく変更・追加される場合があります。
> 正式版公開までの参考資料としてご利用ください。

---

## LUNA システムとは

**LUNA**（ルナ）は、**ESP32 上で動く MCP サーバー + Lua スクリプトエンジン**の統合システムです。

```
┌─────────────────────────────────────────┐
│              LUNA システム               │
│                                         │
│   MCP サーバー        Lua エンジン       │
│   ───────────   +   ───────────────     │
│   Claude と通信       スクリプトを       │
│   ツールを提供        ESP32 上で実行     │
│                                         │
│              ESP32                      │
└─────────────────────────────────────────┘
        ↕ WiFi (MCP)          ↕ Serial
     Claude (AI)          外部機器・センサー
```

Claude は LUNA に「スクリプトを渡す」だけです。
あとは LUNA が**自律的に**機器を制御し、完了を通知します。

| 項目 | 内容 |
|------|------|
| 正式名称 | LUNA システム（AiBridgeMCP v3.9.1 LUNA Edition） |
| 動作環境 | ESP32（約500円〜のマイコン） |
| スクリプト言語 | Lua 5.1 |
| 通信プロトコル | MCP（Model Context Protocol）over HTTP |
| ライセンス | MIT（無償・商用利用可） |

---

## はじめに

LUNA では、ESP32 の中で **Lua スクリプト**を動かすことができます。

これまでの版では、Claude がコマンドを1つずつ直接送っていました。
LUNA では、Claude が書いたスクリプトを ESP32 に渡して**自律実行**させることができます。

```
【従来】Claude → コマンド送信 → 機器 → Claude → 次のコマンド…（往復が続く）

【LUNA】Claude → スクリプトを渡す → ESP32 が自律実行 → 完了を通知 → Claude が結果を確認
```

Claude はスクリプトの生成を代行してくれます。
Lua を知らなくても、やりたいことを日本語で伝えるだけで使えます。

---

## 第1章　クイックスタート

### 1-1. 必要なもの

- ESP32 （AiBridgeMCP v3.9.0 書き込み済み）
- Claude などの MCP 対応 AI クライアント
- 制御したい外部機器（RS-232C 接続）

### 1-2. 最初のスクリプトを実行する

Claude に以下のように伝えるだけです。

> 「LED を5回点滅させて」

Claude は次のようなスクリプトを生成して実行します。

```lua
for i = 1, 5 do
    hw.led(1)
    hw.delay(200)
    hw.led(0)
    hw.delay(200)
end
log.write("点滅完了")
notify.set("完了")
```

### 1-3. 基本的な流れ

```
① Claude にやりたいことを伝える
      ↓
② Claude が Lua スクリプトを生成
      ↓
③ lua_exec または lua_save + lua_run で実行
      ↓
④ lua_status で完了を確認
      ↓
⑤ lua_log で結果を確認
```

### 1-4. スクリプトを保存して再利用する

一度動いたスクリプトは名前をつけて保存できます。

```
Claude への指示：「このスクリプトを "volt_check" という名前で保存して」

次回以降：「volt_check を実行して」と伝えるだけ
```

保存したスクリプトは電源を切っても残ります。

---

## 第2章　Lua API リファレンス

### 2-1. serial モジュール ── シリアル通信

#### シリアル通信の2つの経路

LUNA システムでは、外部機器へのシリアル通信に **2つの独立した経路** があります。

```
【経路①】Claude が直接制御（MCP ツール）

  Claude
    ↓ serial_write / serial_read / serial_query（MCP ツール）
    ↓
  ESP32 C++ 層（executeSerialQuery 等）
    ↓ serialMutex で排他制御
  外部機器（Serial2）


【経路②】Lua スクリプトが制御（Lua バインディング）

  Claude
    ↓ lua_exec / lua_run
  Lua スクリプト
    ↓ serial.write / serial.read / serial.query（Lua API）
    ↓
  ESP32 C++ 層（executeSerialQuery 等）← 同じ関数を呼ぶ
    ↓ serialMutex で排他制御
  外部機器（Serial2）
```

**2つの経路は内部で同じ C++ 関数を共有し、`serialMutex` で保護されています。**

---

#### 使い分けの指針

| 用途 | 推奨経路 | 理由 |
|------|---------|------|
| 動作確認・デバッグ・1コマンドの確認 | **① Claude 直接** | 即座に試せる。Lua 不要 |
| マニュアルを見ながら手動で機器を操作 | **① Claude 直接** | 対話的に進められる |
| 繰り返し測定・定期実行 | **② Lua** | 自律動作・Claude 不要 |
| 複数ステップのプロトコル | **② Lua** | ステップ間の状態を保持できる |
| 電源オフ後も継続する処理 | **② Lua** + log.save | 状態を SPIFFS に永続化できる |

---

#### 競合について

`serialMutex` により **物理的なデータ破壊（バイト化け等）は防がれます**。
ただし、**論理的な競合**には注意が必要です。

```
悪い例：Lua スクリプト実行中に Claude が割り込む

  Lua スクリプト → "INIT\r"     → 機器
  Claude         → "STATUS?\r"  → 機器  ← 割り込み！
  機器           → "OK"         → どちらへの応答か不明
  Lua スクリプト → "MEAS?\r"    → 機器
  機器           → データ       → Lua は正しい応答を受け取れない
```

**ルール：Lua スクリプト実行中（`lua_status` が `running`）は `serial_query` を使わない。**

---

#### Lua を使わず Claude が直接シリアル制御する方法

Lua スクリプトを書かなくても、MCP ツールを使って Claude が直接外部機器を制御できます。
**プロトタイプ・動作確認・単発操作** に最適です。

**利用できる MCP ツール（Claude が直接呼ぶ）:**

| ツール | 動作 |
|--------|------|
| `serial_write` | コマンドを送信（応答を待たない） |
| `serial_read` | 受信バッファを読み取る |
| `serial_query` | 送信→受信を一体で行う（★推奨） |

**実際の使い方イメージ:**

```
ユーザー: 「機器に *IDN? を送って、機種名を教えて」

Claude:
  serial_query("*IDN?\r\n", 1000) を実行
  → "KEITHLEY INSTRUMENTS,MODEL 2110,..." を受信
  → 「Keithley 2110 デジタルマルチメータです」と回答
```

```
ユーザー: 「Temma2 の現在位置を教えて」

Claude:
  serial_query("Q\r", 2000) を実行
  → "0535173-052328" を受信
  → 「現在の向き: RA 5h35m17.3s / Dec -5°23'28"」と回答
```

**Lua との比較:**

| 項目 | Claude 直接（MCP） | Lua スクリプト |
|------|-----------------|--------------|
| 準備 | なし（すぐ使える） | スクリプト作成が必要 |
| 繰り返し | Claude が毎回呼ぶ | 自律ループが可能 |
| 複雑な処理 | Claude が判断しながら進める | スクリプトに書いて自律実行 |
| Claude 不在での動作 | 不可 | 可能（lua_run） |
| 向いている用途 | デバッグ・対話操作・探索 | 定期測定・自律制御 |

> 💡 **典型的なワークフロー**: まず Claude 直接で動作を確認 → 慣れたら Lua スクリプトに昇格させる。

---

#### `serial.query(data, timeout_ms)` ★推奨

データを送信し、応答を受け取ります。

| 引数 | 型 | 説明 |
|------|----|------|
| data | string | 送信する文字列（終端文字を含む） |
| timeout_ms | number | 受信待ちタイムアウト（ミリ秒） |

| 戻り値 | 型 | 説明 |
|--------|----|------|
| response | string | 受信した応答。タイムアウトの場合は空文字列 `""` |

```lua
-- 基本的な使い方
local resp = serial.query("*IDN?\r\n", 1000)
if resp ~= "" then
    log.write("応答: " .. resp)
else
    log.write("タイムアウト")
end

-- 数値として受け取る
local raw = serial.query(":MEAS:VOLT:DC?\r\n", 2000)
local val = tonumber(raw)
if val then
    log.write(string.format("電圧: %.4f V", val))
end
```

---

#### `serial.write(data)`

データを送信します。応答は待ちません。

| 引数 | 型 | 説明 |
|------|----|------|
| data | string | 送信する文字列 |

```lua
serial.write("START\r\n")
hw.delay(100)
local resp = serial.read()
```

---

#### `serial.read()`

受信バッファのデータを読み取ります。

| 戻り値 | 型 | 説明 |
|--------|----|------|
| data | string | 受信データ（バッファが空の場合は空文字列） |

> **注意**: タイミングに依存します。確実に応答を得たい場合は `serial.query()` を推奨します。

---

### 2-2. log モジュール ── ログ記録・スクリプト間データ受け渡し

#### log モジュールの2つの役割

log モジュールは LUNA システムの **情報ハブ**です。単なるデバッグ用ログではなく、2つの独立した用途を持ちます。

```
【役割①】スクリプト → Claude へのデータ送信

  Lua スクリプト
    log.write("測定値: 1.234")
         ↓  RAM上のリングバッファ（最新20行）
  Claude
    lua_log ツール → 全20行を一括取得
    → 結果の確認・次の指示に活用

【役割②】スクリプト → スクリプト へのデータ受け渡し（Claude 介在ゼロ）

  スクリプトA
    log.write("CAL=1.0023")
    log.save()  → SPIFFS /luna_log.txt に永続保存
         ↓  電源オフ・別セッションをまたいでも OK
  スクリプトB
    log.load()  → /luna_log.txt を読み込み
    log.read()  → "CAL=1.0023" を取得して処理に使う
```

| 用途 | 書く | 読む | 永続性 |
|------|------|------|--------|
| スクリプト → Claude | `log.write()` | `lua_log`（MCP ツール） | RAM のみ（揮発） |
| スクリプト → スクリプト（同セッション） | `log.write()` | `log.read()` | RAM のみ（揮発） |
| スクリプト → スクリプト（電源オフ跨ぎ） | `log.write()` + `log.save()` | `log.load()` + `log.read()` | SPIFFS（永続） |
| Claude が SPIFFS を直接確認 | `log.save()` | `fs_read("/luna_log.txt")` | SPIFFS（永続） |

> 💡 **実務上の価値**: `log.save()` / `log.load()` を使えば、校正スクリプト・測定スクリプト・解析スクリプトが Claude を介さず自律的に連携できます。定期実行スクリプトが前回の状態を引き継ぐことも可能です。

---

#### `log.write(message)`

ログバッファにメッセージを記録します。Claude は `lua_log` ツールでこの内容を取得します。

| 引数 | 型 | 説明 |
|------|----|------|
| message | string | 記録するメッセージ |

```lua
log.write("処理開始")
log.write(string.format("測定値: %.3f V", value))
log.write("エラー: 応答なし")
```

> **制限**: リングバッファ方式で **最新 20行** を保持します。
> 21行目以降は古い行から上書きされます。

---

#### `log.read([n])` ── ログ行の読み出し（Phase 6 / 6b）

リングバッファから1行返します。引数なしで最新行（後方互換）、インデックス指定で任意行にアクセスできます。

| 引数 | 型 | 説明 |
|------|----|------|
| n | number（省略可） | 省略/-1=最新、0=最古、1,2,...=古い方から、-2,-3,...=最新から遡る |

| 戻り値 | 型 | 説明 |
|--------|----|------|
| line | string / nil | 該当行。バッファ空・範囲外は `nil` |

```lua
log.read()      -- 最新行（引数なし・従来と同じ）
log.read(-1)    -- 最新行（同上）
log.read(0)     -- 最古行
log.read(1)     -- 古い方から2番目
log.read(-2)    -- 最新から2番目
log.read(99)    -- 範囲外 → nil
```

> ✅ **実機確認済み（2026-03-18）**
> ```
> -- log.write("LINE0") 〜 log.write("LINE3") の後
> read()=LINE3 | read(0)=LINE0 | read(1)=LINE1 | read(-1)=LINE3 | read(-2)=LINE2 | read(99)=nil
> ```

#### 応用例① 最新行だけ使う（最もシンプル）

受け渡したい値を**最後に `log.write()`** すると、`log.read()` 引数なしで取れます。

```lua
-- スクリプトA：受け渡す値を最後に書く
log.write("TS=" .. hw.millis())     -- 付加情報
log.write("CAL=1.0023")            -- ← 受け渡す値を最後に
log.save()
notify.set("完了")

-- スクリプトB：log.read() で最新行を取得（Claude 介在ゼロ）
log.load()
local cal = tonumber((log.read() or ""):match("CAL=(.+)")) or 1.0
local v = tonumber(serial.query(":MEAS?\r\n", 1000)) * cal
return string.format("%.4f", v)
```

#### 応用例② 全行スキャンで特定キーを探す

書き込み順序を気にせず、キー名で値を検索できます。

```lua
log.load()
local function find(key)
    for i = 0, 19 do
        local line = log.read(i)
        if not line then break end
        local v = line:match(key .. "=(.+)")
        if v then return v end
    end
    return nil
end

local cal   = tonumber(find("CAL"))   or 1.0
local offset = tonumber(find("OFFSET")) or 0.0
log.write(string.format("cal=%.4f offset=%.4f", cal, offset))
```

#### 応用例③ 最新2行から複数の値を取り出す

```lua
log.load()
local line1 = log.read(-1)   -- 最新
local line2 = log.read(-2)   -- その前
-- 例: "VOLT=1.234" と "TEMP=25.6" が保存されている場合
local volt = tonumber((line1 or ""):match("VOLT=(.+)"))
local temp = tonumber((line2 or ""):match("TEMP=(.+)"))
return string.format("V=%.3f T=%.1f", volt or 0, temp or 0)
```

---

#### `log.save()` ── SPIFFS への永続保存（Phase 6）

リングバッファの内容を SPIFFS の `/luna_log.txt` に書き出します。電源オフを跨いだ状態保存に使います。

| 戻り値 | 型 | 説明 |
|--------|----|------|
| result | boolean | 成功で `true`、失敗で `false` |

```lua
log.write("校正値=1.0023")
local ok = log.save()
if ok then
    log.write("保存完了")
end
```

> ⚠️ **フラッシュ寿命に注意**: `log.save()` はスクリプト**終了時に1回だけ**呼ぶ設計にしてください。
> ループ内で毎回呼ぶと SPIFFS の書き込み回数が急増し、フラッシュの寿命を縮めます。

---

#### `log.load()` ── SPIFFS からの状態復元（Phase 6）

`/luna_log.txt` の内容をリングバッファに読み込みます。前回のスクリプトが保存した状態を復元します。

| 戻り値 | 型 | 説明 |
|--------|----|------|
| result | boolean | 成功で `true`、ファイルなし/失敗で `false` |

> ⚠️ **重要**: `log.load()` は呼び出し時に**リングバッファを一度クリア**してからファイルを読み込みます。呼び出し前にバッファに書き込んでいた内容は失われます。必ずスクリプトの**先頭で呼ぶ**設計にしてください。

```lua
-- 正しいパターン：スクリプト先頭で load()
local ok = log.load()
if ok then
    local last = log.read()
    log.write("前回状態: " .. tostring(last))
else
    log.write("初回起動（保存データなし）")
end
-- この後に処理を書く

-- 誤ったパターン（やってはいけない）
log.write("処理開始")   -- この行は load() で消える
local ok = log.load()   -- ← バッファクリアされる
```

#### Claude から /luna_log.txt を直接確認する

`log.save()` で保存したファイルは Claude が `fs_read` ツールで直接読めます。スクリプトが自律実行した結果を後から確認するのに便利です。

```
Claude の操作:
  fs_read("/luna_log.txt")
  → スクリプトが保存した全行を取得
```

これにより「スクリプトが自律的に蓄積したデータを Claude があとで回収する」という非同期ワークフローが実現できます。

---

### 2-3. hw モジュール ── ハードウェア制御

#### `hw.delay(ms)`

指定した時間だけ処理を停止します。

| 引数 | 型 | 説明 |
|------|----|------|
| ms | number | 待機時間（ミリ秒） |

```lua
hw.delay(500)     -- 0.5秒待機
hw.delay(1000)    -- 1秒待機
hw.delay(60000)   -- 1分待機
```

> **重要な特徴**: `hw.delay()` は Lua タスクのみを停止します。
> ESP32 の HTTP サーバーはブロックされず、Claude からの操作は待機中も受け付けます。

---

#### `hw.millis()`

ESP32 起動からの経過時間をミリ秒で返します。

| 戻り値 | 型 | 説明 |
|--------|----|------|
| ms | number | 起動からの経過ミリ秒 |

```lua
local start = hw.millis()
-- 処理
local elapsed = hw.millis() - start
log.write(string.format("処理時間: %d ms", elapsed))
```

---

#### `hw.free_heap()`

現在の空きヒープメモリをバイト単位で返します。

| 戻り値 | 型 | 説明 |
|--------|----|------|
| bytes | number | 空きヒープサイズ（バイト） |

```lua
log.write("空きヒープ: " .. hw.free_heap() .. " bytes")

-- メモリ不足チェック
if hw.free_heap() < 10000 then
    log.write("警告: メモリ不足")
    notify.set("メモリ不足")
    return
end
```

---

#### `hw.led(state)`

ESP32 の内蔵 LED を制御します。

| 引数 | 型 | 説明 |
|------|----|------|
| state | number | `1` で点灯、`0` で消灯 |

```lua
hw.led(1)        -- 点灯
hw.delay(200)
hw.led(0)        -- 消灯
```

```lua
-- 点滅の例
for i = 1, 5 do
    hw.led(1)
    hw.delay(100)
    hw.led(0)
    hw.delay(100)
end
```

> ⚠️ **注意**: `hw.led(true)` / `hw.led(false)`（boolean）はエラーになります。
> 必ず数値 `0` または `1` を使用してください。

---

### 2-4. notify モジュール ── 完了通知

#### `notify.set(message)`

Claude に通知メッセージを送ります。メッセージ文字列を引数に渡すことができます。

| 引数 | 型 | 説明 |
|------|----|------|
| message | string | Claude に伝えるメッセージ（必須） |

```lua
notify.set("処理完了: 測定値 1.234V")
```

Claude 側は `check_notify` ツールでフラグとメッセージを受け取ります。

```json
{"flag": 1, "message": "処理完了: 測定値 1.234V", "uptime": 120, "heap": 235000}
```

フラグは `check_notify` を呼ぶと自動リセットされます（2回目以降は `flag: 0`）。

> **推奨**: 長時間動作するスクリプトの末尾に記述し、完了状態や測定値を含むメッセージを渡してください。

```lua
-- 典型的なスクリプト構造
log.write("処理開始")

-- メイン処理
local last_val = ""
for i = 1, 10 do
    local v = serial.query(":MEAS:VOLT?\r\n", 1000)
    log.write(string.format("%d: %s", i, v))
    last_val = v
    hw.delay(1000)
end

log.write("処理完了")
notify.set("測定完了: 最終値 " .. last_val)    -- ← メッセージを渡す
```

#### 通知の確認方法（ポーリング）

LUNA の通知はポーリング方式です。Claude が `check_notify` を呼んだときにメッセージを受け取ります。

**Claude Desktop（通常利用）**:

スクリプト実行後、Claude に一言伝えるだけで確認できます。

```
「notify を確認して」
```

**Claude Code（上級者向け）**:

`/loop` スキルで定期的に自動確認できます。

```
/loop 1m check_notify を呼んで flag=1 なら報告して
```

→ 1分ごとに自動確認し、通知があれば即報告します。

---

### 2-5. luna モジュール ── スクリプトチェーン実行（v3.9.1）

#### `luna.run(name)` ── 次のスクリプトを自動起動

現在のスクリプトが終了した後、指定したスクリプトを自動的に起動します。
**Claude の介入なしに**複数のスクリプトを順番に実行できます。

| 引数 | 型 | 説明 |
|------|----|------|
| name | string | 次に実行するスクリプト名（`lua_save` で保存済みのもの） |

| 戻り値 | 型 | 説明 |
|--------|----|------|
| result | boolean | 予約成功で `true`、チェーン上限超過で `false` |

```lua
-- スクリプト "init" の末尾
log.write("初期化完了")
log.save()
luna.run("measure")   -- 終了後 "measure" を自動起動
notify.set("完了")
```

#### 動作の仕組み：「キューして終了」方式

`luna.run()` を呼んでも**現在のスクリプトはすぐには止まりません**。スクリプトが最後まで実行された後、ESP32 が次のスクリプトを自動的に読み込んで起動します。

```
Claude
  ↓ lua_run("init") を1回呼ぶだけ
ESP32（自律動作）
  init 実行 → luna.run("measure") → init 終了
  measure 実行 → luna.run("analyze") → measure 終了
  analyze 実行 → notify.set("完了") → 終了
  ↓
Claude に通知（全シーケンス完了）
```

#### チェーン上限：最大10本

無限ループを防ぐため、チェーンは最大10本（最初の1本＋9本）に制限されています。

```lua
-- NG例（無限ループ）
-- Script "a": luna.run("b")
-- Script "b": luna.run("a")  ← 10回目で自動停止
-- log に "chain limit reached" が記録される
```

上限に達した場合：
- `luna.run()` が `false` を返す
- log に `"luna.run: chain limit (10) reached"` が記録される
- 現在のスクリプトはそのまま続行（クラッシュしない）

#### 実用例：観測シーケンス

```lua
-- lua_save("obs_init", ...) で保存
log.write("obs: 初期化開始")
-- 機器初期化処理
log.write("obs: 初期化完了")
log.save()
luna.run("obs_measure")   -- 測定スクリプトへ
notify.set("完了")

-- lua_save("obs_measure", ...) で保存
log.load()
log.write("obs: 測定開始")
for i = 1, 5 do
    local v = serial.query(":MEAS?\r\n", 1000) or "err"
    log.write(string.format("obs: %d=%s", i, v))
    hw.delay(10000)
end
log.save()
luna.run("obs_finish")    -- 終了処理スクリプトへ
notify.set("完了")

-- lua_save("obs_finish", ...) で保存
log.load()
log.write("obs: シーケンス完了")
notify.set("全スクリプト完了")   -- Claude に最終完了を通知
```

> ⚠️ **注意**: `luna.run()` で起動するスクリプトは事前に `lua_save` で保存しておく必要があります。存在しないスクリプト名を指定した場合はエラーが log に記録されてチェーンが終了します。

---

### 2-6. http モジュール — HTTP通信・Alpaca 連携（v5.9.1）

> **v5.9.1 新機能** — LUNA Observatory から追加されました。

LAN 内の HTTP デバイスへリクエストを送信します。
**AiBridge（ASCOM/Alpaca 対応）** との連携に最適です。

#### `http.get(url, timeout_ms)` ★推奨（状態取得）

```lua
local body = http.get(url, timeout_ms)
```

| 引数 | 説明 |
|------|------|
| `url` | リクエスト先 URL（HTTP のみ。HTTPS 非対応） |
| `timeout_ms` | タイムアウト（省略時 2000ms） |
| 戻り値 | レスポンスボディ文字列。エラー・タイムアウト時は `""` |

```lua
-- Alpaca 気温取得（AiBridge observingconditions）
local t = http.get("http://192.168.3.7/api/v1/observingconditions/0/temperature", 1000)
if t == "" then return '{"error":"no response"}' end
return '{"temp":' .. t .. '}'
```

#### `http.put(url, body, timeout_ms)` — コマンド送信

```lua
local resp = http.put(url, body, timeout_ms)
```

| 引数 | 説明 |
|------|------|
| `url` | リクエスト先 URL |
| `body` | リクエストボディ（`application/x-www-form-urlencoded` 形式） |
| `timeout_ms` | タイムアウト（省略時 2000ms） |
| 戻り値 | レスポンスボディ文字列。エラー時は `""` |

```lua
-- Alpaca スイッチ ON（AiBridge switch）
local r = http.put(
  "http://192.168.3.7/api/v1/switch/0/setswitch",
  "Id=0&State=true",
  1000
)
return r ~= "" and "OK" or "error"
```

#### AiBridge Alpaca 連携スクリプト例

**気象センサーを Resource として公開（lua_resource_weather.lua）**

```lua
local temp = http.get("http://192.168.3.7/api/v1/observingconditions/0/temperature", 1000)
local hum  = http.get("http://192.168.3.7/api/v1/observingconditions/0/humidity",    1000)
if temp == "" then return '{"error":"no response from AiBridge"}' end
return string.format('{"temp":%s,"humidity":%s}', temp, hum)
```

**スイッチポート状態を Resource として公開（lua_resource_switch.lua）**

```lua
local s0 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=0", 1000)
local s1 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=1", 1000)
local s2 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=2", 1000)
local s3 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=3", 1000)
return string.format('{"port0":%s,"port1":%s,"port2":%s,"port3":%s}', s0, s1, s2, s3)
```

#### タイムアウト推奨値

| 用途 | 推奨タイムアウト |
|------|----------------|
| Alpaca GET（センサー・状態取得） | 1000ms |
| Alpaca PUT（コマンド送信） | 2000ms |
| AiBridge `/api/external/discover` | 4000ms（UDP 待ちのため長め） |

#### 注意事項

- **HTTPS 非対応**（LAN 内 HTTP 専用）
- **ブロッキング動作** — リクエスト中は Lua VM が停止。タイムアウトを短めに設定
- エラー・タイムアウト時は `""` を返す。必ずチェックすること
- `log.save()` と同様、ループ内の連続呼び出しは避ける（メモリ断片化）
- AiBridge のファームウェア変更は不要。Alpaca REST サーバーとして動作するだけ

#### 実機確認済み（2026-03-21）

| テスト | 応答時間 |
|--------|---------|
| AiBridgeMCP Console (192.168.4.1) | 35ms ✅ |
| AiBridge Console (192.168.3.7) | 153ms ✅ |
| switch/maxswitch → Value:4 | ✅ |
| switch getswitch Id=0〜3 → 全 false | 315ms ✅ |
| observingconditions/temperature → -999.0 | ✅ |

---

### 2-7. 標準 Lua 5.1 ライブラリ

以下の標準ライブラリが使用できます。

| ライブラリ | 主な関数 |
|------------|----------|
| `string.*` | format, find, sub, len, rep, upper, lower |
| `math.*` | floor, ceil, sqrt, abs, max, min, pi |
| `table.*` | insert, remove, concat, sort |
| `coroutine.*` | create, resume, yield, status, wrap |
| 基本関数 | tonumber, tostring, type, print, pcall, ipairs, pairs |

```lua
-- よく使う組み合わせ
local resp = serial.query(":MEAS:VOLT?\r\n", 1000)
local val = tonumber(resp)
if val then
    log.write(string.format("%.4f V", val))
else
    log.write("変換失敗: [" .. tostring(resp) .. "]")
end
```

---

### 2-8. 数値型・16進数・ビット演算（実機確認済み）

#### 数値型

Lua 5.1 の数値はすべて **double（倍精度浮動小数点）** で保持されます。
整数・実数の区別はなく、同じ number 型として扱われます。

```lua
local a = 42          -- 整数として使える
local b = -7
local f = 3.14159     -- 実数

log.write(a + b)      -- → 35
log.write(a * b)      -- → -294
log.write(math.floor(a / 5))  -- → 8（整数除算は math.floor を使う）
```

#### 16進数（実機確認済み ✅）

```lua
-- 16進数リテラル（0x プレフィックス）
local h1 = 0xFF       -- → 255
local h2 = 0x1A       -- → 26
local h3 = 0xDEAD     -- → 57005

-- 10進数 → 16進数文字列
string.format("0x%X",   255)   -- → "0xFF"
string.format("0x%04X", 1234)  -- → "0x04D2"（4桁ゼロ埋め）
string.format("%02X",   10)    -- → "0A"

-- 16進数文字列 → 数値
tonumber("FF",   16)   -- → 255
tonumber("1A2B", 16)   -- → 6699
tonumber("dead", 16)   -- → 57005（小文字も可）
```

#### ビット演算（実機確認済み ❌）

`bit` ライブラリは本バージョンでは**使用できません**。
代替として、以下の純粋 Lua での実装が使えます。

```lua
-- AND の代替
local function band(a, b)
    local result = 0
    local bit = 1
    while a > 0 and b > 0 do
        if a % 2 == 1 and b % 2 == 1 then result = result + bit end
        a = math.floor(a / 2)
        b = math.floor(b / 2)
        bit = bit * 2
    end
    return result
end

-- OR の代替
local function bor(a, b)
    local result = 0
    local bit = 1
    while a > 0 or b > 0 do
        if a % 2 == 1 or b % 2 == 1 then result = result + bit end
        a = math.floor(a / 2)
        b = math.floor(b / 2)
        bit = bit * 2
    end
    return result
end

-- 左シフトの代替
local function lshift(a, n)
    return a * (2 ^ n)
end

-- 右シフトの代替
local function rshift(a, n)
    return math.floor(a / (2 ^ n))
end

-- 使用例
log.write(band(0xFF, 0x0F))    -- → 15
log.write(bor(0xF0, 0x0F))     -- → 255
log.write(lshift(1, 4))        -- → 16
log.write(rshift(0xFF, 4))     -- → 15
```

> 💡 **ヒント**: 機器の応答が16進数フォーマットの場合、`tonumber(resp, 16)` で数値に変換できます。
> ビット演算が多い用途（バイナリプロトコル等）では、Claude に代替関数ごとスクリプトに含めてもらいましょう。

---

### 2-9. 使用できない機能

| 機能 | 結果 | 理由 |
|------|------|------|
| `io.*`（ファイル読み書き） | ❌ | SPIFFS へのアクセスは MCP ツール経由で行う |
| `os.*`（OS 操作） | ❌ | 組み込み環境のため未サポート |
| `require()`（外部モジュール） | ❌ | 単一スクリプト環境のため未サポート |
| ネットワーク通信（HTTP・TCP等） | ❌ | 現バージョン未実装 |
| `bit.*`（ビット演算ライブラリ） | ❌ | 未実装（純粋 Lua で代替可能） |
| 複数スクリプトの同時実行 | ❌ | 1スクリプトのみ（コルーチンで疑似並行は可能） |

---

### 2-10. クイックリファレンス一覧

```lua
-- シリアル通信
serial.write(data)                  -- 送信のみ
serial.read()                       -- 受信のみ
serial.query(data, timeout_ms)      -- 送信→受信（推奨）

-- ログ
log.write(message)                  -- ログ記録（最新20行保持）
log.read([n])                       -- 行取得（省略/-1=最新、0=最古、負=末尾から）
log.save()                          -- リングバッファ → SPIFFS /luna_log.txt（終了時1回のみ）
log.load()                          -- SPIFFS /luna_log.txt → リングバッファ

-- ハードウェア
hw.delay(ms)                        -- 待機（HTTPサーバをブロックしない）
hw.millis()                         -- 起動からの経過ミリ秒
hw.free_heap()                      -- 空きヒープ（バイト）
hw.led(1 or 0)                      -- 内蔵LED制御（数値のみ）

-- 通知
notify.set("完了")                  -- Claude に完了を通知

-- スクリプトチェーン（v3.9.1）
luna.run(name)                      -- 現在のスクリプト終了後に name を自動起動（最大10本）

-- HTTP通信・Alpaca 連携（v5.9.1）
http.get(url, timeout_ms)           -- HTTP GET。レスポンスボディを返す（エラー時 ""）
http.put(url, body, timeout_ms)     -- HTTP PUT。レスポンスボディを返す（エラー時 ""）
```

---

## 第3章　実用スクリプト集

### 3-1. 機器の動作確認

```lua
-- 機器が応答するか確認する
local resp = serial.query("*IDN?\r\n", 2000)
if resp ~= "" then
    log.write("機器確認OK: " .. resp)
else
    log.write("エラー: 応答なし（ケーブル・ボーレートを確認してください）")
end
notify.set("完了")
```

---

### 3-2. 定期測定（複数回）

```lua
-- 1秒おきに10回測定してログに記録
local count = 10
log.write(string.format("測定開始 (%d回)", count))

local sum = 0
for i = 1, count do
    local raw = serial.query(":MEAS:VOLT:DC?\r\n", 2000)
    local val = tonumber(raw) or 0
    sum = sum + val
    log.write(string.format("[%d/%d] %.6f V", i, count, val))
    if i < count then hw.delay(1000) end
end

local avg = sum / count
log.write(string.format("平均値: %.6f V", avg))
log.write(string.format("経過時間: %d ms", hw.millis()))
notify.set("測定完了")
```

---

### 3-3. 条件分岐制御

```lua
-- 応答内容によって次の操作を変える
local status = serial.query(":STAT?\r\n", 1000)
log.write("状態: " .. status)

if string.find(status, "READY") then
    log.write("準備完了 → 測定開始")
    local val = serial.query(":MEAS?\r\n", 2000)
    log.write("測定値: " .. val)
elseif string.find(status, "BUSY") then
    log.write("ビジー → 2秒後に再試行")
    hw.delay(2000)
    local val = serial.query(":MEAS?\r\n", 2000)
    log.write("測定値（再試行）: " .. val)
else
    log.write("不明な状態: " .. status)
end

notify.set("完了")
```

---

### 3-4. エラーハンドリング付き測定

```lua
-- pcall でエラーをキャッチして安全に実行
local ok, err = pcall(function()
    for i = 1, 5 do
        local resp = serial.query(":MEAS:VOLT?\r\n", 1500)
        if resp == "" then
            error("タイムアウト（" .. i .. "回目）")
        end
        local val = tonumber(resp)
        if not val then
            error("数値変換失敗: [" .. resp .. "]")
        end
        log.write(string.format("%d: %.4f V", i, val))
        hw.delay(1000)
    end
end)

if ok then
    log.write("全測定完了")
else
    log.write("エラー発生: " .. tostring(err))
end
notify.set("完了")
```

---

### 3-5. LED を使った動作インジケーター

```lua
-- 処理中は LED 点滅、完了で3回速点滅
log.write("処理開始")

-- 処理中インジケーター（5回点滅しながら処理）
for i = 1, 5 do
    hw.led(1)
    local resp = serial.query(":MEAS?\r\n", 1000)
    log.write(string.format("%d: %s", i, resp))
    hw.led(0)
    hw.delay(500)
end

-- 完了サイン（速点滅3回）
for i = 1, 3 do
    hw.led(1)
    hw.delay(100)
    hw.led(0)
    hw.delay(100)
end

log.write("処理完了")
notify.set("完了")
```

---

### 3-6. 複数コマンドの順次実行

```lua
-- 初期化 → 設定 → 測定 → 終了 の手順を自動化
local function send(cmd, wait_ms)
    wait_ms = wait_ms or 500
    local resp = serial.query(cmd, wait_ms)
    log.write(cmd:gsub("\r\n","") .. " → " .. (resp ~= "" and resp or "OK"))
    return resp
end

log.write("=== 初期化シーケンス開始 ===")
send("*RST\r\n", 2000)        -- リセット
hw.delay(1000)
send(":CONF:VOLT:DC\r\n")     -- DC電圧モード設定
send(":VOLT:RANG 10\r\n")     -- レンジ設定
send(":TRIG:SOUR IMM\r\n")    -- トリガ設定
hw.delay(500)

log.write("=== 測定開始 ===")
for i = 1, 3 do
    local val = send(":READ?\r\n", 2000)
    hw.delay(1000)
end

log.write("=== 完了 ===")
notify.set("完了")
```

---

### 3-7. コルーチンによる疑似並行処理

v3.9.0 では1スクリプトしか実行できませんが、**Lua のコルーチン**を使うと、1つのスクリプト内で複数の処理を交互に進める「疑似並行」ができます。

#### コルーチンとは

処理を途中で一時停止（`yield`）して、別の処理に切り替えられる仕組みです。
真の並列実行ではありませんが、「2つの処理を交互に進める」ような場面で有効です。

```
タスクA: ステップ1 → yield → ステップ2 → yield → ステップ3 → 完了
タスクB:              ステップ1 → yield → ステップ2 → yield → ステップ3 → 完了
```

#### 基本的な使い方（実機確認済み）

```lua
-- 2つのタスクを交互に実行する例
local function task_a()
    for i = 1, 3 do
        log.write("タスクA: ステップ " .. i)
        coroutine.yield()       -- ここで一時停止、制御を返す
    end
    log.write("タスクA: 完了")
end

local function task_b()
    for i = 1, 3 do
        log.write("タスクB: ステップ " .. i)
        coroutine.yield()
    end
    log.write("タスクB: 完了")
end

-- コルーチンを生成
local co_a = coroutine.create(task_a)
local co_b = coroutine.create(task_b)

-- スケジューラ：交互に実行
while true do
    local a_alive = coroutine.status(co_a) ~= "dead"
    local b_alive = coroutine.status(co_b) ~= "dead"
    if not a_alive and not b_alive then break end
    if a_alive then coroutine.resume(co_a) end
    if b_alive then coroutine.resume(co_b) end
end

log.write("全タスク完了")
notify.set("完了")
```

**実行結果:**
```
タスクA: ステップ 1
タスクB: ステップ 1
タスクA: ステップ 2
タスクB: ステップ 2
タスクA: ステップ 3
タスクB: ステップ 3
タスクA: 完了
タスクB: 完了
全タスク完了
```

#### 実用例：測定と LED 点滅を同時進行

```lua
-- 測定中に LED を点滅させる
local function led_blinker(count)
    for i = 1, count do
        hw.led(1)
        coroutine.yield()
        hw.led(0)
        coroutine.yield()
    end
end

local function measure(count)
    local results = {}
    for i = 1, count do
        local v = serial.query(":MEAS:VOLT?\r\n", 1000)
        log.write(string.format("[%d] %s V", i, v))
        coroutine.yield()
        hw.delay(500)
        coroutine.yield()
    end
    log.write("測定完了")
end

local co_led  = coroutine.create(function() led_blinker(10) end)
local co_meas = coroutine.create(function() measure(5) end)

while true do
    local l = coroutine.status(co_led)  ~= "dead"
    local m = coroutine.status(co_meas) ~= "dead"
    if not l and not m then break end
    if l then coroutine.resume(co_led)  end
    if m then coroutine.resume(co_meas) end
end

notify.set("測定完了")
```

#### コルーチン関数一覧

| 関数 | 説明 |
|------|------|
| `coroutine.create(f)` | 関数 f からコルーチンを生成する |
| `coroutine.resume(co)` | コルーチンを（再）開始する |
| `coroutine.yield()` | 呼び出し元に制御を返して一時停止 |
| `coroutine.status(co)` | 状態を返す（`"running"` / `"suspended"` / `"dead"`） |
| `coroutine.wrap(f)` | resume を関数として呼び出せるラッパーを返す |

> **注意**: コルーチン内で `hw.delay()` を呼ぶと、その間は他のコルーチンも停止します。
> 疑似並行のポイントは `yield` と `delay` の組み合わせを工夫することです。

---

### 3-8. MCP Resources — リソーススクリプトの書き方（v5.9.1）

> **📖 詳細**: `LUNA_Observatory_Resources_Guide.md` を参照してください。

#### Resources とは何か

LUNA Observatory（v5.9.1）では、Lua スクリプトを「計器」として Claude に公開できます。
Claude が「今どこ向いてる？」と聞かれたとき、serial_query を2回呼ぶ代わりに、
`luna://lua/mount` を1回読むだけで即座に回答できます。

```
【3-1〜3-7 の通常スクリプト】   lua_exec / lua_run で実行 → 処理を行う
【3-8 のリソーススクリプト】     保存するだけで自動公開 → Claude が読む
```

#### 命名規則

| ファイル名 | URI（Claude が参照する名前） |
|-----------|--------------------------|
| `lua_resource_mount.lua` | `luna://lua/mount` |
| `lua_resource_weather.lua` | `luna://lua/weather` |
| `lua_resource_switch.lua` | `luna://lua/switch` |

`lua_resource_` で始まるファイルを SPIFFS に保存すると、**自動的に Resource として公開**されます。ファームウェアの変更は不要です。

#### 基本テンプレート

```lua
-- lua_resource_{名前}.lua
-- ① 機器からデータを取得
local value = serial.query("コマンド", 100)

-- ② エラーチェック（必須）
if value == "" then
  return '{"error":"no response"}'
end

-- ③ JSON 文字列を return で返す（必須）
return string.format('{"key":"%s"}', value)
```

**重要な規約:**

| 項目 | 規約 |
|------|------|
| 戻り値 | `return` で JSON 文字列を返す（必須） |
| エラー時 | `'{"error":"メッセージ"}'` を返す |
| 副作用 | 機器を動かすコマンドは含めない（読み取りのみ） |
| 実行時間 | できるだけ短く（`serial.query` は 100〜200ms 推奨） |
| `log.save()` | リソーススクリプト内では**呼ばない**（フラッシュ寿命） |

#### 実例① RS-232C 機器（赤道儀 LX200互換）

```lua
-- lua_resource_mount.lua
local ra  = serial.query(":GR#", 100)
local dec = serial.query(":GD#", 100)
if ra == "" or dec == "" then
  return '{"error":"no response from mount"}'
end
return string.format('{"ra":"%s","dec":"%s"}', ra, dec)
```

#### 実例② AiBridge（Alpaca 気象センサー）

```lua
-- lua_resource_weather.lua（http モジュールを使用）
local temp = http.get("http://192.168.3.7/api/v1/observingconditions/0/temperature", 1000)
local hum  = http.get("http://192.168.3.7/api/v1/observingconditions/0/humidity",    1000)
if temp == "" then return '{"error":"no response from AiBridge"}' end
return string.format('{"temp":%s,"humidity":%s}', temp, hum)
```

#### 実例③ AiBridge（スイッチポート状態）

```lua
-- lua_resource_switch.lua
local s0 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=0", 1000)
local s1 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=1", 1000)
local s2 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=2", 1000)
local s3 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=3", 1000)
return string.format('{"port0":%s,"port1":%s,"port2":%s,"port3":%s}', s0, s1, s2, s3)
```

#### 保存方法

Claude に次のように指示するだけで保存できます：

```
「以下の Lua スクリプトを resource_mount という名前で保存して」
（スクリプト内容を貼り付ける）
```

→ `lua_save("resource_mount", ...)` が実行され `/lua_resource_mount.lua` として保存
→ `luna://lua/mount` が自動的に Resources に現れる

#### Claude Desktop での使い方

保存後、Claude Desktop から自然な会話で使えます：

```
ユーザー:「今の望遠鏡の向きを教えて」
Claude:  luna://lua/mount を読む → RA/Dec を即答

ユーザー:「今夜観測できそう？」
Claude:  luna://lua/weather も読んで → 気温・湿度も含めて総合判断
```

---

## 第4章　使い方のコツと注意事項

### 4-1. Claude との連携の基本

**① まず動作確認から始める**

```
Claude への指示：「*IDN? コマンドで機器が応答するか確認して」
→ Claude が serial_query で直接確認
→ 応答が確認できたら Lua スクリプトへ進む
```

**② スクリプトは小さく試してから拡張する**

```
悪い例：いきなり複雑なスクリプトを作る
良い例：1コマンドのテスト → 3回ループ → 条件分岐追加 と段階的に
```

**③ マニュアルを渡すと Claude がコマンドを調べてくれる**

```
Claude への指示：
「このマニュアル（PDF）を読んで、電圧を測定するコマンドを確認して、
 1秒おきに5回測定するスクリプトを作って」
```

---

### 4-2. log.write の使い方のコツ

20行の制限を有効活用するために：

```lua
-- 悪い例：毎回 log.write すると大量測定時に溢れる
for i = 1, 100 do
    log.write("測定: " .. serial.query(":MEAS?\r\n", 1000))
end

-- 良い例：10回ごとに集計して記録
local sum = 0
for i = 1, 100 do
    sum = sum + (tonumber(serial.query(":MEAS?\r\n", 1000)) or 0)
    if i % 10 == 0 then
        log.write(string.format("[%d/100] 小計平均: %.4f", i, sum/i))
    end
end
log.write(string.format("最終平均: %.4f", sum/100))
```

---

### 4-3. タイムアウト値の目安

| 機器の種類 | 推奨タイムアウト |
|------------|----------------|
| 応答の早い機器（測定器等） | 500〜1000 ms |
| 動作を伴う機器（望遠鏡等） | 2000〜5000 ms |
| 初期化・リセットコマンド | 2000〜5000 ms |
| 不明な場合 | 2000 ms から始める |

---

### 4-4. よくあるトラブルと対処法

| 症状 | 原因 | 対処法 |
|------|------|--------|
| `serial.query` が空文字を返す | タイムアウト・終端文字の違い | タイムアウトを伸ばす / `\r\n` → `\r` に変更 |
| `tonumber()` が nil を返す | 余分な空白や改行が含まれている | `string.gsub(resp, "%s+", "")` で除去 |
| スクリプトが止まる | 無限ループ・長い hw.delay | `lua_stop` ツールで強制停止 |
| 並行実行エラー | 既に別スクリプトが動いている | `lua_status` で確認 → 完了まで待つ |
| スクリプトが見つからない | 名前の誤り・未保存 | `lua_list` で保存済み一覧を確認 |

---

### 4-5. スクリプト名のルール

| 条件 | 内容 |
|------|------|
| 使える文字 | 英数字、アンダースコア `_`、ハイフン `-` |
| 最大文字数 | **22文字**（SPIFFS の制約） |
| 大文字・小文字 | 区別されます |
| サブディレクトリ | **使用不可**（スラッシュ `/` は使えない） |

```
良い例: volt_check, led_test, temp_monitor, my-script-01
悪い例: 電圧測定（日本語不可）, volt check（スペース不可）, lua/test（スラッシュ不可）
```

> **⚠️ 重要：サブディレクトリは作れません**
>
> SPIFFS はフラット構造のため、ディレクトリの概念がありません。
> すべてのスクリプトは同じ階層に保存されます。
>
> ```
> /lua_led_test.lua       ← すべてフラットに並ぶ
> /lua_volt_avg.lua
> /lua_measure01.lua
> ```

#### カテゴリ分けの推奨命名規則

スクリプトが増えてきたら、**プレフィックスでカテゴリを表現**することを推奨します。

```
serial_check      ← シリアル通信確認系
serial_volt       ← シリアル通信系
measure_temp      ← 測定系
measure_volt_avg  ← 測定系
led_blink         ← LED制御系
led_test          ← LED制御系
scope_trigger     ← オシロスコープ系
temma_goto        ← 望遠鏡制御系
```

`lua_list` で一覧表示したときに、カテゴリごとにまとまって見えるようになります。

---

### 4-6. 同時実行について

v3.9.0 では **1つのスクリプトのみ** 実行できます。

```lua
-- 複数の処理をしたい場合は、1つのスクリプトにまとめる
-- 処理 A
local v = serial.query(":MEAS:VOLT?\r\n", 1000)
log.write("電圧: " .. v)

-- 処理 B
local t = serial.query(":MEAS:TEMP?\r\n", 1000)
log.write("温度: " .. t)

notify.set("完了")
```

---

## 第5章　Claude と一緒に使うインタラクティブプログラミング

### 5-1. これは何か

AiBridgeMCP v3.9.0 は、単なる制御ツールではありません。
Claude と会話しながら、**モニターを見て結果を確認しながら**、その場でスクリプトを作り・動かし・育てていける、**AI 時代のインタラクティブプログラミング環境**です。

Lua を知らなくても大丈夫です。**「やりたいこと」を日本語で伝えるだけ**で使えます。

---

### 5-2. 基本的な進め方

#### ステップ① まず機器と話してみる

難しいことは考えず、最初は Claude に確認を頼みます。

```
ユーザー：「繋いだ機器が応答するか確認して。
           コマンドは *IDN? で、終端は CR+LF です。」

Claude   ：（serial_query で確認）
           → 応答：KEITHLEY INSTRUMENTS, MODEL 2110
           → 正常に通信できています。
```

> ポイント：最初の1コマンドだけ試す。動けば次へ進む。

---

#### ステップ② やりたいことを言葉で伝える

動作確認が取れたら、やりたいことを日本語で伝えます。

```
ユーザー：「電圧を1秒おきに5回測定して、結果を表示して」

Claude   ：以下のスクリプトを実行します。

           for i = 1, 5 do
               local v = serial.query(":MEAS:VOLT:DC?\r\n", 2000)
               log.write(string.format("[%d] %.4f V", i, tonumber(v) or 0))
               hw.delay(1000)
           end
           notify.set("完了")

           → 実行中...（5秒後）
           → [1] 1.2345 V
           → [2] 1.2347 V
           → ...
```

---

#### ステップ③ 結果を見て、修正を依頼する

結果を見て、気になる点があれば言葉で伝えるだけです。

```
ユーザー：「10回に増やして、最後に平均値も出して」

Claude   ：修正して再実行します。
           （修正版を生成 → 即実行）

ユーザー：「タイムアウトが発生することがある」

Claude   ：タイムアウト値を 1000ms → 2000ms に変更して
           エラー処理も追加します。
           （修正版を生成 → 即実行）
```

> ポイント：エラーが出ても怒らない。Claude が原因を説明して直してくれる。

---

#### ステップ④ 動いたら保存する

満足のいく結果が出たら、名前をつけて保存します。

```
ユーザー：「これを volt_check という名前で保存して」

Claude   ：lua_save("volt_check", ...) → 保存完了
           次回から「volt_check を実行して」と言えば
           すぐに動きます。
```

---

#### ステップ⑤ 次回は名前だけで呼び出す

保存したスクリプトは電源を切っても残ります。

```
ユーザー：「volt_check を実行して」

Claude   ：lua_run("volt_check") → 実行
           → 結果表示
```

---

### 5-3. マニュアルを渡してスクリプトを作らせる

機器のコマンドを知らなくても、マニュアルを Claude に渡すだけで使えます。

```
ユーザー：「この測定器のマニュアル（PDF）です。
           温度と電圧を交互に5回ずつ測って、
           CSV形式でログに残すスクリプトを作って」

Claude   ：マニュアルを確認しました。
           温度は :MEAS:TEMP?、電圧は :MEAS:VOLT:DC? ですね。
           以下のスクリプトはどうでしょう。

           log.write("time_ms,type,value")
           for i = 1, 5 do
               local t = serial.query(":MEAS:TEMP?\r\n", 2000)
               local v = serial.query(":MEAS:VOLT:DC?\r\n", 2000)
               local ms = hw.millis()
               log.write(ms .. ",TEMP," .. (t or "ERR"))
               log.write(ms .. ",VOLT," .. (v or "ERR"))
               hw.delay(2000)
           end
           notify.set("完了")

           テストしますか？
```

> **これがこのシステムの最大の特長です。**
> ユーザーはコマンドを覚える必要がありません。
> マニュアルを渡して「やりたいこと」を言葉にするだけで、
> Claude がスクリプトを生成・実行・修正まで代行します。

---

### 5-4. うまく進めるためのコツ

#### ① 一度に全部やろうとしない

```
悪い例：「初期化して、設定して、100回測定して、統計処理して、
         異常値を検出して、アラームを出して」
         → 一気に作ると、どこで失敗したか分からない

良い例：まず「1回測れるか確認して」
        → 動いたら「10回測って」
          → 動いたら「統計処理を追加して」
            → 段階的に積み上げる
```

#### ② エラーはチャンス

```
エラーが出たとき：
「こんなエラーが出ました。原因を教えて、修正して」

Claude が：
  - 原因を日本語で説明
  - 修正版を生成
  - 再実行
```

プログラミングの知識がなくても、Claude がデバッグを代行します。

#### ③ 動いたらすぐ保存

```
「少し動いた」という段階でも保存しておく。
→ 後で「あの時のスクリプトに戻して」と言えば復元できる。
```

#### ④ ログは結果の「窓口」

`log.write()` は Claude との情報共有の手段です。
スクリプトの要所要所に記録を残すことで、
Claude がどこまで動いたかを正確に把握できます。

```lua
log.write("=== 開始 ===")         -- 開始を記録
log.write("初期化完了")
log.write("測定値: " .. val)       -- 重要な値を記録
log.write("=== 完了 ===")          -- 終了を記録
notify.set("完了")
```

#### ⑤ LED で「生きている」を確認する

長時間スクリプトの場合、LED を点滅させると
「動いているか止まっているか」が画面を見なくても分かります。

```lua
-- 処理のたびに LED を光らせる
for i = 1, 10 do
    hw.led(1)
    local v = serial.query(":MEAS?\r\n", 1000)
    log.write(v)
    hw.led(0)
    hw.delay(1000)
end
```

---

### 5-5. 従来のプログラミングとの違い

| 項目 | 従来のプログラミング | LUNA システム |
|------|---------------------|-------------|
| コードを書く | 自分で書く | Claude が生成する |
| 実行環境 | IDE・コンパイラが必要 | Claude に話しかけるだけ |
| エラー対処 | 自分でデバッグ | Claude が原因を説明・修正 |
| 機器仕様の調査 | 自分でマニュアルを読む | マニュアルを渡せば Claude が読む |
| 結果の確認 | ターミナル・画面 | `lua_log` で即確認 |
| 保存・再利用 | ファイル管理 | 名前を付けて一言で保存 |
| 必要な前提知識 | Lua の文法・APIを知る必要あり | **日本語で伝えるだけ** |

---

### 5-6. LUNA システムでできることのまとめ

```
話しかける  →  Claude がスクリプトを書く
               ↓
            ESP32 上で実行
               ↓
            結果を lua_log で確認
               ↓
話しかける  →  Claude が修正・改良する
               ↓
            満足したら名前をつけて保存
               ↓
            次回から一言で呼び出せる
```

> **「プログラミングの知識がなくても、**
> **やりたいことを言葉にできれば、機器を自動制御できる。」**
>
> これが LUNA システムの目指す姿です。

---

## 付録　MCP ツール一覧（Claude が使うツール）

| ツール | 説明 |
|--------|------|
| `lua_exec` | Lua コードを直接実行（テスト向け） |
| `lua_status` | 実行中スクリプトの状態を確認 |
| `lua_stop` | 実行中スクリプトを強制停止 |
| `lua_log` | ログバッファの内容を取得 |
| `lua_save` | スクリプトを名前付きで SPIFFS に保存 |
| `lua_run` | 保存済みスクリプトを名前で実行 |
| `lua_list` | 保存済みスクリプトの一覧を表示 |
| `serial_query` | Claude が直接シリアルコマンドを送受信 |

---

## 付録B　Lua 言語 最小リファレンス

> このリファレンスは、AiBridgeMCP で使う範囲に絞った最小限の Lua 5.1 言語解説です。
> Claude が生成したスクリプトを読んで理解する、または小さな修正を自分で行うことを目的としています。

---

### B-1. 変数と型

変数は `local` で宣言します。型の宣言は不要です。

```lua
local a = 42          -- 数値（整数）
local b = 3.14        -- 数値（小数）
local s = "hello"     -- 文字列
local s2 = 'world'    -- 文字列（シングルクォートも可）
local flag = true     -- 真偽値（true / false）
local nothing = nil   -- 値なし（未定義・削除）
```

**型の確認：**
```lua
type(42)        -- → "number"
type("hello")   -- → "string"
type(true)      -- → "boolean"
type(nil)       -- → "nil"
```

---

### B-2. 演算子

**算術演算子：**

| 演算子 | 意味 | 例 | 結果 |
|--------|------|----|------|
| `+` | 加算 | `3 + 2` | `5` |
| `-` | 減算 | `3 - 2` | `1` |
| `*` | 乗算 | `3 * 2` | `6` |
| `/` | 除算 | `7 / 2` | `3.5` |
| `%` | 余り | `7 % 2` | `1` |
| `^` | べき乗 | `2 ^ 8` | `256` |

```lua
local x = math.floor(7 / 2)   -- 整数除算 → 3
```

**比較演算子：**

| 演算子 | 意味 |
|--------|------|
| `==` | 等しい |
| `~=` | 等しくない（`!=` ではなく `~=`）|
| `<` / `>` | 小さい / 大きい |
| `<=` / `>=` | 以下 / 以上 |

```lua
10 == 10    -- → true
10 ~= 5     -- → true（不等号は ~=）
```

**論理演算子：**

| 演算子 | 意味 |
|--------|------|
| `and` | かつ |
| `or` | または |
| `not` | 否定 |

```lua
true and false   -- → false
true or false    -- → true
not true         -- → false
```

**文字列結合：**

```lua
"Hello" .. " " .. "World"   -- → "Hello World"
"値: " .. 42                -- → "値: 42"
```

> ⚠️ 数値を文字列に連結するときは自動変換されますが、
> 確実にするには `tostring(42)` を使います。

---

### B-3. 条件分岐（if）

```lua
if 条件 then
    -- 条件が true のとき
elseif 別の条件 then
    -- 別の条件が true のとき
else
    -- どれにも当てはまらないとき
end
```

**実例：**
```lua
local val = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000))

if val == nil then
    log.write("エラー: 数値変換失敗")
elseif val > 5.0 then
    log.write("高電圧: " .. val .. " V")
elseif val > 1.0 then
    log.write("正常: " .. val .. " V")
else
    log.write("低電圧: " .. val .. " V")
end
```

**よく使うパターン：**
```lua
-- nil チェック
if val then
    log.write("値あり: " .. val)
end

-- 文字列の中身を確認
if string.find(resp, "OK") then
    log.write("応答OK")
end
```

---

### B-4. 繰り返し（for / while）

**数値 for（最も頻出）：**

```lua
for 変数 = 開始, 終了 do
    -- 処理
end

for 変数 = 開始, 終了, ステップ do
    -- 処理
end
```

```lua
-- 1 から 5 まで
for i = 1, 5 do
    log.write("i = " .. i)
end

-- 10 から 1 まで（逆順）
for i = 10, 1, -1 do
    log.write(i)
end

-- 2 飛ばし
for i = 0, 10, 2 do
    log.write(i)    -- 0, 2, 4, 6, 8, 10
end
```

**while（条件が true の間繰り返す）：**

```lua
while 条件 do
    -- 処理
end
```

```lua
local count = 0
while count < 5 do
    log.write("count = " .. count)
    count = count + 1
end
```

**break（ループを途中で抜ける）：**

```lua
for i = 1, 100 do
    local v = tonumber(serial.query(":MEAS?\r\n", 1000)) or 0
    log.write(v)
    if v > 10.0 then
        log.write("閾値超え → 停止")
        break
    end
end
```

---

### B-5. 関数

```lua
local function 関数名(引数1, 引数2)
    -- 処理
    return 戻り値
end
```

```lua
-- 基本的な関数
local function add(a, b)
    return a + b
end
log.write(add(3, 5))    -- → 8

-- 引数にデフォルト値を設定する
local function measure(cmd, timeout)
    timeout = timeout or 1000    -- 省略時は 1000
    return serial.query(cmd, timeout)
end

-- 複数の戻り値
local function min_max(t)
    local mn, mx = t[1], t[1]
    for _, v in ipairs(t) do
        if v < mn then mn = v end
        if v > mx then mx = v end
    end
    return mn, mx
end

local lo, hi = min_max({3, 1, 4, 1, 5, 9, 2, 6})
log.write("最小: " .. lo .. " 最大: " .. hi)
```

---

### B-6. テーブル（配列として使う）

Lua のテーブルは配列・辞書どちらにも使えます。
ここでは最もよく使う「配列」の使い方を説明します。

```lua
-- 配列の作成（インデックスは 1 から始まる）
local data = {}
local data2 = {10, 20, 30}   -- 初期値あり

-- 値の追加
table.insert(data, 1.234)
table.insert(data, 5.678)

-- 値の取得
log.write(data[1])    -- → 1.234
log.write(data[2])    -- → 5.678

-- 要素数
log.write(#data)      -- → 2

-- 全要素を処理
for i, v in ipairs(data) do
    log.write(string.format("[%d] %.3f", i, v))
end
```

**実用例：測定値を配列に蓄積して平均を出す**
```lua
local values = {}

for i = 1, 10 do
    local v = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000)) or 0
    table.insert(values, v)
    hw.delay(500)
end

-- 平均計算
local sum = 0
for _, v in ipairs(values) do
    sum = sum + v
end
log.write(string.format("平均: %.4f V", sum / #values))
```

---

### B-7. 文字列操作（よく使うもの）

```lua
-- フォーマット（printf スタイル）
string.format("%.4f", 3.14159)       -- → "3.1416"
string.format("%d", 255)             -- → "255"
string.format("0x%X", 255)           -- → "0xFF"
string.format("[%d/%d]", 3, 10)      -- → "[3/10]"

-- 数値変換
tonumber("3.14")                     -- → 3.14
tonumber("FF", 16)                   -- → 255（16進数）
tonumber("abc")                      -- → nil（変換失敗）

-- 文字列 → 数値の安全な変換
local val = tonumber(resp) or 0      -- 失敗時は 0 を使う

-- 文字列の検索
string.find("READY OK", "OK")        -- → 7, 8（位置を返す）
string.find("ERROR", "OK")           -- → nil（見つからない）

-- 文字列の長さ
string.len("hello")                  -- → 5
#"hello"                             -- → 5（同じ）

-- 空白・改行の除去
string.gsub(resp, "%s+", "")         -- 全空白を除去
string.gsub(resp, "[\r\n]", "")      -- 改行を除去

-- 大文字・小文字変換
string.upper("hello")                -- → "HELLO"
string.lower("HELLO")                -- → "hello"
```

---

### B-8. エラー処理（pcall）

エラーが発生してもスクリプトを止めずに続けたい場合に使います。

```lua
local ok, err = pcall(function()
    -- エラーが起きるかもしれない処理
    local v = serial.query(":MEAS?\r\n", 1000)
    if v == "" then error("タイムアウト") end
    log.write("値: " .. v)
end)

if ok then
    log.write("成功")
else
    log.write("エラー: " .. tostring(err))
end
```

**リトライ処理の例：**
```lua
local function safe_measure(retries)
    for i = 1, retries do
        local ok, result = pcall(function()
            local v = serial.query(":MEAS?\r\n", 1000)
            if v == "" then error("タイムアウト") end
            return v
        end)
        if ok then return result end
        log.write("リトライ " .. i .. "/" .. retries)
        hw.delay(500)
    end
    return nil
end

local val = safe_measure(3)
if val then
    log.write("測定値: " .. val)
else
    log.write("測定失敗（3回リトライ）")
end
notify.set("完了")
```

---

### B-9. よく使うパターン集

```lua
-- ① 測定値を安全に数値化する
local raw = serial.query(":MEAS?\r\n", 1000)
local val = tonumber(raw) or 0

-- ② 応答に特定の文字列が含まれるか確認する
if string.find(resp, "ERROR") then
    log.write("エラー応答: " .. resp)
end

-- ③ 繰り返し測定してログに記録する
for i = 1, 10 do
    local v = serial.query(":MEAS?\r\n", 1000)
    log.write(string.format("[%02d] %s", i, v))
    hw.delay(1000)
end

-- ④ 閾値を超えたら停止する
for i = 1, 100 do
    local v = tonumber(serial.query(":MEAS?\r\n", 1000)) or 0
    if v > 5.0 then
        log.write("閾値超え: " .. v)
        break
    end
    hw.delay(500)
end

-- ⑤ 複数の値をまとめて記録する
local t = serial.query(":MEAS:TEMP?\r\n", 1000)
local v = serial.query(":MEAS:VOLT?\r\n", 1000)
log.write(string.format("%d,%.3f,%.3f", hw.millis(),
          tonumber(t) or 0, tonumber(v) or 0))
```

---

## 付録C　Lua から Claude へのデータ受け渡し方法（実機確認済み）

> 外部機器からデータを受け取った Lua スクリプトが、そのデータを Claude に渡す方法を説明します。

---

### ⚠️ 重要：スクリプト間でグローバル変数は引き継がれない

`lua_exec` / `lua_run` を呼ぶたびに Lua の実行環境はリセットされます。
前回のスクリプトで定義した変数は、次のスクリプトからは見えません。

```lua
-- スクリプト①（lua_exec で実行）
x = 42          -- グローバル変数に代入しても…

-- スクリプト②（別の lua_exec で実行）
log.write(x)    -- → エラー！x は nil（前回の値は消えている）
```

> **理由**: メモリリーク防止のため、スクリプト実行のたびに Lua 内部状態を生成・破棄しています。

#### スクリプト間でデータを引き継ぐ方法

v3.9.0（Phase 6）から、`log.read()` / `log.save()` / `log.load()` を使えば **Claude を介さずに**スクリプト間でデータを受け渡せます。

| 方法 | Claude 介在 | 電源オフ跨ぎ | 用途 |
|------|:-----------:|:-----------:|------|
| **`log.write()` → `log.read()`** | 不要 | ✗（RAM のみ） | 同セッション内のスクリプト間受け渡し |
| **`log.save()` → `log.load()`** | 不要 | ✅（SPIFFS 保存） | 電源オフ跨ぎの状態保存・復元 |
| **`return` → Claude が次スクリプトに組み込む** | 必要 | ✗ | 同セッション内（Claude が仲介） |
| **`return` → Claude が `fs_write` → `fs_read`** | 必要 | ✅ | セッション跨ぎ（Claude が仲介） |

```
【Claude 不要パターン — log.save / load を使う】

スクリプトA（校正スクリプト）
  local cal = ...測定...
  log.write("CAL=" .. cal)
  log.save()        ← SPIFFS /luna_log.txt に保存
  notify.set("完了")
      ↓ （電源オフ・別セッション・時間経過 OK）
スクリプトB（測定スクリプト）
  log.load()        ← /luna_log.txt から復元
  local cal = tonumber(log.read():match("CAL=(.+)")) or 1.0
  local v = tonumber(serial.query(":MEAS?\r\n", 1000)) * cal
  return v
```

---

### C-1. データを渡す方法：どれを選ぶか

| 方法 | 容量 | 用途 | 特徴 |
|------|------|------|------|
| **`log.write()` + `log.read()`** | 最新20行 | スクリプト間の値受け渡し | **Claude 不要**・即時利用可 |
| **`log.save()` + `log.load()`** | 最新20行 | 電源オフ跨ぎの状態保存 | **Claude 不要**・SPIFFS 永続化 |
| **`return`** | 制限なし（大容量可） | 測定結果・配列・CSV | 実行完了後に Claude が取得 |
| **`log.write()` + `return` 併用** | ─ | 長時間処理の全記録 | 最も確実な方法 |

---

### C-2. 測定データを丸ごと渡す：`return` の使い方（★推奨）

スクリプトの最後に `return` を書くと、`lua_status` の `last_result.return` に値が入ります。
**行数制限なし**で大量データを渡せます。

#### シンプルな値を返す

```lua
local v = serial.query(":MEAS:VOLT?\r\n", 2000)
return "電圧: " .. v .. " V"
```

`lua_status` の結果：
```json
{"last_result": {"status":"ok", "return":"電圧: 1.2345 V"}}
```

#### 配列を文字列にまとめて返す（実機確認済み ✅）

```lua
local results = {}
for i = 1, 5 do
    local v = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000)) or 0
    table.insert(results, v)
    hw.delay(1000)
end

local out = ""
for i, v in ipairs(results) do
    out = out .. string.format("[%d]=%.4f ", i, v)
end
return out
```

`return` の内容：
```
[1]=1.2355 [2]=1.2365 [3]=1.2375 [4]=1.2385 [5]=1.2395
```

#### CSV 形式で返す（実機確認済み ✅）

```lua
local csv = "index,voltage,timestamp\n"
for i = 1, 10 do
    local v = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000)) or 0
    csv = csv .. string.format("%d,%.4f,%d\n", i, v, hw.millis())
    hw.delay(1000)
end
return csv
```

`return` の内容：
```
index,voltage,timestamp
1,1.2355,14783456
2,1.2365,14783556
...
```

> Claude はこの CSV をそのまま解析・集計・グラフ化の指示に使えます。

---

### C-3. 変数・配列の中身をその場で確認する

スクリプト実行中または実行後に、特定の変数の状態を確認したい場合は、
`return` または `log.write()` で明示的に出力します。

```lua
-- 変数の状態を一括で返す（実機確認済み ✅）
local x = 42
local name = "測定器A"
local data = {1.234, 5.678, 9.012}
local flag = true

return string.format(
    "x=%d, name=%s, flag=%s, data=[%.3f, %.3f, %.3f], heap=%d",
    x, name, tostring(flag),
    data[1], data[2], data[3],
    hw.free_heap()
)
```

`return` の内容：
```
x=42, name=測定器A, flag=true, data=[1.234, 5.678, 9.012], heap=201800
```

> **Claude への指示例**：
> 「スクリプトを実行して、data 配列の中身を確認して」
> → Claude が `return` の内容から変数を読み取って説明してくれます。

---

### C-4. 長時間処理で両方使う：`log`（進捗）＋ `return`（全データ）

```lua
-- 推奨パターン：log で進捗、return で最終結果
log.write("=== 測定開始 ===")

local csv = "index,voltage\n"
for i = 1, 20 do
    local v = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000)) or 0
    csv = csv .. string.format("%d,%.4f\n", i, v)

    -- 5回ごとに進捗をログに残す
    if i % 5 == 0 then
        log.write(string.format("進捗: %d/20 完了", i))
    end
    hw.delay(1000)
end

log.write("=== 測定完了 ===")
notify.set("測定完了")
return csv    -- 全データは return で渡す
```

| 出力先 | 内容 |
|--------|------|
| `lua_log` | 進捗状況（5回ごとの報告） |
| `return` | 全20件の CSV データ |

---

### C-5. 用途別・最適な方法の選び方

```
【少量のデータ・進捗報告】
  → log.write()（最新20行・実行中も書ける）

【測定結果・配列・大量データ】
  → return（行数制限なし・CSV形式も可）

【変数・配列の中身を確認したい】
  → return string.format(...) で明示的に出力
  → Claude が lua_status から値を読み取る

【長時間処理の完全な記録】
  → log（進捗） + return（全データ）の併用
```

---

## 付録D　実機サンプル：高橋製作所 Temma2 天体望遠鏡制御

LUNA システムを使って高橋製作所の天体望遠鏡マウント **Temma2** を制御するサンプルです。
RS-232C シリアル通信経由で現在位置取得・自動導入（Goto）を行います。

> **対象機種**: Temma PC / Temma2 / Temma2 Jr. / EM-10/200/400/500 Temma2 系列

---

### D-1. 接続構成

Temma2 は RS-232C (±12V) のため、ESP32 (3.3V TTL) との間に **MAX3232 変換モジュール**が必要です。

```
Temma2                  MAX3232              ESP32
Mini-DIN4               モジュール
  ピン1 (TXD) ────── R1IN → R1OUT ──── GPIO16 (RX2)
  ピン2 (RXD) ────── T1IN ← T1OUT ──── GPIO17 (TX2)
  ピン3 (GND) ────── GND  ──────────── GND
  ピン4       ─ 未接続
```

| 通信設定 | 値 |
|---------|-----|
| ボーレート | **19200 bps** |
| データビット | 8 |
| パリティ | なし (N) |
| ストップビット | 1 |
| コマンド終端 | CR (`\r`) |

> ⚠️ Temma2 本体の **Computer Standby スイッチを ON** にしないと RS-232C 通信が無効です。

---

### D-2. 座標フォーマット

Temma2 は ASCII 10進数形式の独自座標フォーマットを使用します。

#### 赤経 (RA)：`HHMMSSs`（7桁）

| フィールド | 桁数 | 内容 |
|-----------|------|------|
| HH | 2 | 時（00〜23） |
| MM | 2 | 分（00〜59） |
| SSs | 3 | 秒×10（17.3秒 → `173`） |

例: `0535173` = 05h 35m 17.3s

#### 赤緯 (Dec)：`SDDMMSS`（7桁）

| フィールド | 桁数 | 内容 |
|-----------|------|------|
| S | 1 | 符号（`+` または `-`） |
| DD | 2 | 度（00〜90） |
| MM | 2 | 分（00〜59） |
| SS | 2 | 秒（00〜59） |

例: `-052328` = -05° 23' 28"

---

### D-3. スクリプト① temma_q ── 現在位置取得（疎通確認）

接続確認に最初に実行します。`Q\r` コマンド1本で現在位置を取得・表示します。

```lua
-- temma_q: Temma2 現在位置取得（疎通確認用）
-- Q\r → <HHMMSSs><SDDMMSS> 14文字
local resp = serial.query("Q\r", 2000)

if resp and #resp >= 14 then
  local ra_h  = resp:sub(1, 2)
  local ra_m  = resp:sub(3, 4)
  local ra_ss = tonumber(resp:sub(5, 7)) or 0   -- 秒×10
  local dec   = resp:sub(8, 14)

  local ra_s  = ra_ss / 10.0
  log.write(string.format("RA  = %sh %sm %.1fs", ra_h, ra_m, ra_s))
  log.write(string.format("Dec = %s", dec))
  log.write("RAW = " .. resp)
else
  log.write("ERROR: " .. (resp or "応答なし"))
end

notify.set("完了")
return resp
```

**実行と確認:**
```
Claude の操作:
  lua_save("temma_q", <上記コード>)
  lua_run("temma_q")
  lua_log
    → RA  = 10h 30m 45.6s
    → Dec = -321030
    → RAW = 1030456-321030
```

---

### D-4. スクリプト② temma_goto ── 自動導入（Goto）

指定した天体座標へマウントを自動導入します。`ra_target` と `dec_target` を書き換えて使用してください。

```lua
-- temma_goto: Temma2 自動導入スクリプト（LUNA v3.9.0）
--
-- ターゲット座標を変更して使用してください。
-- 例: M42（オリオン大星雲）RA 05h35m17.3s  Dec -05°23'28"
--
-- RA  フォーマット: HHMMSSs （秒の1/10まで。例 17.3s → 173）
-- Dec フォーマット: SDDMMSS （S=符号 + or -）
--
-- ⚠️ 初回使用時は mount 本体の Computer Standby SW を ON にすること

local ra_target  = "0535173"   -- 05h 35m 17.3s  ← ここを変える
local dec_target = "-052328"   -- -05° 23' 28"   ← ここを変える

-- Step1: 現在位置取得（通信確認）
log.write("=== temma_goto start ===")
local pos = serial.query("Q\r", 2000)
if pos and #pos >= 14 then
  log.write("現在位置 RAW = " .. pos)
else
  log.write("WARNING: Q コマンド応答なし -> " .. tostring(pos))
end

-- Step2: Goto コマンド送信
-- G<HHMMSSs><SDDMMSS>\r
local cmd = "G" .. ra_target .. dec_target .. "\r"
log.write("Goto cmd = G" .. ra_target .. dec_target)
local resp = serial.query(cmd, 3000)

-- 応答は ">" または無応答（どちらも正常な場合あり）
if resp and #resp > 0 then
  log.write("応答 = " .. resp)
else
  log.write("応答なし（導入開始の場合あり）")
end

log.write("=== temma_goto end ===")
notify.set("完了")
return {current=pos, goto_resp=resp}
```

**実行と確認:**
```
Claude の操作:
  lua_save("temma_goto", <上記コード>)  ← ra_target / dec_target を書き換えてから
  lua_run("temma_goto")
  lua_log
    → === temma_goto start ===
    → 現在位置 RAW = 1030456-321030
    → Goto cmd = G0535173-052328
    → 応答なし（導入開始の場合あり）
    → === temma_goto end ===
```

---

### D-4a. 最小 Goto スクリプト ── RA/Dec 文字列を直接指定

疎通確認（Q コマンド）を省いた、Goto コマンドのみの最小構成です。プロトコルを学ぶ最初の一歩として最適です。

```lua
-- Temma2 Goto 最小スクリプト
-- RA/Dec を Temma 形式の文字列で直接指定する版
--
-- RA  形式: HHMMSSs  （秒 × 10。例: 45.6s → 456）
-- Dec 形式: SDDMMSS  （S = 符号 + or -）

local ra_str  = "1030456"   -- 10h 30m 45.6s
local dec_str = "-321030"   -- -32° 10' 30"

-- Goto コマンド送信: G<HHMMSSs><SDDMMSS>\r
local cmd = "G" .. ra_str .. dec_str .. "\r"
log.write("Send: " .. cmd:gsub("\r", "\\r"))

local resp = serial.query(cmd, 5000)   -- 5秒待ち
if resp and resp ~= "" then
    log.write("Resp: " .. resp)
else
    log.write("Goto 送信完了（応答なしは正常の場合あり）")
end

notify.set("完了")
```

> 💡 **Temma2 の応答について**: G コマンドに対して応答なしまたは `>` が返る実装があります。どちらも正常です。導入開始は応答ではなくマウントの動作音で確認してください。

---

### D-4b. 座標変換付き Goto スクリプト ── 時間・度数で入力

RA を時間単位（例: 10.5127h）、Dec を度単位（例: -32.175°）で指定すると、内部で Temma 形式に変換して Goto するスクリプトです。天文計算ソフトや Stellarium から座標をコピーしてそのまま使えます。

```lua
-- Temma2 座標変換付き Goto スクリプト
-- ra_hours : RA を時間単位で指定  （例: 10h30m45.6s ≒ 10.512666h）
-- dec_deg  : Dec を度単位で指定   （例: -32°10'30" ≒ -32.175°）

-- RA（時間単位）→ Temma HHMMSSs 形式
-- 整数演算で浮動小数点誤差を回避
local function ra_to_temma(ra_hours)
    local s10_total = math.floor(ra_hours * 36000 + 0.5)  -- 0.1秒単位の整数
    if s10_total < 0       then s10_total = 0 end
    if s10_total >= 864000 then s10_total = 863999 end
    local h   = math.floor(s10_total / 36000)
    local m   = math.floor((s10_total - h * 36000) / 600)
    local s10 = s10_total - h * 36000 - m * 600
    return string.format("%02d%02d%03d", h, m, s10)
end

-- Dec（度単位）→ Temma SDDMMSS 形式
-- 整数演算で浮動小数点誤差を回避
local function dec_to_temma(dec_deg)
    local sign = "+"
    if dec_deg < 0 then sign = "-"; dec_deg = -dec_deg end
    if dec_deg > 90.0 then dec_deg = 90.0 end
    local arcsec = math.floor(dec_deg * 3600 + 0.5)  -- 秒単位の整数
    local d = math.floor(arcsec / 3600)
    local m = math.floor((arcsec - d * 3600) / 60)
    local s = arcsec - d * 3600 - m * 60
    return string.format("%s%02d%02d%02d", sign, d, m, s)
end

-- ★ここをユーザーが変更する★
local ra_hours = 10.512666   -- 例: 10h 30m 45.6s
local dec_deg  = -32.175     -- 例: -32° 10' 30"

-- 変換
local ra_str  = ra_to_temma(ra_hours)
local dec_str = dec_to_temma(dec_deg)

log.write("RA  (Temma): " .. ra_str)
log.write("Dec (Temma): " .. dec_str)

-- Goto コマンド送信
local cmd = "G" .. ra_str .. dec_str .. "\r"
log.write("Send: " .. cmd:gsub("\r", "\\r"))

local resp = serial.query(cmd, 5000)
if resp and resp ~= "" then
    log.write("Resp: " .. resp)
else
    log.write("Goto 送信完了（応答なしは正常の場合あり）")
end

notify.set("完了")
return {ra=ra_str, dec=dec_str, resp=resp}
```

**変換例の確認:**

| 入力 | 変換後 |
|------|--------|
| `ra_hours = 10.512666` | `"1030456"` （10h 30m 45.6s） |
| `dec_deg  = -32.175` | `"-321030"` （-32° 10′ 30″） |
| `ra_hours = 5.588139` | `"0535173"` （M42: 5h 35m 17.3s） |
| `dec_deg  = -5.391` | `"-052328"` （M42: -5° 23′ 28″） |

> 💡 **Claude に任せる場合**: 「M42 に導入して」と伝えると Claude が座標を調べて `ra_hours` / `dec_deg` を計算し、このスクリプトを `lua_save` + `lua_run` で実行します。

---

### D-5. よく使う天体の座標早見表

`ra_target` / `dec_target` をそのまま貼り付けて使えます。

| 天体 | `ra_target` | `dec_target` | 備考 |
|------|------------|--------------|------|
| M42 オリオン大星雲 | `"0535173"` | `"-052328"` | 冬の定番 |
| M45 プレアデス星団 | `"0347000"` | `"+240600"` | 冬の定番 |
| M31 アンドロメダ銀河 | `"0042443"` | `"+411609"` | 秋の定番 |
| M13 ヘルクレス球状星団 | `"1641420"` | `"+362755"` | 夏の定番 |
| M57 こと座リング星雲 | `"1853570"` | `"+330200"` | 夏の定番 |
| 北極星 (Polaris) | `"0231480"` | `"+892542"` | 極軸確認用 |

> ⚠️ 惑星・月は日時により座標が変わります。天文計算ソフトや Stellarium で最新座標を取得してください。

---

### D-6. 初期化コマンド（必要な場合）

マウントを初めて使用する際や、電源再投入後に導入精度が出ない場合は初期化が必要です。
Claude に「Temma2 を初期化して」と指示するか、以下のコマンドを `serial_query` で順番に送ります。

```
1. 緯度設定:  I+354600\r   （例: 北緯 35°46'00"）
2. 時刻設定:  T203045\r    （例: 20:30:45 JST）
3. 設定確定:  E\r
```

> ⚠️ 各コマンドの正確な応答形式は実機により異なる場合があります。

---

### D-7. トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| `temma_q` が "応答なし" | Computer Standby SW が OFF | マウント本体のスイッチを ON に |
| 応答が文字化け | ボーレート不一致 | LUNA ファームウェアは 19200bps 固定です。他の機器が接続されていないか確認してください |
| `temma_q` が応答するが `temma_goto` で動かない | 初期化未実施 | D-6 の初期化コマンドを実行 |
| 起動時に文字化け | ESP32 起動メッセージが Temma2 に流入 | ESP32 を先に起動し、その後 Temma2 電源 ON |

> **注意**: `serial_config` ツールによるボーレート変更は Pro Edition の機能です。LUNA（free 版）は **19200bps 固定**のため、ファームウェアの設定変更は不要です。

---

### D-8. luna.run() を使った自律観測シーケンス（v3.9.1）

`luna.run()` を使うと、Claude の介在なしにスクリプトが次のスクリプトを自動起動します。
以下は **疎通確認 → 自動導入 → 導入確認** の3段階を完全自律で実行する例です。

```
Claude が lua_run("temma_init") を呼ぶだけ
  ↓
temma_init → temma_goto → temma_confirm
              （自動）        （自動）
  ↓
完了後に notify.set("完了") → Claude に通知
```

#### スクリプト① temma_init（疎通確認・パラメータ設定）

```lua
-- temma_init: 疎通確認 + 導入先座標を log に保存
local resp = serial.query("Q\r", 2000)
if not resp or resp == "" then
    log.write("ERROR: Temma2 応答なし")
    log.write("CHECK: Computer Standby SW を ON にしてください")
    notify.set("エラー")
    return
end
log.write("OK: Temma2 応答確認")
log.write("POS=" .. resp)

-- 導入先座標を設定（ここを書き換える）
log.write("TARGET_RA=0535173")    -- M42: 05h 35m 17.3s
log.write("TARGET_DEC=-052328")   -- M42: -05° 23′ 28″

log.save()                         -- SPIFFS に保存
luna.run("temma_goto")             -- 次のスクリプトを自動起動
```

#### スクリプト② temma_goto（自動導入）

```lua
-- temma_goto: log から座標を読んで Goto 実行
log.load()

-- log から TARGET_RA / TARGET_DEC を探す
local ra, dec
for i = 0, 19 do
    local line = log.read(i)
    if not line then break end
    ra  = ra  or line:match("^TARGET_RA=(.+)")
    dec = dec or line:match("^TARGET_DEC=(.+)")
end

if not ra or not dec then
    log.write("ERROR: 座標が見つかりません")
    notify.set("エラー")
    return
end

local cmd = "G" .. ra .. dec .. "\r"
log.write("GOTO: " .. cmd:gsub("\r","\\r"))
local resp = serial.query(cmd, 5000)
log.write("RESP=" .. (resp or "なし"))
log.save()

luna.run("temma_confirm")          -- 導入確認スクリプトへ
```

#### スクリプト③ temma_confirm（導入確認）

```lua
-- temma_confirm: 導入完了後の位置確認
hw.delay(3000)                     -- 導入完了を少し待つ

local pos = serial.query("Q\r", 2000)
if pos and pos ~= "" then
    log.write("ARRIVED=" .. pos)
else
    log.write("WARN: 導入後の位置確認できず")
end
log.save()
notify.set("完了")                 -- Claude に完了通知
```

#### 使い方

```
1. 3本のスクリプトを lua_save で保存:
   lua_save("temma_init",    <上記コード①>)
   lua_save("temma_goto",    <上記コード②>)
   lua_save("temma_confirm", <上記コード③>)

2. Claude から最初の1本を起動:
   lua_run("temma_init")

3. あとは ESP32 が自律実行。完了後に lua_log で結果を確認。
```

> 💡 **チェーン上限**: `luna.run()` は最大 10 本まで連鎖できます（`LUA_MAX_CHAIN=10`）。
> 上限に達した場合は log に `"chain limit reached"` が記録され、それ以上の自動起動は行われません。

---

## ライセンス

LUNA システム（AiBridgeMCP v3.9.1 LUNA Edition）は **MIT ライセンス** で公開しています。
商用・非商用を問わず自由にご利用いただけます。

---

*AiBridgeMCP プロジェクト: https://github.com/OnStepNinja/AiBridgeMCP*
*作者: Nishioka Sadahiko*
