# 3. LUNA Lua ガイド

**対象ファームウェア**: AiBridgeMCP v3.9.1 LUNA / v5.9.1 LUNA Observatory / v5.9.2 / v5.9.3 / v5.9.4 / v5.9.4a / v5.9.5 / v5.9.6 / v6.9.2 / v6.9.3 INDI Edition / v8.9.3 / v8.9.4 / v5.9.8/v6.9.5/v8.9.5 Lab Edition / v8.9.6 Lab Edition / **v8.9.7 + LUNA Node v1.0.4**
**ドキュメントリビジョン**: r1.30
**最終更新**: 2026-05-15
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
| r1.9  | 2026-03-22 | 3-8 新設：MCP Resources リソーススクリプトの書き方（v5.9.1） |
| r1.10 | 2026-03-24 | serial 通信図の「Serial2」→「シリアルブリッジポート」に修正・TARGET_BOARD別ポート対応表追加・v2.x との違いを注記 |
| r1.7 | 2026-03-20 | lua_load → lua_save にツール名変更（全コード例・APIリファレンス更新）・目次内リンクをプレーンテキスト化 |
| r1.11 | 2026-03-26 | v5.9.2 対応：http.post() 追加（NINA Advanced API / Stellarium Remote Control 対応）・セクション 2-6 更新・クイックリファレンス更新 |
| r1.12 | 2026-03-27 | NINA Advanced API v2・Stellarium Remote Control 実機接続・操作を確認済みに更新 |
| r1.13 | 2026-03-28 | v5.9.3 対応：udp モジュール追加（Vixen Wireless Unit / 汎用 UDP 通信）・セクション 2-7 新設・2-7〜2-8 を 2-8〜2-9 に繰り下げ |
| r1.14 | 2026-03-28 | v5.9.3 対応：gpio モジュール追加（GPIO 直接制御・フォーカサーモーター対応）・Output ピン定義（OnStepNinja V2）・セクション 2-8 新設・2-8〜2-10 を 2-9〜2-11 に繰り下げ。実機全ピン確認済み |
| r1.15 | 2026-03-28 | v5.9.3 対応：sensor モジュール追加（DHT22 温湿度センサー・GPIO5）・エラー値 -999.0（AiBridge 統一仕様）・lua_resource_weather MCP Resource 追加・セクション 2-9 新設・2-9〜2-12 を 2-10〜2-13 に繰り下げ。実機確認済み |
| r1.16 | 2026-03-29 | v5.9.4 対応：json モジュール追加（json.parse / json.encode）・ArduinoJson v7 対応・セクション 2-10 新設 |
| r1.17 | 2026-03-29 | v5.9.4 対応：ntp モジュール追加（ntp.sync / synced / unix / utc / jd）・NTP 時刻同期・ユリウス日対応・セクション 2-11 新設 |
| r1.18 | 2026-03-30 | v5.9.4a 対応：tcp モジュール追加（TCP サーバー・クライアント完全対応）・SkySafari 実機確認・セクション 2-12 新設・旧 2-12〜2-14 を 2-13〜2-15 に繰り下げ |
| r1.19 | 2026-03-30 | v5.9.5 対応：autostart 機能追加（電源ON自律起動・セーフモード・クラッシュ検出）・セクション 2-13 新設・MCP ツール一覧に set/get/clear_autostart 追加 |
| r1.20 | 2026-04-13 | v6.9.2 INDI Edition 対応：INDI MCP ツール 10 個追加（indi_connect/disconnect/status/devices/properties/get/state/set_number/set_switch/set_text）・luna_guide セクション 2-13 参照案内追加 |
| r1.21 | 2026-04-15 | v5.9.6/v6.9.3 対応：ui モジュール追加（ブラウザ UI 連携・リアルタイム表示・ボタン受信）・セクション 2-14 新設・旧 2-14〜2-17 を 2-16〜2-19 に繰り下げ |
| r1.22 | 2026-04-15 | Console UI / File Manager セクション追加（2-15 新設）・hw.free_heap() 誤記削除（非実装）・Lua 5.1 整数除算注意追記・クイックリファレンス更新 |
| r1.23 | 2026-04-19 | v6.9.3 対応：hw.rgb / hw.rgb_blink / hw.rgb_flag 追加（NeoPixel WS2812B GPIO48 ESP32-S3 専用）・セクション 2-3 更新・クイックリファレンス更新。★実機確認済み |
| r1.24 | 2026-04-20 | v6.9.2/v6.9.3 対応：indi モジュール（Lua API）追加・セクション 2-13 新設・旧 2-13〜2-19 を 2-14〜2-20 に繰り下げ。詳細は `get_lua_guide` セクション 2-13 参照 |
| r1.25 | 2026-04-26 | v8.9.3 Beacon Edition 対応：ble モジュール追加（Phase 1〜4 + Phase 7 ビーコン）・セクション 2-17 新設・旧 2-17〜2-20 を 2-18〜2-21 に繰り下げ。hw.device_id() / hw.sign() 追加（Phase 8）。Phase 5 BLE HID 削除記録。★実機確認済み |
| r1.26 | 2026-04-28 | v8.9.4 対応：gpio.pwm() / gpio.pwm_stop() 追加（PWM 出力・LED 調光・モーター速度制御）・セクション 2-8 更新・クイックリファレンス更新。★実機確認済み（GPIO2 調光・フェードアウト）。fs_write / fs_append 書き込み上限を 1KB → 4KB に更新（v8.9.4 実装済み） |
| r1.27 | 2026-04-29 | v5.9.8/v6.9.5/v8.9.5 Lab Edition 対応：adc/dac モジュール追加（アナログ入出力）・セクション 2-18 新設・旧 2-18〜2-21 を 2-19〜2-22 に繰り下げ・クイックリファレンス更新。ESP32=GPIO32-39 ADC1 + GPIO25/26 DAC。ESP32-S3=GPIO1-10 ADC1・DAC はスタブ（gpio.pwm()+RC フィルター推奨）。★全機種実機確認済み（2026-04-29） |
| r1.28 | 2026-05-10 | v8.9.6 Lab Edition 対応：ble モジュール Phase 3 / Phase 7 送信を NOT SUPPORTED に更新（CONFIG_BT_NIMBLE_EXT_ADV 無効化のため）。`ble.parse_luna_beacon()` 追加（LUNA Sub v1.0.3+ 健康テレメトリ解析）。セクション 2-17 更新。★実機確認済み |
| r1.29 | 2026-05-11 | v8.9.6 / LUNA Sub v1.0.3 Phase 7 対応：`sub` モジュール追加（`sub.send_file` / `sub.lua_run` / `sub.lua_stop`）・セクション 2-17a 新設。`hw.delay()` が BLE Notify キューを内部処理するように変更された旨を 2-3 に追記。クイックリファレンス（2-22）に sub モジュールエントリ追加。★実機確認済み |
| r1.30 | 2026-05-15 | v8.9.7 / **LUNA Node** v1.0.4 対応。「LUNA Sub」の公式名称を **LUNA Node** に変更。sub モジュール大幅拡張：`sub.send` / `sub.to_hex` / `sub.from_hex` / `sub.decode` / `sub.get_log` / `sub.read_file` / `sub.net_load` / `sub.net_save` / `sub.pair` / `sub.forget` 追加。LUNA Node ネットワーク管理（`/sub_network.json` DB）・Phase 9（Beacon `log_count`）・PAIR UUID 追加。`json.encode` ネストテーブルバグ修正。`dev.addr` フィールド名修正（`dev.address` は誤り）。セクション 2-17a 全面更新・2-22 クイックリファレンス更新。★全項目実機確認済み |

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
  - 2-7. udp モジュール — UDP 通信・Vixen Wireless Unit（v5.9.3）
  - 2-8. gpio モジュール — GPIO 直接制御・PWM 出力（v5.9.3 / v8.9.4）
  - 2-9. sensor モジュール — 温湿度センサー（v5.9.3）
  - 2-10. json モジュール — JSON parse/encode（v5.9.4）
  - 2-11. ntp モジュール — NTP 時刻同期（v5.9.4）
  - 2-12. tcp モジュール — TCP サーバー・クライアント（v5.9.2/v5.9.4a）
  - 2-13. indi モジュール — INDI 自律制御（v6.9.2）
  - 2-14. autostart — 電源ON自律起動（v5.9.5）
  - 2-15. ui モジュール — ブラウザ UI 連携（v5.9.6/v6.9.3）
  - 2-16. Console UI / File Manager — 運用ダッシュボード（v5.9.6/v6.9.3）
  - 2-17. ble モジュール — BLE スキャン・ビーコン・GATT クライアント（v8.9.3〜）
  - 2-17a. sub モジュール — LUNA Node BLE リモート制御（v8.9.7）
    - LUNA Node とは
    - UUID 一覧
    - API 一覧
    - 重要な注意事項（接続・no-op・ペアリング）
    - 標準接続フロー
    - ログ取得（sub.get_log）
    - ファイル読み出し（sub.read_file）
    - LUNA Node ネットワーク管理（net_load / net_save / pair / forget）
    - GATT CMD/RSP プロトコル早見表
  - 2-18. adc/dac モジュール — アナログ入出力（v5.9.8/v6.9.5/v8.9.5 Lab Edition）
  - 2-19. 標準 Lua 5.1 ライブラリ
  - 2-20. 数値型・16進数・ビット演算（実機確認済み）
  - 2-21. 使用できない機能
  - 2-22. クイックリファレンス一覧
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
┌──────────────────────────────────────────────────────────────┐
│                       LUNA システム                           │
│                                                              │
│  ┌──────────────────────────────┐                            │
│  │  LUNA Bridge（Main）          │  ← WiFi 接続・MCP サーバー │
│  │  AiBridgeMCP v8.9.7          │    Claude と通信            │
│  │  Lua エンジン + BLE Client   │    Lua スクリプト実行       │
│  └──────────────────────────────┘                            │
│         ↕ WiFi (MCP)          ↕ BLE (GATT)                  │
│      Claude (AI)        LUNA Node（×最大複数台）              │
│                         ┌────────────────────┐               │
│                         │  LUNA Node v1.0.4   │              │
│                         │  WiFi 不要・BLE のみ │              │
│                         │  Lua エンジン内蔵   │              │
│                         └────────────────────┘               │
│                              ↕ GPIO / ADC / センサー          │
│                           現場デバイス・センサー類             │
└──────────────────────────────────────────────────────────────┘
```

Claude は Bridge に「スクリプトを渡す」だけです。  
Bridge は BLE 経由で LUNA Node を制御し、結果を Claude に返します。

### LUNA Node（旧称: LUNA Sub）とは

**LUNA Node** は、LUNA Bridge に BLE で接続される WiFi 非搭載の ESP32-S3 デバイスです。  
独立した Lua エンジンを持ち、Bridge からのコマンドで自律的にスクリプトを実行します。

```
Bridge → ble.connect() → LUNA Node
Bridge → sub.send_file() → LUNA Node SPIFFS にスクリプト保存
Bridge → sub.lua_run() → LUNA Node でスクリプト実行開始
LUNA Node → log.write() / センサー計測 → Beacon log_count でログ通知
Bridge → ble.scan() → log_count > 0 を検出 → sub.get_log() → ログ収集
```

> 開発コード「LUNA Sub」「Sub」は **LUNA Node** の旧称です。v1.0.4 以降は **LUNA Node** を正式名称として使用します。  
> Bridge 側 Lua API では関数名プレフィックスとして `sub.` を引き続き使用します（互換性維持）。

| 項目 | 内容 |
|------|------|
| 正式名称 | LUNA システム（AiBridgeMCP v3.9.1 LUNA Edition〜） |
| Bridge | AiBridgeMCP v8.9.7（ESP32-S3、WiFi + BLE） |
| LUNA Node | LUNA Node v1.0.4（ESP32-S3、BLE のみ） |
| スクリプト言語 | Lua 5.1（Bridge / Node 両方） |
| Bridge-Claude 通信 | MCP（Model Context Protocol）over HTTP |
| Bridge-Node 通信 | BLE GATT（LUNA独自UUID）|
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
  外部機器（シリアルブリッジポート）


【経路②】Lua スクリプトが制御（Lua バインディング）

  Claude
    ↓ lua_exec / lua_run
  Lua スクリプト
    ↓ serial.write / serial.read / serial.query（Lua API）
    ↓
  ESP32 C++ 層（executeSerialQuery 等）← 同じ関数を呼ぶ
    ↓ serialMutex で排他制御
  外部機器（シリアルブリッジポート）
```

