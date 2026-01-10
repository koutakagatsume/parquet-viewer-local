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

## 画面イメージ
<img width="1112" height="1864" alt="image" src="https://github.com/user-attachments/assets/dd54660a-7991-49be-bb96-68292d215157" />


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
