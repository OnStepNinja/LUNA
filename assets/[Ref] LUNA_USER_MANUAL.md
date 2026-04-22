# 2. LUNA ユーザーマニュアル
# 完全版リファレンス

**ブランド名**: LUNA / LUNA Observatory / LUNA INDI Edition（AiBridgeMCP v3.9.1〜v6.9.3）
**対象**: RS-232C機器・Alpaca 対応機器・INDI 対応天文機器・天文撮影ソフトを制御したいユーザー・技術者
**リビジョン**: r2.7
**最終更新**: 2026-04-15
**作成日**: 2026-03-19

| リビジョン | 日付 | 変更内容 |
|-----------|------|---------|
| r2.9 | 2026-04-16 | v6.9.3/v5.9.6: セクション5「LUNA アプリ配布システム」追加 |
| r2.8 | 2026-04-15 | v6.9.3: Console UI に Uptime / Free Heap 表示追加・複数 MCP 登録ガイド追加 |
| r2.7 | 2026-04-15 | v6.9.3: Console UI に Open UI ボタン追加・HTML 命名規則確立 |
| r2.6 | 2026-04-14 | v6.9.3: Console UI Lua Script パネル・File Manager 30分自動開放・MCP タイムアウト動作確認 |
| r2.5 | 2026-04-14 | v6.9.3: Lua UI フレームワーク実装・実機確認済み（ui モジュール・セクション追加） |
| r2.4 | 2026-04-14 | v6.9.2: ESP32-S3 へ移行（ハードウェア要件に追記） |
| r2.3 | 2026-04-13 | v6.9.2 INDI Edition 対応（INDI MCP ツール 10 個・セクション 3 新設） |
| r2.2 | 2026-03-30 | v5.9.5 autostart / セーフモード対応 |
| r2.1 | 2026-03-27 | v5.9.2 NINA 統合・USER_MANUAL r2.0 相当 |

---

> **【重要事項】** 本ソフトウェアは MITライセンス（無償・商用利用可）に基づき、**バイナリ形式**で配布されます。ソースコードは非公開です。

---

## はじめに

初めて LUNA をお使いの方は、まず以下をご覧ください：

> 📄 **LUNA_Quick_Setup_Guide.md** — 初回セットアップ（3ステップで完了）
>
> LUNA（ESP32）と Claude Desktop を接続するための初回設定ガイドです。
> 接続プロキシ（connector）の配置と Claude Desktop の設定ファイル編集を
> 3ステップで説明します。まずここから始めてください。

本マニュアルは全機能のリファレンスです。セットアップ完了後にご参照ください。

---