**2つの経路は内部で同じ C++ 関数を共有し、`serialMutex` で保護されています。**

> **📌 シリアルポートについて（v5.9.x）**
> v5.9.x では外部機器接続用のシリアルブリッジは **1チャンネルのみ** です。
> 使用するポートは `TARGET_BOARD` の設定によって決まります：
>
> | TARGET_BOARD | ブリッジポート | デバッグ出力 |
> |---|---|---|
> | TARGET_STANDARD | Serial2（GPIO16/17） | Serial（USB） |
> | TARGET_ONSTEPNINJA_V2 | Serial（UART0） | 無効（NullStream） |
>
> v2.x では Serial / Serial2 の動的切り替えに対応していましたが、v5.9.x では廃止されました。

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
>
> **v8.9.6 追記**: `hw.delay()` は内部で 10ms ごとに BLE Notify キューを処理します。
> `ble.on("notify", cb)` で登録したコールバックは `hw.delay()` の待機中にも呼び出されます。
> これにより `ble.notify_wait()` を使わずとも `hw.delay()` ポーリングループで RSP 受信が可能です。

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

> ⚠️ **注意**: `hw.free_heap()` は Lua API に**存在しません**。空きヒープを確認するには、Claude 側から `lua_status()` MCP ツールを使用してください（レスポンスに `free_heap` が含まれます）。

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

#### `hw.rgb(r, g, b)` — RGB LED 単色点灯（v6.9.3 ESP32-S3 専用）

> **v6.9.3 新機能** — FREENOVE ESP32-S3 WROOM の内蔵 NeoPixel WS2812B（GPIO48）を制御します。

NeoPixel を指定色で点灯します。`hw.rgb(0, 0, 0)` で消灯。

| 引数 | 型 | 説明 |
|------|----|------|
| r | number | 赤 0〜255 |
| g | number | 緑 0〜255 |
| b | number | 青 0〜255 |

```lua
hw.rgb(255, 0, 0)    -- 赤点灯
hw.delay(1000)
hw.rgb(0, 255, 0)    -- 緑点灯
hw.delay(1000)
hw.rgb(0, 0, 0)      -- 消灯
```

---

#### `hw.rgb_blink(r, g, b, duration_ms [, interval_ms])` — RGB LED 時間指定点滅（v6.9.3 ESP32-S3 専用）

指定した時間だけ点滅して自動消灯します。`hw.delay()` と同様にブロッキングですが、HTTP サーバーに CPU を譲りながら待機します。

| 引数 | 型 | 説明 |
|------|----|------|
| r, g, b | number | 色（0〜255） |
| duration_ms | number | 点滅する合計時間（ミリ秒） |
| interval_ms | number | 点灯/消灯の切り替え間隔（省略時 250ms） |

```lua
-- 黄色1秒チカチカ → 自動消灯
hw.rgb_blink(255, 200, 0, 1000)

-- 赤を2秒、100ms間隔で激しく点滅
hw.rgb_blink(255, 0, 0, 2000, 100)
```

---

#### `hw.rgb_flag(state [, r, g, b])` — RGB LED ノンブロッキング点滅（v6.9.3 ESP32-S3 専用）

バックグラウンドで点滅し続けるフラグです。`hw.delay()` と違い **Lua スクリプトの実行をブロックしません**。loop() 内で 500ms ごとに ON/OFF を切り替えます。

| 引数 | 型 | 説明 |
|------|----|------|
| state | number | `1` で開始、`0` で停止・消灯 |
| r, g, b | number | 色（省略時は黄 255, 200, 0） |

```lua
-- GoTo 開始: 黄点滅スタート（スクリプトは次の処理へ進む）
hw.rgb_flag(1, 255, 200, 0)

-- GoTo 中に他の処理を実行できる
serial.write(":MS#")
hw.delay(3000)   -- 待機中も NeoPixel は点滅し続ける

-- GoTo 完了: 消灯 → 緑で成功表示
hw.rgb_flag(0)
hw.rgb(0, 255, 0)
hw.delay(1000)
hw.rgb(0, 0, 0)
```

> ⚠️ **ESP32-S3 専用**: `hw.rgb` / `hw.rgb_blink` / `hw.rgb_flag` は v6.9.3（ESP32-S3）のみで使用できます。

---

#### `hw.device_id()` — デバイス固有 ID 取得（v8.9.3）

ESP32 の EFuse MAC アドレスを 12桁の大文字 HEX 文字列で返します。

```lua
local id = hw.device_id()
log.write("Device ID: " .. id)   -- 例: "74ABF84EB580"
```

- 電源を切っても変わらない固有 ID です
- ビーコン送信時の送信元識別子として使用します
- ライセンス認証の Device ID と同一の値です

---

#### `hw.sign(msg)` — HMAC-SHA256 署名（v8.9.3）

Activation Key を秘密鍵として、メッセージの HMAC-SHA256 署名を生成します。  
署名は 64桁の小文字 HEX 文字列で返ります。  
Activation Key は C 層で管理されており、Lua スクリプトから直接読み取ることはできません（セキュリティ設計）。

```lua
local sig, err = hw.sign("hello_beacon_2026")
if sig then
  log.write("署名: " .. sig)   -- 64桁 HEX
else
  log.write("エラー: " .. err)  -- Activation Key 未設定など
end
```

- **前提**: Web 設定画面（`http://<LUNA-IP>/config`）で Activation Key を設定すること
- Activation Key 未設定時は `nil, "Activation Key not set"` を返します
- ビーコンパケットへの署名付与・改ざん検出に使用します
> v5.9.6（ESP32）では利用できません。

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

### 2-6. http モジュール — HTTP通信・Alpaca 連携（v5.9.1 / v5.9.2）

> **v5.9.1 新機能** — LUNA Observatory から追加されました。http.post() は **v5.9.2** で追加。

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

#### `http.post(url, body, content_type, timeout_ms)` — POST 送信（v5.9.2）

> **v5.9.2 新機能** — NINA Advanced API / Stellarium Remote Control に対応。
> **2026-03-27 実機確認済み** — NINA v3.2 シミュレーターカメラ接続・撮影、Stellarium Remote Control 天体選択・座標取得を確認。

```lua
local resp, code = http.post(url, body, content_type, timeout_ms)
```

| 引数 | 説明 |
|------|------|
| `url` | リクエスト先 URL |
| `body` | リクエストボディ（省略時 `""`） |
| `content_type` | Content-Type（省略時 `"application/json"`） |
| `timeout_ms` | タイムアウト（省略時 2000ms） |
| 戻り値1 `resp` | レスポンスボディ文字列。エラー時は `""` |
| 戻り値2 `code` | HTTP ステータスコード（200=成功、負値=接続エラー） |

> **注意**: `http.get` / `http.put` との違い — `http.post` はステータスコードも返します（2値）。

```lua
-- NINA Advanced API v2 への POST（JSON ボディ）
local resp, code = http.post(
  "http://192.168.3.23:1888/v2/api/equipment/camera/capture",
  '{"Duration":5.0,"SaveImage":true}',
  "application/json",
  10000
)
if code == 200 then
  return '{"result":"capture started"}'
else
  return string.format('{"error":"http %d"}', code)
end

-- Stellarium Remote Control への POST（フォーム形式）
local resp, code = http.post(
  "http://192.168.3.23:8090/api/main/focus",
  "target=100",
  "application/x-www-form-urlencoded",
  3000
)
```

#### 用途別メソッド選択

| 用途 | メソッド | Content-Type |
|------|---------|-------------|
| ASCOM/Alpaca 値読み取り | `http.get` | — |
| ASCOM/Alpaca コマンド送信 | `http.put` | `application/x-www-form-urlencoded`（固定） |
| NINA Advanced API v2 | `http.post` | `application/json`（デフォルト） |
| Stellarium Remote Control | `http.post` | `application/x-www-form-urlencoded` |

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

### 2-7. udp モジュール — UDP 通信（v5.9.3）

> **v5.9.3 新機能**
> UDP データグラム通信を Lua スクリプトから直接行えます。
> UDP ループバックテスト確認済み（2026-03-28）。

#### API リファレンス

```lua
udp.begin(local_port)              -- UDP ソケットを開く（受信ポート指定）
udp.connect(host, port)            -- 送信先ホスト・ポートを設定
udp.write(str)                     -- UDP パケットを送信（true/false）
local resp = udp.read([timeout_ms])-- UDP パケットを受信（デフォルト 1000ms）
udp.close()                        -- ソケットを閉じる
```

#### 基本的な使い方

```lua
-- UDP 送受信の基本パターン
udp.begin(60023)               -- ローカルポートを開く
udp.connect("192.168.x.x", 60023)  -- 送信先を設定
udp.write("コマンド\r\n")     -- 送信
local resp = udp.read(1000)    -- 受信（1000ms タイムアウト）
udp.close()                    -- 閉じる
```

#### 対応機器

| 機器 | IP | ポート |
|------|-----|--------|
| **Vixen Wireless Unit** | 192.168.6.1 | 60023 |
| 汎用 UDP 機器 | 任意 | 任意 |

> **Vixen 機器のコマンド仕様については `prompts/get vixen_guide` を参照してください。**
> コマンドリファレンス（Wireless Unit / STAR BOOK TEN 両対応）が Claude に提供されます。

#### ネットワーク設定の注意

- Vixen Wireless Unit は初期状態で AP モード（IP: 192.168.6.1）で動作
- LUNA と同一ネットワークで使用するには Wireless Unit を Infrastructure モードに変更が必要
- Infrastructure モードへの変更方法は `prompts/get vixen_guide` に記載

---

### 2-8. gpio モジュール ── GPIO 直接制御（v5.9.3 / v8.9.4）

> **v5.9.3 新機能**
> ESP32 の GPIO ピンを Lua スクリプトから直接制御します。
> フォーカサーモーター、リレー、外部 LED などの駆動に使用します。
> Output 1〜4 全ピン 実機確認済み（2026-03-28）。
>
> **v8.9.4 追加**: `gpio.pwm()` / `gpio.pwm_stop()` — PWM 出力（LED 調光・ファン速度制御など）。実機確認済み（2026-04-27）。

#### Output ピン割り当て（OnStepNinja V2）

| Output | GPIO | 用途 |
|--------|------|------|
| Output 1 | GPIO18 | PLS（ステッパー STEP パルス）|
| Output 2 | GPIO0  | DIR（ステッパー方向）|
| Output 3 | GPIO27 | LED（`hw.led()` と共用）|
| Output 4 | GPIO26 | 未割当 |

#### `gpio.output_pin(n)`

Output n（1〜4）に割り当てられた GPIO 番号を返します。ピン番号をハードコードせずに使用できます。

| 引数 | 型 | 説明 |
|------|----|------|
| n | number | Output 番号（1〜4） |

| 戻り値 | 型 | 説明 |
|--------|----|------|
| gpio_num | number | GPIO ピン番号。範囲外の場合は nil |

```lua
local pls = gpio.output_pin(1)  -- → 18 (GPIO18 PLS)
local dir = gpio.output_pin(2)  -- → 0  (GPIO0  DIR)
local led = gpio.output_pin(3)  -- → 27 (GPIO27 LED)
local o4  = gpio.output_pin(4)  -- → 26 (GPIO26)
```

---

#### `gpio.write(pin, value)`

GPIO ピンに HIGH / LOW を出力します。

| 引数 | 型 | 説明 |
|------|----|------|
| pin | number | GPIO ピン番号 |
| value | number | 0 = LOW / 1 = HIGH |

```lua
local pls = gpio.output_pin(1)
gpio.write(pls, 1)  -- HIGH
hw.delay(5)
gpio.write(pls, 0)  -- LOW
```

---

#### `gpio.read(pin)`

GPIO ピンの状態を読み取ります。

| 引数 | 型 | 説明 |
|------|----|------|
| pin | number | GPIO ピン番号 |

| 戻り値 | 型 | 説明 |
|--------|----|------|
| value | number | 0 = LOW / 1 = HIGH |

```lua
local state = gpio.read(gpio.output_pin(3))
log.write("LED pin state: " .. state)
```

---

#### `gpio.mode(pin, mode)`

