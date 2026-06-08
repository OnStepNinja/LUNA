# [AI] LUNA Node Lua Guide

> Target: LUNA Node v1.0.4  
> Source verified: LUNA_Sub_v1_0_4.ino  
> Updated: 2026-06-08

---

## Overview

LUNA Node は ESP32-S3 上で動く BLE 専用の自律 Lua 実行デバイスです。  
WiFi 不要・電源投入で即動作。Bridge から BLE 経由でスクリプトを受け取り、単独で実行します。

```
Bridge Lua  → sub.send_file() → Node SPIFFS にスクリプト保存
Bridge Lua  → sub.lua_run()   → Node でスクリプト実行開始
Node Lua    → log.write()     → Beacon log_count に反映 (接続不要で通知)
Bridge Lua  → sub.get_log()   → Node ログ回収
```

---

## Lua 実行環境

| 項目 | 内容 |
|------|------|
| Lua バージョン | **Lua 5.1** |
| 許可ライブラリ | base / string / table / math |
| 禁止ライブラリ | io / os / debug / package / require |
| タイムアウト | 5000ms（超過で強制終了） |
| ログバッファ | リングバッファ 20行（LUA_LOG_MAX_LINES） |

### Lua 5.1 の注意事項

```lua
-- NG: // 演算子は Lua 5.3+ のみ
local q = 10 // 3   -- エラー

-- OK: math.floor を使う
local q = math.floor(10 / 3)   -- → 3

-- NG: ビット演算子なし
-- bit ライブラリは未搭載

-- NG: 文字列フォーマット
-- string.format のみ使用可
```

---

## Bridge との重要な差異

| 項目 | Bridge | Node |
|------|--------|------|
| `hw.free_heap()` | **非実装**（lua_status で確認） | **実装済み**（ESP.getFreeHeap() を返す） |
| `udp.begin()` | `begin(port)` | **`open(port)`**（Node では関数名が異なる） |
| `adc.read_avg()` | なし | **あり**（Node 追加 API） |
| WiFi | 常時接続 | デフォルト OFF（WIFI_DEFAULT_ON=0） |
| `notify.set()` | Bridge → Claude に通知 | ログバッファ経由のみ（BLE 接続がないと届かない） |

---

## モジュール一覧

| モジュール | 主な用途 |
|-----------|---------|
| hw | 遅延・LED・ヒープ・デバイスID |
| log | ログ記録・Beacon 連携・永続保存 |
| gpio | デジタル入出力・PWM |
| adc | アナログ入力（電圧計測） |
| sensor | DHT22 温湿度センサー |
| fs | SPIFFS ファイル操作 |
| wifi | WiFi 接続/切断 |
| pair | ペアリング状態管理 |
| serial | UART 通信 |
| json | JSON parse/encode |
| ntp | NTP 時刻同期 |
| http | HTTP GET/PUT/POST |
| tcp | TCP サーバー/クライアント |
| udp | UDP 通信 |
| luna | スクリプトチェーン実行 |
| notify | 完了通知フラグ |
| ble | BLE スキャン（Node は GATT Server のみ・Client は非推奨） |
| dac | **スタブ**（ESP32-S3 は HW DAC 非搭載） |
| ui | ブラウザ UI（WiFi ON 時のみ有効） |
| xml | XML パース（常時有効） |

---

## 第1章 hw モジュール — ハードウェア制御

### `hw.delay(ms)`

指定ミリ秒待機。`lua_stop` コマンドを受け付けながら待機します。

```lua
hw.delay(1000)    -- 1秒待機
hw.delay(60000)   -- 1分待機
```

---

### `hw.millis()`

起動からの経過時間（ミリ秒）を返します。

```lua
local start = hw.millis()
-- 処理
local elapsed = hw.millis() - start
log.write("処理時間: " .. elapsed .. " ms")
```

---

### `hw.free_heap()`

空きヒープをバイト数で返します（Node 専用 API）。

```lua
log.write("Heap: " .. hw.free_heap() .. " bytes")
-- 例: "Heap: 165680 bytes" (約162KB)
```

> Bridge では `hw.free_heap()` は存在しません。Node のみで使用可。

---

### `hw.led(state)`

内蔵 LED を制御します。`1` で点灯、`0` で消灯。

```lua
hw.led(1)       -- 点灯
hw.delay(200)
hw.led(0)       -- 消灯
```

