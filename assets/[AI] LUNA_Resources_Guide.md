# 6. LUNA Observatory Resources Guide
**Version:** v6.9.2 INDI Edition
**リビジョン:** r1.4
**対象:** LUNA Observatory / LUNA INDI Edition ユーザー
**更新日:** 2026-04-13
**作者:** Nishioka Sadahiko

---

> **[重要事項]** 本ソフトウェアは MIT ライセンスのもと**バイナリ形式**で配布されています（無償・商用利用可）。ソースコードは公開されていません。

---

## 1. Resources とは何か

LUNA Observatory は、MCP（Model Context Protocol）の3つの機能を実装しています。

| 機能 | 役割 | たとえ |
|------|------|--------|
| **Tools** | Claude がデバイスを**操作する** | ボタンを押す |
| **Prompts** | Claude が**ガイドを読む** | マニュアルを参照する |
| **Resources** | Claude がデバイスの**状態を読む** | 計器を見る |

**Resources は「計器」です。**
望遠鏡の現在位置、ドームの状態、気象データなど、リアルタイムの情報を
Claude が自然に読み取れるようにします。

### Tools との違い

**Tools のみの場合（従来）:**
```
ユーザー:「今どこ向いてる？」
Claude: serial_query(":GR#") を呼び出す → 結果を受け取る
Claude: serial_query(":GD#") を呼び出す → 結果を受け取る
→ 2回のツール呼び出しが必要
```

**Resources 実装後:**
```
ユーザー:「今どこ向いてる？」
Claude: luna://lua/mount を読む → 即答
→ 文脈の中で自然に参照
```

---

## 2. 仕組み

LUNA Observatory の Resources は **Lua スクリプトで定義**します。

```
SPIFFS（ESP32 内部ストレージ）
    └── lua_resource_mount.lua    → luna://lua/mount
    └── lua_resource_dome.lua     → luna://lua/dome
    └── lua_resource_weather.lua  → luna://lua/weather
```

LUNA が SPIFFS を自動検索して `lua_resource_*.lua` を見つけ、
Resources として公開します。**スクリプトを追加すれば Resource が増え、
削除すれば Resource が消えます。** firmware の変更は不要です。

### 処理フロー

```
① Claude が resources/read "luna://lua/mount" を要求
      ↓
② LUNA が /lua_resource_mount.lua を SPIFFS から読み込む
      ↓
③ LUNA が Lua スクリプトを実行
      ↓
④ スクリプトが望遠鏡にコマンドを送り RA/Dec を取得
      ↓
⑤ スクリプトが JSON 文字列を return する
      ↓
⑥ LUNA が MCP レスポンスとして Claude に返す
      ↓
⑦ Claude が RA/Dec を受け取り回答する
```

---

## 3. リソーススクリプトの作り方

### 命名規則

| 項目 | 規則 |
|------|------|
| ファイル名 | `lua_resource_{名前}.lua` |
| 名前に使える文字 | 英数字・アンダースコア・ハイフン |
| URI | `luna://lua/{名前}` |
| 戻り値 | JSON 文字列（`return` で返す） |

### 基本テンプレート

```lua
-- lua_resource_{名前}.lua
-- 処理（機器へのコマンド送信など）
local value = serial.query("コマンド", 100)

-- エラーチェック
if value == "" then
  return '{"error":"no response"}'
end

-- JSON 文字列を return で返す
return string.format('{"key":"%s"}', value)
```

### 実例① 赤道儀（LX200互換）

```lua
-- lua_resource_mount.lua
local ra  = serial.query(":GR#", 100)
local dec = serial.query(":GD#", 100)
if ra == "" or dec == "" then
  return '{"error":"no response from mount"}'
end
return string.format('{"ra":"%s","dec":"%s"}', ra, dec)
```

### 実例② ドーム

```lua
-- lua_resource_dome.lua
local status = serial.query("DOME_STATUS\r", 300)
if status == "" then
  return '{"error":"no response from dome"}'
end
return '{"status":"' .. status .. '"}'
```

### 実例③ 複数の値をまとめる

```lua
-- lua_resource_mount.lua（詳細版）
local ra     = serial.query(":GR#", 100)
local dec    = serial.query(":GD#", 100)
local status = serial.query(":GW#", 100)
if ra == "" then
  return '{"error":"no response from mount"}'
end
return string.format(
  '{"ra":"%s","dec":"%s","status":"%s"}',
  ra, dec, status
)
```

### 実例④ ログデータを公開する

```lua
-- lua_resource_log.lua
log.load()
local lines = log.read(5)
return '{"recent_log":"' .. lines .. '"}'
```

### 規約まとめ