GPIO ピンのモードを設定します。Output 1〜4 は setup() で自動的に OUTPUT に設定済みのため、通常は不要です。

| 引数 | 型 | 説明 |
|------|----|------|
| pin | number | GPIO ピン番号 |
| mode | number | 0 = INPUT / 1 = OUTPUT |

```lua
gpio.mode(19, 0)  -- GPIO19 を INPUT に設定
gpio.mode(19, 1)  -- GPIO19 を OUTPUT に設定
```

---

#### `gpio.pwm(pin, freq, duty)` ── PWM 出力（v8.9.4）

指定した GPIO ピンで PWM 信号を出力します。LED 調光・ファン速度制御・ブザー駆動などに使用します。

| 引数 | 型 | 説明 |
|------|----|------|
| pin | number | GPIO ピン番号 |
| freq | number | 周波数（Hz）。例：1000（1kHz） |
| duty | number | デューティ比（0〜255）。0=0% / 128=50% / 255=100% |

> **内部動作**: LEDC チャネルを自動割り当て。最大 8ch 同時使用可能。

```lua
-- LED を 50% の明るさで点灯
gpio.pwm(2, 1000, 128)

-- フェードアウト
for d = 255, 0, -5 do
  gpio.pwm(2, 1000, d)
  hw.delay(20)
end
gpio.pwm_stop(2)
```

---

#### `gpio.pwm_stop(pin)` ── PWM 停止（v8.9.4）

PWM 出力を停止し、LEDC チャネルを解放します。

| 引数 | 型 | 説明 |
|------|----|------|
| pin | number | GPIO ピン番号 |

```lua
gpio.pwm_stop(2)  -- GPIO2 の PWM を停止
```

---

#### ステッパーモーター制御の例（フォーカサー）

```lua
-- focuser_step.lua: ステッパーモーターを指定ステップ数動かす
local pls = gpio.output_pin(1)  -- GPIO18 STEP
local dir = gpio.output_pin(2)  -- GPIO0  DIR
local STEP_DELAY = 5            -- ms（200 steps/sec）
local steps = 200               -- ステップ数

-- IN方向に移動
gpio.write(dir, 1)
hw.delay(1)
for i = 1, steps do
  gpio.write(pls, 1); hw.delay(STEP_DELAY)
  gpio.write(pls, 0); hw.delay(STEP_DELAY)
end
log.write("Moved " .. steps .. " steps IN")
```

> ⚠️ **注意**: `hw.led()` と `gpio.write(gpio.output_pin(3), ...)` はどちらも GPIO27 を制御します。混在しても問題ありませんが、どちらかに統一することを推奨します。

---

### 2-9. sensor モジュール ── 温湿度センサー（v5.9.3）

> **v5.9.3 新機能**
> DHT22 温湿度センサーを Lua スクリプトから直接読み取ります。
> GPIO5 に接続し、温度・湿度・露点の取得に使用します。
> エラー値は -999.0（AiBridge 統一仕様）。実機確認済み（2026-03-28）。

#### ピン定義

| ピン | GPIO | センサー端子 |
|------|------|------------|
| DHT_PIN | GPIO5 | DHT22 DATA |

---

#### `sensor.dht22([pin])`

DHT22 センサーから温度と湿度を読み取ります。内部で最大3回リトライします。

| 引数 | 型 | 説明 |
|------|----|------|
| pin | number | GPIO ピン番号（省略時 = GPIO5）|

| 戻り値 | 型 | 説明 |
|--------|----|------|
| temperature | number | 温度（℃）。読み取り失敗時は -999.0 |
| humidity | number | 湿度（%）。読み取り失敗時は -999.0 |

```lua
-- 基本使用例
local temp, humi = sensor.dht22()
if temp == -999.0 then
  log.write("DHT22 read error")
else
  log.write(string.format("T=%.1f°C H=%.1f%%", temp, humi))
end

-- ピン番号明示
local temp, humi = sensor.dht22(5)  -- GPIO5
```

---

#### 露点計算（Magnus 公式）

```lua
local temp, humi = sensor.dht22()
if temp ~= -999.0 then
  local a, b = 17.27, 237.7
  local alpha = (a * temp / (b + temp)) + math.log(humi / 100.0)
  local dew = b * alpha / (a - alpha)
  log.write(string.format("T=%.1f°C H=%.1f%% Dew=%.1f°C", temp, humi, dew))
end
```

---

#### MCP Resource: `luna://lua/weather`

`lua_resource_weather.lua` を SPIFFS に保存することで Claude が直接読み取れます。

```lua
-- lua_resource_weather.lua の内容
local temp, humi = sensor.dht22()
if temp == -999.0 then
  return '{"temperature":-999.0,"humidity":-999.0,"dewpoint":-999.0,"error":"DHT22 read failed"}'
end
local a, b = 17.27, 237.7
local alpha = (a * temp / (b + temp)) + math.log(humi / 100.0)
local dew = b * alpha / (a - alpha)
return string.format(
  '{"temperature":%.1f,"humidity":%.1f,"dewpoint":%.1f}',
  temp, humi, dew)
```

Claude が `resources/read → luna://lua/weather` を呼ぶと以下が返ります：
```json
{"temperature":19.1,"humidity":59.3,"dewpoint":11.0}
```

---

### 2-10. json モジュール ── JSON parse/encode（v5.9.4）

> **v5.9.4 新機能**
> ArduinoJson v7 を Lua にバインドした JSON 変換モジュールです。
> HTTP レスポンスや MCP Resource の JSON を簡単に扱えます。実機確認済み（2026-03-29）。

---

#### `json.parse(str)`

JSON 文字列を Lua テーブルに変換します。

| 引数 | 型 | 説明 |
|------|----|------|
| str | string | JSON 文字列 |

| 戻り値 | 型 | 説明 |
|--------|----|------|
| table | table | パース結果。失敗時は nil |
| err | string | エラーメッセージ（成功時は nil）|

```lua
local resp = http.get("http://192.168.3.23:1888/api/camera", 3000)
local data, err = json.parse(resp)
if data then
  log.write("Camera: " .. tostring(data.name))
else
  log.write("parse error: " .. tostring(err))
end
```

---

#### `json.encode(val)`

Lua テーブル・数値・文字列を JSON 文字列に変換します。

| 引数 | 型 | 説明 |
|------|----|------|
| val | any | Lua 値（テーブル・数値・文字列・真偽値）|

| 戻り値 | 型 | 説明 |
|--------|----|------|
| str | string | JSON 文字列 |

```lua
-- オブジェクト（文字列キー → JSON object）
local body = json.encode({temperature=19.5, humidity=58, active=true})
-- {"temperature":19.5,"humidity":58,"active":true}

-- 配列（整数キー 1..n → JSON array）
local arr = json.encode({10, 20, 30})
-- [10,20,30]
```

---

#### MCP Resource での活用

```lua
-- lua_resource_camera.lua
local resp = http.get("http://192.168.3.23:1888/api/camera", 3000)
local cam = json.parse(resp)
if cam then
  return json.encode({name=cam.name, temp=cam.temperature, cooling=cam.canSetTemperature})
end
return '{"error":"camera not available"}'
```

---

### 2-11. ntp モジュール ── NTP 時刻同期（v5.9.4）

> **v5.9.4 新機能**
> ESP32 内蔵 SNTP クライアント（configTime()）を Lua から利用できます。
> 追加ライブラリ不要。天体観測の時刻管理・ユリウス日計算に対応。実機確認済み（2026-03-29）。

---

#### `ntp.sync([server])`

NTP サーバーへの同期を開始します（非ブロッキング）。

| 引数 | 型 | 説明 |
|------|----|------|
| server | string | NTP サーバー（省略時: "pool.ntp.org"）|

```lua
ntp.sync()              -- デフォルトサーバー（世界中で利用可能）
ntp.sync("ntp.nict.jp") -- 日本: NICT（国立研究開発法人情報通信研究機構）
```

---

#### `ntp.synced()`

時刻同期済みなら `true` を返します。

```lua
ntp.sync("ntp.nict.jp")
hw.delay(5000)  -- 同期完了を待つ
if ntp.synced() then
  log.write("同期完了")
end
```

---

#### `ntp.unix()`

UTC Unix タイムスタンプ（1970-01-01 からの秒数）を返します。

---

#### `ntp.utc()`

UTC 時刻を文字列で返します。

```lua
log.write("UTC: " .. ntp.utc())  -- "2026-03-29 04:08:35"
```

---

#### `ntp.jd()`

ユリウス日（Julian Date）を返します。天体計算の基準時刻として使用します。

```lua
log.write("JD: " .. string.format("%.5f", ntp.jd()))  -- "2461128.67263"
```

---

#### 使用例（NTP 同期 → 観測記録）

```lua
ntp.sync("ntp.nict.jp")
hw.delay(5000)
if ntp.synced() then
  local utc = ntp.utc()
  local jd  = ntp.jd()
  log.write(utc .. " 観測開始")
  log.write(string.format("JD=%.5f", jd))
  log.save()
end
```

---

### 2-12. tcp モジュール ── TCP サーバー・クライアント（v5.9.2 / v5.9.4a）

> **v5.9.2 新機能** / **v5.9.4a で tcp.disconnect() 追加**
> LUNA を TCP サーバーまたは TCP クライアントとして動作させます。
> SkySafari（天文アプリ）からの LX200 WiFi 接続に対応。実機確認済み（2026-03-30）。

---

#### TCP サーバー（LUNA = 待ち受け側）

SkySafari などのアプリが LUNA に接続してくるパターン。

| 関数 | 戻り値 | 説明 |
|------|--------|------|
| `tcp.begin(port)` | なし | 指定ポートで TCP サーバーを起動 |
| `tcp.connected()` | bool | クライアント接続確認。未接続なら新規接続を自動受付 |
| `tcp.read()` | string | 受信データ取得（`#` 終端検出・最大 20ms 待機） |
| `tcp.write(str)` | なし | クライアントへデータ送信 |
| `tcp.disconnect()` | なし | **[v5.9.4a]** クライアント切断・サーバーは継続（次接続を即受付） |
| `tcp.stop()` | なし | サーバー停止・全リソース解放 |

---

#### TCP クライアント（LUNA = 接続する側）

PHD2・NINA・Stellarium など外部サービスへ LUNA から接続するパターン。

| 関数 | 戻り値 | 説明 |
|------|--------|------|
| `tcp.client_connect(host, port [, ms])` | bool | 外部サーバーへ接続。デフォルト timeout=1000ms |
| `tcp.client_connected()` | bool | 接続中か確認 |
| `tcp.client_write(str)` | なし | データ送信（バイナリ対応） |
| `tcp.client_read([ms])` | string | データ受信。デフォルト timeout=500ms |
| `tcp.client_close()` | なし | 接続を切断 |

---

#### 使用例 1: SkySafari LX200 WiFi ブリッジ（TCP サーバー）

SkySafari から LUNA に接続し、NS-5000 などの LX200 互換望遠鏡を WiFi で制御します。

**SkySafari 設定:**
- Telescope Type: Meade LX200 Classic
- Connection: WiFi
- IP: LUNA の STA IP（`get_network_status` で確認）
- Port: 4030

```lua
-- luna_skysafari_bridge.lua
-- SkySafari 実機確認済み（2026-03-30）: NS-5000 + GoTo 動作確認

local PORT = 4030
tcp.begin(PORT)
log.write("Ready on port " .. PORT)

while true do
  if tcp.connected() then
    local idle = 0

    -- 同一接続内で複数コマンドを処理するループ
    -- SkySafari は :RC# などの応答なしコマンドの後に同じ接続で :GD# を送る場合がある
    while idle < 5 do
      local cmd = tcp.read()
      if cmd and #cmd > 0 then
        idle = 0
        local resp = serial.query(cmd, 100)
        if resp and #resp > 0 then
          tcp.write(resp)
          break  -- 応答ありコマンド: 送信後に接続終了
        end
        -- 応答なしコマンド (:U# :RC# 等): 接続維持して次コマンドを待つ
      else
        idle = idle + 1
        hw.delay(20)
      end
    end

    tcp.disconnect()
  end
end
```

> **設計上の注意（2026-03-30 実測判明）**
> SkySafari は基本的に「1接続1コマンド」方式ですが、`:RC#` のような
> 応答なしコマンドの後に同じ TCP 接続内で `:GD#` を続けて送信するケースがあります。
> 上記スクリプトのループ構造がこれに対応しています。

