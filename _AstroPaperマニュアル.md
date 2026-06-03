# AstroPaper 使い方マニュアル（日本語・v6対応）

> 対象バージョン: AstroPaper v6（Astro v6 / Tailwind v4 ベース）
> 用途: hayatom.com の学習ログメディア構築
> 最終更新: 2026-06-03

このマニュアルは、AstroPaper 公式ドキュメント（README・「記事の追加」・「設定」）を読み込み、日本語で実践的に再構成したものです。手元のプロジェクトに置いておき、迷ったときに参照できる「逆引き」を意識して作っています。

---

## 1. AstroPaper とは

AstroPaper は、ミニマル・高速・アクセシブル・SEOに強いことを売りにした、Astro製のブログテーマです。ブログに必要な機能が最初からひと通り揃っているため、「記事を書いて蓄積する」ことにすぐ集中できます。

主な機能は次のとおりです。

- 型安全な Markdown（frontmatter をTypeScriptで検証）
- 非常に高速なパフォーマンス（Lighthouseで高スコア）
- キーボード操作・スクリーンリーダー対応のアクセシビリティ
- スマホ〜デスクトップまでのレスポンシブ対応
- ライト/ダークモード切り替え
- 静的サイト内検索（Pagefind）
- 下書き（draft）機能とページネーション
- サイトマップ・RSSフィードの自動生成
- MDXサポート（Markdown内でコンポーネントが使える）
- 折りたたみ可能な目次（TOC）
- 記事ごとのOG画像の動的生成
- 多言語化（i18n）対応の素地

### 技術スタック

| 役割 | 採用技術 |
|---|---|
| フレームワーク | Astro |
| 型チェック | TypeScript |
| スタイリング | Tailwind CSS |
| サイト内検索 | Pagefind |
| アイコン | Tabler Icons |
| コード整形 | Prettier |
| Lint | ESLint |
| OG画像の動的生成 | Satori + Sharp + Astro Fonts |

---

## 2. プロジェクト構造

AstroPaper をインストールすると、おおむね次のような構成になります。最初に「どこに何があるか」を把握しておくと、後の作業が一気に楽になります。

```
/
├── public/                  # そのまま配信される静的ファイル
│   ├── pagefind/            # ビルド時に自動生成される検索インデックス
│   ├── favicon.svg
│   └── default-og.jpg       # デフォルトのOG画像
├── src/
│   ├── assets/              # 最適化対象の画像・アイコン
│   │   ├── icons/
│   │   └── images/
│   ├── components/          # 再利用コンポーネント（Header.astro など）
│   ├── content/
│   │   ├── pages/           # 固定ページ（about.md など）
│   │   └── posts/           # ★ブログ記事はここに書く
│   ├── i18n/                # 多言語化用
│   ├── layouts/             # ページの土台（Layout.astro など）
│   ├── pages/               # ルーティング
│   ├── scripts/
│   ├── styles/              # global.css / theme.css
│   ├── types/
│   ├── utils/
│   ├── config.ts
│   └── content.config.ts    # コンテンツのスキーマ定義
├── astro-paper.config.ts    # ★あなたが触る設定ファイル
└── astro.config.ts          # Astro本体の設定（フォント・Markdown処理など）
```

覚えておくべき最重要ポイントは2つだけです。

- **記事を書く場所** = `src/content/posts/`
- **サイト全体の設定をいじる場所** = `astro-paper.config.ts`

---

## 3. よく使うコマンド一覧

すべてプロジェクトのルートで実行します。公式は pnpm 例ですが、ここでは npm で記載します（pnpm/yarn/bun でも同様）。

| コマンド | 役割 |
|---|---|
| `npm install` | 依存パッケージのインストール |
| `npm run dev` | ローカル開発サーバーを起動（`localhost:4321`） |
| `npm run build` | 型チェック → ビルド → Pagefind検索インデックス生成 → `public/pagefind/` へコピー |
| `npm run preview` | デプロイ前にビルド結果をローカルで確認 |
| `npm run sync` | Astroモジュールの型定義を再生成 |
| `npm run astro ...` | `astro add` や `astro check` などのCLIを実行 |

> ⚠️ Windowsの注意: `build` はPagefindインデックスを `cp` でコピーするため、Windowsでは `npm run build:win` を使う必要があります（Mac/Linuxは通常の `build` でOK）。

