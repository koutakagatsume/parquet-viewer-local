# Parquet Viewer

ローカルでParquetファイルを閲覧するツール


## 必要なもの

- VSCode
- 拡張機能「Live Server」（ritwickdey.LiveServer）

## 使い方

1. VSCodeでこのフォルダを開く
2. `index.html` を右クリック → 「Open with Live Server」
3. ブラウザでParquetファイルをドラッグ＆ドロップ

## 機能

- Parquetファイルの閲覧
- カラムソート
- テキスト検索
- 統計情報表示
- CSV出力

## ディレクトリ構成

```
├── cdn/
│   └── parquet-viewer.html  # ※ 使用時注意：オンライン版（ネット接続必要）
├── lib/                      # hyparquetライブラリ
├── index.html                # オフライン版
├── README.md
└── test.parquet              # テスト用ファイル
```

*公開ok*