# REV-C Canonical Login Screen — 詳細リファレンス

> 2026-07-08 時点で実ファイルを読んで作成。コード引用は実装の要点を伝えるための抜粋/全文コピーであり、
> 実際の移植時は必ず引用元リポジトリの最新版を直接確認すること(このドキュメントはスナップショット)。

## 0. なぜこのデザインが正本なのか

- `https://admin-license.rev-c.org/login` と `https://license.rev-c.org/login` は、
  rev_license リポジトリの単一 Cloudflare Worker(`apps/dashboard-next`)が配信する
  **同一のログイン画面**である。ホスト名が違うだけで、`LoginPage` コンポーネント自体は
  1つしかない(ログイン成功後の遷移先だけが `defaultLandingPath()` によって
  `/admin/dashboard` か `/app/dashboard` かに分岐する)。
- `rev_ad_dashboard`(広告運用ダッシュボード、別リポジトリ・別フレームワーク)も、
  独立に同じ視覚デザイン(WebGL シェーダー背景 + backdrop-blur カード、REV-C ネイビー
  ティント)で `/login` を実装している。
- `DESIGN.md`(rev_license)§14.4 に明記されている通り、rev_license 側のシェーダー背景は
  「rev_ad_dashboard の `shader4` から移植した」ものであり、**両者は同一の意匠を共有する
  ことがユーザー承認済みの既定路線**である(6章の "no large colored blur" 原則に対する
  明示的な例外としてスコープされている)。
- したがって、REV-C が新しいプロダクトを立ち上げるたびにログイン画面のビジュアルを
  ゼロから考える必要はない。**この2つのどちらかを土台に移植すればよい。**

## 1. 見た目の仕様(共通の視覚的柱)

| 要素 | 仕様 |
| --- | --- |
| 背景 | フルスクリーンの WebGL シェーダー(`@react-three/fiber` + `three`)。REV-C ネイビー `#32435D`(`vec3(0.196, 0.263, 0.365)`)にティントされた、光の筋が流れる抽象パターン |
| アニメーション | `u_time` を毎フレーム更新。`prefers-reduced-motion: reduce` の場合は `frameloop="demand"` にして静止画化(GPU/RAF 負荷を止める) |
| カード | 背景の上に中央配置。`border border-white/10` + 半透明黒背景(`bg-black/60` または `bg-[#111111]/60`)+ `backdrop-blur-lg` + `shadow-2xl` + 角丸(`rounded-2xl`) |
| ロゴ | カード上部(rev_license)またはカード内上部(rev_ad_dashboard)。**常にダーク背景用ロゴ**(テーマ非依存)。rev_license は `forceTheme="dark"` prop、rev_ad_dashboard は反転済み SVG(`invert` クラス) |
| 見出し | 「ログイン」/「Login」+ 1行の説明文(白文字、`text-white` / `text-white/70`) |
| フォーム | メールアドレス → パスワード → primary button の縦積み。入力欄は `border-white/20 bg-white/5 text-white placeholder:text-white/40` |
| エラー表示 | `role="alert"` の赤系ボックス(destructive トークン使用) |
| フッター | 招待制の場合は「アカウントは招待制です」の案内のみ。セルフサインアップを許可する場合のみ「アカウント作成」リンクを出す |