日常の流れはシンプルです。**`npm run dev` で書きながら確認 → 完成したら commit & push → 自動でビルド＆公開**。

---

## 4. サイト全体の設定（`astro-paper.config.ts`）

v6では設定が `astro-paper.config.ts` に集約され、`defineAstroPaperConfig()` で囲むことでエディタの補completion（IntelliSense）が効くようになっています。中身は大きく `site` / `posts` / `features` / `socials` / `shareLinks` に分かれます。

```ts
import { defineAstroPaperConfig } from "./src/types/config";

export default defineAstroPaperConfig({
  site: {
    url: "https://hayatom.com/",          // ← 本番URLに必ず変更
    title: "はやとの開発ログ",              // ← サイト名
    description: "プログラミング学習の過程を記録するメディア",
    author: "Hayato",
    profile: "https://hayatom.com",
    ogImage: "default-og.jpg",
    lang: "ja",                            // ← 日本語サイトなら "ja"
    timezone: "Asia/Tokyo",                // ← 日本時間に
    dir: "ltr",
  },
  posts: {
    perPage: 4,
    perIndex: 4,
    scheduledPostMargin: 15 * 60 * 1000,
  },
  features: {
    lightAndDarkMode: true,
    dynamicOgImage: true,
    showArchives: true,
    showBackButton: true,
    editPost: { enabled: true, url: "https://github.com/あなた/hayatom-blog/edit/main/" },
    search: "pagefind",
  },
  socials: [
    { name: "github", url: "https://github.com/あなた" },
    { name: "x", url: "https://x.com/あなた" },
    { name: "mail", url: "mailto:you@example.com" },
  ],
  shareLinks: [
    { name: "x", url: "https://x.com/intent/post?url=" },
    { name: "mail", url: "mailto:?subject=See%20this%20post&body=" },
  ],
});
```

### `site`（サイト基本情報）

| 項目 | 意味 |
|---|---|
| `url` | デプロイ後のサイトURL。canonical・OG画像・RSS・サイトマップに使われる。**本番では必ず正しい値に**。 |
| `title` | サイト名 |
| `description` | サイトの説明（SEO・SNS共有に使用） |
| `author` | 既定の記事著者名 |
| `profile` | 自分のポートフォリオ等のURL（構造化データ用。無ければ `undefined`） |
| `ogImage` | `/public` 内のデフォルトOG画像ファイル名 |
| `lang` | `<html lang="...">` の値。日本語サイトは `"ja"` |
| `timezone` | 記事日時のタイムゾーン（IANA形式）。`"Asia/Tokyo"` でローカルと本番の時刻表示を統一 |
| `dir` | 文字方向。`"ltr"` / `"rtl"` / `"auto"` |
| `googleVerification` | Google Search Console の確認用メタタグ値（任意） |

### `posts`（記事一覧の挙動）

| 項目 | 意味 |
|---|---|
| `perPage` | 一覧ページの1ページあたり記事数（既定4） |
| `perIndex` | トップページの「最近の記事」表示数（既定4） |
| `scheduledPostMargin` | この時間内（ミリ秒）の未来日時の記事は公開扱いにする（既定15分） |

### `features`（機能のオン/オフ）

| 項目 | 意味 |
|---|---|
| `lightAndDarkMode` | ライト/ダーク切り替えの有効化（既定true） |
| `dynamicOgImage` | frontmatterにOG画像が無いとき記事ごとに自動生成（既定true） |
| `showArchives` | `/archives` ページとヘッダーリンクの表示（既定true） |
| `showBackButton` | 記事ページの「戻る」ボタン表示（既定true） |
| `editPost` | 記事下の「Edit page」リンク。`enabled` とリポジトリの編集URLを指定 |
| `search` | 検索プロバイダ。既定 `"pagefind"`。`false` で検索を無効化 |

---

## 5. 見た目のカスタマイズ

「AstroPaperを少しずつ自分のものにする」ための入口です。最初は深入りせず、必要になったら触る程度でOK。

### レイアウト幅

サイト全体の `max-width` は既定で `768px`（`max-w-3xl`）。変更したい場合は `src/styles/global.css` の `max-w-app` ユーティリティを編集します。

```css
@utility max-w-app {
  @apply max-w-4xl xl:max-w-5xl;  /* 例: 少し広げる */
}
```