> ## 💬 LUNA について質問できます
>
> 最新の **LUNA Docs**（ドキュメント集）を NotebookLM で公開しています。
> LUNA の使い方・設定・Lua スクリプトなど、何でも直接ご質問いただけます。
>
> 🔗 **[LUNA Docs を開く](https://notebooklm.google.com/notebook/f2ce997f-18c2-44c4-8738-973b204c190c)**
>
> ※ Google アカウントをお持ちの方はそのままご利用いただけます。

---

## 目次

1. システム概要
   - 1.1 できること
   - 1.2 対応機器
   - 1.3 必要なもの
2. autostart — 電源ON自律起動（v5.9.5）
3. LUNA INDI Edition — INDI 対応望遠鏡制御（v6.9.2）
4. Lua UI フレームワーク — ブラウザ連携 UI（v5.9.6 / v6.9.3） ★新機能★
5. LUNA アプリ配布システム — 3ファイルで完結（v5.9.6 / v6.9.3） ★画期的★
6. LUNA + NINA 統合天文撮影システム（v5.9.2）
   - 5.1 NINA とは
   - 5.2 LUNA + NINA で何が変わるか
   - 5.3 フル自動化の構成
   - 5.4 初心者へのメリット
   - 5.5 接続のしかた
6. 初期設定（APモード）
7. ハードウェア構成
8. Claude Desktop の設定
9. MCPツール リファレンス
   - 9.1 シリアルブリッジ系（機器制御）
   - 9.2 Lua自律制御系
   - 9.3 通知系
   - 9.4 システム情報系
   - 9.5 ファイルシステム系（SPIFFS）
   - 9.6 Resources 系（v5.9.1）
10. Lua自律制御
    - 10.1 Lua APIリファレンス
    - 10.2 スクリプトチェーン（luna.run）
11. リセットモード
12. トラブルシューティング
13. 仕様一覧

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

### 1.1 できること

| 機能 | 説明 |
|------|------|
| **機器へのコマンド送信** | `serial_query` で送信→応答を1ステップで取得 |
| **機器からのデータ受信** | `serial_read` で機器が自発的に送るデータを受信 |
| **Lua スクリプト実行** | ESP32 上で Lua 5.1 スクリプトを非同期実行 |
| **スクリプト永続保存** | `lua_save` で SPIFFS に保存、電源断後も有効 |
| **自律スクリプトチェーン** | `luna.run()` で複数スクリプトを順番に自律実行 |
| **Claude への非同期通知** | `notify.set("メッセージ")` でスクリプトから Claude に通知 |
| **LUNA 使い方の自動通知** | 接続時に `prompts/get luna_guide` が自動実行 |
| **MCP Resources（v5.9.1）** | 機器の状態を Claude が計器のように自然に読み取る |
| **http モジュール（v5.9.1）** | Lua から LAN 内の Alpaca/HTTP デバイスを直接制御 |
| **NINA 統合（v5.9.2）** | 天文撮影ソフト NINA を Claude から自然言語で制御 |
| **Stellarium 連携（v5.9.2）** | プラネタリウムソフト Stellarium と連動 |
| **autostart（v5.9.5）** | 電源ON時にスクリプトを自動実行・MCPサーバー不要のスタンドアロン動作 |
| **セーフモード（v5.9.5）** | クラッシュループ・暴走スクリプトを自動検出して保護 |
| **INDI MCP ツール（v6.9.2）** | INDI サーバー経由で望遠鏡・カメラ等を Claude から直接制御（10 ツール） |

### 1.2 対応機器（例）

RS-232Cポートを持つ機器であれば原則として何でも制御できます。
v5.9.2 からは LAN 上の PC ソフトウェアとの連携も拡大しました。

| 分野 | 機器・ソフト例 |
|------|-------|
| **産業・製造** | NC工作機械、PLC、インバータ、ロボットコントローラ |
| **計測・研究** | マルチメータ、オシロスコープ、スペクトラムアナライザ、電源装置 |
| **天文（機器）** | Takahashi Temma2、Celestron 等の望遠鏡マウント |
| **天文（ソフト）** | NINA（撮影）、Stellarium（プラネタリウム）、ASCOM/Alpaca 対応機器全般 |
| **天文（INDI）** | INDI 対応機器全般（望遠鏡・カメラ・フォーカサー等）RPi/Linux + indiserver 経由（v6.9.2） |
| **放送・AV** | スイッチャー、VTRコントローラ |

### 1.3 必要なもの

LUNA を使い始めるために必要なものは2つだけです。

| 必要なもの | 入手先 | 費用 |
|-----------|--------|------|
| **Claude Desktop** | https://claude.ai/download | 無料 |
| **LUNA ファームウェア** | GitHub Releases | 無料 |

> **💡 Claude Code は不要です。** Claude Desktop（無料）だけで、LUNA の全機能を利用できます。JSON 設定ファイルの作成や機器の操作も、Claude Desktop 上で Claude に相談しながら進めることができます。

---

## 3. LUNA INDI Edition — INDI 対応望遠鏡制御（v6.9.2）

### 3.1 INDI Edition とは

**INDI（Instrument-Neutral Distributed Interface）** は Linux/RPi 環境で広く使われる天文機器制御プロトコルです。
v6.9.2 INDI Edition では、LUNA が INDI サーバーに接続し、Claude Desktop から 10 個の MCP ツールで望遠鏡・カメラ等を直接制御できます。

```
Claude Desktop
    │ MCP tools
    ▼
LUNA（ESP32-S3）── WiFi ──→ indiserver（RPi / Linux PC / WSL2）
                                  │
                                  ├── indi_simulator_telescope
                                  ├── indi_lx200generic
                                  ├── indi_asi_ccd
                                  └── その他 INDI ドライバー
```

### 3.2 INDI MCP ツール一覧

| ツール | 説明 |
|--------|------|
| `indi_connect` | INDI サーバー接続（ホスト・ポート指定） |
| `indi_disconnect` | 切断（キャッシュ保持） |
| `indi_status` | 接続状態・受信バイト数・デバイス数 |
| `indi_devices` | 接続デバイス一覧 |
| `indi_properties` | デバイスのプロパティ一覧（name/type/state） |
| `indi_get` | 指定エレメントの値取得 |
| `indi_state` | プロパティ state 取得（Idle/Ok/Busy/Alert） |
| `indi_set_number` | Number プロパティ送信（GoTo 座標など） |
| `indi_set_switch` | Switch プロパティ送信（接続・モード切替など） |
| `indi_set_text` | Text プロパティ送信 |

### 3.3 GoTo の基本手順

```
1. indi_connect {"host":"192.168.x.x", "port":7624}
2. （3秒待機）
3. indi_status {} → devices > 0 を確認
4. indi_set_switch CONNECTION → CONNECT: On
5. indi_set_switch ON_COORD_SET → SLEW: On
6. indi_set_number EQUATORIAL_EOD_COORD → RA/DEC 送信
7. indi_state EQUATORIAL_EOD_COORD → Busy = スルー中
8. 座標安定（indi_get で確認）= GoTo 完了
```

> 詳細は **4. LUNA_RPi_INDI_Design** および `prompts/get luna_guide` セクション 2-13 を参照してください。

---

## 2. autostart — 電源ON自律起動（v5.9.5）

> **実機確認済み（2026-03-30）**: NS-5000 + SkySafari / 接続・アライメント・GoTo 動作確認

### 2.1 autostart とは

LUNA の電源を入れると、設定されたスクリプトが **自動的に起動**する機能です。

従来は SkySafari ブリッジなどを使うたびに Claude Desktop から `lua_run` を呼ぶ必要がありました。
v5.9.5 の autostart を設定すると、**電源を入れるだけで即使える**スタンドアロン機器になります。

```
【従来】
電源ON → Claude Desktop 接続 → lua_run("skysafari_bridge") → SkySafari 接続

【v5.9.5 以降】
電源ON → skysafari_bridge 自動起動 → SkySafari 接続  ← Claude 不要
```

### 2.2 設定方法

**1. スクリプトを保存する**（Claude Desktop から）

```
lua_save: name="skysafari_bridge", code=<スクリプト内容>
```

**2. autostart に設定する**（Claude Desktop から）

```
set_autostart: name="skysafari_bridge"
```

**3. 再起動する**

```
hard_reset
```

以後は電源を入れるだけでスクリプトが自動起動します。

**autostart の解除：**

```
clear_autostart
```

### 2.3 セーフモード

スクリプトにバグがあり起動のたびにクラッシュした場合、LUNA が自動的に保護状態に入ります。

| 条件 | 動作 |
|------|------|
| 連続クラッシュ3回（RTC カウンター） | **自動セーフモード** |
| 起動時 GPIO6 長押し（3秒以上） | **手動セーフモード** |

**セーフモード時：**
- autostart がスキップされる
- LED（GPIO2 内蔵 LED）が**点灯**してセーフモードを明示
- File Manager が自動開放される（スクリプトを修正・削除可能）
- MCP サーバーは通常通り動作（Claude から操作可能）

**セーフモードの解除：**
GPIO6 を押さずに再起動（または電源入れ直し）するだけで通常モードに戻ります。

### 2.4 状態の確認

`get_system_info` の応答に autostart 情報が含まれます：

```json
{
  "autostart_script": "skysafari_bridge",
  "safe_mode": false,
  "boot_count": 0
}
```

| フィールド | 内容 |
|-----------|------|
| `autostart_script` | 設定中のスクリプト名（未設定時は `""`） |
| `safe_mode` | セーフモード中は `true` |
| `boot_count` | 連続失敗回数（3以上でセーフモード） |

### 2.5 パーティション設定の注意

autostart を含む LUNA v5.9.5 は **Huge APP (3MB No OTA/1MB SPIFFS)** パーティションが必要です。

| 環境 | 設定 |
|------|------|
| Arduino Cloud | Huge APP (3MB No OTA/1MB SPIFFS) |
| Arduino IDE | Huge APP (3MB No OTA/1MB SPIFFS) |

> **重要**: Cloud と IDE で異なるパーティションで書き込むと SPIFFS の位置がずれ、
> WiFi 設定・スクリプト・autostart 設定が消えます。必ず両環境を揃えてください。

---

## 3. LUNA + NINA 統合天文撮影システム（v5.9.2）

> **v5.9.2 新機能・実機確認済み（2026-03-27）**

### 2.1 NINA とは

**NINA（Nighttime Imaging 'N' Astronomy）** は、天体撮影を自動化するための無料ソフトウェアです。
カメラ・望遠鏡・フォーカサー・フィルターホイール・ガイドシステムをまとめて制御でき、
天文ファンの間では世界標準のツールになっています。

```
https://nighttime-imaging.eu/  （無料ダウンロード）
```

---

### 2.2 LUNA + NINA で何が変わるか

NINA は単体でも非常に高機能ですが、使いこなすには「シーケンス」と呼ばれる撮影手順を
事前に組む必要があります。これが初心者には大きな壁になっています。

**LUNA を加えることで、この壁がなくなります。**

```
NINA 単体
  ↓
「シーケンス」を事前に組む（30〜50 箇所の設定が必要）
  ↓
設定が終わって初めて撮影できる

─────────────────────────────────

LUNA + NINA
  ↓
Claude に話しかけるだけ
「M42 を 3 分露出で 20 枚撮って」
  ↓
すぐに撮影が始まる
```

| 操作 | NINA 単体 | LUNA + NINA |
|------|---------|------------|
| 撮影開始 | シーケンスを事前に作成して実行 | Claude に言葉で頼むだけ |
| 対象天体の変更 | シーケンスを修正して再実行 | 「次は M31 にして」で完了 |
| 天候悪化への対応 | 事前に条件を設定しておく必要がある | Claude が気象センサーを監視して自動判断 |
| 設定の知識 | ゲイン・HFR・ディザリング等の知識が必要 | Claude が補完・提案してくれる |
| 機器トラブル時 | エラーログを自分で調べる | Claude が状況を説明・対処法を提案 |

---

### 2.3 フル自動化の構成

LUNA + NINA を組み合わせると、以下のすべてを Claude への一言で実行できます。

```
「今夜 M42 を撮影して」

  ① 気象センサー確認 → 観測可能か判断       （LUNA + AiBridge/Alpaca）
  ② 望遠鏡を M42 へ自動導入                 （Alpaca）
  ③ オートフォーカス実行                    （NINA HTTP API）
  ④ テスト撮影 → プレートソルブ → 座標補正  （NINA HTTP API）
  ⑤ 本番シーケンス開始                      （NINA HTTP API）
  ⑥ 定期的なフォーカス再調整                （NINA HTTP API）
  ⑦ 気象悪化を検知したら自動中断            （LUNA 監視）
```

**NINA は「腕」、Claude が「頭脳」、LUNA が「神経」です。**
NINA の自律機能はそのまま活かしながら、AI による高度な判断層が加わります。

---

### 2.4 LUNA + NINA が初心者にもたらすメリット

**① 天文知識がなくても始められる**
「M42 を撮りたい」と言えば、Claude が推奨の露出・ゲイン・フィルターを提案します。
専門用語を覚えなくても、撮影を始められます。

**② NINA の設定が難しくてもサポートしてくれる**
設定の意味が分からなければ Claude に聞けます。
「なぜこのゲイン値にしたのか」「ディザリングとは何か」もその場で答えてくれます。

**③ 段階的にスキルアップできる**
最初は Claude に全部任せて、慣れてきたら自分でシーケンスを作る——という
段階的な学習が自然にできます。

**④ 複数機器を統合して管理できる**
望遠鏡・カメラ・気象センサー・ドームを横断して Claude が一括管理します。
NINA だけでは管理できない機器も LUNA が橋渡しします。

---

### 2.5 NINA との接続方法

**必要なもの**
- NINA v3.x（https://nighttime-imaging.eu/ から無料ダウンロード）
- NINA Advanced API プラグイン（NINA 内から無料インストール）
- LUNA と NINA PC が同じ WiFi ネットワーク上にあること

**NINA Advanced API のセットアップ**

```
① NINA を起動
② 左メニューの「プラグイン」アイコンをクリック
③「利用可能」タブ → 検索欄に「Advanced API」と入力
④「NINA Advanced API」をインストール
⑤ NINA を再起動
⑥「Advanced API」タブでポート 1888・API Enabled ON を確認
```

**接続確認（LUNA から）**

```lua
-- NINA への接続テスト
local resp, code = http.get("http://[NINAのPC-IP]:1888/v2/api/application", 3000)
```

**実機確認済みの操作（2026-03-27）**

| 操作 | エンドポイント | 方式 |
|------|--------------|------|
| カメラ情報取得 | `/v2/api/equipment/camera/info` | GET |
| 撮影開始 | `/v2/api/equipment/camera/capture` | POST |
| オートフォーカス | `/v2/api/equipment/focuser/auto-focus` | POST |
| フィルター切替 | `/v2/api/equipment/filterwheel/change-filter` | POST |
| シーケンス開始 | `/v2/api/sequence/start` | POST |
| ガイド開始 | `/v2/api/equipment/guider/start` | POST |

> 詳しい Lua スクリプト例は **LUNA_Observatory_Resources_Guide.md** と **LUNA_Lua_Guide.md** を参照してください。

---

## 3. 初期設定（APモード）

### なぜ APモードがあるのか

LUNA は購入後または初期化後、**必ず APモードで起動**します。
APモードとは、本体自身が WiFi アクセスポイントになる仕組みです。

1. **初回の WiFi 設定を行うため**
2. **DHCP で割り当てられた IP アドレスを確認するため**
3. **ルーターを変更したときに再設定できるようにするため**

AP は常時有効なので、どんな状態でも必ず本体に接続できます。

### 初回設定の流れ

> ### ⚠️ 重要：SSID・パスワード設定の前に必ず SPIFFS をフォーマットしてください
>
> **v5.9.4 以降はパーティション構成が変更されました（SPIFFS: 0.8MB）。**
> 新規ファームウェア書き込み後、初めて接続した際は WiFi 設定（SSID/パスワード入力）を行う前に、
> 必ず一度 SPIFFS フォーマットを実行してください。
>
> **手順：** AP（`LUNA_{LocationName}`）に接続 → `http://192.168.4.1` → File Manager → **Format** ボタン → 確認
>
> フォーマットを行わないと WiFi 設定が正しく保存されない場合があります。
> フォーマット後は本体が再起動します。その後 ① から設定を進めてください。

#### ① AP に接続する

Wi-Fi 設定で以下を選択します：

```
LUNA_{LocationName}
```

> ※ パスワードは不要です。そのまま接続してください。

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

> **💡 推奨**: 自宅 WiFi（STA）への接続設定が完了したら、スマホ・PCも自宅 WiFi に戻してください。通常の運用は自宅 WiFi 経由で行うことを推奨します。AP は緊急時の接続手段として常時待機しています。

> **⚠️ 複数台使用時の注意**
> 複数の LUNA を同じネットワークで使用する場合は、**必ず異なる Station Name を設定してください**（例: `LUNA1`、`LUNA2`）。
> Station Name は AP の SSID（`LUNA_LUNA1`）および MCP サーバー名（`LUNA-LUNA1`）に使われます。
> 同じ名前のままだと、複数台の初期設定時に接続先を誤る可能性があります。

### File Manager の開き方

v5.9.6 / v6.9.3 より、**電源投入後 30 分間は自動的に File Manager にアクセスできます**。GPIO39 ボタン操作は不要です。

| 状態 | アクセス |
|------|---------|
| 電源投入後 30 分以内 | 自動的に開放（ボタン不要） |
| 電源投入後 30 分経過 | アクセス不可（再起動で再開放） |

> **セキュリティについて**: LUNA は家庭内 LAN 内での使用を想定しています。電源ONが物理的な認証を兼ねる設計のため、GPIO ボタンなしで 30 分間アクセスできます。  
> 30 分を過ぎた場合は電源を入れ直すことで再び 30 分間アクセスできます。

> **セーフモード時**: クラッシュ検出（連続3回）または GPIO6 長押しでセーフモードが有効になると、30 分制限に関わらず File Manager に常時アクセスできます。

---

## 3. ハードウェア構成

### 対応ハードウェア（バイナリ選択）

LUNA ファームウェアは基板に合わせて 2 種類のバイナリを提供します。

| バイナリ | 対象基板 | RS-232C ポート |
|---------|---------|--------------|
| `LUNA_v3.9.1_Standard.bin` | 汎用 ESP32 開発ボード | Serial2（GPIO16 / GPIO17） |
| `LUNA_v3.9.1_OnStepNinja_V2.bin` | OnStepNinja V2 基板 | Serial（UART0） |

使用している基板に合ったバイナリを書き込んでください。
現在動作中のバイナリは `get_network_status` の `target_board` フィールドで確認できます。

```
例: {"target_board": "Standard", "serial_baud": 9600, ...}
```

### 必要な機材

| 品目 | 仕様 | 備考 |
|------|------|------|
| ESP32 開発ボード | ESP32-WROOM-32 推奨 | LUNA ファームウェア書き込み済み |
| RS-232C 変換モジュール | MAX3232 等 | 3.3V ↔ RS-232C 変換（必須） |
| RS-232C ケーブル | D-sub 9ピンまたは 25ピン | 機器に合わせて選択 |
| WiFi ルーター | 2.4GHz 対応 | インターネット接続済み |
| PC（Claude Desktop 用） | Windows / Mac | Claude Desktop インストール済み |

> **v6.9.3 は ESP32-S3（FREENOVE WROOM N8R8 推奨）専用です。** 標準の ESP32-WROOM-32 では動作しません。

### GPIO ピン配置（v6.9.3 / ESP32-S3 FREENOVE WROOM N8R8）

#### システムピン

| GPIO | 機能 | 備考 |
|------|------|------|
| GPIO 2  | 内蔵 LED / File Manager LED | 起動表示・File Manager 動作中表示 |
| GPIO 6  | セーフモードボタン | 起動時 LOW 長押し（3秒）でセーフモード |
| GPIO 14 | Reset Mode Select | HIGH=ソフトリセット / LOW=ハードリセット |
| GPIO 17 | UART2 TX | ESP32-S3 → 機器（RS-232C経由） |
| GPIO 18 | UART2 RX | 機器 → ESP32-S3（RS-232C経由） |
| GPIO 21 | USER LED | 動作表示 LED（Output Pin 3 と共用） |
| GPIO 43 | UART0 TX (USB) | PC 接続・書き込み・デバッグ用 |
| GPIO 44 | UART0 RX (USB) | PC 接続・書き込み・デバッグ用 |
| GPIO 48 | NeoPixel RGB LED | WS2812B 内蔵 RGB（`hw.rgb` / `hw.rgb_blink` / `hw.rgb_flag`） |

#### Output ピン（Lua: `gpio.output_pin(n)`）

| GPIO | Output番号 | 機能 | 備考 |
|------|-----------|------|------|
| GPIO 38 | Output 1 | PLS | ステッパー STEP パルス |
| GPIO 39 | Output 2 | DIR | ステッパー方向制御 |
| GPIO 21 | Output 3 | LED | USER LED と共用 |
| GPIO 47 | Output 4 | — | 汎用出力（未割当） |

> **GPIO 禁止ピン（ESP32-S3 N8R8）**: GPIO26〜32（Flash バス）、GPIO35〜37（PSRAM バス）は使用禁止。

### 配線

```
【LUNA (ESP32-S3)】          【RS-232C機器】

  TX2 (GPIO17) ──────────────► RXD
  RX2 (GPIO18) ◄────────────── TXD
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

### 複数の LUNA を同時接続する

2台以上の LUNA を Claude Desktop から同時制御できます。サーバー名を変えて並べるだけです。

```json
{
  "mcpServers": {
    "AiBridgeMCP": {
      "command": "C:\\path\\to\\aibridge-connector-v2.exe",
      "args": ["192.168.3.50"]
    },
    "AiBridgeMCP2": {
      "command": "C:\\path\\to\\aibridge-connector-v2.exe",
      "args": ["192.168.3.154"]
    }
  }
}
```

登録後に Claude Desktop を再起動すると、ツール一覧に `mcp__AiBridgeMCP__*` と `mcp__AiBridgeMCP2__*` が並び、2台を同時に操作できます。

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

#### `lua_save`

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
| **パラメータ** | `name`（必須）: `lua_save` で保存したスクリプト名 |
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

Lua スクリプトの `notify.set()` による通知フラグとメッセージを確認します。

| 項目 | 内容 |
|------|------|
| **パラメータ** | なし |
| **戻り値（通知なし）** | `{"flag":0, "uptime":..., "heap":...}` |
| **戻り値（通知あり）** | `{"flag":1, "message":"完了メッセージ", "uptime":..., "heap":...}` |

**使い方**: Lua スクリプト内で `notify.set("メッセージ文字列")` を呼ぶと、Claude が `check_notify` でフラグとメッセージを受け取れます。フラグとメッセージは読み取り後に自動リセットされます。

**通知の確認方法（Claude Desktop）**: スクリプト実行後、Claude に「notify を確認して」と伝えるだけで `check_notify` を自動的に呼び出します。

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

### 5.6 Resources 系（v5.9.1 新機能）

> **📖 詳細**: `LUNA_Observatory_Resources_Guide.md` を参照してください。

MCP Resources は「計器」です。望遠鏡の向き・気温・スイッチ状態などを、Claude が会話の中で自然に読み取れるようにします。

#### Tools と Resources の違い

| | Tools | Resources |
|--|-------|-----------|
| **感覚** | ボタンを押す | 計器を見る |
| **用途** | コマンド送信・操作 | 状態・データの読み取り |
| **Claude の使い方** | 明示的に呼び出す | 文脈に応じて自然に参照 |

**例：**

```
【Tools のみ（従来）】
ユーザー:「今どこ向いてる？」
Claude: serial_query(":GR#") を呼ぶ → serial_query(":GD#") を呼ぶ → 回答
→ 2回のツール呼び出しが必要

【Resources 実装後（v5.9.1）】
ユーザー:「今どこ向いてる？」
Claude: luna://lua/mount を読む → 即答
→ 1回で完結・文脈の中で自然に参照
```

#### MCP ツール

| ツール名 | 機能 |
|---------|------|
| `resources/list` | 公開中の Resources 一覧を取得（Claude Desktop が自動参照） |
| `resources/read` | 指定 URI の Resource の現在値を取得 |

#### Resource の作り方（Lua スクリプト）

SPIFFS に `lua_resource_{名前}.lua` という名前でスクリプトを保存するだけで、自動的に Resource として公開されます。

```
ファイル名                    → URI（Claude が参照する名前）
lua_resource_mount.lua    → luna://lua/mount
lua_resource_weather.lua  → luna://lua/weather
lua_resource_switch.lua   → luna://lua/switch
```

**スクリプトの基本形:**

```lua
-- lua_resource_mount.lua（赤道儀 RA/Dec 取得）
local ra  = serial.query(":GR#", 100)
local dec = serial.query(":GD#", 100)
if ra == "" or dec == "" then
  return '{"error":"no response from mount"}'
end
return string.format('{"ra":"%s","dec":"%s"}', ra, dec)
```

**規約:**

| 項目 | 内容 |
|------|------|
| 戻り値 | `return` で JSON 文字列を返す（必須） |
| エラー時 | `'{"error":"メッセージ"}'` を返す |
| 副作用 | 機器を動かすコマンドは含めない（読み取りのみ推奨） |
| 実行時間 | できるだけ短く（`serial.query` は 100〜200ms 推奨） |

#### Claude Desktop での使い方

Claude Desktop に LUNA Observatory が接続されていれば、自然な会話で Resources を参照できます。

```
ユーザー:「今の望遠鏡の向きを教えて」
Claude:  luna://lua/mount を読んで → RA/Dec を回答

ユーザー:「観測できる状態か確認して」
Claude:  luna://lua/mount・luna://lua/weather を読んで総合判断して回答
```

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
  → lua_save → lua_run

【Lua スクリプトの実行状況を確認したい】
  → lua_status

【Lua スクリプトのログを確認したい】
  → lua_log

【Lua スクリプトを途中で止めたい】
  → lua_stop

【保存済みスクリプト一覧を確認したい】
  → lua_list

【Lua スクリプトから完了・異常を Claude に通知したい】
  → スクリプト内で notify.set("メッセージ") → Claude が check_notify で検知

【LUNA が正常動作しているか確認したい】
  → get_system_info または get_network_status

【log.save() で保存したデータを Claude から確認したい】
  → fs_read("/luna_log.txt")

【機器の現在状態を Claude が自然に読み取れるようにしたい（v5.9.1）】
  → lua_resource_{名前}.lua を lua_save で保存 → luna://lua/{名前} として自動公開

【どんな Resources が公開されているか確認したい（v5.9.1）】
  → Claude に「利用可能なリソースを確認して」と依頼（resources/list が実行される）

【LAN 内の Alpaca/HTTP デバイスを Lua から制御したい（v5.9.1）】
  → Lua スクリプト内で http.get() / http.put() を使用
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
  lua_save("voltage_monitor", <スクリプト>) → SPIFFS に保存
  lua_run("voltage_monitor")               → 実行開始

③ Claude は離席してOK（ESP32 が単独で動き続ける）

④ 異常発生時
  notify.set("電圧異常: 0.8234V") が発火
  Claude が check_notify でメッセージを受信
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

#### http モジュール（v5.9.1 新機能）

LAN 内の HTTP デバイスへリクエストを送信します。AiBridge（ASCOM/Alpaca）との連携に使用します。

| API | 説明 |
|-----|------|
| `http.get(url, timeout_ms)` | HTTP GET。レスポンスボディを返す（エラー時 `""`） |
| `http.put(url, body, timeout_ms)` | HTTP PUT。レスポンスボディを返す（エラー時 `""`） |

**注意**:
- HTTPS 非対応（LAN 内 HTTP 専用）
- ブロッキング動作。タイムアウトは短めに設定（GET: 1000ms / PUT: 2000ms 推奨）
- エラー・タイムアウト時は必ず `""` チェックを行うこと

```lua
-- AiBridge スイッチ状態取得
local s0 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=0", 1000)
if s0 == "" then return '{"error":"no response"}' end
return '{"port0":' .. s0 .. '}'

-- AiBridge スイッチ ON
local r = http.put("http://192.168.3.7/api/v1/switch/0/setswitch", "Id=0&State=true", 1000)
```

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
| **Lua自律制御** | lua_exec / lua_save / lua_run / lua_status / lua_stop / lua_log / lua_list |
| **通知** | check_notify |
| **システム情報** | get_system_info / get_network_status / get_license_info / hard_reset |
| **ファイルシステム** | fs_list / fs_read / fs_write / fs_append / fs_delete |

### ドキュメント一覧

| ファイル | 用途 |
|---------|------|
| `LUNA_Quick_Setup_Guide.md` | **初回セットアップ（3ステップ・まずここから）** |
| `LUNA_USER_MANUAL.md` | このファイル（完全版マニュアル） |
| `LUNA_Lua_Guide.md` | Lua スクリプト詳細ガイド |
| `LUNA_SERIAL_BRIDGE_GUIDE.md` | シリアルブリッジ詳細ガイド |
| `LUNA_Observatory_Resources_Guide.md` | MCP Resources・複数機器統合ガイド |
| `Temma_Protocol_Spec.md` | Takahashi Temma2 プロトコル仕様 |

---

*LUNA v5.9.5 / MIT License*
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

---

## 4. Lua UI フレームワーク — ブラウザ連携 UI（v5.9.6 / v6.9.3）

### 4.1 概要

v5.9.6 / v6.9.3 から、Lua スクリプトがブラウザ（スマホ・タブレット・PC）と **リアルタイムで通信**できる UI フレームワークを内蔵しています。

```
ブラウザ（スマホ等）
    │ GET /api/ui/messages（1秒ポーリング）
    │ POST /api/ui/button?name=xxx
    ↓
LUNA（ESP32）
    │ ui.send() / ui.check()
    ↓
Lua スクリプト（制御ループ）
```

Lua スクリプトが観測・制御を行いながら、ブラウザに状態を表示し、ボタン操作を受け取ることができます。

---

### 4.2 ui モジュール API

| 関数 | 説明 |
|------|------|
| `ui.send(id, msg)` | 指定 ID のブロックにメッセージ送信（複数ブロック個別更新可） |
| `ui.check()` | ノンブロッキング: ボタン名 or `"none"` 返却・取得後自動リセット |
| `ui.poll(ms)` | ブロッキング: 最大 ms ミリ秒待機。タイムアウト時 `"timeout"` 返却 |
| `ui.switch(url)` | ブラウザを指定 URL に切り替え（GET レスポンス後自動リセット） |
| `ui.clear()` | 全メッセージ・ボタン・switch_target をリセット |

---

### 4.3 HTTP エンドポイント（ブラウザ側）

| エンドポイント | メソッド | 説明 |
|--------------|---------|------|
| `/api/ui/messages` | GET | 全メッセージ + switch_target を JSON で返す |
| `/api/ui/button` | POST | `?name=xxx` でボタン名を送信 |

---

### 4.4 基本的な使用例

```lua
-- 起動時に UI 初期化
ui.clear()
ui.send("status", "観測準備中")
ui.send("target", "M42 オリオン大星雲")

local running = true
while running do
  -- 望遠鏡座標を取得して表示
  local ra = serial.query(":GR#", 200)
  ui.send("ra", ra)

  -- ボタン確認（ノンブロッキング）
  local btn = ui.check()
  if btn == "stop" then
    running = false
  elseif btn == "goto" then
    serial.write(":MGo#")
    ui.send("status", "GoTo 実行中...")
  elseif string.match(btn, "^input:") then
    local val = btn:sub(7)   -- "input:" の後の文字列
    ui.send("target", val)
  end

  hw.delay(300)
end

ui.send("status", "終了")
notify.set("観測完了")
```

---

### 4.5 HTML テンプレート（SPIFFS に保存）

ブラウザ側の HTML は SPIFFS（`/lua_ui.html` 等）に置き、Claude が `fs_write()` で書き込みます。

**最小テンプレート構成:**
- メッセージブロック: `<div id="ui-{id}">` に対応した要素
- ボタン: `fetch('/api/ui/button?name=xxx', {method:'POST'})` を送信
- 入力: `fetch('/api/ui/button?name=input:'+値, {method:'POST'})` を送信
- ポーリング: `setInterval(() => fetch('/api/ui/messages').then(...)`, 1000)`

詳細テンプレートは `LUNA_Lua_Guide_EN.md` Section 2-14 を参照してください。

---

### 4.6 二層 UI アーキテクチャ

```
電源 ON
  ↓
PROGMEM Console UI（/dashboard）← 常に存在・フォールバック
  ↓ autostart 実行
Lua スクリプト起動
  ↓ ui.switch("/lua_ui.html")
カスタム Lua UI（SPIFFS）← Claude が事前準備
```

- **Console UI**: 常時利用可能な管理用ダッシュボード（ファームウェア内蔵）
- **Lua UI**: Claude が用途に応じて作成する動的 UI（SPIFFS に格納）

---

### 4.7 Console UI — Lua Script パネル詳細

`http://<LUNA IP>/dashboard`（または `http://192.168.4.1/dashboard`）を開くと Console UI が表示されます。

#### Lua Script パネル

```
┌─────────────────────────────────────────────┐
│ Lua Script                                  │
│ autostart: telescope_ctrl                   │
│ state: ● running — telescope_ctrl           │
│ uptime: 1h 23m 45s   heap: 249.3 KB         │
│ [▶ Run]  [■ Stop]  [↗ Open UI]              │
└─────────────────────────────────────────────┘
```

| 表示項目 | 内容 | 更新 |
|---------|------|------|
| autostart | 登録されたスクリプト名（未設定時は案内表示） | 2秒 |
| state | `● running — スクリプト名` / `○ idle` | 2秒 |
| uptime | 起動からの経過時間（例: `1h 23m 45s`） | 2秒 |
| heap | 空きヒープメモリ（例: `249.3 KB`） | 2秒 |

#### ボタン

| ボタン | 動作 | 有効条件 |
|--------|------|---------|
| `▶ Run` | autostart スクリプトを即時実行 | autostart 登録済み・idle 時 |
| `■ Stop` | 実行中スクリプトを強制停止 | running 時 |
| `↗ Open UI` | Lua UI HTML を新タブで開く | autostart 登録済み時 |

> **heap の目安**: v6.9.3 (ESP32-S3) の通常動作時は 240〜260 KB。Lua スクリプト実行中は 200 KB 以上あれば安定動作します。heap が大きく減少している場合はメモリリークの可能性があります。

---

---

## 5. LUNA アプリ配布システム — 3ファイルで完結（v5.9.6 / v6.9.3）

> **★ 画期的な仕組み** — Claude AI なし・プログラミング知識なし。3ファイルをアップロードするだけでアプリが動きます。

### 5.1 概要

LUNA v5.9.6 / v6.9.3 では、アプリの配布とインストールが**極めてシンプル**になりました。

従来は Claude AI を使ってスクリプトを保存・設定する必要がありました。  
**v5.9.6 以降は File Manager に3ファイルをアップロードするだけです。**

---

### 5.2 配布パッケージの構成

アプリの配布に必要なファイルはたった **3つ** です。

```
my_app/
  ├── lua_my_app.lua        ← Lua スクリプト（制御ロジック）
  ├── my_app.html           ← HTML UI（ブラウザ操作画面）
  └── luna_autostart.txt    ← 起動設定（内容: "my_app" の1行）
```

> **命名規則（重要）**: `lua_{name}.lua` / `{name}.html` / `luna_autostart.txt` の3点セット。  
> `name` の部分を統一することで、Dashboard の **[↗ Open UI]** ボタンが自動的に正しい HTML を開きます。

---

### 5.3 インストール手順

```
1. ブラウザで http://192.168.x.x/dashboard を開く
2. [File Manager] ボタンをクリック
3. 3つのファイルをアップロード
4. 完了
```

**以上です。** 再起動不要。Claude AI 不要。コマンド入力不要。

---

### 5.4 インストール後の動作

```
電源 ON
  ↓ luna_autostart.txt を読み込み
  ↓ Lua スクリプトが自動起動

ブラウザで /dashboard を開く
  ↓ [▶ Run]      → スクリプトを手動起動
  ↓ [■ Stop]     → 実行中スクリプトを停止
  ↓ [↗ Open UI]  → HTML UI を新タブで表示
```

スマホ・タブレット・PC のブラウザから操作できます。

---

### 5.5 従来方式との比較

| 項目 | 従来（v5.9.5 以前） | LUNA v5.9.6+ |
|------|-------------------|-------------|
| スクリプト保存 | Claude に `lua_save` を依頼 | File Manager でアップロード |
| HTML 保存 | `fs_write` を複数回実行 | File Manager でアップロード |
| autostart 設定 | Claude に `set_autostart` を依頼 | `luna_autostart.txt` をアップロード |
| **Claude AI** | **必要** | **不要** |
| **難易度** | **中〜高** | **極めて簡単** |

---

### 5.6 アプリ開発者向け配布方法

配布パッケージは ZIP にまとめて配布できます：

```
my_telescope_app_v1.0.zip
  ├── lua_my_app.lua
  ├── my_app.html
  └── luna_autostart.txt
```

**ユーザーの操作**: ZIP を展開 → File Manager に3ファイルをアップロード → 完了。

アプリ開発者は Lua スクリプトと HTML を作成するだけで、世界中の LUNA ユーザーに配布できます。

---

### 5.7 複数アプリの切り替え

複数のアプリを SPIFFS に保存しておき、`luna_autostart.txt` の内容を変更するだけで切り替えられます。

```
SPIFFS に保存済み:
  /lua_telescope_ctrl.lua   ← 望遠鏡制御アプリ
  /lua_weather_monitor.lua  ← 気象モニターアプリ
  /lua_skysafari_bridge.lua ← SkySafari ブリッジアプリ

luna_autostart.txt の内容を変更するだけで切り替え:
  "telescope_ctrl"  → 望遠鏡制御アプリが起動
  "weather_monitor" → 気象モニターアプリが起動
```

Claude AI に `set_autostart("telescope_ctrl")` と依頼するか、  
File Manager で `luna_autostart.txt` を書き換えるだけです。

---

## 💖 開発継続のためのご支援のお願い

LUNA は個人開発プロジェクトです。世界中の天文ファンの皆様に AI 制御の楽しさを無償でお届けすることを目標としています。

開発を継続し、さらに多くの機器・機能に対応していくには、皆様のご支援が大変大きな励みになります。
**少額でも構いません**。どうかご支援をよろしくお願いいたします。

| 方法 | 詳細 |
|------|------|
| 💳 **PayPal でのご寄付** | **nishioka.sst@gmail.com** 宛にお送りください（[PayPal サイト](https://www.paypal.com/)からメールアドレスで送金） |
| 🛒 **基板のご購入** | **LUNA OnStepNinja V2** 基板をご購入いただくことが開発継続の直接的な支えになります。[GitHub Issues](https://github.com/OnStepNinja/LUNA/issues) よりお問い合わせください |

> ご支援いただいた皆様のおかげで、LUNA はより良いプロジェクトへと成長し続けることができます。心より感謝申し上げます。🙏

---

Legal Disclaimer:
This product is an independently developed hardware by OnStepNinja and is not affiliated with, endorsed by, or sponsored by Anthropic, Takahashi Seisakusho, or any other company.
"Temma", "MCP" and other product names are trademarks of their respective owners.
Use of these names is solely for the purpose of describing compatibility (Nominative Fair Use).

本プロジェクトは各社とは無関係の個人によるプロジェクトです。
