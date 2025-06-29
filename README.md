# Zenn CLI

* [📘 How to use](https://zenn.dev/zenn/articles/zenn-cli-guide)

## 命名規則

### 記事を作成する時の命名規則

| ルール         | Why?                                   | コツ                                         |
|--------------|----------------------------------------|---------------------------------------------|
| 日付先頭      | 並び替え不要、変更もまず起きないので URL 安定 | zenn publish 後に日付を変えるのはNG           |
| カテゴリ 1語   | 大量記事でも ざっくり絞り込み可                | front-matter の tags: と整合を取ると◎         |
| slug を短く    | URL が短く覚えやすい                        | 5〜7語以内／動詞は省略気味                    |
| kebab-case   | Zenn の推奨・SEOにも強い                    | アルファベット・数字・ハイフンのみ             |
| ドラフト接頭辞 | 公開前の一時置き場識別                        | draft-YYYY...-slug.md → 公開時にリネーム      |


## 運用

### 新しい記事の作成手順

1. 記事を作成

```bash
# YYYY-MM-DD-[カテゴリ]-slug.md
zenn new:article --slug $(date +%F)-xrpl-slug
```