この blur/glow 表現は **ログイン画面 1 画面にのみ** 許可されたスコープであり、他の画面
(ダッシュボード、設定画面、モーダル等)へ拡張しないこと(`DESIGN.md` 6章 "no large
colored blur" 原則の例外はここだけ)。

## 2. rev_license 側実装(Vite SPA + react-router)

### 2.1 ルート本体 — `apps/dashboard-next/src/routes/auth/login.tsx`

```tsx
import { useState } from "react";
import { Navigate, useNavigate, useSearchParams } from "react-router-dom";

import { Login6 } from "@/components/login6";
import { RouteFallback } from "@/components/route-fallback";
import { authErrorMessage } from "@/lib/auth-errors";
import { useSession } from "@/lib/auth-store";
import { defaultLandingPath } from "@/lib/hostname";
import { supabase } from "@/lib/supabase";

// /login — email/password sign-in (login6 block). On success Supabase emits a
// session via onAuthStateChange (auth-store), so we just navigate to the app.
// Invite-only policy (本番 Supabase は signup_disabled): no "アカウント作成" CTA
// is shown — signupHref is intentionally omitted so login6 renders the
// invite-only notice instead.
export function LoginPage() {
  const { session, checking } = useSession();
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();
  const callbackError = searchParams.get("error");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [loading, setLoading] = useState(false);
  const [errorMessage, setErrorMessage] = useState(
    callbackError === "link"
      ? "リンクが無効か、有効期限が切れています。パスワード再設定をやり直すか、管理者に再招待を依頼してください。"
      : callbackError === "callback"
        ? "確認リンクの処理に失敗しました。もう一度ログインしてください。"
        : callbackError === "recovery"
          ? "パスワード設定用のセッションを確認できませんでした。メールのリンクをもう一度開いてください。"
          : "",
  );

  if (checking) return <RouteFallback />;
  // Already authenticated → skip the login screen. Landing depends on the
  // domain: /admin/dashboard on the admin hostname, /app/dashboard otherwise.
  if (session)
    return <Navigate to={`${defaultLandingPath()}/dashboard`} replace />;

  const handleSubmit = async () => {
    setLoading(true);
    setErrorMessage("");
    const { error } = await supabase.auth.signInWithPassword({
      email: email.trim(),
      password,
    });
    if (error) {
      setErrorMessage(authErrorMessage(error));
      setLoading(false);
      return;
    }
    navigate(`${defaultLandingPath()}/dashboard`, { replace: true });
  };

  return (
    <Login6
      email={email}
      password={password}
      onEmailChange={setEmail}
      onPasswordChange={setPassword}
      onSubmit={handleSubmit}
      loading={loading}
      errorMessage={errorMessage}
      forgotPasswordHref="/auth/forgot-password"
    />
  );
}
```

要点:
- `defaultLandingPath()`(`src/lib/hostname.ts`)がホスト名から admin/app を判定する。
  `admin-license.rev-c.org` なら `/admin`、それ以外(`license.rev-c.org`)なら `/app` を返す。
  **これが admin/client 兼用ログイン画面の実体**。
- `signupHref` を渡さないことで招待制の案内文に切り替わる(下記 `login6.tsx` 参照)。
- 認証エラーは `authErrorMessage()` で日本語化してから表示する(Supabase の生エラー文言を
  そのまま出さない)。

### 2.2 フォーム UI — `apps/dashboard-next/src/components/login6.tsx`

```tsx
import type { FormEvent } from "react";

import { BrandLogo } from "@/components/brand-logo";
import { ShaderBackground } from "@/components/shader-background";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { cn } from "@/lib/utils";

// Adapted from @shadcnblocks/login6 (radix-nova). The original layout — centered
// card with logo above, email/password fields and a footer "need an account?"
// link — is preserved; the demo's Google OAuth button and CDN logo were removed
// (no third-party JS/img per CSP), copy is Japanese, and the form is wired to a
// real onSubmit handler with error + loading state.
interface Login6Props {
  email: string;
  password: string;
  onEmailChange: (value: string) => void;
  onPasswordChange: (value: string) => void;
  onSubmit: () => void;
  loading?: boolean;
  errorMessage?: string;
  /**
   * Where the "アカウント作成" link points. Omitted under the invite-only policy
   * (本番 Supabase は signup_disabled) so no public self-signup CTA is shown.
   */
  signupHref?: string;
  /** Where the "パスワードをお忘れですか？" link points (self-service reset). */
  forgotPasswordHref?: string;
  className?: string;
}

const Login6 = ({
  email,
  password,
  onEmailChange,
  onPasswordChange,
  onSubmit,
  loading = false,
  errorMessage,
  signupHref,
  forgotPasswordHref,
  className,
}: Login6Props) => {
  const handleSubmit = (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    if (loading) return;
    onSubmit();
  };

  return (
    <section
      className={cn(
        "relative min-h-svh overflow-hidden bg-background",
        className,
      )}
    >
      <ShaderBackground />
      <div className="relative z-10 flex min-h-svh items-center justify-center p-6">
        <div className="flex w-full max-w-sm flex-col items-center gap-6">
          <BrandLogo
            variant="full"
            forceTheme="dark"
            className="[&_img]:w-[140px]"
          />
          <form
            onSubmit={handleSubmit}
            className="flex w-full flex-col items-center gap-y-4 rounded-2xl border border-white/10 bg-black/60 px-6 py-10 shadow-2xl backdrop-blur-lg"
          >
            <div className="flex w-full flex-col gap-1 text-center">
              <h1 className="text-xl font-semibold tracking-tight text-white">
                ログイン
              </h1>
              <p className="text-sm text-white/70">
                REV License ダッシュボードにサインインします。
              </p>
            </div>

            {errorMessage ? (
              <div
                role="alert"
                className="w-full rounded-md border border-destructive/40 bg-destructive/10 px-3 py-2 text-sm text-destructive"
              >
                {errorMessage}
              </div>
            ) : null}

            <div className="flex w-full flex-col gap-2">
              <Label htmlFor="login-email" className="text-white">
                メールアドレス
              </Label>
              <Input
                id="login-email"
                type="email"
                autoComplete="email"
                placeholder="you@example.com"
                value={email}
                onChange={(event) => onEmailChange(event.target.value)}
                className="border-white/20 bg-white/5 text-sm text-white placeholder:text-white/40"
                required
              />
            </div>
            <div className="flex w-full flex-col gap-2">
              <div className="flex items-center justify-between">
                <Label htmlFor="login-password" className="text-white">
                  パスワード
                </Label>
                {forgotPasswordHref ? (
                  <a
                    href={forgotPasswordHref}
                    className="text-xs text-white/70 underline-offset-4 hover:text-white hover:underline"
                  >
                    パスワードをお忘れですか？
                  </a>
                ) : null}
              </div>
              <Input
                id="login-password"
                type="password"
                autoComplete="current-password"
                placeholder="パスワード"
                value={password}
                onChange={(event) => onPasswordChange(event.target.value)}
                className="border-white/20 bg-white/5 text-sm text-white placeholder:text-white/40"
                required
              />
            </div>
            <Button type="submit" className="w-full" disabled={loading}>
              {loading ? "サインイン中…" : "ログイン"}
            </Button>
          </form>
          {/*
            Invite-only: no public self-signup CTA. The link is only rendered
            when an explicit signupHref is provided (it is not, in production).
          */}
          {signupHref ? (
            <div className="flex justify-center gap-1 text-sm text-white/70">
              <p>アカウントをお持ちでない方</p>
              <a
                href={signupHref}
                className="font-medium text-white hover:underline"
              >
                アカウント作成
              </a>
            </div>
          ) : (
            <p className="text-center text-sm text-white/70">
              アカウントは招待制です。ご利用には管理者にお問い合わせください。
            </p>
          )}
        </div>
      </div>
    </section>
  );
};

export { Login6 };
```

### 2.3 ロゴのテーマ固定パターン — `apps/dashboard-next/src/components/brand-logo.tsx` の `forceTheme`

```tsx
interface BrandLogoProps {
  variant?: "full" | "icon";
  className?: string;
  /**
   * Pin the rendered asset to one theme regardless of the app's active theme.
   * Used on the login screen, where the logo sits over the (always-dark)
   * shader background rather than the themed page background.
   */
  forceTheme?: "light" | "dark";
}
```

`forceTheme="dark"` を渡すと、テーマトグルの状態に関わらず常にダーク背景用のロゴ画像
(`dark_revc_logo_hirizontal.png` 等)を単一の `<img>` で描画する(未指定時は
`dark:hidden`/`dark:block` の2枚差し替えでページテーマに追従する)。ログイン画面の背景は
常に暗い(シェーダー)ため、ページのライト/ダーク設定と無関係に「常にダーク版」を強制する
必要がある — これが `forceTheme` prop が存在する理由そのもの。新規プロダクトで同じ
パターンを移植する場合、ロゴコンポーネントに同様の「テーマ非依存の強制表示モード」を
用意すること。

### 2.4 シェーダー背景 — `apps/dashboard-next/src/components/shader-background.tsx`

```tsx
import { Canvas, useFrame, useThree } from "@react-three/fiber";
import { useEffect, useMemo, useRef, useState } from "react";
import * as THREE from "three";

import { cn } from "@/lib/utils";

// Ported from rev_ad_dashboard's shader4 (REV-C brand-tinted WebGL login
// background). Vite has no SSR step, so unlike the Next.js source this needs
// no ssr:false wrapper — the whole /login route is already its own lazy
// chunk (src/routes/lazy.ts), so three.js only loads when the login page does.
function usePrefersReducedMotion() {
  const [reduced, setReduced] = useState(
    () => window.matchMedia("(prefers-reduced-motion: reduce)").matches,
  );

  useEffect(() => {
    const query = window.matchMedia("(prefers-reduced-motion: reduce)");
    const onChange = (event: MediaQueryListEvent) => setReduced(event.matches);
    query.addEventListener("change", onChange);
    return () => query.removeEventListener("change", onChange);
  }, []);

  return reduced;
}

interface ShaderPlaneProps {
  vertexShader: string;
  fragmentShader: string;
  uniforms: { [key: string]: { value: unknown } };
  animate: boolean;
}

const ShaderPlane = ({
  vertexShader,
  fragmentShader,
  uniforms,
  animate,
}: ShaderPlaneProps) => {
  const meshRef = useRef<THREE.Mesh>(null);
  const { size } = useThree();

  useFrame((state) => {
    if (meshRef.current) {
      const material = meshRef.current.material as THREE.ShaderMaterial;
      // prefers-reduced-motion 時は u_time を止め、静止画として表示する。
      if (animate) {
        material.uniforms.u_time.value = state.clock.elapsedTime;
      }
      material.uniforms.u_resolution.value.set(size.width, size.height, 1.0);
    }
  });

  return (
    <mesh ref={meshRef}>
      <planeGeometry args={[2, 2]} />
      <shaderMaterial
        vertexShader={vertexShader}
        fragmentShader={fragmentShader}
        uniforms={uniforms}
        side={THREE.FrontSide}
        depthTest={false}
        depthWrite={false}
      />
    </mesh>
  );
};

interface ShaderBackgroundProps {
  className?: string;
}

const VERTEX_SHADER = `
  varying vec2 vUv;
  void main() {
    vUv = uv;
    gl_Position = vec4(position, 1.0);
  }
`;

const FRAGMENT_SHADER = `
  precision highp float;
  varying vec2 vUv;
  uniform float u_time;
  uniform vec3 u_resolution;
  uniform vec3 u_tint;

  float lightImpulses(vec2 v, float time) {
      float streak = 0.0;

      for (int j = 0; j < 4; j++) {
          float seed = float(j) * 1.37;
          float phase = dot(v, normalize(vec2(sin(seed*12.3), cos(seed*4.7))));
          float speed = 0.4 + fract(sin(seed*77.7)*43758.5);

          float pulse = exp(-30.0 * pow(fract(phase*0.2 + time*speed) - 0.5, 2.0));

          streak += pulse;
      }

      return streak;
  }

  vec4 getScene(vec2 fragCoord, vec2 resolution) {
      float i = .13, a;
      vec2 r = resolution;

      vec2 p = (fragCoord+fragCoord - r) / r.y / .9;
      vec2 d = vec2(-1,1);
      vec2 b = p - i*d;
      vec2 c = p * mat2(1, 1, d/(.1 + i/dot(b,b)));
      vec2 v = c * mat2(cos(.5*log(a=dot(c,c))))/i;
      vec2 w = vec2(0.0);

      for(; i++<9.; w += 1.1+sin(v))
          v += 0.9* sin(v.yx/i+u_time) / i + .3;

      i = length(5.0);

      vec4 base = 0.9 - exp( -exp( c.x * vec4(0.0,0.0,0,0) )
                     /  vec4(length(w))
                     / ( 2. + i*i/4. - i )
                     / ( .5 + 1. / a )
                     / ( .03 + abs( length(p)-.7 ) )
               );

      float streak = lightImpulses(v, u_time);
      base.rgb += streak * 0.015;

      return base;
  }

  void mainImage(out vec4 fragColor, in vec2 fragCoord)
  {
      vec2 r = u_resolution.xy;
      vec2 uv = fragCoord / r;

      float u_aberration = 10.0;
      float ChromaticAberration = u_aberration;
      vec2 texel = 1.0 / r;
      vec2 coords = (uv - 0.5) * 2.0;
      float coordDot = dot(coords, coords);
      vec2 precompute = ChromaticAberration * coordDot * coords;

      vec2 uvR = uv - texel * precompute;
      vec2 uvB = uv + texel * precompute;

      vec2 fragCoordR = uvR * r;
      vec2 fragCoordB = uvB * r;

      vec4 colR = getScene(fragCoordR, r);
      vec4 colG = getScene(fragCoord , r);
      vec4 colB = getScene(fragCoordB, r);

      vec3 rgb = vec3(colR.r, colG.g, colB.b);
      // REV-C ブランドカラー(ネイビー)でティントする。輝度を保持しつつ
      // u_tint の色相へブレンドすることで、白黒基調の光の筋がネイビー系に見えるようにする。
      float luma = dot(rgb, vec3(0.299, 0.587, 0.114));
      vec3 tintNorm = u_tint / max(max(u_tint.r, u_tint.g), max(u_tint.b, 0.0001));
      vec3 tinted = luma * tintNorm;
      rgb = mix(rgb, tinted, 0.85);

      fragColor = vec4(rgb, 1.0);
  }

  void main() {
      vec4 fragColor;
      vec2 fragCoord = vUv * u_resolution.xy;
      mainImage(fragColor, fragCoord);
      gl_FragColor = fragColor;
  }
`;

export function ShaderBackground({ className }: ShaderBackgroundProps) {
  const uniforms = useMemo(
    () => ({
      u_time: { value: 0 },
      u_resolution: { value: new THREE.Vector3(1, 1, 1) },
      // REV-C ブランドカラー ネイビー #32435D。
      u_tint: { value: new THREE.Vector3(0.196, 0.263, 0.365) },
    }),
    [],
  );
  const reducedMotion = usePrefersReducedMotion();

  return (
    <div
      className={cn(
        "pointer-events-none absolute inset-0 overflow-hidden",
        className,
      )}
      aria-hidden="true"
    >
      <Canvas frameloop={reducedMotion ? "demand" : "always"}>
        <ShaderPlane
          vertexShader={VERTEX_SHADER}
          fragmentShader={FRAGMENT_SHADER}
          uniforms={uniforms}
          animate={!reducedMotion}
        />
      </Canvas>
    </div>
  );
}
```

Vite は SSR ステップを持たないため、`ssr: false` のようなラッパーは不要。ただし
`three`/`@react-three/fiber` は重量級なので、`/login` ルート自体を lazy chunk
(`src/routes/lazy.ts` 経由の `React.lazy`)にして、ログイン画面を開いた時だけ
バンドルが読み込まれるようにしてある。新規 Vite プロダクトへ移植する際もこの
lazy-loading は必ず踏襲すること。

## 3. rev_ad_dashboard 側実装(Next.js App Router + Server Actions)

### 3.1 ルート本体 — `src/app/login/page.tsx`

```tsx
import { LoginForm } from "./login-form";
import { LoginShaderBackground } from "./login-shader-background";

/**
 * ログイン画面（`/login`）。メール+パスワードで Supabase Auth にサインインする（§7-2-1）。
 * 成功時は Server Action（loginAction）が `/clients` へリダイレクトする。
 * 本ページ自体は認可判断を行わない（role はサーバ側 supabase-session.ts が唯一の信頼源）。
 * 見た目は shadcnblocks の login3 ブロックのレイアウトに、shader4 の WebGL シェーダー背景
 * （REV-C ブランドカラーにティント）を重ねている。
 */
export default function LoginPage() {
  return (
    <section className="relative h-screen overflow-hidden bg-background">
      <LoginShaderBackground />
      <div className="relative z-10 flex h-full items-center justify-center">
        <LoginForm />
      </div>
    </section>
  );
}
```

### 3.2 フォーム UI — `src/app/login/login-form.tsx`

```tsx
"use client";

/**
 * ログインフォーム（client component）。Server Action（loginAction）を useActionState で呼ぶ。
 * §7-2-1: ここでロール等は一切扱わない。認証結果のリダイレクト先固定（/clients）とエラー表示のみ。
 * 見た目は shadcnblocks の login3 ブロックのレイアウトに合わせている（ロゴ+見出し+縦積みフォーム）。
 */

import Image from "next/image";
import { useActionState } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { loginAction, type LoginActionState } from "./actions";

const INITIAL_STATE: LoginActionState = { error: null };

export function LoginForm() {
  const [state, formAction, isPending] = useActionState(loginAction, INITIAL_STATE);

  return (
    <div className="flex w-full max-w-sm flex-col items-center gap-y-4 rounded-2xl border border-white/10 bg-[#111111]/60 px-6 py-12 shadow-2xl backdrop-blur-lg">
      <Image
        src="/brand/revc_logo.svg"
        alt="REV-C"
        width={100}
        height={14}
        className="mb-4 h-auto w-[100px] invert"
        priority
      />
      <div className="flex flex-col items-center gap-1 text-center">
        <h1 className="text-sm font-semibold text-white">Login</h1>
        <p className="text-sm text-white/70">
          広告運用ダッシュボードにログインしてください。
        </p>
      </div>

      <form action={formAction} className="flex w-full flex-col gap-4">
        <div className="flex w-full flex-col gap-2">
          <Label htmlFor="email" className="text-white">
            メールアドレス
          </Label>
          <Input
            id="email"
            name="email"
            type="email"
            autoComplete="email"
            required
            disabled={isPending}
            className="border-white/20 bg-white/5 text-sm text-white placeholder:text-white/40"
          />
        </div>

        <div className="flex w-full flex-col gap-2">
          <Label htmlFor="password" className="text-white">
            パスワード
          </Label>
          <Input
            id="password"
            name="password"
            type="password"
            autoComplete="current-password"
            required
            disabled={isPending}
            className="border-white/20 bg-white/5 text-sm text-white placeholder:text-white/40"
          />
        </div>

        {state.error && (
          <p role="alert" className="text-sm text-destructive">
            {state.error}
          </p>
        )}

        <Button type="submit" disabled={isPending} className="mt-2 w-full">
          {isPending ? "ログイン中..." : "ログイン"}
        </Button>
      </form>
    </div>
  );
}
```

ロゴはここでは `forceTheme` prop 方式ではなく、SVG に直接 `invert` クラスを常時付与する
方式(黒地ロゴを白反転)。rev_license 側と実装方法は違うが、狙いは同じ(常にダーク背景用の
見え方に固定する)。移植先がどちらの方式を採るかは、ロゴアセットが PNG(明暗2枚持ち)か
SVG(1枚を CSS フィルタで反転)かによって決めればよい。

### 3.3 シェーダー背景ラッパー — `src/app/login/login-shader-background.tsx`

```tsx
"use client";

/**
 * ログイン画面専用のシェーダー背景ラッパー。
 * `@react-three/fiber` / `three` は WebGL 依存のクライアント専用ライブラリのため、
 * `next/dynamic` + `ssr: false` でこのファイル配下にのみ閉じ込め、
 * `/login` 以外のルートの JS バンドルに含まれないようにする。
 */

import dynamic from "next/dynamic";

const Shader4 = dynamic(
  () => import("@/components/shader4").then((mod) => mod.Shader4),
  { ssr: false },
);

export function LoginShaderBackground() {
  return (
    <Shader4 className="absolute inset-0 h-full max-h-none min-h-0 w-full" />
  );
}
```

**Next.js への移植で最も重要な注意点**: `three`/`@react-three/fiber` は
`window`/`WebGL` に依存するクライアント専用ライブラリであり、Next.js の SSR/RSC
パイプラインでそのまま import すると失敗する。`next/dynamic(..., { ssr: false })` で
明示的にクライアント専用チャンクへ隔離すること。Vite(rev_license 側)では SSR ステップが
無いためこの隔離が不要 — フレームワークによって要否が変わる代表的な差分。

### 3.4 シェーダー本体 — `src/components/shader4.tsx`

GLSL の vertex/fragment shader コードは rev_license 側 `shader-background.tsx` と
**ほぼ完全に同一**(`u_tint` の値 `vec3(0.196, 0.263, 0.365)` = REV-C ネイビー
`#32435D` も一致)。差分は主に以下:

- rev_ad_dashboard 版は `vertexShader`/`fragmentShader`/`uniforms` を props で
  上書き可能にしている(デフォルト値として shader コードを持つ)。rev_license 版は
  モジュール定数 `VERTEX_SHADER`/`FRAGMENT_SHADER` として固定している。
- rev_ad_dashboard 版のコンテナは `<section className="relative h-svh max-h-[1200px]
  min-h-[600px] w-full overflow-hidden">`。rev_license 版は
  `<div className="pointer-events-none absolute inset-0 overflow-hidden">` +
  `aria-hidden="true"`(装飾要素として明示的にアクセシビリティツリーから除外)。
  **新規移植時は rev_license 版の `aria-hidden="true"` を踏襲することを推奨** —
  シェーダー背景は完全に装飾用途であり、スクリーンリーダーに晒す情報を持たない。

完全なソースは `src/components/shader4.tsx`(rev_ad_dashboard リポジトリ)を直接参照
すること(GLSL コード全体は §2.4 の rev_license 版とほぼ同一なので本書では省略)。

## 4. 新規プロダクトへの移植手順チェックリスト

- [ ] 移植先のフレームワークを確認する(Vite/SPA なら rev_license 版、Next.js/SSR なら
      rev_ad_dashboard 版を土台にする)。
- [ ] `@react-three/fiber` と `three` を依存に追加する(package.json)。
- [ ] Next.js の場合、シェーダーコンポーネントを `next/dynamic(..., { ssr: false })` で
      クライアント専用に隔離する専用ラッパーファイルを作る(§3.3 参照)。
- [ ] `/login` ルート自体を lazy-load / code-split する(3D ライブラリを他ルートの
      バンドルに含めない)。Vite: `React.lazy` + ルート定義側での lazy import。
      Next.js: App Router のルートセグメントは既定で分割されるため通常は追加対応不要。
- [ ] GLSL シェーダーの `u_tint` は REV-C ネイビー `vec3(0.196, 0.263, 0.365)`
      (`#32435D`)を既定値として使う。プロダクト固有のアクセントカラーに変える場合は
      デザイン承認を得ること(勝手に変えない — `DESIGN.md` 6章の保護対象に準ずる扱い)。
- [ ] `prefers-reduced-motion` 対応(`frameloop`/`animate` の分岐)を必ず踏襲する。
      モーション低減設定のユーザーに常時アニメーションを強制しない。
- [ ] ロゴは常にダーク背景用に固定する(`forceTheme` prop 方式、または反転済み
      SVG/PNG のどちらか、アセット形式に応じて選ぶ)。ページ全体のライト/ダークトグルに
      連動させない。
- [ ] カードは `border-white/10` + 半透明黒背景 + `backdrop-blur-lg` + `rounded-2xl`
      + `shadow-2xl`。入力欄は `border-white/20 bg-white/5 text-white
      placeholder:text-white/40`。
- [ ] エラー表示は `role="alert"` + destructive トークン。生の SDK エラー文言を
      そのまま出さず、日本語の運用者向けメッセージにマッピングする層を挟む
      (rev_license の `authErrorMessage()` 相当)。
- [ ] 招待制ポリシーの場合、セルフサインアップ CTA を出さない(条件付きレンダリング。
      `signupHref` が無ければ「招待制です」の案内文に切り替える rev_license のパターンを
      踏襲)。
- [ ] この blur/glow 表現をログイン画面以外に拡張しない。他画面で似た表現が欲しくなったら、
      新規の承認が必要という前提で `DESIGN.md` を更新してから着手する。
- [ ] 移植後、実機(実ブラウザ)でシェーダーが正しく描画されること、
      `prefers-reduced-motion` 有効時に静止画になること、モバイル/デスクトップ双方の
      レイアウト崩れが無いことを確認する。lint/typecheck だけでは検出できない
      (WebGL の実描画確認は必須)。

## 5. 関連ドキュメント

- `DESIGN.md`(rev_license リポジトリルート)
  - §6 Shell And Navigation → Protected Shell Patterns(ログイン画面は保護対象。
    構造・配置・サイズ感・ロゴ位置・主要文言・主要余白の変更にはユーザー承認が必要)。
  - §14.3 Sidebar (Dark-Locked, Explicit Exception)
  - §14.4 Login Screen (Explicit Exception) — 本スキルが詳細化した箇所の要約がここにある。
- `docs/design/theme-portability-contract.md`(rev_ad_dashboard)— コンポーネントの
  他プロジェクトへの移植に伴う技術的前提(`radix-nova` スタイル、Tailwind v4 専用構文、
  統合パッケージ `radix-ui`、`tw-animate-css` 等)。ログイン画面を rev_ad_dashboard 側の
  実装から移植する場合は、シェーダー/レイアウト以外にこれらの前提も満たす必要がある。
