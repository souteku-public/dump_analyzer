# Minidump Analyzer

ブラウザで動作する Windows クラッシュダンプ (.dmp) 解析ツール。
TouchDesigner をはじめとする Windows アプリケーションのクラッシュダンプを、
WinDbg 等の専用ツールなしで解析できます。

**[ツールを開く →](https://souteku-public.github.io/minidump_analyzer/)**

## 特徴

- **インストール不要** — ブラウザでURLを開くだけで使えます
- **完全ローカル処理** — ファイルデータは一切外部に送信されません
- **大容量対応** — 10GB超のフルメモリダンプも解析可能
- **3つのダンプ形式に対応** — MDMP, PAGEDUMP, PAGEDU64

## 対応フォーマット

|
 シグネチャ 
|
 形式 
|
 説明 
|
|
-----------
|
------
|
------
|
|
`MDMP`
|
 Windows Minidump 
|
 ミニダンプ（MiniDumpWriteDump で生成） 
|
|
`PAGEDUMP`
|
 32-bit Crash Dump 
|
 Windows 32-bit フル/カーネルクラッシュダンプ 
|
|
`PAGEDU64`
|
 64-bit Crash Dump 
|
 Windows 64-bit フル/カーネルクラッシュダンプ 
|

## 解析できる情報

### 共通
- **ヘッダー情報** — シグネチャ、バージョン、タイムスタンプ、ファイルサイズ
- **システム情報** — OS バージョン、CPU アーキテクチャ、プロセッサ数
- **例外情報** — 例外コード（ACCESS_VIOLATION 等）、例外アドレス、パラメータ

### MDMP（ミニダンプ）
- **モジュール一覧** — ロードされた DLL/EXE のベースアドレス、サイズ、パス
- **スレッド一覧** — スレッドID、優先度、スタック情報
- **クラッシュモジュール特定** — 例外アドレスからクラッシュしたモジュールを自動判定
- **TouchDesigner 関連モジュール** — TD 関連の DLL を自動検出
- **ストリームディレクトリ** — 全ストリームの種別・サイズ一覧

### PAGEDUMP / PAGEDU64（フルクラッシュダンプ）
- **BugCheck (BSOD) 情報** — BugCheck コード（IRQL_NOT_LESS_OR_EQUAL 等）とパラメータ
- **ダンプ種別** — Full Dump / Kernel Dump / Bitmap Dump

### JSON エクスポート
解析結果を JSON ファイルとしてダウンロード可能。他のツールでの二次分析や記録に活用できます。

## 使い方

1. [ツールのURL](https://souteku-public.github.io/minidump_analyzer/) をブラウザで開く
2. `.dmp` ファイルをドラッグ＆ドロップ（またはクリックして選択）
3. 解析結果が画面に表示される
4. 必要に応じて「JSON エクスポート」でレポートをダウンロード

## 大容量ファイル対応

10GB を超えるフルメモリダンプ（`MiniDumpWithFullMemory`）にも対応しています。

ファイル全体をメモリに読み込まず、`File.slice()` で必要なヘッダー・メタデータ部分のみを段階的に読み込むため、ブラウザのメモリ制限に影響されません。

|
 ダンプ形式 
|
 読み込み方式 
|
 実際の読み込み量 
|
|
-----------
|
-------------
|
----------------
|
|
 PAGEDUMP / PAGEDU64 
|
 先頭 8KB のみ 
|
 8 KB 
|
|
 MDMP（小） 
|
 ファイル全体 
|
 ファイルサイズと同等 
|
|
 MDMP（大・10GB等） 
|
 ヘッダー → ディレクトリ → メタデータのみ 
|
 通常 数MB 以下 
|

## プライバシー・セキュリティ

- **すべての処理はブラウザ内で完了** します
- ファイルデータがネットワークに送信されることは **一切ありません**
- サーバーサイドの処理は存在しません
- オフライン環境でも動作します（HTMLファイルをローカルに保存して使用可能）

## 技術仕様

### 使用している Web API

|
 API 
|
 用途 
|
|
-----
|
------
|
|
`File`
 / 
`File.slice()`
|
 ファイルの部分読み込み（大容量対応） 
|
|
`FileReader.readAsArrayBuffer()`
|
 バイナリデータの読み込み 
|
|
`DataView`
|
 リトルエンディアンでのバイナリ解析 
|
|
`TextDecoder('utf-16le')`
|
 モジュール名の UTF-16 文字列デコード 
|
|
`Blob`
 / 
`URL.createObjectURL()`
|
 JSON エクスポートのダウンロード生成 
|
|
 Drag and Drop API 
|
 ファイルのドラッグ＆ドロップ 
|

### 外部依存

なし。CDN、npm パッケージ、外部ライブラリへの依存は一切ありません。

### 対応する例外コード（一部）

|
 コード 
|
 名前 
|
|
--------
|
------
|
|
`0xC0000005`
|
 ACCESS_VIOLATION 
|
|
`0xC00000FD`
|
 STACK_OVERFLOW 
|
|
`0xC0000409`
|
 STACK_BUFFER_OVERRUN 
|
|
`0xE06D7363`
|
 CPP_EXCEPTION 
|
|
`0xC0000094`
|
 INTEGER_DIVIDE_BY_ZERO 
|

### 対応する BugCheck コード（一部）

|
 コード 
|
 名前 
|
|
--------
|
------
|
|
`0x0000000A`
|
 IRQL_NOT_LESS_OR_EQUAL 
|
|
`0x00000050`
|
 PAGE_FAULT_IN_NONPAGED_AREA 
|
|
`0x00000116`
|
 VIDEO_TDR_TIMEOUT_DETECTED 
|
|
`0x00000139`
|
 KERNEL_SECURITY_CHECK_FAILURE 
|
|
`0x000000EF`
|
 CRITICAL_PROCESS_DIED 
|

## 動作環境

- Chrome 67+ / Edge 79+ / Firefox 68+ / Safari 15+
- `BigInt` および `DataView.getBigUint64()` をサポートするモダンブラウザが必要
- Internet Explorer は非対応