> 必ず数値 `0` / `1` を使用。`true/false` はエラーになります。

---

### `hw.version()`

ファームウェアバージョン文字列を返します。

```lua
local v = hw.version()
log.write("FW: " .. v)   -- "FW: 1.0.4"
```

---

### `hw.license()`

MIT ライセンス情報を返します。

```lua
log.write(hw.license())
-- "LUNA Node v1.0.4\nLicense: MIT\n(c) 2026 Nishioka Sadahiko\nhttps://..."
```

---

### `hw.device_id()`

EFuse MAC を 12桁大文字 HEX 文字列で返します（電源切断後も不変）。

```lua
local id = hw.device_id()
log.write("ID: " .. id)   -- "ID: ACA704FD1671"
```

---

### `hw.sign(msg)`

Activation Key を秘密鍵として HMAC-SHA256 署名を生成します。  
64桁小文字 HEX を返します。

```lua
local sig, err = hw.sign("beacon_payload_2026")
if sig then
  log.write("sig: " .. sig)
else
  log.write("err: " .. err)  -- "Activation Key not set"
end
```

---

### `hw.rgb(r, g, b)` — NeoPixel ESP32-S3 専用

GPIO48 の NeoPixel WS2812B を指定色で点灯。`(0,0,0)` で消灯。

```lua
hw.rgb(255, 0, 0)    -- 赤
hw.rgb(0, 255, 0)    -- 緑
hw.rgb(0, 0, 0)      -- 消灯
```

---

### `hw.rgb_blink(r, g, b, duration_ms [, interval_ms])`

指定時間だけ点滅して自動消灯（ブロッキング）。

```lua
hw.rgb_blink(255, 200, 0, 1000)        -- 黄色1秒点滅
hw.rgb_blink(255, 0, 0, 2000, 100)     -- 赤を2秒、100ms間隔
```

---

### `hw.rgb_flag(state [, r, g, b])`

ノンブロッキング点滅フラグ。`state=1` で開始、`state=0` で停止。

```lua
hw.rgb_flag(1, 0, 0, 255)   -- 青点滅開始（スクリプトは次へ進む）
-- 別の処理
hw.delay(3000)              -- 待機中も点滅継続
hw.rgb_flag(0)              -- 停止・消灯
```

---

## 第2章 log モジュール — ログ記録・Beacon 連携

Node の log モジュールは Bridge の log と同一の API を持ちます。  
重要な点: `log.write()` は Beacon の `log_count` フィールドに反映されるため、**Bridge は接続なしでログ有無を検出**できます。

### `log.write(message)`

ログバッファ（最新20行リングバッファ）に記録します。

```lua
log.write("計測開始")
log.write(string.format("Temp=%.1f Humi=%.0f", 25.3, 62.0))
```

> ⚠️ `log.write()` を呼ぶたびに `luaLogCount++` → Beacon Byte 18 に反映。  
> Bridge は `parse_luna_beacon().log_count > 0` で未読ログを検出可能。

---

### `log.read([n])`

バッファから1行取得。インデックスの意味：

| 引数 | 意味 |
|------|------|
| 省略 / -1 | 最新行 |
| 0 | 最古行 |
| 1, 2, ... | 古い方から n 番目 |
| -2, -3, ... | 最新から遡って n 番目 |
| 範囲外 / 空 | nil |

```lua
local latest = log.read()    -- 最新行
local oldest = log.read(0)   -- 最古行
local prev   = log.read(-2)  -- 最新の1つ前
```

---

### `log.save()`

バッファを SPIFFS `/luna_log.txt` に保存。成功で `true`。

```lua
log.write("TEMP=25.3")
log.write("HUMI=62")
log.save()   -- 電源オフをまたいで保持
```

> ⚠️ ループ内で毎回呼ばないこと（フラッシュ寿命）。スクリプト末尾に1回だけ。

---

### `log.load()`

`/luna_log.txt` をバッファに読み込む。成功で `true`。  
呼び出し時にバッファをクリアしてから読み込みます。スクリプト先頭で呼ぶこと。

```lua
local ok = log.load()
if ok then
  local last = log.read()
  log.write("前回値: " .. tostring(last))
else
  log.write("初回起動")
end
```

---

### 典型パターン: センサー自律計測 + Beacon 通知