---

#### 使用例 2: PHD2 ガイドソフト連携（TCP クライアント）

```lua
-- PC 上の PHD2 に接続してガイド状態を取得
local ok = tcp.client_connect("192.168.3.23", 4400, 2000)
if ok then
  tcp.client_write('{"method":"get_app_state","id":1}\r\n')
  local resp = tcp.client_read(1000)
  log.write("PHD2: " .. tostring(resp))
  tcp.client_close()
end
```

---

#### 使用例 3: Stellarium Remote Control（TCP クライアント）

```lua
-- Stellarium の現在選択天体を取得（ポート 8090）
local ok = tcp.client_connect("192.168.3.23", 8090, 2000)
if ok then
  -- ※ Stellarium は HTTP プロトコル。http.get() 推奨
  tcp.client_close()
end
```

> Stellarium・NINA との連携は `http.get()` / `http.post()` を使う方が簡便です（セクション 2-6 参照）。

---

#### Readout Rate の目安

SkySafari の Readout Rate（設定画面）と安定性の関係：

| Rate | 間隔 | 安定性 |
|------|------|--------|
| 1〜6/sec | 167ms〜1000ms | ✅ 安定 |
| 7〜10/sec | 100〜143ms | ⚠ serial.query タイムアウトと競合する場合あり |

**推奨: 4/sec**（デフォルト値・実機確認済み）

---

### 2-13. indi モジュール ── INDI 自律制御（v6.9.2）

> **詳細リファレンス**: `get_lua_guide` の **セクション 2-13** を参照してください。GoTo ワークフロー・全 API・スクリプト例が記載されています。

INDI（Instrument-Neutral Distributed Interface）経由で望遠鏡・天体機器を Lua スクリプトから自律制御するモジュールです。FreeRTOS バックグラウンドタスクで常時 XML ストリームを受信・キャッシュします。**v6.9.3（ESP32-S3）専用**。

#### API 一覧

| 関数 | 説明 |
|------|------|
| `indi.connect(host, port)` | INDI サーバー接続・BG タスク起動 |
| `indi.disconnect()` | 切断（キャッシュ保持） |
| `indi.status()` | 接続状態・統計テーブル返却 |
| `indi.cache_size()` | キャッシュ内デバイス数 |
| `indi.devices()` | デバイス名の配列 |
| `indi.properties(device)` | プロパティ一覧（name/type/state） |
| `indi.get(device, prop, elem)` | 要素値取得（文字列） |
| `indi.state(device, prop)` | 状態取得（`Idle`/`Ok`/`Busy`/`Alert`） |
| `indi.set_number(device, prop, {name=val,...})` | Number プロパティ送信 |
| `indi.set_switch(device, prop, {name=val,...})` | Switch プロパティ送信 |
| `indi.set_text(device, prop, {name=val,...})` | Text プロパティ送信 |
| `indi.wait_state(device, prop, state, ms)` | 状態変化待機（true/false） |
| `indi.set_blob(device, mode)` | BLOB モード設定（`'Never'` 推奨） |

> ⚠️ `indi.connect()` 後は `hw.delay(3000)` でキャッシュ構築を待つこと。  
> ⚠️ `indi.on_update()` は未実装（スタブ）。  
> ⚠️ MCP ツール（`indi_connect` 等）と同時呼び出し不可。

---

### 2-14. autostart — 電源ON自律起動（v5.9.5）

> **実機確認済み（2026-03-30）**: NS-5000 + SkySafari / 接続・アライメント・GoTo 動作確認

autostart は LUNA の電源を入れるだけで指定スクリプトを自動実行する機能です。
MCPサーバーへの依頼は不要になり、LUNA がスタンドアロン機器として動作します。

---

#### 基本的な使い方

**1. スクリプトを保存する**

```
lua_save("skysafari_bridge", <スクリプト内容>)
```

**2. autostart に設定する（MCP ツール）**

```
set_autostart: name = "skysafari_bridge"
```

**3. 再起動する**

```
hard_reset
```

以後、電源を入れるたびに `skysafari_bridge` が自動起動します。

---

#### 動作の流れ

```
電源ON
  ↓
setup()：WiFi 接続・MCP サーバー起動
  ↓
/luna_autostart.txt を読み込む
  ↓
スクリプト名が設定されていれば → lua_run() で自動実行
  ↓
MCP サーバーも同時に動作（Claude からの操作も可能）
```

---

#### MCP ツール（Claude が使う）

| ツール | 機能 |
|--------|------|
| `set_autostart` | 起動スクリプトを設定（name パラメーター） |
| `get_autostart` | 現在の設定を確認 |
| `clear_autostart` | autostart を無効化 |

**get_autostart のレスポンス例：**

```json
{
  "autostart_script": "skysafari_bridge",
  "safe_mode": false,
  "boot_count": 0
}
```

**get_system_info の拡張フィールド（v5.9.5）：**

```json
{
  "autostart_script": "skysafari_bridge",
  "safe_mode": false,
  "boot_count": 0
}
```

---

#### autostart の解除

```
clear_autostart
```

または `/luna_autostart.txt` を削除：

```
fs_delete: path = "/luna_autostart.txt"
```

---

#### セーフモード

autostart が暴走・クラッシュループに陥った場合の保護機能です。

| 条件 | 動作 |
|------|------|
| **連続クラッシュ3回**（RTC カウンター ≥ 3） | 自動セーフモード |
| **起動時 GPIO39 長押し**（3秒以上・5秒以内） | 手動セーフモード |

**セーフモード時の動作：**

- autostart をスキップ（スクリプトは実行されない）
- File Manager が自動開放される（ボタン不要）
- LED（GPIO25）が点灯してセーフモードを明示
- MCP サーバーは通常通り動作（Claude から修正可能）
- `boot_count` がリセットされる → 次回は通常起動

**セーフモードの確認：**

```
get_system_info
→ "safe_mode": true
```

**セーフモードの解除：**
GPIO39 を押さずにリセット（または電源入れ直し）するだけで通常モードに戻ります。
セーフモード自体は1回の起動限り有効です。

---

#### クラッシュ検出の仕組み

```
autostart 実行 → rtcAutostartActive = true（RTC メモリ）
  ↓（クラッシュ・ウォッチドッグ）
再起動 → rtcAutostartActive を検出 → boot_count++
  ↓（3回繰り返すと）
boot_count ≥ 3 → セーフモード発動
```

**正常動作時：**
スクリプト起動から30秒後に `boot_count` が自動でリセットされます。

> **注意**: `hard_reset` MCP ツール（内部で `ESP.restart()` を使用）は RTC メモリを保持します。
> autostart 起動後30秒以内に `hard_reset` を3回繰り返すと、意図せずセーフモードになる可能性があります。
> 開発中の頻繁な再起動には電源断→投入を推奨します。

---

#### 耐性テスト結果（2026-03-30）

| テスト | 条件 | 結果 |
|--------|------|------|
| ファイル削除 | `/luna_autostart.txt` なし | autostart スキップ・正常動作 ✅ |
| 存在しないスクリプト | `"nonexistent_script"` | エラー無視・クラッシュなし ✅ |
| 不正な内容 | `"!!!INVALID@#$%%%"` | エラー無視・クラッシュなし ✅ |
| 手動セーフモード | GPIO39 長押し + リセット | `safe_mode: true`・Lua idle ✅ |
| セーフモード後復帰 | GPIO39 なしでリセット | 通常モード・autostart 実行 ✅ |

---

#### 実用例：SkySafari スタンドアロンアダプター

```
1. lua_save("skysafari_bridge", <ブリッジスクリプト>)
2. set_autostart: name = "skysafari_bridge"
3. 以後は電源を入れるだけ
```

→ **天文台でLUNA の電源を入れる → SkySafari で即接続できる**

---

### 2-15. ui モジュール — ブラウザ UI 連携（v5.9.6/v6.9.3）

#### 概要

`ui` モジュールは、Lua スクリプトとブラウザ上の HTML ページをリアルタイムで連携させるフレームワークです。
Claude との通信なしに、**スマートフォン・PC のブラウザから直接スクリプトを操作**できます。

```
┌─────────────┐   HTTP ポーリング   ┌──────────────────┐
│  ブラウザ   │ ←── /api/ui/messages │  LUNA (ESP32)    │
│  HTML UI    │                     │  Lua スクリプト  │
│  ボタン操作 │ ──── /api/ui/button ─→│  ui.check()      │
└─────────────┘                     └──────────────────┘
```

> **⚠️ /api/ui/button のボタン送信形式**  
> ボタン名は **クエリパラメータ `?name=xxx`** で送ります。JSON body ではありません。  
> ```javascript
> // ✅ 正しい
> fetch('/api/ui/button?name=on', {method:'POST'})
> // ❌ 誤り（ui.check() に届かない）
> fetch('/api/ui/button', {method:'POST', body:'{"name":"on"}'})
> ```

> **⚠️ /api/ui/messages のレスポンス形式**  
> `{"key1":"value1","key2":"value2"}` の **JSON Object** を返します（配列 `[]` ではありません）。  
> JavaScript で受け取る際は `for...of`（配列専用）ではなく `Object.entries()` を使います。
>
> ```javascript
> const d = await (await fetch('/api/ui/messages')).json();
> for (const [k, v] of Object.entries(d)) {
>   const e = document.getElementById(k);
>   if (e) e.textContent = v;
> }
> ```

#### API リファレンス

| 関数 | 説明 |
|------|------|
| `ui.send(id, message)` | ブラウザ側の要素 `id` にメッセージを送信（即座に表示） |
| `ui.check()` | ブラウザから送られたボタン名を取得（ノンブロッキング）。なければ `""` を返す |
| `ui.poll(timeout_ms)` | ボタン入力を待機（ブロッキング）。タイムアウト時は `""` を返す |
| `ui.switch(url)` | ブラウザを指定 URL にリダイレクト |
| `ui.clear()` | ブラウザ側メッセージキューをクリア |

#### ui.send() — メッセージ送信

```lua
-- ブラウザの id="msg" 要素にテキストを表示
ui.send("msg", "温度: 25.3°C")

-- 複数の要素に送信
ui.send("status", "● 観測中")
ui.send("ra",     string.format("RA: %s", ra_str))
ui.send("dec",    string.format("Dec: %s", dec_str))
```

HTML 側では、`id` に一致する要素（`div`, `span`, `p` など）が自動更新されます:
```html
<div id="status">---</div>
<div id="ra">---</div>
<div id="dec">---</div>
```

#### ui.check() — ノンブロッキングボタン受信

```lua
-- メインループ内で定期チェック
while true do
    local btn = ui.check()
    if btn == "stop" then
        log.write("停止コマンド受信")
        break
    elseif btn == "measure" then
        do_measurement()
    end
    -- その他の処理
    hw.delay(100)
end
```

HTML 側のボタン定義（`data-btn` 属性でボタン名を指定）:
```html
<button data-btn="stop">停止</button>
<button data-btn="measure">測定</button>
```

#### ui.poll() — ブロッキング待機

```lua
-- ユーザー入力を最大10秒待つ
local btn = ui.poll(10000)
if btn == "" then
    log.write("タイムアウト")
elseif btn == "start" then
    log.write("開始ボタンが押されました")
end
```

#### テキスト入力・スライダーの受信

ブラウザ側でテキスト入力やスライダーを使う場合、`input:値` や `slider:値` の形式でボタン名として送信されます:

```lua
local btn = ui.poll(30000)
if btn:sub(1, 6) == "input:" then
    local text = btn:sub(7)
    log.write("入力値: " .. text)
elseif btn:sub(1, 7) == "slider:" then
    local val = tonumber(btn:sub(8)) or 0
    log.write("スライダー: " .. val)
end
```

HTML 側:
```html
<!-- テキスト入力 -->
<input type="text" id="txt-input">
<button onclick="sendInput()">送信</button>
<script>
function sendInput() {
  fetch('/api/ui/button?name=input:' + document.getElementById('txt-input').value);
}
</script>

<!-- スライダー -->
<input type="range" min="0" max="100"
  oninput="fetch('/api/ui/button?name=slider:'+this.value)">
```

#### 完全なサンプルスクリプト