| 項目 | 規約 |
|------|------|
| ファイル名 | `lua_resource_{名前}.lua` |
| 戻り値 | `return` で JSON 文字列を返す（必須） |
| エラー時 | `{"error":"メッセージ"}` を返す |
| 実行時間 | できるだけ短く（serial.query は 100〜200ms 推奨） |
| log.save() | リソーススクリプト内では**呼ばない**（フラッシュ寿命） |
| 副作用 | 機器を動かすコマンドは含めない（読み取りのみ推奨） |

---

## 4. スクリプトの保存方法（lua_save）

Claude から直接スクリプトを保存できます。

### Claude への指示例

```
「以下の Lua スクリプトを resource_mount という名前で保存して」

local ra  = serial.query(":GR#", 100)
local dec = serial.query(":GD#", 100)
if ra == "" or dec == "" then
  return '{"error":"no response from mount"}'
end
return string.format('{"ra":"%s","dec":"%s"}', ra, dec)
```

### lua_save ツール

```
lua_save("resource_mount", "スクリプト内容")
→ /lua_resource_mount.lua として SPIFFS に保存
→ luna://lua/mount が自動的に Resources に現れる
```

---

## 5. Resources の確認方法

### 5-1. resources/list — 利用可能な Resources を確認

Claude Desktop では自動的に参照されます。
手動で確認する場合は curl を使います：

```bash
curl -X POST http://{LUNA_IP}/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"resources/list","params":{}}'
```

**レスポンス例:**
```json
{
  "resources": [
    {
      "uri": "luna://lua/mount",
      "name": "mount",
      "description": "Resource: /lua_resource_mount.lua",
      "mimeType": "application/json"
    }
  ]
}
```

### 5-2. resources/read — Resource の値を読む

```bash
curl -X POST http://{LUNA_IP}/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"resources/read","params":{"uri":"luna://lua/mount"}}'
```

**レスポンス例:**
```json
{
  "contents": [
    {
      "uri": "luna://lua/mount",
      "mimeType": "application/json",
      "text": "{\"ra\":\"20:41.4#\",\"dec\":\"+45:16#\"}"
    }
  ]
}
```

### 5-3. Claude Desktop から確認する

Claude Desktop に LUNA Observatory が接続されていれば、
自然な会話で Resources を参照できます：

```
ユーザー:「今の望遠鏡の向きを教えて」
Claude:  luna://lua/mount を読んで → RA/Dec を回答

ユーザー:「観測できる状態か確認して」
Claude:  luna://lua/mount・luna://lua/weather を読んで総合判断
```

---

## 6. LUNA Observatory のアーキテクチャ

1台の LUNA が 1台の機器を担当します。
Claude Desktop が複数の LUNA に同時接続することで天文台全体を制御します。

```
Claude Desktop（AI の脳）
    │
    ├── LUNA #1（シリアル赤道儀）      192.168.x.x
    │     Serial → NS-5000 / Temma2（LX200・Temma プロトコル）
    │     luna://lua/mount
    │
    ├── LUNA #2（ドーム）              192.168.x.x
    │     luna://lua/dome
    │
    ├── LUNA #3（気象センサー）        192.168.x.x
    │     luna://lua/weather
    │
    └── LUNA #4（AiBridge ゲートウェイ）  192.168.x.x
          http.get/put → AiBridge v7.14（Alpaca REST + LX200 API）
          Resources:
            luna://lua/mount_aibridge  ← OnStep 赤道儀（/api/status）
            luna://lua/weather         ← 気象センサー（DHT22）
            luna://lua/switch          ← リレー4ポート
```

### 複数 LUNA の識別方法

各 LUNA の `location` 名を変えるだけで識別できます。

| 項目 | 設定方法 |
|------|----------|
| **location 名** | `LUNA1` / `LUNA2` のように各デバイスで変える |
| **IP アドレス** | DHCP で自動割り当て、または固定 |
| **Claude Desktop 設定** | `claude_desktop_config.json` でサーバー名と IP を登録 |

```json
{
  "mcpServers": {
    "luna-mount":   { "args": ["http://192.168.3.154/mcp"] },
    "luna-dome":    { "args": ["http://192.168.3.155/mcp"] },
    "luna-weather": { "args": ["http://192.168.3.156/mcp"] }
  }
}
```

Claude は `get_network_status` の `location` フィールドで各デバイスを区別します。

### LUNA と LUNA Observatory の関係

LUNA Observatory は LUNA の**完全上位互換**です。

| 機能 | LUNA v3.9.1 | LUNA Observatory v5.9.1 |
|------|:-----------:|:-----------------------:|
| Tools（全ツール） | ✅ | ✅ |
| Prompts（luna_guide） | ✅ | ✅ |
| Lua エンジン・luna.run() | ✅ | ✅ |
| notify.set() / log モジュール | ✅ | ✅ |
| **Resources** | ❌ | ✅ |

単体接続でも LUNA Observatory をそのまま使えます。
複数機器を統合したとき、Observatory の真価が発揮されます。

