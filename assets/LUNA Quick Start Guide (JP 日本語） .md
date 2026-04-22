# LUNA クイックスタートガイド

**対象バージョン**: v5.9.6 (ESP32) / v6.9.3 (ESP32-S3)
**更新日**: 2026-04-21

---

> 💡 **マニュアルを読まなくても使えます。**
> 繋いで Claude に話しかければ、Claude が案内してくれます。

---

## 必要なもの

| 必要なもの | 入手先 | 費用 |
|-----------|--------|------|
| **Claude Desktop** | [https://claude.ai/download](https://claude.ai/download) | 無料 |
| **LUNA ファームウェア** | [GitHub Releases](https://github.com/OnStepNinja/LUNA/releases) | 無料 |
| **Windows PC** | LUNA と同じ Wi-Fi に接続済みのもの | — |

> 💡 **Claude Code は不要です。** Claude Desktop（無料）だけで LUNA の全機能を利用できます。
> 設定や操作に迷ったときも、Claude Desktop 上で Claude に相談しながら進められます。

---

## LUNA で制御できる機器

RS-232C ポートを持つ機器であれば、原則として何でも制御できます。

| 分野 | 機器・ソフト例 |
|------|--------------|
| **産業・製造** | NC工作機械・PLC・インバータ・ロボットコントローラ |
| **計測・研究** | マルチメータ・オシロスコープ・電源装置・スペクトラムアナライザ |
| **天文（機器）** | Takahashi Temma2・Celestron 等の望遠鏡マウント |
| **天文（ソフト）** | NINA・Stellarium・ASCOM/Alpaca 対応機器全般 |
| **天文（INDI）** | INDI 対応機器全般（望遠鏡・カメラ・フォーカサー等）v6.9.3 専用 |
| **放送・AV** | スイッチャー・VTRコントローラ |

> 機器のマニュアル（PDF等）を Claude に渡すだけで、すぐに制御を開始できます。

---

## 困ったときは

> 🔗 **[LUNA Docs を開く（NotebookLM）](https://notebooklm.google.com/notebook/f2ce997f-18c2-44c4-8738-973b204c190c)**
>
> LUNA の使い方・設定・Lua スクリプトなど、何でも直接質問できます。
> Google アカウントをお持ちの方はそのままご利用いただけます。

---

## セットアップ（3ステップ）

### 1. フォルダを配置する

以下から `connector` フォルダをダウンロードしてください（無料）:

> 📦 **BOOTH**: [https://onstepninja.booth.pm/items/8237213](https://onstepninja.booth.pm/items/8237213)

ダウンロードした ZIP を解凍し、`connector` フォルダを PC の任意の場所（例: `C:\LUNA\`）に置く。
**一度置いたら移動しない**（パスが変わると接続できなくなる）。

---

### 2. LUNA の IP アドレスを確認する

LUNA が自宅ルーターに接続済みの状態で（STA モード）、割り当てられた IP アドレスを確認します。

確認方法:
- ルーターの DHCP クライアント一覧で **LUNA** のアドレスを確認
- または LUNA Web コンソール `http://192.168.4.1`（AP モード）→ **Network Information** → **STA IP** を確認

この **STA IP アドレス**（例: `192.168.3.50`）を次のステップで使います。

---

### 3. Claude Desktop の設定を書き換える

#### ① 設定ファイルを開く

キーボードの **Windows キー + R** を同時に押します。
「ファイル名を指定して実行」が開いたら、以下をそのままコピーして貼り付けて **Enter**:

```
%APPDATA%\Claude
```

フォルダが開きます。その中に **`claude_desktop_config.json`** があります。
（ない場合は新規作成してください）

このファイルを**メモ帳**などのテキストエディタで開きます。

> 💡 **操作に迷ったら、Claude に質問するのが一番確実です。**

---

#### ③ 内容を書き換えて保存する

以下をそのままコピーして、ファイルの内容を**全て書き換えて**保存します。

```json
{
  "mcpServers": {
    "LUNA": {
      "command": "C:\\LUNA\\connector\\aibridge-connector-v2.exe",
      "args": ["192.168.3.xxx"]
    }
  }
}
```

**書き換える箇所は1か所だけです:**

`"192.168.3.xxx"` → Step 2 で確認した LUNA の STA IP アドレスに変える
（例: `"192.168.3.50"`）

> ⚠️ **注意**: `command` のパス内の `\\` は2つ重ねたバックスラッシュです。
> `connector` フォルダを別の場所に置いた場合はパスも変更してください。
> （例: デスクトップに置いた場合 → `"C:\\Users\\あなたの名前\\Desktop\\connector\\aibridge-connector-v2.exe"`）

---

#### ③ Claude Desktop を完全に終了する

設定を保存したら、Claude Desktop を完全に終了します。
最も確実な方法は**タスクマネージャー**を使うことです:

1. **Ctrl + Shift + Esc** を押してタスクマネージャーを開く
2. 「プロセス」一覧から **Claude** を見つける
3. 右クリック → **「タスクの終了」**

> 💡 **操作に迷ったら、Claude に質問するのが一番確実です。**
> 終了前に「タスクマネージャーで Claude を終了する方法を教えて」と聞いてみましょう。

---

#### ④ 接続を確認する

Claude Desktop を起動し、チャット入力欄の左下にある **「+」** をクリック。
**「Connectors」** を選んで **LUNA** が 🔵 青色（ON）になっていれば接続成功です。

> 💡 **分からないことがあれば、Claude AI に質問しましょう。**
> セットアップ手順でつまずいた場合も、Claude に状況を説明すれば一緒に解決してくれます。

---

### 接続確認

Claude に話しかける：

> 「LUNA デバイスの状態を教えて」

IP アドレス・ファームウェアバージョン・空きメモリが返ってきたら正常動作。

---

## LUNA でできること — シリアルブリッジという革命

RS-232C ポートを持つ**あらゆる機器**を、Claude AI で直接制御できます。

```
従来のアプローチ:
  機器購入 → 専門家がマニュアル習得 → 制御プログラム開発 → テスト
  合計: 数ヶ月〜数年、数百万円のコスト
  問題: 担当者が退職すると誰も使えなくなる

LUNA + AI のアプローチ:
  機器購入 → RS-232C ケーブルで LUNA に接続（数分）
           → マニュアルを Claude に渡す（数分）
           → 「この機器で○○して」と話しかけるだけ
  合計: 数十分、追加開発コストほぼゼロ
  利点: マニュアルさえあれば、誰でも、いつでも使える
```

工場の生産設備・研究室の計測器・天文機器——**機器のマニュアルを Claude に渡すだけで AI システムになります。**

さらに Lua スクリプトを使えば、Claude が不在でも ESP32 が自律的に動き続けます。

---

## Claude への話しかけ方

接続できたら、あとは自由に話しかけるだけです。

```
「今どんなことができるか教えて」

「望遠鏡（NS-5000）の現在の RA/Dec を教えて」

「SkySafari で接続できるように設定して」

「NINA と連携して 30 秒露光を1枚撮って」

「Stellarium と連携して現在の望遠鏡位置を表示して」

「Lua スクリプトで10秒ごとに RA を記録するものを作って」
```

---

## Claude に伝えておきたい3つのこと

### 📖 get_lua_guide — まず最初に

> 「get_lua_guide を取得して」

LUNA には **Lua スクリプト**という仕組みがあり、望遠鏡の自動制御や定期観測などを ESP32 単体で自律実行できます。Claude がこの機能を使いこなすには、LUNA の設計書（Lua API リファレンス）を事前に読ませる必要があります。

`get_lua_guide` はその設計書を Claude に届けるツールです。**複雑な操作を依頼する前に、まずこれを取得させると、より正確なスクリプトを書いてくれます。**

---

### 🔭 Vixen 対応 — ビクセン製赤道儀をつなぐ（v5.9.6 / v6.9.3）

> 「Vixen の赤道儀を WiFi で接続する Lua スクリプトを作って」

LUNA は **Vixen Wireless Unit** および **STAR BOOK TEN** のプロトコルに対応しています。ビクセン製の赤道儀をお持ちの方は、専用コントローラーなしに Claude から直接操作できます。

対応機種: AXD / SXP / SXD2 / AP / SX2 / AXJ など

**`get_vixen_guide` ツールに Vixen 専用の使い方が含まれています。** まず以下を Claude に伝えてください:

> 「get_vixen_guide を取得して」

> ⚠️ **Vixen プロトコル情報は非公開です。**
> 本実装に使用したプロトコル仕様は機密情報です。第三者への開示・転載はご遠慮ください。

---

### 📷 NINA 連携 — 天体撮影を AI で自動化（v5.9.6 / v6.9.3）

> 「NINA と連携して 30 秒露光を1枚撮って」

**NINA**（Nighttime Imaging 'N' Astronomy）は Windows 向けの天体撮影ソフトです。LUNA は NINA の **Advanced API プラグイン**を通じて、カメラの制御・撮影開始・露光完了の確認などを Claude の会話から直接行えます。

できること:
- カメラの接続状態・温度・ゲインの確認
- 露光時間・ゲインを指定して撮影開始
- 露光完了を自動で待機して通知

**get_lua_guide に NINA Advanced API の使い方が含まれています。** まず `get_lua_guide` を取得させてから依頼してください。

---

### 🔌 ASCOM / Alpaca 連携 — Windows 標準の機器制御（v5.9.6 / v6.9.3）

> 「Alpaca 経由で望遠鏡の情報を取得して」

**ASCOM** は Windows における天文機器制御の標準規格です。**Alpaca** はその WiFi 版で、HTTP 経由で望遠鏡・カメラ・フォーカサーなどを操作できます。LUNA は Lua の `http` モジュールを使って Alpaca デバイスに直接アクセスできます。

できること:
- 望遠鏡の現在位置（RA/Dec）の取得
- GoTo コマンドの送信
- フォーカサー・カメラなど Alpaca 対応機器全般の操作

**get_lua_guide に Alpaca 連携の使い方が含まれています。** まず `get_lua_guide` を取得させてから依頼してください。

---

### 📡 Resources — リアルタイム状態を Claude が自動で読む（v5.9.6 / v6.9.3）

LUNA には **Resources（リソース）** という仕組みがあります。望遠鏡の現在位置・気象データ・カメラ状態などを「計器」として登録しておくと、Claude が会話の中で自動的に参照できるようになります。

```
登録なし: 「今どこ向いてる？」→ Claude がその都度コマンドを送って取得
登録あり: 「今どこ向いてる？」→ Claude が即座に答える
```

`lua_resource_mount.lua` のような名前のスクリプトを SPIFFS に保存するだけで有効になります。スクリプトの作成は Claude に任せてください。

> 「望遠鏡の RA/Dec を自動で読める Resource スクリプトを作って」

---

### 🌐 INDI サーバー連携 — 天文台全体を AI で制御（v6.9.3 専用）

> 「INDI サーバーに接続して、望遠鏡を RA=18.6 DEC=38.8 に向けて」

**INDI** は Linux（Raspberry Pi など）で動く天文台制御の標準規格です。KStars / Ekos / PHD2 などと連携し、望遠鏡・カメラ・フォーカサーなどを一元管理できます。

v6.9.3（ESP32-S3）では INDI サーバーに直接接続でき、Claude との会話だけで**GoTo・撮影・ガイドの指示**が可能になります。Raspberry Pi と LUNA を組み合わせることで、低コストな DIY 天文台自動化システムが実現します。

INDI には専用の MCP ツールが用意されており、Claude が直接操作します:

| ツール名 | 役割 |
|---------|------|
| `indi_connect` | INDI サーバーに接続 |
| `indi_devices` | 接続中のデバイス一覧を取得 |
| `indi_get` | デバイスの値を取得（RA/Dec など） |
| `indi_set_number` | 数値を設定（GoTo 座標など） |
| `indi_set_switch` | スイッチを操作（追尾ON/OFF など） |

まず以下を Claude に伝えるだけで始められます:

> 「indi_connect で 192.168.x.x の INDI サーバーに接続して」

---

## 2台以上の LUNA を使う場合

`"LUNA-mount"` や `"LUNA-aux"` の部分は **MCP サーバーの名前**です。自由に決められます。
Claude への指示でもこの名前で区別されます（例:「赤道儀の RA を教えて」）。

```json
{
  "mcpServers": {
    "LUNA-mount": {
      "command": "C:\\LUNA\\connector\\aibridge-connector-v2.exe",
      "args": ["192.168.3.50"]
    },
    "LUNA-aux": {
      "command": "C:\\LUNA\\connector\\aibridge-connector-v2.exe",
      "args": ["192.168.3.154"]
    }
  }
}
```

---

## トラブルシューティング

| 症状 | 対処 |
|------|------|
| Connectors に LUNA が出ない | JSON の書式エラー（カンマ・括弧）を確認。再起動。 |
| ファイルエラー | `aibridge-connector-v2.exe`（`-v2` 付き）を使っているか確認。 |
| 通信エラー | ESP32 と PC が**同じ Wi-Fi**にいるか確認。IP が正しいか確認。 |
| 再起動後 IP が変わる | ルーターで LUNA の IP を固定（DHCP 予約）。 |

---

## v6.9.3 (ESP32-S3) と v5.9.6 (ESP32) の違い

| 機能 | v5.9.6 | v6.9.3 |
|------|--------|--------|
| シリアル・Lua・UI・TCP・HTTP | ✅ | ✅ |
| INDI サーバー連携（天文台制御）| ❌ | ✅ |
| RGB LED ステータス（NeoPixel）| ❌ | ✅ |

それ以外の全機能は共通です。

---

*LUNA — Bridge Across Decades © 2026 Nishioka Sadahiko*
*ご支援: PayPal → nishioka.sst@gmail.com / 基板購入 → GitHub Issues*