```lua
-- ui_demo.lua — ブラウザ UI デモ
local count = 0
local running = true

ui.send("title", "LUNA UI デモ")
ui.send("status", "● 動作中")

while running do
    count = count + 1
    ui.send("counter", string.format("カウント: %d", count))
    ui.send("uptime",  string.format("稼働: %d 秒", math.floor(hw.millis() / 1000)))

    local btn = ui.check()
    if btn == "stop" then
        running = false
    elseif btn == "reset" then
        count = 0
        ui.send("status", "リセットしました")
    end

    hw.delay(1000)
end

ui.send("status", "● 停止")
log.write("スクリプト終了")
notify.set("完了")
```

#### HTML ファイル命名規則（重要）

**スクリプト名 = HTML ファイル名** の規則を守ることで、Console UI の **[↗ Open UI]** ボタンが自動的に正しい HTML を開きます。

| autostart スクリプト名 | SPIFFS 保存ファイル名 | Open UI が開く URL |
|----------------------|--------------------|--------------------|
| `ui_demo` | `ui_demo.html` | `/ui_demo.html` |
| `telescope_ctrl` | `telescope_ctrl.html` | `/telescope_ctrl.html` |
| `weather_monitor` | `weather_monitor.html` | `/weather_monitor.html` |

#### HTML を SPIFFS に保存する方法

`fs_write` / `fs_append` は **1回あたり最大 4KB（4096バイト）** まで書き込めます（v8.9.4 以降）。  
HTML が 4KB を超える場合は分割書き込みを使います：

```
1. fs_write:  path="/my_app.html", content=<HTMLの最初の部分（〜3800B）>
2. fs_append: path="/my_app.html", content=<続き（〜3800B）>
3. fs_append: path="/my_app.html", content=<残り>
```

または **File Manager**（`/dashboard` → File Manager ボタン）で直接アップロード（推奨）。

#### アプリ展開ワークフロー（確立版）

```
1. lua_save("my_app", <スクリプト>)          ← Lua スクリプトを保存
2. fs_write / fs_append で my_app.html 保存  ← HTML UI を保存
   （または File Manager でアップロード）
3. set_autostart: name = "my_app"            ← autostart に登録
4. /dashboard を開く
   → [▶ Run] でスクリプト起動
   → [↗ Open UI] で HTML UI を新タブで表示
5. 電源ON時は autostart が自動起動
```

#### 実機確認済みシーケンス（2026-04-14〜15）

```
テスト環境: ESP32-S3  STA IP: 192.168.3.50
スクリプト: ui_browser_test.lua
HTML: ui_browser_test.html

・ui.send()  → ブラウザにリアルタイム表示 ✅
・ボタン stop/goto/next/prev → Lua 側受信 ✅
・テキスト入力 input:xxx パターン → 受信・ログ確認 ✅
・スライダー slider:値 パターン → 受信確認 ✅
・メモリ安定性: free_heap 250KB 台維持 ✅
```

> ⚠️ **Lua 5.1 注意**: `//` 演算子（整数除算）は Lua 5.3 以降のみ。  
> Lua 5.1 では `math.floor(x / y)` を使ってください。

---

### 2-16. Console UI / File Manager — 運用ダッシュボード（v5.9.6/v6.9.3）

#### Console UI（ダッシュボード）とは

`/dashboard` で開く管理パネルです。Claude（MCP）なしに、**ブラウザだけで LUNA を操作**できます。

```
http://192.168.3.50/dashboard
```

```
┌──────────────────────────────────────────────┐
│ LUNA v6.9.3 INDI Edition                    │
│                                              │
│ Lua Script                                   │
│ autostart: ui_browser_test                   │
│ state: ● running — ui_browser_test           │
│ [▶ Run]  [■ Stop]  [↗ Open UI]              │
│                                              │
│ [File Manager] （電源ON後30分間有効）         │
└──────────────────────────────────────────────┘
```

#### HTTP API

| エンドポイント | 説明 |
|--------------|------|
| `GET /dashboard` | Console UI HTML |
| `GET /api/dashboard/status` | firmware, edition, uptime, heap, lua_running, lua_script, autostart |
| `POST /api/lua/run` | autostart スクリプトを即時実行（二重起動防止） |
| `POST /api/lua/stop` | 実行中スクリプトを強制停止 |

#### [▶ Run] / [■ Stop] ボタン

| ボタン | 有効条件 | 動作 |
|--------|---------|------|
| `[▶ Run]` | autostart 設定済み・Lua 停止中 | autostart スクリプトを起動 |
| `[■ Stop]` | Lua 実行中 | スクリプトを強制停止 |
| `[↗ Open UI]` | autostart 設定済み | `/{autostart名}.html` を新タブで開く |

ダッシュボードは **2秒ごとに `/api/dashboard/status` をポーリング**して状態を自動更新します。

#### File Manager

電源投入後 **30分間**、ファイルのアップロード・削除が可能です。

| 項目 | 内容 |
|------|------|
| アクセス可能期間 | 電源ON後 30 分間（自動・ボタン不要） |
| アクセス方法 | `/dashboard` → [File Manager] ボタン |
| 主な用途 | HTML ファイルのアップロード・Lua スクリプト管理 |
| 注意 | 30 分経過後は再起動が必要 |

> ✅ **GPIO39 の変更（v5.9.6/v6.9.3）**: 旧バージョンでは GPIO39 ボタン押下で File Manager を有効化していましたが、v5.9.6/v6.9.3 から**電源ON後 30 分間自動開放**に変更。GPIO39 は現在 **Safe Mode 専用**（起動5秒以内に3秒 LOW 長押し）です。

#### Safe Mode との連携

セーフモード（GPIO39 長押し起動 or 連続クラッシュ検出）では、**File Manager が常時有効**になります。  
暴走スクリプトを削除・修正する際の緊急アクセスとして利用できます。

#### 1台1アプリ設計方針

LUNA は「1台で1つのアプリを動かす」シンプル設計です。

- `luna_autostart.txt` がアプリ選択を兼ねる
- ドロップダウン・設定画面は不要
- 複数アプリを入れ替える場合 → File Manager でファイルを差し替えるだけ

---

### 2-17. ble モジュール — BLE スキャン・ビーコン・GATT クライアント（v8.9.3〜）

> **必要環境**: ESP32-S3 + NimBLE-Arduino ライブラリ  
> v8.9.3 LUNA Beacon Edition 以降専用モジュールです。

> ⚠️ **ESP32-S3 BLE 対応範囲**: BLE 5.0 標準レンジ（1M PHY）のみ対応。  
> **Coded PHY（Long Range）および 2M PHY は ESP32-S3 では非対応**。

#### 概要

| Phase | 機能 | API | 状態 |
|-------|------|-----|------|
| Phase 1 | BLE スキャン | `ble.scan()` / `ble.scan_stop()` / `ble.on("scan", cb)` | ✅ 有効 |
| Phase 2 | Legacy アドバタイズ | `ble.advertise()` / `ble.discover_luna()` | ✅ 有効 |
| Phase 3 | Extended アドバタイズ送信 | `ble.advertise_ext()` / `ble.advertise_ext_stop()` | ❌ 非対応（v8.9.6） |
| Phase 3 | Extended データ受信解析 | `ble.parse_luna_ext()` | ✅ 有効 |
| Phase 4 | GATT クライアント | `ble.connect()` / `ble.read()` / `ble.notify()` 等 | ✅ 有効 |
| Phase 5 | LUNA Sub Beacon 解析 | `ble.parse_luna_beacon()` | ✅ 有効（v8.9.6） |
| Phase 6 | Long Range（Coded PHY） | — | ❌ 非対応（ESP32-S3 ハード制約） |
| Phase 7 | Beacon 送信 | `ble.beacon_start()` / `ble.beacon_stop()` | ❌ 非対応（v8.9.6） |
| Phase 7 | Beacon 受信解析 | `ble.parse_beacon()` | ✅ 有効 |

> ⚠️ **Phase 3 送信・Phase 7 送信は v8.9.6 では非対応**。`CONFIG_BT_NIMBLE_EXT_ADV` を LUNA Sub の GATT Server との共存のため無効化したことによる制約。`advertise_ext()` / `beacon_start()` は `nil + エラーメッセージ` を返します。  
> ⚠️ **BLE HID は非対応**。専用 ESP32 を使用してください。

---

#### Phase 1: スキャン

```lua
-- スキャンコールバック登録
ble.on("scan", function(dev)
  -- dev: { addr, rssi, name, data(hex), extended(bool) }
  log.write(dev.addr .. " rssi=" .. dev.rssi .. " name=" .. (dev.name or ""))
end)

ble.scan({ duration=10000, active=false })   -- 10秒スキャン（ブロッキング）
ble.scan_stop()                              -- 強制停止

-- Xiaomi LYWSD03MMC（ATC カスタムファームウェア）温湿度パーサー
local temp, hum = ble.parse_xiaomi(dev.data)  -- temp°C, hum% | nil, nil
```

---

#### Phase 2: Legacy アドバタイズ（BLE 4.x 互換）

```lua
ble.advertise()                          -- LUNA ビーコン放送（IP 埋め込み）
ble.advertise({ ip="192.168.3.50" })     -- IP を手動指定
ble.advertise_stop()

-- 近傍の LUNA を自動発見
ble.discover_luna(function(dev)
  log.write("LUNA found: " .. dev.ip .. " rssi=" .. dev.rssi)
end, 10000)
```

---

#### Phase 3: Extended アドバタイズ（BLE 5.0）

> ❌ **送信（`advertise_ext` / `advertise_ext_stop`）は v8.9.6 で非対応**。呼び出すと `nil, "not supported"` が返ります。  
> ✅ **受信解析（`parse_luna_ext`）は有効**です。

```lua
-- ❌ 送信: 非対応
-- ble.advertise_ext({ id="LUNA01", temp=23.5 })  -- nil + エラーを返す

-- ✅ 受信: UUID 0xFE55 ペイロードから JSON を抽出（スキャン受信時に使用）
local json_str = ble.parse_luna_ext(dev.data)  -- JSON文字列 | nil
```

---

#### Phase 4: GATT クライアント

```lua
-- 接続・切断
local ok, err = ble.connect("AA:BB:CC:DD:EE:FF", 5000)   -- デフォルト5秒タイムアウト
ble.disconnect("AA:BB:CC:DD:EE:FF")
local connected = ble.is_connected("AA:BB:CC:DD:EE:FF")

-- サービス・キャラクタリスティック探索
local svcs  = ble.services("AA:BB:CC:DD:EE:FF")                    -- { "1800", "180F", ... }
local chars = ble.characteristics("AA:BB:CC:DD:EE:FF", "180F")     -- { {uuid, props}, ... }

-- 読み書き
local hex = ble.read("AA:BB:CC:DD:EE:FF", "180F", "2a19")          -- HEX 文字列
local ok  = ble.write("AA:BB:CC:DD:EE:FF", "180F", "2a19", "0101")

-- Notify 購読
ble.on("notify", function(evt)
  -- evt: { address, char(uuid), data(hex) }
  log.write("notify: " .. evt.char .. " = " .. evt.data)
end)
ble.notify("AA:BB:CC:DD:EE:FF", "1809", "2a1c")
ble.notify_wait(30000)     -- 30秒待機
ble.notify_stop("AA:BB:CC:DD:EE:FF", "1809", "2a1c")

-- SwitchBot パーサー（Company ID 0x0969）
local info = ble.parse_switchbot(dev.data)
-- Bot:   { model="Bot",   state=bool, battery=n }
-- Meter: { model="Meter", humidity=n, temp=n, battery=n }
```

> ⚠️ **接続安全注意**: 未知の BLE 機器への接続は ESP32 WDT リセットを引き起こす場合があります。LUNA-to-LUNA（UUID 0xFE55）は安全です。未確認機器に接続する前にユーザーへ告知してください。

---

#### Phase 5: LUNA Node Beacon 解析（v8.9.6〜）

LUNA Node v1.0.3+ が Manufacturer Data（Company ID `0xFFFF`、24バイト）に載せた状態テレメトリを解析します。  
v1.0.4 以降は **Byte 18 に `log_count`** が追加され、接続なしに未読ログの有無を判定できます（Phase 9）。

> ⚠️ Node の Beacon データは **Scan Response** に含まれるため、**`active=true` スキャンが必須**です。  
> ⚠️ スキャンコールバックのデバイスフィールドは **`dev.addr`** です（`dev.address` は誤り）。

