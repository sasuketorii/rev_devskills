---
name: revc-canonical-login-screen
description: REV-Cが開発する全プロダクトのログイン画面(認証UI)は、rev_license(admin-license.rev-c.org/login, license.rev-c.org/login)とrev_ad_dashboardで確立された共通デザイン(REV-Cネイビーティントのshader背景 + backdrop-blurカード + login6/login3レイアウト)を必ず流用する。新規プロダクトのログイン画面を実装する時、既存プロダクトのログイン画面デザインを変更しようとしている時、ログイン画面の共通化・使い回しについて相談された時に使う。
allowed-tools: Read, Bash, Grep, Glob
---

# REV-C Canonical Login Screen

REV-C の各プロダクトのログイン画面は、個別に意匠を起こさない。**rev_license と
rev_ad_dashboard で確立済みの共通デザイン**(WebGLシェーダー背景 + 半透明
backdrop-blurカード)を正本とし、新規プロダクトはこれを移植する。

## 起動条件

- 新規プロダクトのログイン画面(メール/パスワード認証、SSO、招待制サインイン等)を
  ゼロから実装しようとしている時。
- 既存プロダクト(rev_license / rev_ad_dashboard を含む)のログイン画面の見た目
  (レイアウト、シェーダー背景、カラー、ロゴ扱い)を変更しようとしている時。
- 「ログイン画面 使い回し」「ログイン画面 共通化」「認証UI どう作る」といった相談を
  受けた時。
- `DESIGN.md`(rev_license)の 14.4 Login Screen セクション、または本スキルの
  参照先を更新する時。

## 契約(Contract)

- ログイン画面の**視覚的な柱**(以下は削らない・変えない):
  1. REV-C ブランドネイビー(`#32435D`)にティントされた WebGL シェーダー背景
     (光の筋が流れる、`u_time` でアニメーション、`prefers-reduced-motion` で静止画化)。
  2. 背景の上に浮かぶ、`border-white/10` + `bg-black/60`(または `bg-[#111111]/60`)+
     `backdrop-blur-lg` の半透明カード。
  3. カード内はロゴ(常にダーク背景用ロゴ、テーマ非依存)→ 見出し → email/password
     フォーム → primary button の縦積みレイアウト。
  4. カード内の入力欄・ラベルは白文字・半透明白背景(`bg-white/5`、`text-white`)。
- これは `DESIGN.md` 6章の "no large colored blur" 原則に対する**明示的な例外**であり、
  ログイン画面 1 画面にのみ許可されたスコープである。他の画面(ダッシュボード、設定画面等)
  へこの blur/glow 表現を拡張しない。
- 新規プロダクトへ移植する際は、そのプロダクトのフレームワーク(Vite SPA か、
  Next.js SSR/Server Actions か)に応じて実装形態を選ぶ。詳細な移植手順・両実装の
  ソース全文は `references/canonical-login-screen.md` を参照すること(**必読**)。
- ロゴは常にダーク背景用(`forceTheme="dark"` 相当、または反転済みSVG)を使う。ページ全体の
  ライト/ダークトグルに連動させない — シェーダー背景は常に暗いため、ライトロゴだと視認性が
  崩れる。
- shadcnblocks の `login6`(rev_license 側)/ `login3`(rev_ad_dashboard 側)ブロックの
  レイアウト骨格は保つ。デモの Google OAuth ボタンや外部 CDN ロゴなど、第三者 JS/img を
  読み込む要素は REV-C プロダクトでは持ち込まない(CSP 制約)。
- 認証ロジック(Supabase Auth の実装詳細、Server Action、セッション確立)はプロダクトごとに
  異なってよい。本スキルが強制するのは**視覚的デザインの共通化**のみ。

## 正本(Canonical Reference Implementation)

実装の正本は以下の2箇所。**新規プロダクトはこの2つのどちらかを土台に移植する**。

- **rev_license**(Vite SPA + react-router、単一 Cloudflare Worker が
  `license.rev-c.org` と `admin-license.rev-c.org` の両方を配信):
  - `apps/dashboard-next/src/routes/auth/login.tsx` — ルート本体。Supabase Auth
    `signInWithPassword`、成功後は `defaultLandingPath()` で admin/app を出し分け。
  - `apps/dashboard-next/src/components/login6.tsx` — フォーム UI
    (`@shadcnblocks/login6` ベース)。
  - `apps/dashboard-next/src/components/shader-background.tsx` — WebGL シェーダー
    背景(`@react-three/fiber` + `three`)。
  - `apps/dashboard-next/src/components/brand-logo.tsx` — `forceTheme="dark"` で
    テーマ非依存にロゴを固定するパターン。
  - **`https://admin-license.rev-c.org/login` と `https://license.rev-c.org/login`
    は同一コンポーネント(`LoginPage` → `Login6` → `ShaderBackground`)を共有している**。
    ホスト名で admin/client を分けているのはログイン後の遷移先のみで、ログイン画面自体は
    1つしかない。

- **rev_ad_dashboard**(Next.js App Router + Server Actions):
  - `src/app/login/page.tsx` — ルート本体。
  - `src/app/login/login-form.tsx` — フォーム UI(`useActionState` + Server Action)。
  - `src/app/login/login-shader-background.tsx` — `next/dynamic` + `ssr: false` で
    シェーダーをクライアント専用チャンクに隔離するラッパー。
  - `src/components/shader4.tsx` — WebGL シェーダー本体(GLSL コードは rev_license 側
    `shader-background.tsx` とほぼ同一)。

両実装の完全なソース、GLSL シェーダーコードの解説、フレームワーク別の移植手順は
`references/canonical-login-screen.md` に収録している。

## 関連

- `DESIGN.md`(rev_license リポジトリルート)§14.3 Sidebar / §14.4 Login Screen —
  UI/UX の正。ログイン画面を変更する場合はこのセクションも更新すること。
- `revc-shadcn-frontend-workflow` スキル — shadcn/ui を使った REV-C フロントエンド
  全般のワークフロー。ログイン画面固有の意匠は本スキルが正本を持つ。
- `docs/design/theme-portability-contract.md`(rev_ad_dashboard)— コンポーネントを
  他プロジェクトへ移植する際の技術的前提(`radix-nova` スタイル、Tailwind v4 構文、
  統合パッケージ `radix-ui` 等)。ログイン画面を rev_ad_dashboard 側から移植する場合は
  併読すること。
