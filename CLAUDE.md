# CLAUDE.md - AIアシスタント向けガイド

このドキュメントは、Parquet Viewerコードベースで作業するAIアシスタント向けの包括的なガイドです。

## プロジェクト概要

**Parquet Viewer**は、Apache Parquetファイルをローカルで閲覧・分析するためのブラウザベースツールです。バックエンドサーバー不要で、すべてのデータ処理はクライアントサイドで行われます。

- **言語:** 日本語
- **技術スタック:** Vanilla JavaScript (ES6+モジュール)、HTML、CSS
- **コアライブラリ:** hyparquet (v1.14.0) - Parquetファイル解析用
- **ビルド不要:** 純粋なブラウザベースアプリケーション

## 開発クイックスタート

1. VSCodeでこのフォルダを開く
2. 拡張機能「Live Server」(`ritwickdey.LiveServer`) をインストール
3. `index.html` を右クリック → 「Open with Live Server」
4. `.parquet` または `.parq` ファイルをドラッグ＆ドロップしてテスト

## ディレクトリ構成

```
parquet-viewer-local/
├── index.html              # メインのオフライン版（主要エントリーポイント）
├── cdn/
│   └── parquet-viewer.html # オンラインCDN版（SQL対応、要インターネット）
├── lib/                    # hyparquetライブラリモジュール（25個のJSファイル）
│   ├── index.js            # メインエクスポートファイル
│   ├── types.d.ts          # TypeScript型定義
│   ├── hyparquet.min.js    # jsDelivrからの圧縮版ライブラリ
│   ├── read.js             # Parquet読み込みのコアロジック
│   ├── metadata.js         # ファイルメタデータ解析
│   ├── schema.js           # スキーマツリー構築
│   ├── column.js           # カラム処理
│   ├── encoding.js         # RLE/ビットパックエンコーディング
│   ├── convert.js          # 型変換
│   ├── datapage.js         # データページデコード
│   ├── snappy.js           # Snappy圧縮
│   ├── thrift.js           # Thriftデシリアライゼーション
│   └── ...                 # その他のエンコーディング/処理モジュール
├── test.parquet            # テスト用サンプルファイル（247KB）
├── README.md               # ユーザードキュメント
└── CLAUDE.md               # このファイル
```

## アプリケーションバージョン

### オフライン版 (`index.html`)
- **用途:** インターネット不要のローカルファイル閲覧
- **機能:** ドラッグ＆ドロップ、ページネーション、ソート、検索、統計情報、CSV出力
- **依存:** ローカルの `lib/` フォルダのみ

### オンラインCDN版 (`cdn/parquet-viewer.html`)
- **用途:** DuckDB WASMによるSQLクエリ対応
- **要件:** CDNリソース取得のためインターネット接続必須
- **追加機能:** DuckDB WASM (v1.28.0) によるSQLクエリエディタ

## 主要機能

| 機能 | 説明 |
|------|------|
| ファイルアップロード | ドラッグ＆ドロップまたはクリックして選択 |
| ページネーション | 50、100、500、1000行/ページ |
| カラムソート | ヘッダークリックでソート（昇順/降順） |
| テキスト検索 | 全カラム横断のライブ検索 |
| 統計情報 | 行数/列数、データ型、NULL率、最小/最大値、ユニーク数 |
| CSV出力 | UTF-8 BOMエンコード |

## コーディング規約

### JavaScriptパターン

1. **ES6モジュール:** すべてのファイルで `import`/`export` 構文を使用
   ```javascript
   import { parquetRead } from './lib/hyparquet.min.js'
   export { functionName }
   ```

2. **JSDocコメント:** 型ドキュメントにJSDocを使用
   ```javascript
   /**
    * @param {ArrayBuffer} buffer - ファイルバッファ
    * @returns {Promise<Object>} 解析済みデータ
    */
   ```

3. **Async/Await:** ファイル操作にはasync/awaitを優先
   ```javascript
   async function readFile(file) {
     const buffer = await file.arrayBuffer()
     // ...
   }
   ```

4. **コールバックパターン:** ストリーミングデータにはコールバックを使用
   ```javascript
   parquetRead({ file, onComplete: callback })
   ```

### HTML/CSSパターン

1. **セマンティックHTML:** 適切なタグを使用（`<table>`, `<thead>`, `<tbody>`）
2. **CSS変数:** プライマリカラー `#2563eb`、危険色 `#dc2626`
3. **Flexboxレイアウト:** モダンなレスポンシブデザイン
4. **BEM風クラス名:** `.drop-zone`, `.info-bar`, `.table-container`

### セキュリティ考慮事項

- **XSS防止:** ユーザーデータには必ず `escapeHtml()` を使用
  ```javascript
  function escapeHtml(str) {
    return String(str)
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#039;')
  }
  ```