```lua
ble.on("scan", function(dev)
  local b = ble.parse_luna_beacon(dev.data)  -- table | nil（非 LUNA デバイスは nil）
  if b then
    print(string.format("[%s] FW:%s Heap:%dKB paired:%s log_count:%d crc:%s",
      b.device_id, b.fw_version, b.heap_kb,
      tostring(b.paired), b.log_count, tostring(b.crc_ok)))
  end
end)
ble.scan({ duration=5000, active=true })  -- active=true 必須
```

**戻り値テーブルフィールド**:

| フィールド | 型 | 内容 |
|---|---|---|
| `device_id` | string | 送信元 MAC フラグメント（8桁 hex） |
| `fw_version` | string | Node ファームウェアバージョン（例: `"1.0.4"`） |
| `heap_kb` | number | Node の空きヒープ（KB） |
| `stack_bytes` | number | Node Lua タスクのスタック残量 HWM（bytes）※Lua 実行後に更新 |
| `paired` | bool | Node がペアリング済みか |
| `wifi` | bool | Node の WiFi 接続状態 |
| `autorun` | bool | autostart スクリプトが有効か |
| `lua_running` | bool | Node で Lua タスク実行中か |
| `log_count` | number | 未読ログ行数（0〜20）★v1.0.4 Phase 9 |
| `seq` | number | シーケンス番号（送信毎にインクリメント） |
| `crc_ok` | bool | CRC16/CCITT 検証結果 |

**Phase 9: log_count を使った省電力ログ取得**（接続不要でログ有無を確認）:

```lua
-- log_count > 0 のときだけ接続してログ取得 → 不要な BLE 接続を省略
local SVC = "ae9a2e94-5878-408b-882d-3c8ea1ed7169"
local CMD = "60a99508-94f7-4592-8f4d-8a2a5561a500"
local RSP = "f04de0cc-38e7-4bb5-8439-b7362539669b"

ble.on("scan", function(dev)
  local b = ble.parse_luna_beacon(dev.data)
  if b and b.log_count > 0 then
    ble.scan_stop()
    ble.connect(dev.addr)           -- dev.addr（dev.address ではない）
    ble.notify(dev.addr, SVC, RSP)
    hw.delay(300)
    ble.on("notify", function() end)  -- !! no-op 必須（get_log 前）
    local logs = sub.get_log(dev.addr, SVC, CMD, 8000)
    log.write(logs or "")
    ble.disconnect(dev.addr)
  end
end)
ble.scan({ duration=5000, active=true })
```

---

#### Phase 6: Long Range（Coded PHY）— ESP32-S3 非対応

> ❌ **ESP32-S3 は Coded PHY（Long Range）をハードウェアレベルで非対応**。  
> `coded=true` オプションは無効です。使用しないでください。  
> Long Range 対応は将来の別チップ（ESP32-C6 等）への移植時に実装予定です。

---

#### Phase 7: Beacon プロトコル（LUNA 地域コミュニティ網）

> ## ⚠️ 実装状態（v8.9.7 現在）
>
> | 機能 | 状態 | 備考 |
> |------|------|------|
> | **送信** `beacon_start()` / `beacon_stop()` | ❌ **未実装**（スタブ） | `CONFIG_BT_NIMBLE_EXT_ADV` 無効のため。呼び出すと `nil, "not supported"` を返す |
> | **受信解析** `parse_beacon()` | ✅ **動作する** | スキャン受信した ADV データから UUID 0xFE55 パケットを解析 |
> | **iGate**（受信→HTTP転送） | ✅ **動作する** | `parse_beacon` + `http.post` の組み合わせで実現可能 |
>
> **「Beaconネットワークで送受信できる」という説明は誤りです。現状は受信のみです。**

UUID 0xFE55 の Service Data に構造化 JSON を格納するコミュニティ放送プロトコルです（設計仕様）。  
送信機能は将来の実装課題です。

**Beacon パケットフォーマット**（設計仕様）:
```json
{ "type": "broadcast", "id": "74ABF84EB580", "msg": "Hello!", "ts": 1714123456 }
```

| フィールド | 内容 |
|-----------|------|
| `type` | `"broadcast"` / `"like"` / `"cmd"` / `"ack"` / `"ai_post"` |
| `id` | 送信元デバイス ID（`hw.device_id()` の値） |
| `msg` | メッセージ本文（UTF-8, 最大約150文字） |
| `ts` | Unix タイムスタンプ（`ntp.unix()` の値） |

```lua
-- ❌ 送信: 未実装。呼び出してもエラーを返すだけ
-- ble.beacon_start({ msg="Hello!", type="broadcast" })  -- → nil, "not supported"
-- ble.beacon_stop()                                     -- → no-op

-- ✅ 受信のみ有効: スキャンコールバック内で parse_beacon を呼ぶ
ble.on("scan", function(dev)
  local b = ble.parse_beacon(dev.data or "")
  if b then
    log.write("受信: id=" .. b.id:sub(1,8) .. " msg=" .. b.msg)
    -- b: { type, id, msg, ts, raw }
  end
end)
ble.scan({ duration=10000 })
```

**✅ iGate（BLE受信 → インターネット中継）— これは動作します**:
```lua
-- parse_beacon で受信し、HTTP で転送する（送信機能は不要）
ble.on("scan", function(dev)
  local b = ble.parse_beacon(dev.data or "")
  if b and b.type == "broadcast" and b.id ~= hw.device_id() then
    http.post("http://luna.example.com/mcp",
      '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"lua_exec",' ..
      '"arguments":{"code":"log.write(\'relay:' .. b.msg:sub(1,20) .. '\')"}}}',
      "application/json", 5000)
  end
end)
ble.scan({ duration=0 })   -- duration=0 で継続スキャン
```

---

### 2-17a. sub モジュール — LUNA Node BLE リモート制御（v8.9.7）

> **必要環境**: Bridge 側が v8.9.7 以降 / LUNA Node 側が v1.0.4 以降

**LUNA Node**（旧称: LUNA Sub）は、LUNA Bridge に BLE で接続される WiFi 非搭載デバイスの公式名称です。  
Bridge 側の Lua API では引き続き `sub.` プレフィックスを使用します。

---

#### LUNA Node 固定 UUID

| 種別 | UUID | 用途 |
|------|------|------|
| Service | `ae9a2e94-5878-408b-882d-3c8ea1ed7169` | GATT サービス |
| CMD | `60a99508-94f7-4592-8f4d-8a2a5561a500` | Bridge→Node 書き込み |
| RSP | `f04de0cc-38e7-4bb5-8439-b7362539669b` | Node→Bridge Notify |
| INFO | `55b29db3-c1aa-408b-86de-b07e489ddeec` | デバイス情報 Read |
| PAIR | `c0b11fea-1607-4952-b598-bb57e9ef4453` | ペアリング制御 |

---

#### API 一覧（v8.9.7）

| 関数 | 戻り値 | 説明 |
|------|--------|------|
| `sub.send(addr, svc, cmd, text)` | `true` \| `nil, err` | テキストコマンド送信（hex 変換を内部処理） |
| `sub.send_file(addr, svc, cmd, path, content [,chunk])` | `chunks, bytes` \| `nil, err` | Node SPIFFS にファイル転送 |
| `sub.lua_run(addr, svc, cmd, name)` | `true` \| `nil, err` | `/lua_name.lua` を Node で起動 |
| `sub.lua_stop(addr, svc, cmd)` | `true` \| `nil, err` | 実行中スクリプトを停止 |
| `sub.get_log(addr, svc, cmd [,ms])` | `log_text` \| `nil, err` | Node ログを一括取得（GETLOG プロトコル） |
| `sub.read_file(addr, svc, cmd, path [,ms])` | `content` \| `nil, err` | Node SPIFFS ファイルを Bridge に転送 |
| `sub.to_hex(str)` | `hex_string` | 文字列→16進数文字列 |
| `sub.from_hex(hex)` | `string` | 16進数文字列→文字列 |
| `sub.decode(evt)` | `string` | `ble.on("notify")` の `evt.data`（hex）を文字列に変換 |
| `sub.net_load()` | `json_string` | `/sub_network.json` 読み込み（未作成時は `{"subs":[]}` ） |
| `sub.net_save(json_str)` | `true` \| `nil, err` | `/sub_network.json` 書き込み |
| `sub.pair(addr, svc, pair_uuid [,ms])` | `rsp` \| `nil, err` | PAIR char に "ENTER" 送信・ペアリング実行 |
| `sub.forget(addr, svc, pair_uuid [,ms])` | `rsp` \| `nil, err` | PAIR char に "FORGET" 送信・ペアリング解除 |

---

#### !! 重要な注意事項

**① `ble.write()` に直接テキストを渡さない**

```lua
-- NG: byteLen=0 でサイレント失敗
ble.write(SUB, SVC, CMD, "STOP")

-- OK: sub.send() を使う（hex 変換を内部処理）
sub.send(SUB, SVC, CMD, "STOP")

-- OK: 手動 hex 変換
ble.write(SUB, SVC, CMD, sub.to_hex("STOP"))
```

**② `ble.on("notify")` コールバックの形式**

```lua
-- NG: 引数を address, char, data の順で受け取ろうとする（古い誤解）
ble.on("notify", function(address, char, data) end)

-- OK: 引数は TABLE 1つ。sub.decode(evt) で hex→文字列変換
ble.on("notify", function(evt)
  local s = sub.decode(evt)  -- evt.data (hex) を文字列化
  log.write(s)               -- "OK:result" / "OUT:text" / "LOG:line" 等
end)
```

**③ `sub.get_log` / `sub.read_file` 前に必ず `ble.on` を no-op にする（最重要）**

```lua
-- !! これを忘れると get_log / read_file が無限ハングします
-- 理由: アクティブな ble.on コールバックが gattNotifyQueue を先に消費し、
--       get_log / read_file が LOG:END / FEND: を受け取れない

ble.on("notify", function() end)   -- no-op にリセット
local logs = sub.get_log(SUB, SVC, CMD, 8000)
```

**④ `lua_stop` 前に `ble.disconnect()` を呼ぶ**

```lua
-- lua_stop は Lua タスクを止めるが BLE 接続は残存する
-- 次の ble.connect() が同一アドレスにハングする
ble.disconnect(SUB)    -- 先に切断
sub.lua_stop(SUB, SVC, CMD)
```

**⑤ スキャンコールバックのフィールド名**

```lua
-- OK: dev.addr
ble.on("scan", function(dev)
  log.write(dev.addr .. " rssi=" .. dev.rssi)  -- dev.addr が正しい
end)
-- NG: dev.address（存在しない）
```

---

#### 標準接続フロー

```lua
local SVC = "ae9a2e94-5878-408b-882d-3c8ea1ed7169"
local CMD = "60a99508-94f7-4592-8f4d-8a2a5561a500"
local RSP = "f04de0cc-38e7-4bb5-8439-b7362539669b"
local NODE = "ac:a7:04:fd:16:71"   -- LUNA Node の BLE アドレス

-- ① コールバック登録（sub.decode で hex→文字列）
local out = {}
ble.on("notify", function(evt) table.insert(out, sub.decode(evt)) end)

-- ② 接続・RSP 購読
ble.connect(NODE)
hw.delay(500)
ble.notify(NODE, SVC, RSP)
hw.delay(300)

-- ③ コマンド送信（inline Lua 実行）
sub.send(NODE, SVC, CMD, "return hw.free_heap()")
hw.delay(500)
-- out に "OK:12345" が届く

-- ④ get_log / read_file の前は必ず no-op
ble.on("notify", function() end)
local logs = sub.get_log(NODE, SVC, CMD, 8000)

-- ⑤ 切断
ble.disconnect(NODE)
```

---

#### ログ取得（sub.get_log）

LUNA Node 上で `log.write()` に書き込まれたログを一括取得します。  
取得後、Node 側のログバッファはクリアされ（`log_count` → 0）、次の Beacon に反映されます。

```lua
-- !! ble.on を no-op にしてから呼ぶこと（必須）
ble.on("notify", function() end)

local logs, err = sub.get_log(NODE, SVC, CMD, 8000)
if logs then
  log.write("=== Node Log ===\n" .. logs)
else
  log.write("get_log failed: " .. tostring(err))
end
```

---

#### ファイル読み出し（sub.read_file）

Node SPIFFS 上のファイルを Bridge に転送します（READFILE プロトコル、480B/チャンク）。