```lua
-- /lua_sensor_log.lua
-- このスクリプトを RUN:sensor_log で起動
local temp, humi = sensor.dht22()
if temp ~= -999.0 then
  log.write(string.format("T=%.1f H=%.0f ts=%d", temp, humi, hw.millis()))
end
-- log.write() だけで Beacon log_count が増える。log.save() は不要（揮発で OK な場合）
```

Bridge 側でログ回収:
```lua
-- Beacon で log_count > 0 を検出 → 接続 → GETLOG
ble.on("notify", function() end)
local logs = sub.get_log(NODE_ADDR, SVC, CMD, 8000)
```

---

## 第3章 gpio モジュール — デジタル入出力・PWM

### `gpio.write(pin, value)`

GPIO ピンに HIGH(1) / LOW(0) を出力。

```lua
gpio.write(38, 1)   -- GPIO38 HIGH
gpio.write(38, 0)   -- GPIO38 LOW
```

---

### `gpio.read(pin)`

GPIO ピンの状態を返します（0 または 1）。

```lua
local val = gpio.read(39)
log.write("GPIO39: " .. val)
```

---

### `gpio.mode(pin, mode)`

GPIO のモードを設定します。

| mode | 意味 |
|------|------|
| 0 | INPUT |
| 1 | OUTPUT |
| 2 | INPUT_PULLUP |

```lua
gpio.mode(38, 1)   -- OUTPUT
gpio.mode(39, 0)   -- INPUT
gpio.mode(40, 2)   -- INPUT_PULLUP
```

> **注意**: `gpio.input_pin()` は存在しません。入力は `gpio.mode(pin, 0)` + `gpio.read(pin)` を使ってください。

---

### `gpio.output_pin(n)`

Output スロット n (1〜4) に割り当てられた GPIO 番号を返します。

```lua
local p1 = gpio.output_pin(1)   -- Output 1 の GPIO 番号
local p2 = gpio.output_pin(2)   -- Output 2 の GPIO 番号
```

> Output ピンの割り当ては基板設計によります。Node v1.0.4 では OnStepNinja V2 準拠（GPIO38, GPIO39, ...）。

---

### `gpio.pwm(pin, freq, duty)`

PWM 出力を開始します。

| 引数 | 説明 |
|------|------|
| pin | GPIO 番号 |
| freq | 周波数（Hz） |
| duty | デューティ比 0〜255（128=50%） |

```lua
gpio.pwm(38, 1000, 128)    -- 1kHz, 50%
-- フェードアウト
for d = 255, 0, -5 do
  gpio.pwm(38, 1000, d)
  hw.delay(20)
end
gpio.pwm_stop(38)
```

---

### `gpio.pwm_stop(pin)`

PWM を停止して LEDC チャネルを解放します。

```lua
gpio.pwm_stop(38)
```

---

### スイッチ入力の例

```lua
gpio.mode(40, 2)   -- INPUT_PULLUP
local pressed = 0
for i = 1, 50 do   -- 5秒間ポーリング
  if gpio.read(40) == 0 then   -- LOW = 押された
    pressed = 1
    break
  end
  hw.delay(100)
end
log.write(pressed == 1 and "Pressed" or "Not pressed")
```

---

## 第4章 adc モジュール — アナログ入力

対応ピン: **GPIO1〜10（ADC1 のみ）**  
ADC2（GPIO11+）は WiFi と競合するため使用不可。

### `adc.atten(pin, attenuation)`

減衰設定で測定レンジを変更します。

| attenuation | 測定レンジ |
|-------------|-----------|
| 0 | 0〜800 mV |
| 1 | 0〜1100 mV |
| 2 | 0〜1350 mV |
| 3 | 0〜3100 mV（推奨） |

```lua
adc.atten(4, 3)   -- GPIO4を 0〜3.1V レンジに設定
```

---

### `adc.read(pin)`

RAW ADC 値（0〜4095）を返します。

```lua
local raw = adc.read(4)
log.write("ADC raw: " .. raw)
```

---

### `adc.read_mv(pin)`

電圧をミリボルト単位で返します。

```lua
adc.atten(4, 3)
local mv = adc.read_mv(4)
log.write(string.format("Voltage: %d mV (%.2f V)", mv, mv / 1000.0))
```

---

### `adc.read_avg(pin, n)` — Node 追加 API

n 回平均した電圧（mV）を返します。ノイズ低減に有効。