## 実装の重要事項

### データ処理フロー

1. **ファイル読み込み:** ユーザーがファイルをドロップ/選択 → `FileReader` → `ArrayBuffer`
2. **解析:** hyparquetの `parquetRead()` → 構造化データ
3. **表示:** HTMLテーブルにページネーション付きでレンダリング
4. **フィルタリング:** クライアントサイドで全カラム検索
5. **エクスポート:** UTF-8 BOMエンコードでCSVに変換

### 対応Parquet機能

- **圧縮:** SNAPPY, GZIP, LZO, BROTLI, LZ4, ZSTD, LZ4_RAW
- **エンコーディング:** PLAIN, RLE, BIT_PACKED, DELTA_BINARY_PACKED, DELTA_LENGTH_BYTE_ARRAY, DELTA_BYTE_ARRAY, BYTE_STREAM_SPLIT
- **データ型:** ネスト構造を含むすべての標準Parquet型
- **地理空間:** GeoParquetおよびWKBジオメトリ対応

### 主要ライブラリファイル

| ファイル | 目的 |
|---------|------|
| `lib/read.js` | Parquetファイル読み込みのメインエントリーポイント |
| `lib/metadata.js` | ファイルメタデータとフッターの解析 |
| `lib/schema.js` | メタデータからスキーマツリーを構築 |
| `lib/column.js` | 個別カラムの処理 |
| `lib/datapage.js` | データページのデコード |
| `lib/encoding.js` | RLEおよびビットパックエンコーディング処理 |
| `lib/thrift.js` | Thriftプロトコルデシリアライゼーション |

## 変更の加え方

### ビューアUIの変更

`index.html` を直接編集。主要セクション:
- **1-150行:** HTML構造
- **150-300行:** CSSスタイル（`<style>` タグ内）
- **300行以降:** JavaScriptロジック（`<script type="module">` 内）

### 新機能の追加

1. クライアントサイドのみを維持（サーバー依存なし）
2. UIテキストは日本語を維持
3. 既存のコードパターンと規約に従う
4. `test.parquet` サンプルファイルでテスト
5. 該当する場合、オフライン版（`index.html`）とオンライン版（`cdn/parquet-viewer.html`）の両方が動作することを確認

### ライブラリの変更

`lib/` フォルダにはhyparquetライブラリが含まれています。変更は稀であるべき:
- ライブラリはjsDelivr CDNから取得（v1.14.0）
- 型定義は `types.d.ts`
- 修正はアップストリームへの貢献を検討

## テスト

自動テストは存在しません。手動テストワークフロー:
1. Live Serverで開く
2. `test.parquet` をドラッグ＆ドロップで読み込み
3. テーブルが正しく表示されることを確認
4. ページネーション、ソート、検索をテスト
5. CSV出力をテスト
6. 統計情報表示を確認

## よくある作業

### 新しいUI機能を追加
1. 適切なセクションにHTML要素を追加
2. `<style>` タグにCSSスタイリングを追加
3. `<script>` タグにJavaScriptハンドラーを追加

### hyparquetライブラリの更新
1. jsDelivrから新バージョンをダウンロード
2. `lib/hyparquet.min.js` を置き換え
3. 必要に応じて個別モジュールファイルを更新
4. サンプルファイルで徹底的にテスト

### 新しい言語サポートを追加
1. すべてのUI文字列は日本語でハードコードされている
2. 必要に応じてローカライゼーションオブジェクトを作成
3. ハードコードされた文字列を参照に置き換え

## コード内の既知のTODO

コードコメントより:
- `thrift.js`: MAP, SET, UUID型のサポート
- `read.js`: オプションの入力バリデーション
- `delta.js`: ビットパック読み取りの最適化
- `rowgroup.js`: フラット化操作の最適化
- `query.js`: 並列フェッチ、null処理
- `column.js`: 空チャンク出力、配列アロケーション最適化

## Gitワークフロー

- **メインブランチ:** `main`
- **機能ブランチ:** `claude/*` プレフィックス
- コミットメッセージは日本語または英語
- CI/CDパイプラインは未設定

## 依存関係

このプロジェクトは依存関係が最小限:
- **実行時:** ES6モジュール対応のモダンブラウザ
- **開発時:** VSCode + Live Server拡張機能
- **npm/package.json なし:** 純粋なブラウザベースアプリケーション

## パフォーマンス考慮事項

- 大きなファイル（100MB以上）はブラウザメモリ問題を引き起こす可能性
- ページネーションによりレンダリングパフォーマンス問題を防止
- すべての処理はブラウザのメインスレッドでシングルスレッド
- 大きなファイルの最適化には将来的にWeb Workersを検討

---

*最終更新: 2026年1月*
