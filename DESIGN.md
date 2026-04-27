# English Theme Randomizer — 設計書

## プロジェクト概要

英会話コミュニティで使うテーマをランダム表示するWebアプリ。
SNSグループ等でテーマを募集し、`themes.json` を手動編集してデプロイする静的サイト構成。

---

## 設計思想

**テーマ設定者の認知負荷をゼロに近づける。**

テーマを「誰が決めるか」問題は、コミュニティ運営の小さなボトルネック。
このアプリはその決定をランダムに委譲することで、
進行役が「今日は何にしよう」と悩む時間と心理的コストを取り除く。

- UIはボタン1つで完結すること
- フィルターは補助機能（なくても使えること）
- 「ランダムが決めた」という中立性を体験として感じさせること

この思想から、以下の優先順位を設ける：

1. **ランダム表示** — コア機能。これだけで成立する
2. **視認性** — カードの文字は大きく、瞬時に読める
3. **フィルター** — 補助。デフォルトは常に「全件対象」

---

## ディレクトリ構成

```
english-theme-randomizer/
├── index.html          # メインUI（1ファイル完結）
├── themes.json         # テーマデータ（唯一の編集対象）
├── README.md           # 運用手順
└── .github/
    └── workflows/
        └── deploy.yml  # GitHub Pages 自動デプロイ
```

---

## データ設計

### themes.json

```json
[
  {
    "id": 1,
    "theme": "A trip that changed how you see the world",
    "category": "travel",
    "difficulty": "intermediate",
    "added": "2026-04"
  },
  {
    "id": 2,
    "theme": "A habit you've tried to build (and failed)",
    "category": "lifestyle",
    "difficulty": "beginner",
    "added": "2026-04"
  }
]
```

### フィールド定義

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| `id` | number | ✅ | 連番（追加時にインクリメント） |
| `theme` | string | ✅ | テーマ本文（英語） |
| `category` | string | ✅ | `travel` / `lifestyle` / `culture` / `work` / `personal` / `society` |
| `difficulty` | string | ✅ | `beginner` / `intermediate` / `advanced` |
| `added` | string | ✅ | 追加年月 `YYYY-MM` |

> **運用メモ：** `category` と `difficulty` はフィルター用の補助情報。迷う場合は近いものを選べばよく、厳密な分類は不要。

---

## UI 設計

### 画面構成（シングルページ）

```
┌─────────────────────────────────────┐
│  🎲 English Theme Randomizer        │
│  ─────────────────────────────────  │
│                                     │
│  ┌───────────────────────────────┐  │
│  │                               │  │
│  │   A trip that changed         │  │
│  │   how you see the world       │  │
│  │                               │  │
│  │   [travel] [intermediate]     │  │
│  └───────────────────────────────┘  │
│                                     │
│       [ 🎲 Next Theme ]             │
│                                     │
│  ── Filter（補助）────────────────── │
│  Category: [All▼]  Level: [All▼]   │
│                                     │
│  24 themes available                │
└─────────────────────────────────────┘
```

### 機能要件

1. **ランダム表示** — ボタンを押すと `themes.json` からランダムに1件選択
2. **重複スキップ** — 直近5件を除外して表示（連続重複を防ぐ）
3. **テーマ総数表示** — 現在の条件での件数を表示
4. **フィルター（補助）** — カテゴリ・難易度で絞り込み。デフォルトは全件・折りたたみ表示
5. **レスポンシブ** — スマホ・タブレット・プロジェクター投影に対応

### 非機能要件

- 外部ライブラリなし（Vanilla JS）
- `themes.json` を `fetch()` で読み込む
- ローカル開発: `npx serve .` で動作確認可能
- テーマカードの文字サイズ: `1.8rem` 以上（プロジェクター投影を考慮）

---

## ロジック設計

### ランダム選択（重複スキップ付き）

```javascript
const HISTORY_SIZE = 5;
let recentIds = [];

function getRandomTheme(themes) {
  const pool = themes.filter(t => !recentIds.includes(t.id));
  const candidates = pool.length > 0 ? pool : themes; // 全部既出なら全件対象に戻す
  const picked = candidates[Math.floor(Math.random() * candidates.length)];

  recentIds.push(picked.id);
  if (recentIds.length > HISTORY_SIZE) recentIds.shift();

  return picked;
}
```

### フィルター

```javascript
function getFilteredThemes(allThemes, { category, difficulty }) {
  return allThemes.filter(t => {
    const catMatch = category === 'all' || t.category === category;
    const diffMatch = difficulty === 'all' || t.difficulty === difficulty;
    return catMatch && diffMatch;
  });
}
```

---

## デプロイ設計

### GitHub Pages（手動有効化）

1. GitHub リポジトリを public で作成
2. Settings → Pages → Source: `main` ブランチ / `/ (root)`
3. URL: `https://{username}.github.io/english-theme-randomizer/`

### 自動デプロイ（.github/workflows/deploy.yml）

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/deploy-pages@v4
```

> **Note:** 静的ファイルのみなのでビルドステップ不要。push → 数十秒で反映。

---

## 運用フロー

```
① SNSグループ等でテーマ募集
      ↓
② 管理者が themes.json に追記
   （id をインクリメント、カテゴリ・難易度を付与）
      ↓
③ git add themes.json && git commit -m "add theme: ..."
   git push
      ↓
④ GitHub Actions が自動デプロイ
      ↓
⑤ 次のミーティングで使用
```

---

## Claude Code への指示テンプレート

このファイルをプロジェクトルートに `DESIGN.md` として置いた上で、以下を最初のプロンプトとして渡す：

```
DESIGN.md の設計に従って english-theme-randomizer を実装してください。

優先順：
1. themes.json（サンプルデータ10件）
2. index.html（ランダム表示 + フィルター）
3. README.md（運用手順）
4. .github/workflows/deploy.yml

外部ライブラリは使わず、Vanilla JS で実装してください。
設計思想「テーマ設定者の認知負荷をゼロに近づける」を意識し、
ボタンを画面中央に大きく配置、テーマカードの文字は 1.8rem 以上にしてください。
フィルターはデフォルト折りたたみ表示で補助的に扱ってください。
```

---

## 将来の拡張候補（今は作らない）

- テーマに「使用済みフラグ」をつけてミーティングで管理
- QRコード表示（参加者がスマホで見られるように）
- テーマの「お気に入り」機能（localStorage）
- 多言語表示（テーマの日本語訳を併記）