```lua
adc.atten(4, 3)
local mv = adc.read_avg(4, 16)   -- 16回平均
log.write("Avg voltage: " .. mv .. " mV")
```

> Bridge の Lua Guide には記載なし。Node 専用 API です。

---

### DAC について

ESP32-S3 はハードウェア DAC を持ちません。`dac.write()` / `dac.stop()` は**常にエラー**を返します。  
疑似アナログ出力には `gpio.pwm()` + RC フィルタを使用してください。

---

## 第5章 sensor モジュール — DHT22 温湿度センサー

### `sensor.dht22([pin])`

温度(℃) と 湿度(%) を返します。失敗時は両方 `-999.0`。内部で最大3回リトライ。

| 引数 | 説明 |
|------|------|
| pin | GPIO 番号（省略時 GPIO5） |

```lua
local temp, humi = sensor.dht22()
if temp ~= -999.0 then
  log.write(string.format("T=%.1f°C H=%.1f%%", temp, humi))
else
  log.write("DHT22 read error")
end
```

#### 露点計算（Magnus 公式）

```lua
local temp, humi = sensor.dht22()
if temp ~= -999.0 then
  local a, b = 17.27, 237.7
  local alpha = (a * temp / (b + temp)) + math.log(humi / 100.0)
  local dew   = b * alpha / (a - alpha)
  log.write(string.format("T=%.1f H=%.0f Dew=%.1f", temp, humi, dew))
end
```

---

## 第6章 fs モジュール — SPIFFS ファイル操作

### 保護ファイル（書き込み・削除禁止）

以下のファイルは `fs.write()` / `fs.append()` / `fs.delete()` でブロックされます。

```
/luna_autostart.txt   -- autostart 設定
/luna_pair.txt        -- ペアリング情報
/sub_config.json      -- Node 設定ファイル
```

`fs.read()` は制限なし。

---

### `fs.write(path, content)`

ファイルを新規作成または上書きします。

```lua
local ok, err = fs.write("/data.txt", "Hello Node")
if not ok then log.write("Error: " .. tostring(err)) end
```

---

### `fs.append(path, content)`

ファイルに追記します。

```lua
fs.append("/log.txt", os.time() .. ",25.3,62\n")
-- ※ os.time() は使用不可なので hw.millis() を使う
fs.append("/log.txt", hw.millis() .. ",25.3,62\n")
```

---

### `fs.read(path)`

ファイル内容を文字列で返します。存在しない場合は空文字列。

```lua
local content = fs.read("/data.txt")
log.write("Content: " .. content)
```

---

### `fs.delete(path)`

ファイルを削除します。成功で `true`。

```lua
local ok = fs.delete("/old_data.txt")
```

---

### `fs.exists(path)`

ファイルの存在を確認します。

```lua
if fs.exists("/config.json") then
  local cfg = json.parse(fs.read("/config.json"))
end
```

---

### `fs.list()`

SPIFFS のファイル一覧を JSON 文字列で返します。

```lua
local list = json.parse(fs.list())
for _, f in ipairs(list.files) do
  log.write(f.name .. " (" .. f.size .. " bytes)")
end
```

戻り値形式: `{"files":[{"name":"/xxx.txt","size":123},...]}`

---

### `fs.format()`

SPIFFS を全消去・再マウントします。成功で `true`。

```lua
fs.format()   -- 全ファイル削除・SPIFFS 再初期化
```

> ⚠️ ペアリング情報を含む全ファイルが削除されます。実行後は**再ペアリングが必要**です。  
> 初回フラッシュ後の初期化時のみ使用してください。

---

## 第7章 wifi モジュール

Node は `WIFI_DEFAULT_ON=0`（WiFi オフ）で起動します。必要時にオンデマンドで接続します。

### `wifi.set_config(ssid [, pass])`

認証情報を SPIFFS に保存します。接続しません（ノンブロッキング）。

```lua
wifi.set_config("MyNetwork", "mypassword")
-- または
wifi.set_config("OpenNetwork")   -- パスワードなし
```

---

### `wifi.on([ssid [, pass]])`

WiFi に接続します（最大15秒ブロッキング）。  
引数なしの場合は `wifi.set_config()` で保存済みの認証情報を使用。

```lua
-- 初回: 認証情報を設定してから接続
wifi.set_config("MyNetwork", "mypassword")
local ok, err = wifi.on()
if ok then
  log.write("Connected: " .. wifi.status().ip)
else
  log.write("Failed: " .. tostring(err))
end

-- 2回目以降: 引数なしで接続
wifi.on()
```

