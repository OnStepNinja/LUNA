# LUNA ユーザーマニュアル
# 完全版リファレンス

**ブランド名**: LUNA（AiBridgeMCP v3.9.1）
**対象**: RS-232C機器を制御したいユーザー・技術者
**リビジョン**: r1.0
**最終更新**: 2026-03-19
**作成日**: 2026-03-19

---

## 目次

1. [システム概要](#1-システム概要)
2. [初期設定（APモード）](#2-初期設定apモード)
3. [ハードウェア構成](#3-ハードウェア構成)
4. [Claude Desktop の設定](#4-claude-desktop-の設定)
5. [MCPツール リファレンス](#5-mcpツール-リファレンス)
   - 5.1 シリアルブリッジ系（機器制御）
   - 5.2 Lua自律制御系
   - 5.3 通知系
   - 5.4 システム情報系
   - 5.5 ファイルシステム系（SPIFFS）
6. [Lua自律制御](#6-lua自律制御)
   - 6.1 Lua APIリファレンス
   - 6.2 スクリプトチェーン（luna.run）
7. [リセットモード](#7-リセットモード)
8. [トラブルシューティング](#8-トラブルシューティング)
9. [仕様一覧](#9-仕様一覧)

---

## 1. システム概要

### LUNA とは

ESP32 マイコンを使って、RS-232Cポートを持つあらゆる機器を Claude AI で制御するブリッジシステムです。
さらに、Luaスクリプトが Claude 不在でも自律的に動き続ける自律ゲートウェイ機能を備えます。

```
【ダイレクト制御モード】
Claude Desktop ── WiFi/MCP ── LUNA(ESP32) ── RS-232C ── 機器
                                                         （計測器・産業機器・天文機器等）

【Lua自律制御モード】
Claude Desktop ── Luaスクリプトを設計・保存
                        ↓
                  LUNA が単独で動き続ける（Claude不要）
                        ↓
                  完了・異常時のみ Claude に通知
```

### できること

| 機能 | 説明 |
|------|------|
| **機器へのコマンド送信** | `serial_query` で送信→応答を1ステップで取得 |
| **機器からのデータ受信** | `serial_read` で機器が自発的に送るデータを受信 |
| **Lua スクリプト実行** | ESP32 上で Lua 5.1 スクリプトを非同期実行 |
| **スクリプト永続保存** | `lua_load` で SPIFFS に保存、電源断後も有効 |
| **自律スクリプトチェーン** | `luna.run()` で複数スクリプトを順番に自律実行 |
| **Claude への非同期通知** | `notify.set()` でスクリプトから Claude に通知 |
| **LUNA 使い方の自動通知** | 接続時に `prompts/get luna_guide` が自動実行 |

### 対応機器（例）

RS-232Cポートを持つ機器であれば原則として何でも制御できます。

| 分野 | 機器例 |
|------|-------|
| **産業・製造** | NC工作機械、PLC、インバータ、ロボットコントローラ |
| **計測・研究** | マルチメータ、オシロスコープ、スペクトラムアナライザ、電源装置 |
| **天文** | Takahashi Temma2、Celestron 等の望遠鏡マウント |
| **放送・AV** | スイッチャー、VTRコントローラ |

---

## 2. 初期設定（APモード）

### なぜ APモードがあるのか

LUNA は購入後または初期化後、**必ず APモードで起動**します。
APモードとは、本体自身が WiFi アクセスポイントになる仕組みです。

1. **初回の WiFi 設定を行うため**
2. **DHCP で割り当てられた IP アドレスを確認するため**
3. **ルーターを変更したときに再設定できるようにするため**

AP は常時有効なので、どんな状態でも必ず本体に接続できます。

### 初回設定の流れ

#### ① AP に接続する

Wi-Fi 設定で以下を選択します：

```
AiBridgeMCP_Ai-{LocationName}
```

ブラウザで開きます：

```
http://192.168.4.1
```

※ AP 接続中はインターネットは使えません。

#### ② 自宅 WiFi を設定する

1. File Manager を開く
2. 自宅ルーターの SSID と password を入力
3. Save を押す
4. 本体を再起動する

#### ③ DHCP アドレスを確認する

再起動後も AP は有効です。再度 AP に接続し `http://192.168.4.1` を開くと、

```
STA Address: 192.168.x.xxx (DHCP)
```

と表示されます。これが Claude Desktop に登録する IP アドレスです。

### File Manager の開き方

| 方法 | 操作 | 有効時間 |
|------|------|---------|
| **電源投入時** | GPIO39 を LOW にして電源 ON | 30分間 |
| **運用中** | GPIO39 を一時的に LOW | 10分間 |

---

## 3. ハードウェア構成

### 必要な機材

| 品目 | 仕様 | 備考 |
|------|------|------|
| ESP32 開発ボード | ESP32-WROOM-32 推奨 | LUNA ファームウェア書き込み済み |
| RS-232C 変換モジュール | MAX3232 等 | 3.3V ↔ RS-232C 変換（必須） |
| RS-232C ケーブル | D-sub 9ピンまたは 25ピン | 機器に合わせて選択 |
| WiFi ルーター | 2.4GHz 対応 | インターネット接続済み |
| PC（Claude Desktop 用） | Windows / Mac | Claude Desktop インストール済み |

### GPIO ピン配置

| GPIO | 機能 | 備考 |
|------|------|------|
| GPIO 16 | UART2 RX | 機器 → ESP32（RS-232C経由） |
| GPIO 17 | UART2 TX | ESP32 → 機器（RS-232C経由） |
| GPIO 27 | LED | 動作表示 LED |
| GPIO 34 | Reset Mode Select | HIGH=ソフトリセット / LOW=ハードリセット |
| GPIO 39 | File Manager Enable | LOW で File Manager 開放 |
| GPIO 25 | File Manager LED | File Manager 動作中表示 |

### 配線

```
【LUNA】                    【RS-232C機器】

  TX2 (GPIO17) ──────────────► RXD
  RX2 (GPIO16) ◄────────────── TXD
  GND          ──────────────── GND

  ※ MAX3232 等の RS-232C レベル変換回路を経由して接続
  ※ フロー制御が必要な機器は RTS/CTS も接続
```

### ボーレート設定

| エディション | ボーレート |
|-------------|-----------|
| **LUNA（Free版）** | 9600 bps 固定（8N1） |
| **SerialBridge Pro（v4.2.0）** | `serial_config` ツールで変更可能 |

機器のボーレートが 9600bps 以外の場合は Pro 版をご検討ください。

### レベル変換回路（MAX3232）

ESP32 の GPIO は 3.3V、RS-232C は ±12V 系です。必ずレベル変換が必要です。

```
【推奨 IC】
  MAX3232（3.3V動作、+/-15kV ESD保護）
  MAX232（5V動作、外付けコンデンサ4個必要）

【基本接続】
  ESP32 TX2(GPIO17) → MAX3232 T1IN → T1OUT → 機器 RXD
  ESP32 RX2(GPIO16) → MAX3232 R1OUT ← R1IN ← 機器 TXD
  ESP32 GND         → MAX3232 GND   ────────── 機器 GND
```

---

## 4. Claude Desktop の設定

### MCP サーバーの登録

Claude Desktop の設定ファイル（`claude_desktop_config.json`）に追加します。

```json
{
  "mcpServers": {
    "LUNA-Lab": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "http://192.168.x.xxx/mcp"
      ]
    }
  }
}
```

`192.168.x.xxx` の部分を「② で確認した IP アドレス」に書き換えてください。

> **注意**: 旧バージョンの `/mcp/sse` は使用しません。LUNA は `/mcp`（Streamable HTTP）を使用します。

### 接続確認

Claude Desktop を起動し、LUNA ツール（`get_system_info` 等）が表示されれば接続成功です。

接続時に LUNA が `prompts/get luna_guide` を自動実行し、Claude が LUNA の使い方を自動的に把握します。

### 機器マニュアルの渡し方

Claude Desktop のチャット画面で機器のマニュアルを渡します。

```
以下の機器を LUNA 経由の RS-232C で制御します。

【機器情報】
  機器名: ○○社製 ○○モデル
  通信設定: 9600bps, 8N1
  コマンド体系: SCPI準拠（または独自コマンド）

【マニュアル】
  [PDF を添付、またはコマンド一覧をテキストで貼り付け]

上記マニュアルを理解した上で、私の指示に従って機器を制御してください。
```

**ポイント**: コマンド一覧、応答フォーマット、初期化手順が含まれていると最も効果的です。

### IP アドレスが変わった場合

DHCP 環境では IP アドレスが変わる場合があります。

1. AP モード（`http://192.168.4.1`）で新しい IP アドレスを確認
2. `claude_desktop_config.json` の IP を書き換えて保存
3. Claude Desktop を再起動

**恒久対策**: ルーターの「DHCP予約（静的割当）」で LUNA の IP を固定することを推奨します。

---

## 5. MCPツール リファレンス

Claude Desktop から LUNA に対して使えるツールの完全一覧です。

---

### 5.1 シリアルブリッジ系（機器制御の主力）

#### `serial_query`（最重要ツール）

コマンドを送信し、指定時間待機してから応答を受信します。

| 項目 | 内容 |
|------|------|
| **パラメータ** | `content`（必須）: 送信するコマンド文字列 |
| | `timeout_ms`（省略可）: 応答待ち時間（ミリ秒）。デフォルト 500ms |
| **戻り値** | 機器からの応答文字列 |
| **使いどき** | コマンドを送ったら機器が応答を返す、典型的な RS-232C 通信 |

```
使用例:
  serial_query("MEAS:VOLT:DC?\r\n")
    → "+1.2345E+00\r\n" を返す

  serial_query("*IDN?\r\n", timeout_ms=1000)
    → "HEWLETT-PACKARD,34401A,..." を返す
```

**ポイント**: コマンド末尾の改行コード（`\r\n` または `\r`）は機器の仕様に合わせてください。

---

#### `serial_write`

シリアルポートにデータを**送信するだけ**のツールです。

| 項目 | 内容 |
|------|------|
| **パラメータ** | `content`（必須）: 送信する文字列 |
| **戻り値** | 送信バイト数 |
| **使いどき** | 応答不要の一方向送信（設定コマンド送信、データ転送など） |

---

#### `serial_read`

シリアルポートの受信バッファにたまっているデータを**読み出すだけ**のツールです。

| 項目 | 内容 |
|------|------|
| **パラメータ** | なし |
| **戻り値** | 受信バッファの内容（データがなければ空文字） |
| **使いどき** | 機器側から自発的に送られてくるデータを受け取る場合 |

**注意**: 通常は `serial_query` を使う方がシンプルです。

---

#### `serial_config`（SerialBridge Pro版のみ）

シリアル通信のボーレートとパリティを変更します。
**LUNA Free版では使用できません（エラーを返します）。**

| 項目 | 内容 |
|------|------|
| **パラメータ** | `baud`（必須）: ボーレート |
| | `parity`（省略可）: `N`=なし（デフォルト）、`E`=偶数、`O`=奇数 |
| **対応ボーレート** | 300 / 600 / 1200 / 2400 / 4800 / 9600 / 19200 / 38400 / 115200 |

---

### 5.2 Lua自律制御系（LUNA 新機能）

> **📖 別冊参照**: Lua プログラミングの詳細（API仕様・サンプルコード・言語ガイド）は
> **`LUNA_Lua_Guide.md`** を参照してください。

#### `lua_exec`

Lua コードをその場で即時実行します。テストや 1 回限りの処理に使います。

| 項目 | 内容 |
|------|------|
| **パラメータ** | `code`（必須）: Lua 5.1 のソースコード |
| **戻り値** | `{"status":"started"}` （非同期・即返） |
| **使いどき** | 動作確認・テスト・簡単な 1 回実行 |

```lua
-- 使用例: 機器 ID を取得
lua_exec('local r = serial.query("*IDN?\\r\\n", 1000); log.write(r)')
```

実行結果は `lua_status` または `lua_log` で取得します。

---

#### `lua_load`

Lua スクリプトを SPIFFS に保存します。電源断後も残ります。

| 項目 | 内容 |
|------|------|
| **パラメータ** | `name`（必須）: スクリプト名（英数字・`_`・`-` のみ、22文字以内） |
| | `code`（必須）: Lua 5.1 のソースコード |
| **保存先** | `/lua_{name}.lua` |

---

#### `lua_run`

保存済みスクリプトを実行します。

| 項目 | 内容 |
|------|------|
| **パラメータ** | `name`（必須）: `lua_load` で保存したスクリプト名 |
| **戻り値** | `{"status":"started"}` （非同期・即返） |
| **使いどき** | 定期実行・本番運用 |

---

#### `lua_status`

Lua VM の実行状態を確認します。

| 項目 | 内容 |
|------|------|
| **戻り値** | `state`（running/idle）、`last_result`、`free_heap`、`uptime` |
| | `chain_index`、`chain_script`（`luna.run()` チェーン進捗） |

---

#### `lua_stop`

実行中の Lua タスクを強制停止します。

---

#### `lua_log`

スクリプト内の `log.write()` で書かれたログを取得します（直近 20 行のリングバッファ）。

---

#### `lua_list`

SPIFFS に保存されている Lua スクリプト一覧を表示します。

---

### 5.3 通知系

#### `check_notify`

Lua スクリプトの `notify.set()` による通知フラグを確認します。

| 項目 | 内容 |
|------|------|
| **パラメータ** | なし |
| **戻り値** | `{"flag":0}` または `{"flag":1, "message":"..."}` |

**使い方**: Lua スクリプト内で `notify.set("メッセージ")` を呼ぶと、Claude が `check_notify` で検知できます。フラグは読み取り後に自動リセットされます。

---

### 5.4 システム情報系

| ツール名 | 機能 | パラメータ |
|---------|------|-----------|
| `get_system_info` | ESP32 のシステム情報取得（location・チップ型番・ヒープ・SPIFFS 使用量・稼働時間） | なし |
| `get_network_status` | WiFi・ネットワーク状態取得（location・AP SSID・STA IP・RSSI） | なし |
| `get_license_info` | ライセンス・エディション・著者情報取得 | なし |
| `hard_reset` | ESP32 を再起動（1秒後に `ESP.restart()` 実行） | なし |

---

### 5.5 ファイルシステム系（SPIFFS）

| ツール名 | 機能 | パラメータ |
|---------|------|-----------|
| `fs_list` | SPIFFS のファイル一覧取得 | なし |
| `fs_read` | ファイル読み込み（最大 4KB） | `path`（必須） |
| `fs_write` | ファイル書き込み（最大 1KB・上書き） | `path`、`content`（必須） |
| `fs_append` | ファイル追記（最大 1KB） | `path`、`content`（必須） |
| `fs_delete` | ファイル削除 | `path`（必須） |

> **注意**: `fs_*` は Claude（MCP経由）から呼ぶツールです。Lua スクリプト内からは呼べません。
> Lua から SPIFFS に書くには `log.save()` を使用してください。

---

### ツール選択チートシート

```
【機器にコマンドを送って応答を受け取りたい】
  → serial_query（timeout_ms を機器に合わせて調整）

【プログラムや設定を一方的に流し込みたい】
  → serial_write（1行ずつ繰り返す）

【機器が自発的に送ってくるデータを受け取りたい】
  → serial_read

【Lua スクリプトをテストしたい】
  → lua_exec

【Lua スクリプトを本番保存・実行したい】
  → lua_load → lua_run

【Lua スクリプトの実行状況を確認したい】
  → lua_status

【Lua スクリプトのログを確認したい】
  → lua_log

【Lua スクリプトを途中で止めたい】
  → lua_stop

【保存済みスクリプト一覧を確認したい】
  → lua_list

【Lua スクリプトから完了・異常を Claude に通知したい】
  → スクリプト内で notify.set() → Claude が check_notify で検知

【LUNA が正常動作しているか確認したい】
  → get_system_info または get_network_status

【log.save() で保存したデータを Claude から確認したい】
  → fs_read("/luna_log.txt")
```

---

## 6. Lua自律制御

> **📖 別冊参照**: Lua プログラミングの詳細（API仕様・サンプルコード・言語ガイド）は
> **`LUNA_Lua_Guide.md`** を参照してください。

Claude 不在でも ESP32 が単独で動き続ける自律制御機能です。

### 基本的な使い方

```
① Claude にスクリプトの仕様を伝える
  「30秒ごとに電圧を測定し、1.0V未満または1.5V超なら通知するスクリプトを作って」

② Claude が Lua スクリプトを生成・保存・実行
  lua_load("voltage_monitor", <スクリプト>) → SPIFFS に保存
  lua_run("voltage_monitor")               → 実行開始

③ Claude は離席してOK（ESP32 が単独で動き続ける）

④ 異常発生時
  notify.set() が発火
  Claude が check_notify で検知
  「電圧異常: 0.8234V」と報告
```

### Lua スクリプト例（電圧監視）

```lua
-- 電圧監視スクリプト（voltage_monitor）
for i = 1, 120 do
  local res = serial.query("MEAS:VOLT:DC?\r\n", 1000)
  local v = tonumber(res)
  if v then
    log.write(string.format("%.4fV", v))
    if v < 1.0 or v > 1.5 then
      notify.set("電圧異常: " .. string.format("%.4f", v) .. "V")
      break
    end
  end
  hw.delay(30000)  -- 30秒待機
end
```

---

### 6.1 Lua APIリファレンス

Lua スクリプトから ESP32 の機能を使うための API です。

#### serial モジュール

| API | 説明 |
|-----|------|
| `serial.query(cmd, timeout_ms)` | コマンド送信→待機→受信。主力 API |
| `serial.write(data)` | 送信のみ |
| `serial.read()` | 受信バッファ読み出し |

**注意**: serial モジュールとシリアル系 MCP ツール（`serial_query` 等）はシリアルポートを共有します。Lua 実行中は MCP 側からの同時使用を避けてください。

#### log モジュール

| API | 説明 |
|-----|------|
| `log.write(msg)` | ログバッファに書き込む（Claude が `lua_log` で読む） |
| `log.read([n])` | ログバッファを読む（スクリプト間データ受け渡し） |
| `log.save()` | ログを `/luna_log.txt` に SPIFFS 保存（**ループ内での呼び出し禁止**） |
| `log.load()` | SPIFFS から読み込みバッファに展開（呼び出し時にバッファをクリア） |

**log モジュールの 2 つの役割**:

| 用途 | 操作 |
|------|------|
| スクリプト → Claude | `log.write()` で書く → Claude が `lua_log` で読む |
| スクリプト → スクリプト | `log.write()` + `log.save()` → 次スクリプトが `log.load()` で読む |

#### hw モジュール

| API | 説明 |
|-----|------|
| `hw.delay(ms)` | 指定ミリ秒待機 |
| `hw.millis()` | 起動からの経過時間（ms） |
| `hw.reset()` | ESP32 を再起動 |

#### notify モジュール

| API | 説明 |
|-----|------|
| `notify.set(msg)` | Claude への通知メッセージをセット。`check_notify` で検知される |

#### luna モジュール

| API | 説明 |
|-----|------|
| `luna.run(name)` | 次スクリプトをキューして現在スクリプトを終了（チェーン最大 10 本） |

---

### 6.2 スクリプトチェーン（luna.run）

`luna.run(name)` を使うと、スクリプトが次のスクリプトを自律起動できます。
これにより「初期化→測定→レポート」などの複数ステップを Claude なしで完結できます。

```lua
-- スクリプト1: init（初期化）
serial.query("*RST\r\n", 2000)
serial.query("CONF:VOLT:DC\r\n", 500)
log.write("init_ok")
luna.run("measure")   -- 次のスクリプトを自律起動して終了
```

```lua
-- スクリプト2: measure（測定）
local results = {}
for i = 1, 10 do
  local r = serial.query("MEAS:VOLT:DC?\r\n", 1000)
  table.insert(results, r)
  hw.delay(1000)
end
log.write(table.concat(results, ","))
log.save()            -- SPIFFS に保存（次スクリプトへ引き渡し）
luna.run("report")    -- 次のスクリプトへ
```

```lua
-- スクリプト3: report（集計・通知）
log.load()            -- 測定データを受け取る（バッファをクリアしてから読込）
local data = log.read()
-- （集計処理）
notify.set("測定完了: " .. data)
```

**チェーン仕様**:

| 項目 | 仕様 |
|------|------|
| 最大チェーン数 | 10 本 |
| 方式 | 「キューして終了」（現スクリプト終了後に次を起動） |
| 進捗確認 | `lua_status` の `chain_index` / `chain_script` で確認 |

---

## 7. リセットモード

### 2 種類のリセット

#### ソフトリセット

> **「ESP32 は動かしたまま、MCP 接続だけをやり直す」**

- ESP32 は動作し続ける
- WiFi 接続はそのまま維持
- SPIFFS データは保持される
- MCP 接続が切断され、Claude Desktop が自動的に再接続する

**いつ発動するか**: AI との通信が 5分間（300秒）途絶え、かつ **GPIO34 が HIGH** のとき自動発動。

#### ハードリセット

> **「ESP32 を完全に再起動する。電源を入れ直すのと同じ効果」**

- WiFi は再接続（SSID / パスワードは SPIFFS から再読込）
- SPIFFS 上のデータ（WiFi設定・Lua スクリプト等）は保持される
- RAM の内容（実行中 Lua タスク等）はすべてクリア
- 再起動後に MCP サーバーが自動起動

**ハードリセットの 3 つのトリガー**:

| 方法 | 操作 | 遅延 |
|------|------|------|
| MCP ツール `hard_reset` | Claude が呼び出し | 1秒 |
| SSE タイムアウト（GPIO34=LOW） | 自動発動 | 1秒 |

### GPIO34 によるリセットモード設定

AI との通信が 5分間途絶えた場合の動作を GPIO34 で切り替えます。

| GPIO34 の状態 | リセット種類 |
|--------------|------------|
| HIGH（プルアップ / オープン） | ソフトリセット（デフォルト） |
| LOW（GND に接続） | ハードリセット |

### ソフト / ハードリセット 比較表

| 項目 | ソフトリセット | ハードリセット |
|------|-------------|-------------|
| ESP32 再起動 | なし | あり |
| WiFi 接続 | 維持 | 再接続（3〜10秒） |
| 実行中 Lua タスク | 継続 | **停止** |
| SPIFFS データ | 保持 | 保持 |
| MCP 接続 | 切断→自動再接続 | 切断→再起動後に再接続 |

### 長期無人運転の推奨設定

天文台・観測所・工場設備など、誰も操作しない状態で長時間動かし続ける場合：

```
GPIO34 = LOW（ハードリセットモード）推奨

理由:
  通信が 5分間途絶える（ネットワーク瞬断・AI セッション切れ）
        ↓
  ハードリセット発動（ESP32 完全再起動）
        ↓
  WiFi 再接続 → MCP サーバー再起動
        ↓
  自動復帰

メモリリーク・スタック破壊なども再起動でリセットされるため、
長時間運用の安定性が向上します。
```

---

## 8. トラブルシューティング

### 通信できない場合

```
確認項目:
  □ ボーレートが機器と一致しているか（LUNA Free版は 9600 固定）
  □ クロス / ストレートケーブルの選択は正しいか
  □ GND は接続されているか
  □ レベル変換回路（MAX3232 等）は正しく動作しているか
  □ フロー制御（RTS/CTS）が必要な機器ではないか
```

### コマンドを送ったが応答がない場合

```
確認項目:
  □ コマンドの末尾の改行コード（CR / LF / CR+LF）は正しいか
  □ 機器のエコーバック設定はどうなっているか
  □ コマンド送信後の待機時間は十分か（timeout_ms を増やす）
  □ 初期化コマンドを先に送る必要がないか
```

### Lua スクリプトが動かない場合

```
確認項目:
  □ lua_status で state を確認（running / idle / error）
  □ lua_log でスクリプト内 log.write() の出力を確認
  □ serial.query のタイムアウト（ms）が短すぎないか
  □ luna.run() チェーンが LUA_MAX_CHAIN(10) を超えていないか
  □ log.save() をループ内で呼んでいないか（フラッシュ寿命に注意）
```

### WiFi 接続が不安定な場合

```
確認項目:
  □ ESP32 とルーターの距離・障害物
  □ 2.4GHz 帯の電波干渉
  □ GPIO34 の設定（長期運用はハードリセットモード推奨）
```

### MCP が繋がらない場合

```
確認項目:
  □ IP アドレスが変わっていないか（AP モードで確認）
  □ claude_desktop_config.json の URL が正しいか（/mcp を使用）
  □ Claude Desktop を再起動する
  □ LUNA の電源が入っているか
```

### ESP32 が応答しなくなった場合

```
対処:
  1. Claude から hard_reset ツールを呼び出す
  2. USB ケーブルを抜き差しして電源リセット
  3. GPIO34 を LOW にしてハードリセットモードで電源 ON
```

---

## 9. 仕様一覧

### ファームウェア仕様

| 項目 | 仕様 |
|------|------|
| ブランド名 | LUNA |
| バージョン | v3.9.1 |
| プラットフォーム | ESP32（Arduino フレームワーク） |
| プロトコル | MCP（Model Context Protocol）Streamable HTTP |
| Web サーバー | Port 80 |
| MCP エンドポイント | `/mcp` |
| Lua エンジン | Lua 5.1 |
| Lua RAM 使用量 | 約 25KB |
| Lua 起動時間 | 約 9ms |
| スクリプト最大チェーン数 | 10 本 |
| ログバッファ | 直近 20 行（リングバッファ） |
| SPIFFS | WiFi 設定・Lua スクリプト・ログ |

### シリアル通信仕様（Free版）

| 項目 | 設定値 |
|------|--------|
| ボーレート | **9600 bps** |
| データビット | **8 bit** |
| ストップビット | **1 bit** |
| パリティ | **なし（N）** |
| フロー制御 | **なし** |

### MCPツール一覧

| カテゴリ | ツール名 |
|---------|---------|
| **シリアルブリッジ** | serial_query / serial_write / serial_read / serial_config（Pro版のみ） |
| **Lua自律制御** | lua_exec / lua_load / lua_run / lua_status / lua_stop / lua_log / lua_list |
| **通知** | check_notify |
| **システム情報** | get_system_info / get_network_status / get_license_info / hard_reset |
| **ファイルシステム** | fs_list / fs_read / fs_write / fs_append / fs_delete |

### ドキュメント一覧

| ファイル | 用途 |
|---------|------|
| `LUNA_README.md` | クイックスタート・接続手順 |
| `LUNA_USER_MANUAL.md` | このファイル（完全版マニュアル） |
| `LUNA_Lua_Guide.md` | RS-232C機器制御の概念・Lua活用例（詳細ガイド） |
| `LUNA_SERIAL_BRIDGE_GUIDE.md` | シリアルブリッジ詳細ガイド |
| `Temma_Protocol_Spec.md` | Takahashi Temma2 プロトコル仕様 |

---

*LUNA (AiBridgeMCP v3.9.1) / MIT License*
*(C) 2026 Nishioka Sadahiko*
*Blog   : https://nskikaku.blog.fc2.com/*
*GitHub : https://github.com/OnStepNinja/LUNA*

---

免責事項 (Disclaimer)
本システム（LUNA / AiBridgeMCP）は MIT ライセンスに基づき、現状のまま提供されます。

自己責任: 本機および提供されるソフトウェアの利用は、すべて利用者の自己責任において行われるものとします。

損害の不保証: 作者は、本機の使用、または使用不能から生じた直接的・間接的な損害（機器の故障、回路の焼損、データの損失、その他一切の不利益）について、その理由の如何を問わず一切の責任を負いません。

理解と同意: 利用者は、本機を RS-232C 機器に接続する際の電気的リスクを十分に理解した上で、本規約に同意したものとみなされます。

---

Legal Disclaimer:
This product is an independently developed hardware by OnStepNinja and is not affiliated with, endorsed by, or sponsored by Anthropic, Takahashi Seisakusho, or any other company.
"Temma", "MCP" and other product names are trademarks of their respective owners.
Use of these names is solely for the purpose of describing compatibility (Nominative Fair Use).

本プロジェクトは各社とは無関係の個人によるオープンソースプロジェクトです。