### ロゴ・タイトル

3通りの方法があります。

1. **テキストのみ（最も簡単）**: `astro-paper.config.ts` の `site.title` を変えるだけ。
2. **SVGロゴ**: `src/assets/` にSVGを置き、`src/components/Header.astro` でimportして `{config.site.title}` と差し替える。`dark:invert` でダークモード時に色反転もできる。
3. **画像ロゴ（SVG以外）**: Astroの `Image` コンポーネントを使って同様に差し替える。

### SNSリンク / シェアリンク

`socials`（ヘッダー等のSNSアイコン）と `shareLinks`（記事の共有ボタン）は、それぞれ配列で管理します。各エントリの `name` は `src/assets/icons/socials/` 内のSVGファイル名（拡張子なし）と一致させる必要があります。デフォルトに無いサービスを足したいときは、SVGアイコンをそのフォルダに追加してから配列にエントリを足します。

### フォント

既定フォントは Google Sans Code（AstroのフォントAPIで最適化込みで読み込み済み）。別フォントにしたい場合は次の3か所を更新します。

1. `astro.config.ts` の `fonts` 設定にフォントを追加
2. `src/layouts/Layout.astro` の `<Font>` コンポーネントを更新
3. `src/styles/theme.css` の `--font-app` 変数を新しいフォントに向ける（これ1か所でサイト全体に反映）

---

## 6. 記事の書き方（最重要セクション）

ここがメディア運用の中心です。基本は「`src/content/posts/` に Markdown ファイルを足すだけ」です。

### 6-1. ファイルの置き場所とURLの関係

`src/content/posts/` 配下に `.md`（または `.mdx`）を作成します。サブフォルダを作るとその名前がURLの一部になります。

```
src/content/posts/very-first-post.md       → /posts/very-first-post
src/content/posts/2025/example-post.md      → /posts/2025/example-post
src/content/posts/_2026/another-post.md     → /posts/another-post
src/content/posts/docs/_legacy/how-to.md    → /posts/docs/how-to
```

**ポイント:** フォルダ名やファイル名の先頭に `_`（アンダースコア）を付けると、ルーティングから除外されます。下書きや内部用メモ、共有素材の置き場として使えます。

### 6-2. frontmatter（記事冒頭のメタ情報）

記事ファイルの先頭にYAML形式で書くメタ情報です。

| プロパティ | 説明 | 必須/既定 |
|---|---|---|
| `title` | 記事タイトル（h1になる） | **必須** |
| `description` | 記事の説明（抜粋・SEO・SNS共有に使用） | **必須** |
| `pubDatetime` | 公開日時（ISO 8601形式） | **必須** |
| `modDatetime` | 更新日時（ISO 8601）。修正したときだけ付ける | 任意 |
| `author` | 著者名 | 既定 = `site.author` |
| `featured` | トップの注目記事欄に出すか | 既定 = false |
| `draft` | 下書き（未公開）扱いにする | 既定 = false |
| `tags` | タグ（YAML配列） | 既定 = `others` |
| `ogImage` | 記事のOG画像（リモートURL or 相対パス） | 既定 = 自動生成 or `site.ogImage` |
| `canonicalURL` | 既出記事がある場合の正規URL（絶対） | 既定 = 自動 |
| `hideEditPost` | この記事だけ編集ボタンを隠す | 既定 = false |
| `timezone` | この記事だけのタイムゾーン（IANA） | 既定 = `site.timezone` |

必須は **`title` / `description` / `pubDatetime` の3つだけ**。タイトルと説明はSEOに直結するので、毎回しっかり書くのが推奨です。

> 💡 ISO 8601の日時は、ブラウザのコンソールで `new Date().toISOString()` を実行すると簡単に取得できます。

### サンプル frontmatter

```yaml
---
title: 初めての記事
author: Hayato
pubDatetime: 2026-06-03T05:17:19Z
featured: true
draft: false
tags:
  - 学習ログ
  - astro
ogImage: ../../assets/images/example.png
description: 学習過程を記録する最初の投稿です。
---
```

### VS Code スニペット（任意）

AstroPaperには記事作成を高速化するスニペットが同梱されています（`.vscode/astro-paper.code-snippets`）。VS Code / Cursor でワークスペースを開けば自動で使えます。