---

### `wifi.off()`

WiFi を切断します。Node モードでは `WiFi.mode(WIFI_OFF)` で完全停止。

```lua
wifi.off()
```

---

### `wifi.status()`

接続状態をテーブルで返します。

```lua
local s = wifi.status()
-- s.connected = true/false
-- s.ip        = "192.168.3.170"
-- s.ssid      = "MyNetwork"
-- s.rssi      = -65

if s.connected then
  log.write("IP: " .. s.ip)
end
```

---

## 第8章 pair モジュール — ペアリング状態管理

### `pair.status()`

現在のペアリング状態をテーブルで返します。

```lua
local p = pair.status()
-- p.paired            = true/false
-- p.pairing_mode      = true/false (ペアリング受付中)
-- p.peer_addr         = "ac:a7:04:fd:16:71" (ペアリング済み Bridge の MAC)
-- p.timeout_remaining = 30 (秒、ペアリングモード中のみ)
-- p.rssi              = -29

if p.paired then
  log.write("Paired with: " .. p.peer_addr)
end
```

---

### `pair.enter()`

ペアリングモードを起動します。  
ペアリング済みの Bridge からの CMD 経由で呼ぶ用途。通常は Bridge の `sub.pair()` を使うため直接呼ぶことは少ない。

```lua
pair.enter()   -- ペアリングモード開始（60秒間有効）
```

---

### `pair.forget()`

ペアリング情報を削除します。成功で `true`。

```lua
local ok, err = pair.forget()
if ok then
  log.write("Pairing cleared")
else
  log.write("Error: " .. tostring(err))   -- "Not paired"
end
```

---

## 第9章 serial モジュール — UART 通信

Node の serial モジュールは Bridge と同一 API です。ただし Node では外部 UART デバイスへの接続が前提。

### `serial.query(data, timeout_ms)` ★推奨

送信して応答を受信します。

```lua
local resp = serial.query("*IDN?\r\n", 1000)
if resp ~= "" then
  log.write("応答: " .. resp)
else
  log.write("タイムアウト")
end
```

---

### `serial.write(data)`

データを送信します（応答を待たない）。

```lua
serial.write("START\r\n")
hw.delay(100)
local resp = serial.read()
```

---

### `serial.read()`

受信バッファのデータを読み取ります。

```lua
local data = serial.read()
if data ~= "" then
  log.write("受信: " .. data)
end
```

---

## 第10章 json / ntp / http / tcp / udp

これらのモジュールは Bridge と同一 API です。WiFi 接続が必要な点に注意。

### json モジュール

```lua
-- parse
local data, err = json.parse('{"temp":25.3,"humi":62}')
if data then
  log.write("temp=" .. data.temp)
end

-- encode
local s = json.encode({temp=25.3, humi=62, active=true})
-- → '{"temp":25.3,"humi":62,"active":true}'
```

---

### ntp モジュール（WiFi 必要）

```lua
wifi.on()
ntp.sync("pool.ntp.org")
hw.delay(5000)
if ntp.synced() then
  log.write("UTC: " .. ntp.utc())
  log.write(string.format("JD: %.5f", ntp.jd()))
end
wifi.off()
```

| 関数 | 戻り値 |
|------|--------|
| `ntp.sync([server])` | true（非ブロッキング） |
| `ntp.synced()` | 同期済みで true |
| `ntp.unix()` | Unix タイムスタンプ（秒） |
| `ntp.utc()` | "YYYY-MM-DD HH:MM:SS" |
| `ntp.jd()` | ユリウス日（浮動小数点） |

---

### http モジュール（WiFi 必要）

```lua
wifi.on()
local body = http.get("http://192.168.3.23:1888/api/camera", 3000)
local cam  = json.parse(body)
if cam then
  log.write("Camera: " .. cam.name)
end

-- POST
local resp, code = http.post(
  "http://192.168.3.23:1888/v2/api/equipment/camera/capture",
  '{"Duration":5.0}', "application/json", 10000)
wifi.off()
```

---

### tcp モジュール

