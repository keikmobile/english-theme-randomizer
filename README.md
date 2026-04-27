# English Theme Randomizer

英会話コミュニティ向け。ボタン1つでランダムにトークテーマを表示する静的Webアプリ。

**Live:** `https://{username}.github.io/english-theme-randomizer/`

---

## ローカル確認

```bash
npx serve docs
# → http://localhost:3000
```

---

## テーマの追加方法

`docs/themes.json` を直接編集する。

```json
{
  "id": 11,
  "theme": "Your new theme sentence here",
  "category": "lifestyle",
  "difficulty": "intermediate",
  "added": "2026-04"
}
```

**フィールド一覧**

| フィールド | 値 |
|---|---|
| `id` | 連番（最後のIDに +1） |
| `theme` | テーマ本文（英語） |
| `category` | `travel` / `lifestyle` / `culture` / `work` / `personal` / `society` |
| `difficulty` | `beginner` / `intermediate` / `advanced` |
| `added` | 追加年月 `YYYY-MM` |

> `category` と `difficulty` は迷ったら近いものを選べばOK。厳密な分類は不要。

---

## デプロイ手順

1. GitHub リポジトリを **public** で作成
2. Settings → Pages → Source: `GitHub Actions`
3. 以下の手順でテーマを追加・更新する

```bash
git add docs/themes.json
git commit -m "add theme: <テーマの概要>"
git push
```

push 後、数十秒で GitHub Pages に自動反映される。

---

## 運用フロー

```
① SNSグループ等でテーマを募集
      ↓
② 管理者が themes.json に追記（id をインクリメント）
      ↓
③ git push
      ↓
④ GitHub Actions が自動デプロイ
      ↓
⑤ 次のミーティングで使用
```
