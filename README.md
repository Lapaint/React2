# 김민우 (202230305) - React2

## [2026-09-16] 3주차: 프로젝트 구성 심화 & Layouts and Pages

### 1. Folder and file conventions (이어서)
- **Route Groups(라우트 그룹)**: 폴더를 `(folderName)`처럼 괄호로 감싸면 URL 경로에 포함되지 않으면서 코드만 정리할 수 있음. 라우팅되지 않는 파일은 `_folder`(비공개 폴더)에 함께 저장함.
- **Parallel / Intercepted Routes(병렬 및 가로채기 라우팅)**: 슬롯 기반 레이아웃, 모달 라우팅 같은 특정 UI 패턴에 적합함. 부모 레이아웃에 렌더링되는 명명된 슬롯에는 `@slot`을 사용하고, 인터셉트 패턴(`(.)folder`, `(..)folder`, `(...)folder`)을 사용하면 URL을 변경하지 않고도 다른 경로를 현재 레이아웃 위에 렌더링할 수 있음(예: 목록 위에 모달로 상세 보기 표시).
- **메타데이터 파일 규칙**: `favicon`, `icon`, `apple-icon` 등 앱 아이콘 파일 / `opengraph-image`, `twitter-image` 등 SNS 공유 이미지 파일 / `sitemap`, `robots` 등 SEO 파일이 규칙적인 이름으로 제공됨.
- **Open Graph Protocol**: 링크를 SNS(페이스북, 인스타그램, X, 카카오톡 등)로 공유할 때 '미리보기'를 생성하는 프로토콜. 페이스북이 주도하는 표준이며, 웹페이지의 `<head>` 메타 태그에 `og:title`, `og:description`, `og:image` 등을 선언함.

### 2. Organizing your project (프로젝트 구성하기)
- Next.js는 파일 구성 방식에 제약이 없지만, 체계적인 구성을 돕는 몇 가지 기능을 제공함.
- **Component hierarchy(컴포넌트 계층 구조)**: `layout.js` → `template.js` → `error.js`(오류 경계) → `loading.js`(서스펜스 경계) → `not-found.js`(오류 경계) → `page.js` 순서로 중첩되어 렌더링됨.
- **Colocation(코로케이션)**: 파일·폴더를 기능별로 그룹화하여 구조를 명확히 정의하는 것. `app` 디렉토리 내 파일은 `page.js`/`route.js`가 없으면 라우팅되지 않으므로 컴포넌트·유틸 파일을 라우팅 세그먼트 안에 안전하게 함께 둘 수 있음.
- **Private folders(비공개 폴더)**: 폴더 앞에 언더스코어를 붙여(`_folderName`) 만들며, 해당 폴더와 하위 폴더 전체가 라우팅에서 제외됨. UI 로직/라우팅 로직 분리, 파일 정렬·그룹화, 이름 충돌 방지 등에 유용함.
- **Route groups(라우팅 그룹)**: 폴더를 괄호로 묶어(`(folderName)`) 사이트 섹션·팀별로 파일을 구성할 수 있음. 동일한 라우팅 세그먼트 레벨에 여러 루트 레이아웃을 만들 때도 사용함.
- **src 디렉토리**: `app`을 포함한 애플리케이션 코드를 `src/` 폴더 안에 선택적으로 저장하여, 프로젝트 루트의 설정 파일과 분리할 수 있음.

### 3. Layouts and Pages
- 이번 장에서는 **레이아웃과 페이지를 만들고 서로 연결하는 방법**을 다룸. Next.js는 파일 시스템 기반 라우팅을 사용하므로 폴더·파일로 경로를 정의함.

#### 3-1. Creating a page (페이지 만들기)
- `page`는 특정 경로에서 렌더링되는 UI임. `app` 디렉토리에 `page` 파일을 추가하고 React 컴포넌트를 `default export`하여 생성함.

```tsx
// app/page.tsx
export default function Page() {
  return <h1>Hello Next.js!</h1>
}
```

#### 3-2. Creating a layout (레이아웃 만들기)
- `layout`은 여러 페이지에서 공유되는 UI이며, 네비게이션 시 **state와 상호작용을 유지**하고 다시 렌더링되지 않음.
- `layout` 파일에서 컴포넌트를 `default export`하며, `children` prop(page 또는 다른 layout)을 반드시 받아야 함.
- `app` 디렉토리 루트에 정의된 레이아웃은 **루트 레이아웃**이라 하며 **필수**이고, `<html>`·`<body>` 태그를 포함해야 함.
-  subpage의 layout은 없어도 되지만, **RootLayout(루트 레이아웃) 컴포넌트는 반드시 있어야 함**. 이름은 특별한 이유가 없다면 `RootLayout`으로 하는 것이 좋음(문서의 `DashboardLayout` 예시는 폴더명이 아닌 라우팅 용도이기 때문).

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

#### 3-3. Opting for loading skeletons on a specific route (특정 라우트에 로딩 스켈레톤 적용)
- 특정 라우트 폴더에만 로딩 스켈레톤을 적용하려면 새 라우팅 그룹(예: `(overview)`)을 만들고 그 안으로 `loading.tsx`를 이동함 → URL 구조에 영향을 주지 않고 해당 라우트에만 로딩 UI가 적용됨.
- 로딩 속도가 빨라 확인이 어려우므로, `await new Promise(resolve => setTimeout(resolve, 3000))`으로 지연시켜 테스트함.

#### 3-4. Creating multiple root layouts (여러 개의 루트 레이아웃 만들기)
- 최상위 `layout.js`를 제거하고 각 라우팅 그룹 내부에 개별 `layout.js`를 추가하면 여러 개의 루트 레이아웃을 만들 수 있음.
- 완전히 다른 UI/UX를 갖는 섹션(예: `(marketing)`, `(shop)`)으로 앱을 분할할 때 유용하며, 각 루트 레이아웃에 `<html>`·`<body>` 태그를 모두 추가해야 함.

---


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