---

## 7. トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| resources/list に出てこない | ファイル名が違う | `lua_resource_` で始まっているか確認 |
| `{"error":"no response"}` が返る | 機器が応答しない | serial.query のタイムアウトを延ばす |
| resources/read がエラー | Lua タスク実行中 | lua_status で idle を確認してから再試行 |
| JSON が壊れている | return の文字列に `"` が含まれる | `string.format` でエスケープする |

---

## 8. http モジュール — AiBridge / Alpaca 連携（v5.9.1 / v5.9.2）

LUNA v5.9.1 では Lua から LAN 内の HTTP デバイスを直接呼び出せます。
これにより **AiBridge（ASCOM/Alpaca 対応）** の全デバイスを Resources として公開できます。
**v5.9.2 では `http.post()` を追加し、NINA Advanced API / Stellarium Remote Control にも対応しました。**
**2026-03-27 実機接続・操作を確認済みです。**

### API

```lua
http.get(url, timeout_ms)                    -- HTTP GET。レスポンスボディを返す
http.put(url, body, timeout_ms)              -- HTTP PUT。レスポンスボディを返す
local resp, code = http.post(url, body, ct, ms)  -- HTTP POST（v5.9.2）。resp + HTTP コードの2値を返す
-- timeout_ms のデフォルト: 2000ms
-- http.get / http.put: エラー・タイムアウト時は空文字列 "" を返す
-- http.post: code が負値の場合は接続エラー（200 = 成功）
-- HTTPS 非対応（LAN 内 HTTP 専用）
```

#### 用途別メソッド選択

| 用途 | メソッド | 備考 |
|------|---------|------|
| ASCOM/Alpaca 値読み取り | `http.get` | |
| ASCOM/Alpaca コマンド送信 | `http.put` | Content-Type: `application/x-www-form-urlencoded`（固定） |
| NINA Advanced API v2（ポート 1888） | `http.post` | Content-Type: `application/json`（デフォルト） |
| Stellarium Remote Control（ポート 8090） | `http.post` | Content-Type: `application/x-www-form-urlencoded` |

### AiBridge Alpaca 連携スクリプト例

#### 気象センサー（observingconditions）

```lua
-- lua_resource_weather.lua
local temp = http.get("http://192.168.3.7/api/v1/observingconditions/0/temperature", 1000)
local hum  = http.get("http://192.168.3.7/api/v1/observingconditions/0/humidity",    1000)
if temp == "" then return '{"error":"no response from AiBridge"}' end
return string.format('{"temp":%s,"humidity":%s}', temp, hum)
```

#### スイッチポート状態（switch）

```lua
-- lua_resource_switch.lua
local s0 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=0", 1000)
local s1 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=1", 1000)
local s2 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=2", 1000)
local s3 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=3", 1000)
return string.format('{"port0":%s,"port1":%s,"port2":%s,"port3":%s}', s0, s1, s2, s3)
```

#### スイッチ制御（PUT）

```lua
-- ヒーター ON（lua_exec から呼ぶ）
local r = http.put("http://192.168.3.7/api/v1/switch/0/setswitch", "Id=0&State=true", 1000)
return r
```

### タイムアウト推奨値

| 用途 | 推奨タイムアウト |
|------|----------------|
| Alpaca GET（センサー・状態取得） | 1000ms |
| Alpaca PUT（コマンド送信） | 2000ms |
| `/api/external/discover` | 4000ms（UDP 待ちのため長め） |

### 実機テスト結果（2026-03-21）

| テスト | 結果 | 応答時間 |
|--------|------|---------|
| `http.get("http://192.168.4.1/")` | AiBridgeMCP Console 応答 | 35ms ✅ |
| `http.get("http://192.168.3.7/")` | AiBridge Console 応答 | 153ms ✅ |
| `switch/0/maxswitch` | Value:4（4ポート確認） | ✅ |
| `switch/0/getswitch?Id=0〜3` | 全ポート false（OFF） | 315ms ✅ |
| `observingconditions/0/temperature` | Value:-999.0（センサー未接続） | ✅ |
| `/api/external/discover` | Discovery 完了（data:[]） | 3386ms ✅ |

### LUNA + AiBridge の統合アーキテクチャ

```
Claude Desktop（AI の脳）
    │
    ├── LUNA #1（赤道儀）
    │     Serial → LX200 / Temma2
    │     Resources: luna://lua/mount
    │
    └── LUNA #2（AiBridge ゲートウェイ）
          http.get/put → AiBridge → Alpaca REST
          Resources:
            luna://lua/weather   ← 気温・湿度・雲量
            luna://lua/switch    ← 出力ポート状態
            luna://lua/focuser   ← フォーカサー位置
```

**AiBridge のファームウェアは変更不要。** Alpaca REST サーバーとして動作し続けるだけ。
LUNA の Lua スクリプトで任意の Alpaca デバイスを Resource として公開できます。

