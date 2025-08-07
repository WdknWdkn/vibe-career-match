# キャリアマッチAI Lite

AIを活用した価値観ベースの求人マッチングWebアプリケーション

## 概要

キャリアマッチAI Liteは、MBTI性格診断と価値観診断を通じて、あなたに最適な求人を見つけるためのツールです。ChatGPTやClaudeのWeb検索機能と連携して、パーソナライズされた求人情報を取得できます。

## 特徴

- 🧠 MBTI性格診断（16タイプ）
- 💡 30問の価値観診断
- 🔍 AI検索プロンプト自動生成
- 💾 ローカルストレージによるデータ保存
- 📱 レスポンシブデザイン
- 🚀 サーバー不要（静的HTMLのみ）

## ローカル環境での動作確認

### 方法1: 直接ブラウザで開く
```bash
# ファイルをブラウザで開く
open index.html  # macOS
xdg-open index.html  # Linux
start index.html  # Windows
```

### 方法2: Python簡易サーバー（推奨）
```bash
# Python 3の場合
python3 -m http.server 8000

# Python 2の場合
python -m SimpleHTTPServer 8000

# ブラウザで http://localhost:8000 にアクセス
```

### 方法3: Node.js簡易サーバー
```bash
# http-serverをインストール
npm install -g http-server

# サーバー起動
http-server -p 8000

# ブラウザで http://localhost:8000 にアクセス
```

### 方法4: VSCode Live Server
1. VSCodeで`index.html`を開く
2. 右クリック → 「Open with Live Server」
3. 自動的にブラウザが開く

## 使い方

1. **STEP 1: 性格診断**
   - 16のMBTIタイプから自分のタイプを選択
   - わからない場合は外部サイトで診断可能

2. **STEP 2: 価値観診断**
   - 30問の質問に1〜5で回答
   - 9つのカテゴリで価値観を分析

3. **STEP 3: AI求人検索**
   - 「プロンプト生成」ボタンをクリック
   - 生成されたプロンプトをコピー
   - ChatGPTやClaudeに貼り付けて検索実行
   - 検索結果をアプリに貼り付けて管理

## デプロイ方法

### GitHub Pages
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/[username]/career-match-lite.git
git push -u origin main
# Settings > Pages > Source: main branch
```

### Vercel
```bash
npm i -g vercel
vercel
```

### Netlify
- https://app.netlify.com にアクセス
- ファイルをドラッグ&ドロップ

## 技術スタック

- HTML5
- Tailwind CSS (CDN)
- Vanilla JavaScript
- LocalStorage API

## ライセンス

MIT License