```lua
-- !! ble.on を no-op にしてから呼ぶこと（必須）
ble.on("notify", function() end)

local content, err = sub.read_file(NODE, SVC, CMD, "/luna_log.txt", 10000)
if content then
  log.write("ファイル内容:\n" .. content)
else
  log.write("read_file failed: " .. tostring(err))
end
```

---

#### LUNA Node ネットワーク管理（v8.9.7）

複数の LUNA Node を「ネットワーク」として管理します。  
Bridge の SPIFFS に `/sub_network.json` を保存し、MAC テーブルを永続管理します。  
**一度ペアリング・登録すれば、次回からスキャン不要で即接続できます。**

##### /sub_network.json 構造

```json
{
  "subs": [
    {"name": "node1", "addr": "ac:a7:04:fd:16:71", "fw": "1.0.4", "note": "温湿度"},
    {"name": "node2", "addr": "aa:bb:cc:dd:ee:ff", "fw": "1.0.4", "note": "リレー"}
  ]
}
```

##### 初回ペアリング + DB 登録

```lua
local SVC  = "ae9a2e94-5878-408b-882d-3c8ea1ed7169"
local PAIR = "c0b11fea-1607-4952-b598-bb57e9ef4453"

-- Step 1: スキャンで未ペアリング Node を探す
local found = nil
ble.on("scan", function(dev)
  local b = ble.parse_luna_beacon(dev.data)
  if b and not b.paired then
    found = dev.addr
    ble.scan_stop()
  end
end)
ble.scan({ duration=5000, active=true })
hw.delay(3500)

-- Step 2: 接続・ペアリング（Node を Bridge の近く 50cm 以内に置くこと）
if found then
  ble.connect(found); hw.delay(500)
  local rsp = sub.pair(found, SVC, PAIR)
  -- → "OK:PAIRED rssi=-29"（成功）/ "ERR:TOO_FAR rssi=N"（近づけて再試行）
  log.write("pair: " .. tostring(rsp))

  if rsp and rsp:sub(1,3) == "OK:" then
    -- Step 3: DB 登録
    local db = json.parse(sub.net_load()) or {subs={}}
    table.insert(db.subs, {name="node1", addr=found, fw="1.0.4", note=""})
    sub.net_save(json.encode(db))
    log.write("登録完了: " .. found)
  end
  ble.disconnect(found)
end
```

##### ネットワーク一括監視（スキャン不要）

```lua
local CMD = "60a99508-94f7-4592-8f4d-8a2a5561a500"
local RSP = "f04de0cc-38e7-4bb5-8439-b7362539669b"

local db = json.parse(sub.net_load())
for _, n in ipairs(db.subs) do
  ble.connect(n.addr); ble.notify(n.addr, SVC, RSP); hw.delay(300)
  ble.on("notify", function() end)   -- no-op 必須
  local logs = sub.get_log(n.addr, SVC, CMD, 5000)
  if logs then log.write(n.name .. ": " .. logs) end
  ble.disconnect(n.addr); hw.delay(300)
end
```

##### ペアリング解除

```lua
local PAIR = "c0b11fea-1607-4952-b598-bb57e9ef4453"
ble.connect(NODE); hw.delay(500)
local rsp = sub.forget(NODE, SVC, PAIR)
log.write("forget: " .. tostring(rsp))  -- → "OK:FORGOTTEN"
ble.disconnect(NODE)
```

---

#### LUNA Node GATT CMD/RSP プロトコル早見表

**CMD（Bridge→Node 書き込み）**:

| コマンド | 動作 | RSP |
|---------|------|-----|
| `STOP` | Lua 停止（実行中でも優先） | `OK:stopped` |
| `RUN:name` | `/lua_name.lua` 起動 | `OK:started:name` / `BUSY` |
| `GETLOG` | ログ全送信（get_log が内部使用） | `LOG:line` × N → `LOG:END` |
| `READFILE:path` | ファイル送信（read_file が内部使用） | `FSTART:size` / `FDATA:` × N / `FEND:` |
| その他テキスト | inline Lua として実行 | `OK:result` / `ERR:msg` |

**RSP Notify プレフィックス（Node→Bridge）**:

| プレフィックス | 意味 |
|-------------|------|
| `OK:` | 成功（result / stopped / started:name） |
| `ERR:` | エラー（`ERR:NOT_PAIRED` はペアリング未完了） |
| `BUSY` | Lua 実行中で CMD 拒否 |
| `OOM` | メモリ不足 |
| `TIMEOUT` | Lua 実行タイムアウト（5000ms） |
| `OUT:text` | `print()` ストリーミング（inline Lua のみ） |
| `LOG:line` | GETLOG 1行 |
| `LOG:END` | GETLOG 終端（ログバッファクリア） |
| `FSTART:size` | READFILE 開始 |
| `FDATA:...` | READFILE チャンク（最大 480B/notify） |
| `FEND:` | READFILE 完了 |
| `FERR:reason` | READFILE エラー |

> **`print()` ストリーミング**: inline Lua（`sub.send()` でコードを送った場合）のみ `OUT:` Notify が届く。  
> `sub.lua_run()` で起動したスクリプト内の `print()` はストリーミングされない → `log.write()` + `sub.get_log()` を使う。

---

#### 3ファイルデプロイ規則（LUNA Node Console 対応）

LUNA Node Console の Run / Open UI ボタンに対応するための命名規則:

```
/lua_<name>.lua      ← Lua スクリプト（lua_ プレフィックス必須）
/<name>.html         ← ブラウザ UI（プレフィックスなし）
/luna_autostart.txt  ← 内容 = '<name>'（パス・拡張子なし）
```

```lua
-- デプロイ例
sub.send_file(NODE, SVC, CMD, "/lua_myapp.lua", lua_code)
sub.send_file(NODE, SVC, CMD, "/myapp.html",    html_code)
sub.send_file(NODE, SVC, CMD, "/luna_autostart.txt", "myapp")
sub.lua_run(NODE, SVC, CMD, "myapp")
```

---

### 2-18. adc/dac モジュール — アナログ入出力（v5.9.8/v6.9.5/v8.9.5 Lab Edition）

「AI が物理世界で仮説検証する」プラットフォーム。DAC/PWM で刺激を与え、ADC で応答を測定し、Claude が推論して次の実験条件を決定する閉ループを実現します。

```
Claude → lua_exec(dac.write / gpio.pwm) → ESP32 → 対象物（刺激）
Claude ←  adc.read_avg()               ← ESP32 ← 対象物（応答）
   ↓
Claude が推論 → 条件変更 → 次の実験
```

#### `adc` モジュール — アナログ入力（12bit ADC）

| 関数 | 説明 |
|------|------|
| `adc.atten(pin, db)` | 入力レンジ設定。0=0-950mV / 1=0-1250mV / 2=0-1750mV / 3=0-3100mV |
| `adc.read(pin)` | 12bit 生値（0〜4095）を返す |
| `adc.read_mv(pin)` | eFuse 校正済みミリボルト値を返す |
| `adc.read_avg(pin, n)` | n 回平均ミリボルト（推奨 n=16〜64）。ノイズ低減に有効 |

**ESP32（v5.9.8）対応ピン**: GPIO32, 33, 34, 35, 36(VP), 39(VN)（ADC1 のみ）  
**ESP32-S3（v6.9.5/v8.9.5）対応ピン**: GPIO1〜10（ADC1 のみ）  
⚠ ADC2 ピンは WiFi と競合するため使用不可  
⚠ ESP32-S3 FREENOVE WROOM N8R8: GPIO2・GPIO6 は出力ピンとも兼用  
推奨フリーピン（ESP32-S3）: GPIO3, 4, 5, 7, 8, 9, 10

```lua
-- 基本的な読み取り
adc.atten(4, 3)                     -- GPIO4、フルレンジ 0-3100mV
local mv  = adc.read_mv(4)          -- キャリブレーション済みミリボルト
local avg = adc.read_avg(4, 32)     -- 32サンプル平均（安定値）
log.write(string.format("mv=%d avg=%d", mv, avg))

-- 実験ループ: PWM デューティを変えながら ADC で応答を計測
adc.atten(4, 3)
for duty = 0, 255, 32 do
  gpio.pwm(38, 1000, duty)          -- 刺激: PWM デューティを段階的に変化
  hw.delay(200)                     -- 整定時間
  local v = adc.read_avg(4, 32)    -- 応答: 32サンプル平均
  log.write(string.format("duty=%d  mv=%d", duty, v))
end
gpio.pwm_stop(38)
```

#### `dac` モジュール — アナログ出力（8bit DAC）

| 関数 | 説明 |
|------|------|
| `dac.write(pin, value)` | 0〜255 の値を出力（0V〜3.3V）。ESP32 専用 |
| `dac.stop(pin)` | DAC を無効化 |

**ESP32（v5.9.8）対応ピン**: GPIO25（DAC1）/ GPIO26（DAC2）  
⚠ GPIO25 は FILE_MANAGER_LED_PIN と共用  
⚠ **ESP32-S3（v6.9.5/v8.9.5）はハードウェア DAC なし** → `dac.write()` はエラーを返す  
　代替: `gpio.pwm()` + RC フィルター（R=10kΩ, C=10μF → カットオフ約 1.6Hz）

```lua
-- ESP32（v5.9.8）のみ: ハードウェア DAC
dac.write(26, 0)    -- GPIO26: 0V
dac.write(26, 128)  -- GPIO26: 約 1.65V（実測 1.72V / 誤差 +4.4%）
dac.write(26, 255)  -- GPIO26: 3.3V
dac.stop(26)        -- DAC 無効化

-- ESP32-S3（v6.9.5/v8.9.5）: gpio.pwm() + RC フィルターで代替
gpio.pwm(38, 1000, 128)   -- duty=128 → 約 1.65V（RC フィルター後）
gpio.pwm_stop(38)
```

#### バイオニック・マッスル実験例（v5.9.8 ESP32）

```lua
-- 周波数スイープで共振点を自動発見
adc.atten(34, 3)
local best_freq, best_mv = 0, 0
for freq = 10, 200, 10 do
  dac.write(26, 128)                    -- 中間電圧を印加
  gpio.pwm(18, freq, 128)              -- 周波数を変えながら刺激
  hw.delay(500)
  local mv = adc.read_avg(34, 64)     -- 応答を計測
  log.write(string.format("freq=%d Hz  response=%d mV", freq, mv))
  if mv > best_mv then
    best_mv = mv
    best_freq = freq
  end
  gpio.pwm_stop(18)
  hw.delay(100)
end
log.write(string.format("Best resonance: %d Hz (%d mV)", best_freq, best_mv))
dac.stop(26)
```

---

### 2-19. 標準 Lua 5.1 ライブラリ  <!-- 旧 2-18 -->

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

### 2-20. 数値型・16進数・ビット演算（実機確認済み）

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

### 2-21. 使用できない機能

| 機能 | 結果 | 理由 |
|------|------|------|
| `io.*`（ファイル読み書き） | ❌ | SPIFFS へのアクセスは MCP ツール経由で行う |
| `os.*`（OS 操作） | ❌ | 組み込み環境のため未サポート |
| `require()`（外部モジュール） | ❌ | 単一スクリプト環境のため未サポート |
| TCP 生ソケット通信 | ❌ | 現リリースでは非公開（将来バージョンで対応予定） |
| `bit.*`（ビット演算ライブラリ） | ❌ | 未実装（純粋 Lua で代替可能） |
| 複数スクリプトの同時実行 | ❌ | 1スクリプトのみ（コルーチンで疑似並行は可能） |

---

### 2-22. クイックリファレンス一覧

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
hw.led(1 or 0)                      -- 内蔵LED制御（数値のみ）
-- ※ hw.free_heap() は存在しない。空きヒープは Claude 側から lua_status() で確認。

-- RGB LED NeoPixel WS2812B GPIO48（v6.9.3 ESP32-S3 専用）
hw.rgb(r, g, b)                     -- 単色点灯 / hw.rgb(0,0,0) で消灯
hw.rgb_blink(r, g, b, ms[, intv])  -- 指定時間点滅して自動消灯（ブロッキング・intv省略=250ms）
hw.rgb_flag(1[, r, g, b])          -- ノンブロッキック点滅開始（省略時黄 255,200,0）
hw.rgb_flag(0)                      -- 点滅停止・消灯

-- 通知
notify.set("完了")                  -- Claude に完了を通知