```lua
-- TCP サーバー
tcp.begin(8080)
while true do
  if tcp.connected() then
    local data = tcp.read()
    if data ~= "" then
      tcp.write("OK:" .. data)
      tcp.disconnect()   -- 1接続1コマンド方式
    end
  end
  hw.delay(10)
end

-- TCP クライアント
local ok = tcp.client_connect("192.168.3.23", 4400, 3000)
if ok then
  tcp.client_write('{"method":"get_equipment_profile"}\r\n')
  hw.delay(200)
  local resp = tcp.client_read(500)
  log.write("PHD2: " .. resp)
  tcp.client_close()
end
```

---

### udp モジュール

> ⚠️ **Node では `udp.open()` を使用。Bridge の `udp.begin()` とは関数名が異なります。**

```lua
udp.open(60023)                       -- ローカルポート開放
udp.connect("192.168.6.1", 60023)     -- 送信先設定
udp.write("$CMD=STATUS\r\n")          -- 送信
local resp = udp.read(1000)           -- 受信（タイムアウト1秒）
udp.close()                           -- クローズ
```

---

## 第11章 luna モジュール — スクリプトチェーン

### `luna.run(name)`

現在のスクリプト終了後、次のスクリプトを自動起動します。  
成功で `true`、チェーン上限（10本）超過で `false`。

```lua
-- /lua_step1.lua
log.write("Step1完了")
log.save()
luna.run("step2")   -- step1 終了後 step2 を自動起動

-- /lua_step2.lua
log.load()
log.write("Step2: 前回値=" .. tostring(log.read()))
-- step2 で luna.run() を呼ばなければチェーン終了
```

上限到達時: `luaLogLines` に `"luna.run: chain limit (10) reached"` が記録される。

---

## 第12章 notify モジュール

Bridge と同じ API ですが、Node では MCP 接続がないため Bridge 経由でないと Claude には届きません。

```lua
notify.set("計測完了")   -- フラグを立てる（BLE 接続時のみ有効）
```

> Node スタンドアロン動作では `log.write()` + Beacon log_count の組み合わせを推奨します。

---

## 第13章 自律動作パターン集

### パターン 1: 定期センサー計測 → Beacon 通知 → Bridge 回収

```lua
-- /lua_periodic_sensor.lua
-- Bridge から RUN:periodic_sensor で起動
-- autostart に設定して電源ON自動起動も可

local INTERVAL_MS = 60000   -- 1分間隔

for i = 1, 10 do
  local temp, humi = sensor.dht22()
  if temp ~= -999.0 then
    log.write(string.format("i=%d T=%.1f H=%.0f heap=%d",
      i, temp, humi, hw.free_heap()))
  else
    log.write("i=" .. i .. " DHT22_ERR")
  end
  if i < 10 then hw.delay(INTERVAL_MS) end
end
-- スクリプト終了 → luaLogCount が増加 → Beacon Byte 18 に反映
-- Bridge は接続なしでログ有無を検出可能
```

---

### パターン 2: 電源投入後にスタンドアロン稼働

```lua
-- /luna_autostart.txt に "boot_task" を設定
-- /lua_boot_task.lua の内容:

log.write("Boot: " .. hw.version())
log.write("Heap: " .. hw.free_heap())

-- 前回の状態を復元
local restored = log.load()
if restored then
  log.write("Restored: " .. tostring(log.read()))
end

-- センサー計測
local temp, humi = sensor.dht22()
if temp ~= -999.0 then
  log.write(string.format("T=%.1f H=%.0f", temp, humi))
  log.save()   -- 次回起動時に参照
end
-- スクリプト終了 → Beacon log_count が増加 → Bridge が回収
```

---

### パターン 3: WiFi を使った HTTP データ送信

```lua
-- センサーデータを HTTP POST で送信する例
wifi.set_config("MyNetwork", "mypassword")
wifi.on()
hw.delay(2000)   -- 接続安定待ち

local s = wifi.status()
if not s.connected then
  log.write("WiFi failed")
  wifi.off()
  return
end

local temp, humi = sensor.dht22()
if temp ~= -999.0 then
  local body = json.encode({
    device_id = hw.device_id(),
    temp      = temp,
    humi      = humi,
    ts        = ntp.unix()
  })
  local resp, code = http.post(
    "http://192.168.3.23:8080/api/sensor",
    body, "application/json", 5000)
  log.write("POST " .. code .. ": " .. resp)
end

wifi.off()
```

---

### パターン 4: ファイルに累積ログを書いて Bridge に一括転送