---

## 8.1 AiBridge LX200 REST API — OnStep 対応機器の制御（v5.9.1）

AiBridge は Alpaca REST サーバーに加え、**LX200 コマンドを HTTP 経由で送受信できる REST API** を持ちます。
LUNA の http モジュールからこの API を呼ぶことで、AiBridge に接続された OnStep 望遠鏡を制御できます。

### エンドポイント

| エンドポイント | 説明 |
|--------------|------|
| `GET /api/status` | RA / Dec / 追尾状態をまとめて取得（JSON） |
| `GET /api/command?cmd=XX` | 任意の LX200 コマンドを送信（`:` `#` 不要）、JSON で返却 |
| `GET /api/goto?ra=HH:MM:SS&dec=±DD:MM:SS` | 指定座標に GoTo |

### Lua スクリプト例

#### AiBridge 経由で赤道儀状態を取得

```lua
-- lua_resource_mount_aibridge.lua
-- AiBridge（192.168.3.7）に接続された OnStep 望遠鏡の RA/Dec/状態を取得
local resp = http.get("http://192.168.3.7/api/status", 2000)
if resp == "" then return '{"error":"no response from AiBridge"}' end
return resp
-- → {"status":"success","data":{"ra":"20:41.4","dec":"+45:16","state":"Tracking",...}}
```

#### 任意の LX200 コマンドを送る

```lua
-- lua_exec から呼ぶ例（GoTo: ターゲット設定 → GoTo 実行）
local r1 = http.get("http://192.168.3.7/api/command?cmd=SrHH:MM:SS", 1000)  -- RA設定
local r2 = http.get("http://192.168.3.7/api/command?cmd=SdDD:MM:SS", 1000)  -- Dec設定
local r3 = http.get("http://192.168.3.7/api/command?cmd=MS", 1000)           -- GoTo実行
return r3
```

### 実機テスト結果（2026-03-21）

| テスト | 結果 | 応答時間 |
|--------|------|---------|
| `GET /api/status` | `{"state":"Tracking","connected":true,...}` | 113ms ✅ |
| `GET /api/command?cmd=GR` | `{"status":"success","message":"Command executed"}` | 206ms ✅ |

※ テスト時は AiBridge に望遠鏡未接続のため RA/Dec は空。API 動作は確認済み。

### シリアル接続との比較

| | LUNA 直接シリアル | LUNA + AiBridge LX200 API |
|--|-----------------|--------------------------|
| 接続 | ケーブル必須 | LAN 経由（WiFi） |
| コマンド形式 | `serial.query(":GR#", 100)` | `http.get(".../api/command?cmd=GR", 1000)` |
| レスポンス | 生文字列 `"20:41.4#"` | JSON `{"data":"20:41.4#"}` |
| 対象機器 | レガシーシリアル機器 | OnStep 搭載機器（AiBridge 経由） |

---

## 9. 動作確認済み環境

| 機器 | 接続方法 | リソーススクリプト | 状態 |
|------|---------|------------------|------|
| NS-5000（LX200互換） | Serial 9600bps | lua_resource_mount.lua | ✅ 実機確認済み |
| AiBridge v7.14 switch | http.get/put | lua_resource_switch.lua | ✅ 実機確認済み |
| AiBridge v7.14 observingconditions | http.get | lua_resource_weather.lua | ✅ 実機確認済み |
| AiBridge v7.14 LX200 REST API | http.get `/api/status` `/api/command` | lua_resource_mount_aibridge.lua | ✅ 実機確認済み |
| **NINA Advanced API v2**（ポート 1888） | http.get / http.post | — | ✅ **実機確認済み**（2026-03-27） |
| **Stellarium Remote Control**（ポート 8090） | http.get / http.post | — | ✅ **実機確認済み**（2026-03-27） |
| Takahashi Temma2 | Serial 19200bps | lua_resource_mount.lua | 📋 作成予定 |

---

## 10. LUNA Factory への展開

LUNA Observatory のアーキテクチャは天文台以外にも応用できます。

**LUNA Factory** は同じ仕組みを産業・工場向けに転用したコンセプトです。

| | LUNA Observatory | LUNA Factory |
|--|-----------------|-------------|
| 機器 | 望遠鏡・ドーム・気象 | CNC・PLC・センサー |
| 通信 | LX200・Temma | 産業用シリアル |
| 自動化 | 観測シーケンス | 生産プロセス |
| AI 判断 | 天候・星位置 | 品質・異常・効率 |
| 位置づけ | 無料・天文コミュニティ | 有料・法人向け |

**firmware の変更はほとんど不要。** Lua スクリプトで対応機器を定義するだけです。

---

*LUNA Observatory v5.9.1b — Bridge Across Decades © 2026 Nishioka Sadahiko*