-- スクリプトチェーン（v3.9.1）
luna.run(name)                      -- 現在のスクリプト終了後に name を自動起動（最大10本）

-- HTTP通信・Alpaca 連携（v5.9.1）、POST 対応（v5.9.2）
http.get(url, timeout_ms)                    -- HTTP GET。レスポンスボディを返す（エラー時 ""）
http.put(url, body, timeout_ms)              -- HTTP PUT。レスポンスボディを返す（エラー時 ""）
local resp, code = http.post(url, body, ct, ms)  -- HTTP POST（v5.9.2）。resp + HTTP コードの2値を返す

-- UDP 通信（v5.9.3）
udp.begin(local_port)           -- UDP ソケットを開く
udp.connect(host, port)         -- 送信先を設定
udp.write(str)                  -- UDP パケット送信
local resp = udp.read([ms])     -- UDP パケット受信（デフォルト 1000ms）
udp.close()                     -- ソケットを閉じる

-- GPIO 直接制御（v5.9.3 / v8.9.4）
gpio.output_pin(n)              -- Output n（1〜4）の GPIO 番号を返す
gpio.write(pin, 0 or 1)        -- GPIO に LOW/HIGH を出力
gpio.read(pin)                  -- GPIO の状態を読み取る（0 or 1）
gpio.mode(pin, 0 or 1 or 2)    -- ピンモード設定（0=INPUT / 1=OUTPUT / 2=INPUT_PULLUP）
gpio.pwm(pin, freq, duty)      -- PWM 出力（freq=Hz, duty=0〜255）★v8.9.4
gpio.pwm_stop(pin)             -- PWM 停止・チャネル解放 ★v8.9.4

-- 温湿度センサー（v5.9.3）
local t, h = sensor.dht22([pin])  -- DHT22 読み取り。エラー時は -999.0, -999.0

-- JSON parse/encode（v5.9.4）
local tbl, err = json.parse(str)  -- JSON 文字列 → Lua テーブル（失敗時 nil, err）
local str = json.encode(val)      -- Lua テーブル → JSON 文字列

-- NTP 時刻同期（v5.9.4）
ntp.sync([server])                -- 同期開始（非ブロッキング。デフォルト: pool.ntp.org）
ntp.synced()                      -- 同期済み: true / 未同期: false
ntp.unix()                        -- UTC Unix タイムスタンプ（秒）
ntp.utc()                         -- UTC 文字列 "YYYY-MM-DD HH:MM:SS"
ntp.jd()                          -- ユリウス日（天体計算用）

-- ブラウザ UI 連携（v5.9.6/v6.9.3）
ui.send(id, message)              -- ブラウザの id 要素にメッセージを送信
ui.check()                        -- ブラウザからのボタン入力を取得（ノンブロッキング。なければ ""）
ui.poll(timeout_ms)               -- ボタン入力を待機（ブロッキング。タイムアウト時 ""）
ui.switch(url)                    -- ブラウザを指定 URL にリダイレクト
ui.clear()                        -- ブラウザ側メッセージキューをクリア
-- ※ ボタン名: 通常は "btn名"、テキスト入力は "input:値"、スライダーは "slider:値"
-- ※ Lua 5.1 注意: 整数除算は math.floor(x/y) を使う（// 演算子は Lua 5.3 以降のみ）

-- ADC アナログ入力（v5.9.8/v6.9.5/v8.9.5 Lab Edition）
adc.atten(pin, db)             -- 入力レンジ設定（0=〜950mV / 1=〜1250mV / 2=〜1750mV / 3=〜3100mV）
adc.read(pin)                  -- 12bit 生値 0〜4095
adc.read_mv(pin)               -- eFuse 校正済みミリボルト
adc.read_avg(pin, n)           -- n 回平均ミリボルト（推奨 n=16〜64）
-- 対応ピン: ESP32=GPIO32-39 / ESP32-S3=GPIO1-10（ADC2 は WiFi 競合・使用不可）

-- DAC アナログ出力（v5.9.8 ESP32 のみ・ESP32-S3 はスタブ）
dac.write(pin, value)          -- 0〜255 → 0V〜3.3V（GPIO25=DAC1 / GPIO26=DAC2）
dac.stop(pin)                  -- DAC 無効化
-- ⚠ ESP32-S3: dac.write() はエラーを返す → gpio.pwm() + RC フィルターで代替

-- BLE スキャン・ビーコン（v8.9.3 Beacon Edition）
-- ⚠ ESP32-S3: 1M PHY のみ対応。Coded PHY (Long Range) / 2M PHY は非対応
ble.on("scan", function(dev) end)            -- スキャンコールバック登録
ble.scan({ duration=ms, active=true })       -- スキャン開始
ble.scan_stop()                              -- スキャン停止
ble.parse_xiaomi(dev.data)                   -- Xiaomi 温湿度パーサー → temp, hum
ble.advertise()                              -- Legacy LUNA ビーコン放送
ble.advertise_stop()
ble.discover_luna(cb [, ms])                 -- 近傍 LUNA を自動発見
ble.advertise_ext({ id="LUNA01", temp=23.5 }) -- Extended 放送（JSON 最大200B）
ble.advertise_ext_stop()
ble.parse_luna_ext(dev.data)                 -- UUID 0xFE55 JSON 抽出
ble.connect(addr [, ms])                     -- GATT 接続（デフォルト5秒）
ble.disconnect(addr)
ble.is_connected(addr)                       -- bool
ble.services(addr)                           -- { "1800", ... }
ble.characteristics(addr, svc)              -- { {uuid, props}, ... }
ble.read(addr, svc, chr)                     -- HEX 文字列
ble.write(addr, svc, chr, hex)
ble.on("notify", function(evt) end)          -- evt: { address, char, data(hex) }
ble.notify(addr, svc, chr)
ble.notify_stop(addr, svc, chr)
ble.notify_wait(ms)
ble.parse_switchbot(dev.data)                -- { model, state/temp/hum, battery }
ble.beacon_start({ msg, type })              -- Beacon 放送（type: "broadcast"等）
ble.beacon_stop()
ble.parse_beacon(dev.data)                   -- { type, id, msg, ts, raw } | nil

-- セキュリティ（v8.9.3）
hw.device_id()                               -- EFuse MAC 12桁 HEX（例: "74ABF84EB580"）
hw.sign(msg)                                 -- HMAC-SHA256(ActivationKey, msg) → 64桁 HEX | nil, err

-- LUNA Node BLE リモート制御（v8.9.7）※旧称 LUNA Sub
-- UUID: SVC=ae9a2e94-... CMD=60a99508-... RSP=f04de0cc-... PAIR=c0b11fea-...
-- !! ble.write() に直接テキストを渡さない → sub.send() を使う（内部で hex 変換）
-- !! ble.on("notify", cb) の evt.data は hex → sub.decode(evt) で文字列化
-- !! sub.get_log / sub.read_file の前に必ず ble.on("notify", function() end) でリセット

-- 送信・実行制御
sub.send(addr, svc, cmd, text)               -- テキスト送信（hex 変換自動）→ true | nil, err
sub.send_file(addr, svc, cmd, path, content) -- Node SPIFFS にファイル転送 → chunks, bytes | nil, err
sub.lua_run(addr, svc, cmd, name)            -- /lua_name.lua を Node で起動 → true | nil, err
sub.lua_stop(addr, svc, cmd)                 -- 実行中スクリプトを停止 → true | nil, err

-- データ取得（ブロッキング・呼び出し前に ble.on no-op 必須）
sub.get_log(addr, svc, cmd [, ms])           -- Node ログ一括取得 → log_text | nil, err  (default 5000ms)
sub.read_file(addr, svc, cmd, path [, ms])   -- Node ファイル転送 → content | nil, err

-- ヘルパー
sub.to_hex(str)                              -- 文字列→hex 文字列
sub.from_hex(hex)                            -- hex 文字列→文字列
sub.decode(evt)                              -- ble.on("notify") evt.data (hex) を文字列化

-- LUNA Node ネットワーク管理（/sub_network.json）
sub.net_load()                               -- {"subs":[...]} を返す（未作成時 {"subs":[]}）
sub.net_save(json_str)                       -- /sub_network.json に保存 → true | nil, err
sub.pair(addr, svc, pair_uuid [, ms])        -- PAIR char に "ENTER" → "OK:PAIRED rssi=N" | nil, err
sub.forget(addr, svc, pair_uuid [, ms])      -- PAIR char に "FORGET" → "OK:FORGOTTEN" | nil, err
-- pair/forget は PAIR char の notify 購読を内部処理（ble.notify() 不要）

-- RSP Notify: "OK:result" / "ERR:msg" / "BUSY" / "OUT:text" / "LOG:line" / "LOG:END"
--             "FSTART:size" / "FDATA:..." / "FEND:" / "FERR:reason"
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
| `set_autostart` | autostart スクリプトを設定・保存（v5.9.5） |
| `get_autostart` | 現在の autostart 設定・safe_mode・boot_count を確認（v5.9.5） |
| `clear_autostart` | autostart を無効化・RTC カウンターリセット（v5.9.5） |
| `indi_connect` | INDI サーバーに接続・BG キャッシュ起動（v6.9.2） |
| `indi_disconnect` | INDI サーバー切断（キャッシュ保持）（v6.9.2） |
| `indi_status` | INDI 接続状態・統計情報取得（v6.9.2） |
| `indi_devices` | INDI デバイス一覧取得（v6.9.2） |
| `indi_properties` | デバイスのプロパティ一覧取得（v6.9.2） |
| `indi_get` | 指定エレメント値取得（v6.9.2） |
| `indi_state` | プロパティ state 取得（v6.9.2） |
| `indi_set_number` | Number プロパティ送信（GoTo 座標など）（v6.9.2） |
| `indi_set_switch` | Switch プロパティ送信（接続・モード切替）（v6.9.2） |
| `indi_set_text` | Text プロパティ送信（v6.9.2） |
| `fs_write` | SPIFFS にファイルを書き込み（最大 4KB）（v5.9.x/v6.9.x/v8.9.x） |
| `fs_append` | SPIFFS ファイルに追記（最大 4KB）（v5.9.x/v6.9.x/v8.9.x） |
| `fs_read` | SPIFFS ファイルを読み込み（最大 4KB）（v5.9.x/v6.9.x） |
| `fs_delete` | SPIFFS ファイルを削除（v5.9.x/v6.9.x） |
| `fs_list` | SPIFFS ファイル一覧（v5.9.x/v6.9.x） |
| `get_system_info` | ファームウェア情報・autostart・safe_mode・boot_count 取得 |
| `hard_reset` | ESP32 を再起動（ESP.restart()）（v5.9.x/v6.9.x） |

> **indi.* Lua API（スクリプト用）**: 本ガイド セクション **2-13** を参照してください。
> **INDI MCP ツール・GoTo ワークフローの詳細**: `get_lua_guide` の **セクション 2-13** を参照してください。
> **ui モジュール・Console UI の詳細**: 本ガイド セクション **2-15 / 2-16** を参照してください。

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
    "x=%d, name=%s, flag=%s, data=[%.3f, %.3f, %.3f], uptime=%d",
    x, name, tostring(flag),
    data[1], data[2], data[3],
    hw.millis()
)
```

`return` の内容：
```
x=42, name=測定器A, flag=true, data=[1.234, 5.678, 9.012], uptime=12345
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

## 💖 開発継続のためのご支援のお願い

LUNA は個人開発プロジェクトです。Lua エンジン・全モジュール・ドキュメントを無償で公開し続けるために、皆様のご支援が大きな励みになります。

少額でも構いません。ぜひご支援をよろしくお願いいたします。

| 方法 | 詳細 |
|------|------|
| 💳 **PayPal でのご寄付** | **nishioka.sst@gmail.com** 宛にお送りください（[PayPal サイト](https://www.paypal.com/)からメールアドレスで送金） |
| 🛒 **基板のご購入** | **LUNA OnStepNinja V2** 基板をご購入いただくことが開発継続の直接的なご支援になります → [GitHub Issues](https://github.com/OnStepNinja/LUNA/issues) よりお問い合わせください |

皆様のご支援により、Lua モジュールの拡充・新機器対応・ドキュメント整備を続けることができます。
心より感謝申し上げます。🙏

---

*AiBridgeMCP プロジェクト: https://github.com/OnStepNinja/AiBridgeMCP*
*作者: Nishioka Sadahiko*
