# LUNA ファームウェア書き込みガイド

**対象バージョン**: v5.x.x (ESP32) / v6.x.x (ESP32-S3)
**更新日**: 2026-04-21

---

## 書き込み済み基板を購入する（推奨）

手間なく始めたい方は、書き込み済みの **LUNA OnStepNinja V2 基板** をご購入ください。

> 🛒 **BOOTH**: [https://onstepninja.booth.pm/](https://onstepninja.booth.pm/)

購入した場合は **この手順は不要**です。[01_QuickStart.md](01_QuickStart.md) へ進んでください。

---

## 自分で書き込む場合

### ファームウェアのダウンロード

以下の GitHub リリースページからファームウェアをダウンロードしてください:

> 📦 **GitHub Releases**: [https://github.com/OnStepNinja/LUNA/releases](https://github.com/OnStepNinja/LUNA/releases)

### ダウンロードしたフォルダに含まれているファイル

- `LUNA_vX.x.x.bootloader.bin`
- `LUNA_vX.x.x.partitions.bin`
- `LUNA_vX.x.x.bin`
- **`flash_command.txt`** ← 書き込みコマンド（確認済み）

---

### Step 1: Python と esptool をインストール

1. [Python.org](https://www.python.org/) から Python をダウンロード・インストール
   - **重要**: インストール時に「**Add Python to PATH**」にチェックを入れる
2. コマンドプロンプトで実行:
   ```
   pip install esptool
   ```

---

### Step 2: ESP32 を PC に接続

1. 基板により、**USB ケーブルもしくは RS-232C ケーブル**をご用意ください
2. COM ポートを確認:
   - スタートボタン右クリック → **デバイスマネージャー** → 「ポート (COM と LPT)」

---

### Step 3: フラッシュモードに入る（必要な場合）

自動で書き込みが始まらない場合:

1. **BOOT ボタン**と **RESET ボタン**を同時に押し続ける
2. **RESET ボタンを先に離す** → その後 **BOOT ボタンを離す**

---

### Step 4: コマンドを実行する

1. ファイルを置いたフォルダで Shift + 右クリック → 「**PowerShell ウィンドウをここで開く**」
2. `flash_command.txt` を開き、`COM3` を実際のポート番号に書き換えてコピー&ペースト
3. Enter で実行

成功すると以下が表示されます:
```
Hash of data verified.
Hard resetting via RTS pin...
```

**RESET ボタン**を1回押してファームウェアを起動します。

---

### Step 5: WiFi 初期設定

初回起動後、LUNA は AP モードで起動します。

1. PC またはスマートフォンの WiFi 設定を開く
2. **`LUNA_XXXX`**（XXXX は基板固有の文字列）に接続
3. ブラウザで **`http://192.168.4.1`** を開く
4. **File Manager** を開く
   - **初回のみ**: **Format SPIFFS** を実行してください
5. **WiFi Configuration** の **Load Config** ボタンを押してから **SSID** と **Password** を入力し、**Save Config** を押してください
6. **RESET** ボタンを押すと LUNA が再起動し、自宅ルーターに接続
7. ルーターの DHCP 一覧または LUNA Web コンソールで IP アドレスを確認

> **推奨**: ルーターで LUNA の IP を固定（DHCP 予約）すると安定します。

WiFi 設定完了後は **[01_QuickStart.md](01_QuickStart.md)** へ進んでください。

---

## トラブルシューティング

| 症状 | 対処 |
|------|------|
| `'python' not found` | `python3` と入力して再試行 |
| `Connection Failed` | Step 3 のボタン操作を再度行う。充電専用ケーブルは使用不可 |
| `LUNA_XXXX` が見つからない | RESET ボタンを1回押して再起動 |
| WiFi 設定後も接続できない | SSID/PW の入力ミスを確認。5GHz WiFi は非対応（2.4GHz を使用） |

---

*LUNA — Bridge Across Decades © 2026 Nishioka Sadahiko*
