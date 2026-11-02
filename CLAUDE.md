# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

GPT-5 / GPT-5.2 への単一プロンプトで生成されたデモアプリのギャラリーです。デモ本体はモデルの出力であり、手作業で編集することは想定されていません。継続的に保守されているコードは `front-end/` の Next.js ギャラリーのみです。ホスト版: https://gpt5-coding-examples.vercel.app/ 。上流リポジトリはコントリビューションを受け付けていません。

ユーザーは日本語を使用します。応答やドキュメントは日本語で記述してください。

## コマンド

すべて `front-end/` で実行します。

```bash
npm install
npm run dev     # ../apps を public/ にコピーしてから next dev --turbopack(localhost:3000)
npm run build   # prebuild で apps をコピーし、静的エクスポート(output: "export")
npm run lint    # next lint
```

テストスイートはありません。

## アーキテクチャ

トップレベルの 3 つの要素が、共通の**スラッグ**(YAML ファイル名から拡張子を除いたもの)で結び付いています。

- `examples/<slug>.yaml`: メタデータ。`title`(または旧形式の `name`)と `prompt` が必須で、どちらかが欠けているファイルは警告なしにスキップされます。任意フィールドは `tags`(文字列リスト)と `camera` / `microphone`(真偽値。`components/app-modal.tsx` で iframe の `allow` 権限になります)。YAML 内の `id` と `screenshot_url` は無視されます。スラッグはファイル名から決まり、ポスター画像は常に `https://cdn.openai.com/devhub/gpt5prompts/<slug>.png` です。
- `apps/<slug>/index.html`: 生成されたアプリ。ほとんどは単一 HTML ファイルですが、一部(例: `asteroid-game`、`espresso`)は `_next/` アセットを含むビルド済みの Next.js 静的エクスポートです。GPT-5.2 版はスラッグに `-5.2` サフィックスが付きます。
- `front-end/`: ギャラリー本体(Next 15 App Router、React 19、Tailwind v4、`components/ui` に shadcn/ui コンポーネント)。

連携の流れ:

1. `front-end/scripts/copy-docs.mjs`(`dev` と `prebuild` で実行)が、`apps/<slug>/` の各ディレクトリを削除・再コピーして `front-end/public/<slug>/` に配置します。`front-end/public/` はコミットされており、`apps/` に存在しない古いスラッグも残っています。`apps/` を正としてください。
2. `lib/code-examples.ts` の `loadApps()` はビルド時に実行されます(`app/page.tsx` は `force-static`)。**リポジトリルート**(`process.cwd()/..`)を再帰的に走査して `.yaml`/`.yml` ファイルを探します(`node_modules`、`.next`、`misc` などは除外)。そのため、`title` と `prompt` を持つ YAML ファイルがリポジトリ内のどこかに紛れ込むと、サンプルとして表示されてしまいます。
3. `next.config.ts` は、`index.html` を含む `public/` 配下の各サブディレクトリについて `/<slug>` を `/<slug>/index.html` にリライトします。リライト一覧は設定読み込み時に計算されるため、アプリを追加したら開発サーバーを再起動してください。
4. モーダルはアプリを iframe(`/<slug>`)で埋め込み、プロンプトを表示し、プロンプトを入力済みの ChatGPT へのリンクを提供します。

つまり、サンプルを追加するには、対応する `examples/<slug>.yaml` と `apps/<slug>/index.html` を追加します。スクリーンショットは OpenAI の CDN から取得されます。

## 守ってほしいこと
- ソースコード内のコメントは日本語で記述すること
- TypeScriptにおいて、必要以上にクラスを利用しないこと
- 既存のディレクトリ構成・命名・状態管理の流儀にそろえること