- `frontmatter` … 推奨のfrontmatterブロックを挿入
- `template` … 目次付きの基本テンプレートを挿入

### 6-3. 目次（TOC）の出し方

記事はデフォルトでは目次なし。出したい位置に `## Table of contents` という見出し（h2）を書くと、そこに目次が生成されます。

```markdown
---
# frontmatter
---

導入文をここに書く。

## Table of contents

<!-- 以降が本文 -->
```

### 6-4. 見出しのルール

記事の大見出し（h1）は frontmatter の `title` が使われます。したがって**本文中の見出しは h2〜h6（`##`〜`######`）を使う**のが推奨です（アクセシビリティ・SEO上の理由）。

### 6-5. シンタックスハイライト

コードブロックは Shiki でハイライトされ、`@shikijs/transformers` による差分表示や行ハイライトなどの拡張も使えます。設定は `astro.config.ts` の `markdown.shikiConfig` にあります。

### 6-6. 画像の入れ方

2つの方法があり、**`src/assets/` 推奨**です。

**① `src/assets/` 配下（推奨・自動最適化される）**
Astroが自動で画像を最適化します。相対パスかエイリアス（`@/assets/`）で参照します。

```markdown
![説明](@/assets/images/example.jpg)
<!-- または -->
![説明](../../assets/images/example.jpg)
```
※ 通常のMarkdownでは `<img>` タグやImageコンポーネントは効きません。スタイルを当てたい最適化画像はMDXを使ってください。

**② `public/` 配下（最適化されない・絶対パス）**
`public/` の画像はAstroに加工されないので、最適化は自分で行います。絶対パスで参照します。

```markdown
![説明](/assets/images/example.jpg)
```

> 特に `public/` の画像は、貼る前に [TinyPNG](https://tinypng.com/) などで圧縮するとサイト全体のパフォーマンスが上がります。

### 6-7. OG画像（SNS共有用画像）

記事にOG画像を指定しなければデフォルト画像が使われます。推奨サイズは **1200 × 640 px**。なお v1.4.0 以降は、指定が無い場合に記事ごとのOG画像を自動生成してくれます（`features.dynamicOgImage`）。

### 6-8. 下書きと予約投稿

- **下書き**: frontmatter に `draft: true` を付けると未公開扱い。`_` プレフィックスのフォルダに入れて除外する方法もある。
- **予約投稿**: `pubDatetime` を未来日時にすると、その時刻まで非公開。`posts.scheduledPostMargin`（既定15分）以内なら公開扱いになる。

---

## 7. 公開までの流れ（おさらい）

1. `npm run dev` で書きながら `localhost:4321` で確認
2. `npm run build` → `npm run preview` で本番同等の確認
3. `git add . && git commit && git push`
4. Cloudflare Workers（Git連携）が自動でビルド＆デプロイ
5. `astro-paper.config.ts` の `site.url` は `https://hayatom.com/` にしておく

---

## 8. 学習ログメディアとしての運用Tips

- **記事をフォルダで年・テーマ別に整理**: `src/content/posts/2026/`, `src/content/posts/react/` のようにすると、URLも `/posts/2026/...` と整理され、後から探しやすい。
- **書きかけは `draft: true` か `_フォルダ`**: 学習途中のメモを気軽に貯めておき、整ったら公開に切り替える運用が相性◎。
- **`tags` を最初に決めておく**: 「学習ログ」「つまずき」「環境構築」など、自分の過程を分類するタグを最初に数個決めておくと、後でテーマ別に振り返れる。
- **`modDatetime` で成長を可視化**: 過去記事を理解が深まった時点で更新し `modDatetime` を付けると、「学び直した過程」自体がコンテンツになる。
- **`title` と `description` は毎回ていねいに**: 検索流入の入口。未来の自分と読者の両方に効く。

---

## 参照元（公式ドキュメント）

- README: https://github.com/satnaing/astro-paper
- 記事の追加: https://astro-paper.pages.dev/posts/adding-new-posts-in-astropaper-theme/
- 設定: https://astro-paper.pages.dev/posts/how-to-configure-astropaper-theme/
- 配色のカスタマイズ: https://astro-paper.pages.dev/posts/customizing-astropaper-theme-color-schemes/
- デモサイト（全記事＝マニュアル）: https://astro-paper.pages.dev/posts/