```lua
-- /lua_accumulate.lua: ループで計測・ファイル累積
for i = 1, 20 do
  local temp, humi = sensor.dht22()
  local line = string.format("%d,%.1f,%.0f\n", hw.millis(), temp, humi)
  fs.append("/sensor_data.csv", line)
  hw.delay(30000)   -- 30秒間隔
end
log.write("Done: 20 samples saved to /sensor_data.csv")
```

Bridge 側でファイル取得:
```lua
ble.on("notify", function() end)   -- no-op 必須
local content, err = sub.read_file(NODE_ADDR, SVC, CMD, "/sensor_data.csv", 15000)
if content then
  -- CSV を解析
end
```

---

## 付録A: モジュール別 API 早見表

### hw
| 関数 | 引数 | 戻り値 |
|------|------|--------|
| `hw.delay(ms)` | ms: 数値 | なし |
| `hw.millis()` | なし | 起動経過ms |
| `hw.free_heap()` | なし | 空きヒープ(B) |
| `hw.led(state)` | 0/1 | なし |
| `hw.version()` | なし | バージョン文字列 |
| `hw.license()` | なし | ライセンス文字列 |
| `hw.device_id()` | なし | 12桁 HEX |
| `hw.sign(msg)` | 文字列 | 64桁 HEX / nil, err |
| `hw.rgb(r,g,b)` | 0-255×3 | なし（S3専用） |
| `hw.rgb_blink(r,g,b,dur[,inv])` | — | なし（S3専用） |
| `hw.rgb_flag(state[,r,g,b])` | 0/1 | なし（S3専用） |

### log
| 関数 | 引数 | 戻り値 |
|------|------|--------|
| `log.write(msg)` | 文字列 | なし |
| `log.read([n])` | インデックス | 文字列/nil |
| `log.save()` | なし | true/false |
| `log.load()` | なし | true/false |

### gpio
| 関数 | 引数 | 戻り値 |
|------|------|--------|
| `gpio.write(pin, val)` | pin, 0/1 | なし |
| `gpio.read(pin)` | pin | 0/1 |
| `gpio.mode(pin, mode)` | pin, 0/1/2 | なし |
| `gpio.output_pin(n)` | 1-4 | GPIO番号/nil |
| `gpio.pwm(pin, freq, duty)` | pin, Hz, 0-255 | true/nil,err |
| `gpio.pwm_stop(pin)` | pin | true |

### adc
| 関数 | 引数 | 戻り値 |
|------|------|--------|
| `adc.atten(pin, n)` | pin, 0-3 | なし |
| `adc.read(pin)` | pin | 0-4095 |
| `adc.read_mv(pin)` | pin | mV |
| `adc.read_avg(pin, n)` | pin, 回数 | mV（n回平均） |

### fs
| 関数 | 引数 | 戻り値 |
|------|------|--------|
| `fs.write(path, content)` | — | true/false,err |
| `fs.append(path, content)` | — | true/false,err |
| `fs.read(path)` | — | 文字列 |
| `fs.delete(path)` | — | true/false |
| `fs.exists(path)` | — | true/false |
| `fs.list()` | — | JSON文字列 |
| `fs.format()` | — | true/nil,err |

### wifi
| 関数 | 引数 | 戻り値 |
|------|------|--------|
| `wifi.on([ssid[,pass]])` | — | true/nil,err |
| `wifi.off()` | — | true |
| `wifi.status()` | — | {connected,ip,ssid,rssi} |
| `wifi.set_config(ssid[,pass])` | — | true |

### pair
| 関数 | 引数 | 戻り値 |
|------|------|--------|
| `pair.status()` | — | {paired,pairing_mode,peer_addr,timeout_remaining,rssi} |
| `pair.enter()` | — | true |
| `pair.forget()` | — | true/nil,err |

---

## 付録B: Node で使用できない機能

| 機能 | 理由 |
|------|------|
| `require` | 禁止（パッケージシステム無効） |
| `io.*` / `os.*` | 禁止（セキュリティ） |
| `debug.*` | 禁止 |
| `dac.write()` / `dac.stop()` | ESP32-S3 は HW DAC 非搭載 → 常にエラー |
| `udp.begin()` | Node では `udp.open()` を使用 |
| `//` 演算子 | Lua 5.1 では未サポート → `math.floor(x/y)` を使用 |

---

*LUNA Node v1.0.4 Lua Guide — Source verified from LUNA_Sub_v1_0_4.ino*
