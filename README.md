# Minidump Analyzer

ブラウザで動作する Windows クラッシュダンプ (.dmp) 解析ツールです。TouchDesigner をはじめとする Windows アプリケーションのクラッシュダンプを、WinDbg 等の専用ツールなしで解析できます。

**[ツールを開く →](https://souteku-public.github.io/dump_analyzer/)**

## 特徴

- **インストール不要** — ブラウザでURLを開くだけで使えます
- **完全ローカル処理** — ファイルデータは一切外部に送信されません
- **大容量対応** — 10GB超のフルメモリダンプも解析可能
- **3つのダンプ形式に対応** — MDMP, PAGEDUMP, PAGEDU64

## 対応フォーマット

| シグネチャ | 形式 | 説明 |
|-----------|------|------|
| MDMP | Windows Minidump | ミニダンプ。MiniDumpWriteDump で生成される小型ダンプ |
| PAGEDUMP | 32-bit Crash Dump | Windows 32-bit フル/カーネルクラッシュダンプ |
| PAGEDU64 | 64-bit Crash Dump | Windows 64-bit フル/カーネルクラッシュダンプ |

## 解析できる情報

### 全フォーマット共通

- **ヘッダー情報** — シグネチャ、バージョン、タイムスタンプ、ファイルサイズ
- **システム情報** — OS バージョン、CPU アーキテクチャ、プロセッサ数
- **例外情報** — 例外コード（ACCESS_VIOLATION 等）、例外アドレス、パラメータ

### MDMP（ミニダンプ）固有

- **モジュール一覧** — ロードされた DLL/EXE のベースアドレス、サイズ、フルパス
- **スレッド一覧** — スレッドID、優先度、スタック開始アドレス、スタックサイズ
- **クラッシュモジュール特定** — 例外アドレスからクラッシュの原因となったモジュールを自動判定
- **TouchDesigner 関連モジュール検出** — TouchDesigner、TouchEngine 等の DLL を自動でフィルタリング表示
- **ストリームディレクトリ** — 全ストリームの種別（ThreadList, ModuleList 等）とサイズの一覧

### PAGEDUMP / PAGEDU64（フルクラッシュダンプ）固有

- **BugCheck (BSOD) 情報** — BugCheck コード（IRQL_NOT_LESS_OR_EQUAL、PAGE_FAULT_IN_NONPAGED_AREA 等）と4つのパラメータ
- **ダンプ種別** — Full Dump、Kernel Dump、Bitmap Dump の判定

### JSON エクスポート

解析結果をJSON形式でダウンロードできます。ヘッダー、システム情報、例外情報、モジュール一覧、スレッド一覧、BugCheck情報を含む完全なレポートが出力されます。

## 使い方

1. [ツールのURL](https://souteku-public.github.io/minidump_analyzer/) をブラウザで開く
2. .dmp ファイルをドラッグ＆ドロップ（またはクリックして選択）
3. 解析結果が画面に表示される
4. 必要に応じて「JSON エクスポート」ボタンでレポートをダウンロード

## 大容量ファイル対応

10GB を超えるフルメモリダンプ（MiniDumpWithFullMemory）にも対応しています。ファイル全体をメモリに読み込まず、File.slice() で必要なヘッダーとメタデータ部分のみを段階的に読み込みます。メモリストリーム（プロセスメモリの実データ）はスキップするため、ブラウザのメモリ制限に影響されません。

| ダンプ形式 | 読み込み方式 | 実際の読み込み量 |
|-----------|-------------|----------------|
| PAGEDUMP / PAGEDU64 | 先頭 8KB のみ読み込み | 8 KB |
| MDMP（数MB程度） | ファイル全体を読み込み | ファイルサイズと同等 |
| MDMP（10GB等の大容量） | ヘッダー → ディレクトリ → メタデータのみ | 通常 数MB 以下 |

## プライバシー

- すべての処理はブラウザ内で完了します
- ファイルデータがネットワークに送信されることは一切ありません
- サーバーサイドの処理は存在しません（GitHub Pages は静的ファイルの配信のみ）
- オフライン環境でも動作します（HTMLファイルをローカルに保存して使用可能）

## 技術仕様

### 使用している Web API

| API | 用途 |
|-----|------|
| File / File.slice() | ファイルのメタ情報取得と部分読み込み（大容量ファイル対応の要） |
| FileReader.readAsArrayBuffer() | 選択されたファイルをバイナリ（ArrayBuffer）として読み込み |
| DataView | ArrayBuffer 上のバイナリデータをリトルエンディアンで解析 |
| BigInt / getBigUint64() | 64ビットアドレス（メモリアドレス等）の精度を保った読み取り |
| TextDecoder('utf-16le') | MINIDUMP_STRING 構造体のモジュール名を UTF-16LE からデコード |
| Blob / URL.createObjectURL() | JSON エクスポート時のファイルダウンロードを生成 |
| Drag and Drop API (dragover/drop) | ファイルのドラッグ＆ドロップ受け付け |

### 外部依存

なし。CDN、npm パッケージ、外部ライブラリへの依存は一切ありません。単一の HTML ファイルに CSS と JavaScript がすべて埋め込まれています。

### 対応する例外コード（主要なもの）

| コード | 名前 | 説明 |
|--------|------|------|
| 0xC0000005 | ACCESS_VIOLATION | 無効なメモリアクセス（読み書き違反） |
| 0xC00000FD | STACK_OVERFLOW | スタック領域の使い切り |
| 0xC0000409 | STACK_BUFFER_OVERRUN | スタックバッファのオーバーラン検出 |
| 0xE06D7363 | CPP_EXCEPTION | C++ の throw による例外 |
| 0xC0000094 | INTEGER_DIVIDE_BY_ZERO | 整数のゼロ除算 |
| 0xC000001D | ILLEGAL_INSTRUCTION | 不正な CPU 命令の実行 |
| 0x80000003 | BREAKPOINT | デバッグブレークポイント |

### 対応する BugCheck コード（主要なもの）

| コード | 名前 | 説明 |
|--------|------|------|
| 0x0000000A | IRQL_NOT_LESS_OR_EQUAL | 不正な割り込みレベルでのメモリアクセス |
| 0x00000050 | PAGE_FAULT_IN_NONPAGED_AREA | 非ページ領域でのページフォルト |
| 0x0000003B | SYSTEM_SERVICE_EXCEPTION | システムサービス内での例外 |
| 0x00000116 | VIDEO_TDR_TIMEOUT_DETECTED | GPU が応答停止（タイムアウト） |
| 0x00000139 | KERNEL_SECURITY_CHECK_FAILURE | カーネルのセキュリティチェック失敗 |
| 0x000000EF | CRITICAL_PROCESS_DIED | 重要なシステムプロセスの異常終了 |
| 0x000000D1 | DRIVER_IRQL_NOT_LESS_OR_EQUAL | ドライバーの不正なメモリアクセス |
| 0x00000124 | WHEA_UNCORRECTABLE_ERROR | ハードウェアの訂正不能エラー |

## 動作環境

| ブラウザ | 最低バージョン |
|---------|--------------|
| Google Chrome | 67 以上 |
| Microsoft Edge | 79 以上 |
| Firefox | 68 以上 |
| Safari | 15 以上 |

BigInt および DataView.getBigUint64() をサポートするモダンブラウザが必要です。Internet Explorer は非対応です。
