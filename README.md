# 김민우 (202230305) - React2


## [2026-09-09] 2주차: 프로젝트 수동 생성 & 프로젝트 구조와 라우팅 규칙

### 1. Installation - 프로젝트 수동 생성 (개념 이해용, 실습은 자동 생성 사용)
- Next.js는 **file-system Routing**을 사용함. `app` 디렉토리 안의 `layout.tsx`(루트 레이아웃, `<html>`/`<body>` 필수)와 `page.tsx`(홈 페이지)로 시작함.
- 수동 생성 시 타입스크립트 환경이 아니므로 `@types/react`, `@types/react-dom`을 `-D`(devDependencies) 옵션으로 추가 설치해야 오류가 없음.
- `public` 디렉토리는 정적 리소스(이미지 등) 저장용이며, 기본 URL(`/`)로 바로 참조 가능함(`public/profile.png` → `/profile.png`).
- `package.json`에 `dev`/`build`/`start`/`lint` 스크립트를 등록하며, **Turbopack**이 기본 번들러임(Webpack은 `--webpack` 옵션으로 사용).
- 개발 서버 실행: `pnpm dev` 후 `http://localhost:3000`에 **수동으로 접속**해야 함(React와 다름).
- import 절대경로 별칭은 `tsconfig.json`의 `paths` 옵션으로 설정함(예: `@/components/button`). 단, `baseUrl`은 TypeScript 6.0부터 Deprecated(7.0에서 삭제 예정)이므로 최신 방식은 `baseUrl` 없이 `paths`에 상대 경로(`./src/*`)를 직접 명시함.

### 2. 자동 생성 방식 (`pnpm create next-app@latest`)
- 실습에서는 이 방식을 사용함. 프로젝트 이름 입력 후 **Yes(권장 기본값) / No, 이전 설정 재사용 / No, 직접 커스터마이즈** 중 선택함.
- 커스터마이즈 시 TypeScript, ESLint, React Compiler, Tailwind CSS, `src/` 디렉토리, App Router, import alias(기본 `@/*`) 여부를 순서대로 선택함.
- 자동 생성 시 `tsconfig.json`, `eslint.config.mjs`, `app/layout.tsx`, `app/page.tsx` 등 수동 생성 시 직접 해야 했던 설정들이 자동으로 처리됨.

### 3. ESLint 설정 방식: `.eslintrc.json` vs `eslint.config.mjs`
- `.eslintrc.json`은 JSON 형식으로 간단하지만 주석·조건문 등을 쓸 수 없어 복잡한 설정이 어려움.
- `eslint.config.mjs`는 JS 모듈(ESM) 형식으로, ESLint v9 이상 공식 권장 방식이며 최신 Next.js의 기본값임. 함수/변수 등을 사용한 동적 설정이 가능함.
- 두 방식 모두 현재 지원되며, 필요 시 서로 마이그레이션 가능함.

### 4. 용어 정의
- **route(라우트)**: 경로 / **routing(라우팅)**: 경로를 찾아가는 과정. path와의 혼동을 피하기 위해 대부분 "라우팅"으로 번역함.
- **directory / folder**: OS에 따라 다른 용어일 뿐 같은 의미로 이해하면 됨.
- **segment**: 라우팅과 관련된 directory의 별칭 정도로 이해하면 됨.

### 5. Folder and file conventions (폴더 및 파일 규칙)
- **최상위 폴더**: `app`(앱 라우터), `pages`(페이지 라우터), `public`(정적 리소스), `src`(선택적 소스 폴더).
- **최상위 파일**: `next.config.js`(Next.js 설정), `package.json`(종속성/스크립트), `.env`류(환경 변수), `tsconfig.json`/`jsconfig.json`(타입/모듈 설정) 등.
- **라우팅 파일**: `layout`(레이아웃), `page`(페이지), `loading`(로딩 UI), `error`/`global-error`(오류 UI), `not-found`(404 UI), `route`(API 엔드포인트) 등 규칙적인 이름의 파일로 구성함.
- **중첩 라우팅(Nested routes)**: 디렉토리 중첩 = URL 세그먼트 중첩. 각 레이아웃은 하위 세그먼트를 감싸며, `page.tsx`나 `route.ts`가 있는 디렉토리만 실제 경로로 공개됨.

### 6. 동적 라우팅 (Dynamic routes) — 3가지 구분
핵심 차이: **하위 경로를 어디까지 허용하는가** / **동적 세그먼트 없는 기본 경로를 처리할 수 있는가**

| 구분 | 디렉토리 구조 | 특징 | 예시 |
|---|---|---|---|
| 일반 동적 라우팅 | `[slug]` | 1개 세그먼트만 매칭. 기본 경로·하위 경로는 404 | `/posts/abc` ◯ / `/posts` ✕ / `/posts/abc/def` ✕ |
| Catch-all 라우팅 | `[...slug]` | 하위 경로 깊이 상관없이 모두 매칭, 단 기본 경로는 불가 | `/shop/clothing`, `/shop/clothing/shirts` ◯ |
| Optional Catch-all | `[[...slug]]` | Catch-all + 기본 경로(파라미터 없음)까지 매칭 | `/posts`, `/posts/abc`, `/posts/abc/def` 모두 ◯ |

- `params` 속성을 통해 slug 값을 받아 page에서 사용함.
- *슬러그(Slug): 신문·잡지 제목처럼 중요한 의미의 단어만으로 구성한 경로 표현.*
