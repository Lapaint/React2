# 김민우 (202230305) - React2

## [2026-09-23] 4주차: 동적 라우팅 심화 & 페이지 연결
 
### 1. Link Component (API Reference 복습)
- `<Link>`는 HTML `<a>` 요소를 확장하여 프리페칭(prefetching)과 라우트 간 클라이언트 사이드 내비게이션 기능을 제공하는 React 컴포넌트임. Next.js에서 라우트 간 이동을 위해 주로 사용됨.
- 주요 prop
| Prop | 예시 | 타입 |
|---|---|---|
| `href` (필수) | `href="/dashboard"` | String or Object |
| `replace` | `replace={false}` | Boolean |
| `scroll` | `scroll={false}` | Boolean |
| `prefetch` | `prefetch={false}` | Boolean, "auto", or null |
| `onNavigate` | `onNavigate={(e) => {}}` | Function |
| `transitionTypes` | `transitionTypes={['slide-in']}` | string[] |
 
- `href`는 문자열뿐 아니라 `{ pathname, query }` 형태의 객체로도 전달 가능함(예: `/about?name=test`).
- `className`이나 `target="_blank"`와 같은 `<a>` 태그 속성을 `<Link>`에 props로 추가하면, 내부의 `<a>` 요소로 그대로 전달됨.
### 2. Creating a layout (복습)
- subpage의 layout은 없어도 되지만, **RootLayout 컴포넌트는 반드시 있어야 함**.
- 문서에서는 root layout의 이름을 `DashboardLayout`이라고 했지만, layout은 결국 routing page를 위한 것이기 때문에 특별한 이유가 없다면 `RootLayout`으로 명명하는 것이 좋음.
### 3. Creating a nested route (중첩 라우트 만들기)
- 중첩 라우트는 다중 URL 세그먼트로 구성된 라우트임. 예를 들어 `/blog/[slug]` 경로는 `/`(Root Segment), `blog`(Segment), `[slug]`(Leaf Segment) 세 개의 세그먼트로 구성됨.
- 폴더는 URL 세그먼트에 매핑되는 경로 세그먼트를 정의하는 데 사용되고, 파일(`page`, `layout` 등)은 세그먼트에 표시되는 UI를 만드는 데 사용됨. 폴더를 중첩하면 중첩된 라우트를 만들 수 있음.
- `/blog`에 대한 경로를 추가하려면 `app` 디렉토리에 `blog` 폴더를 만들고, 공개적으로 액세스할 수 있도록 `page.tsx` 파일을 추가함.
  - 문서 예제 코드(`@/lib/posts`, `@/ui/post` import)를 그대로 복사하면 해당 모듈이 없어 오류가 발생함. 실습에서는 `<li>Post 1</li>` 같은 더미 리스트를 바로 출력하는 방식으로 대체함.
- 폴더 이름을 대괄호(예: `[slug]`)로 묶으면 데이터에서 여러 페이지를 생성하는 데 사용되는 **동적 경로 세그먼트**가 생성됨(예: 블로그 게시물, 제품 페이지 등). 특정 블로그 게시물 경로를 만들려면 `blog` 안에 새 `[slug]` 폴더를 만들고 `page` 파일을 추가함.
### 4. Dynamic Segment - [slug]의 이해
- **슬러그(Slug)**: 신문·잡지 제목처럼 핵심 의미를 포함한 단어만 조합해 간단명료하게 작성한 경로 표현에서 유래함.
- 경로 `/blog/[slug]`의 `[slug]` 부분은 불러올 데이터의 key를 의미하므로, 데이터에는 `slug` key가 반드시 있어야 함(`[foo]`라면 데이터에 `foo` key가 있어야 함).
- 디렉토리 구조: `app/blog/page.tsx`(블로그 메인/목록), `app/blog/[slug]/page.tsx`(블로그 상세 페이지).
```tsx
// app/blog/[slug]/page.tsx
import { posts } from "../posts"
 
export default async function Posts({ params }: { params: { slug: string } }) {
  const { slug } = await params
  const post = posts.find((p) => p.slug === slug)
 
  if (!post) {
    return <h1>게시글을 찾을 수 없습니다!</h1>
  }
 
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}
```
 
- **오류**: `Route "/blog/[slug]" used params.slug. params should be awaited before using its properties.`
  - Next.js 14.2 이후로 `params`와 `searchParams`는 내부적으로 Promise 기반 객체일 수 있어서, 바로 쓰면 안 되고 `await`하거나 props의 구조 분해에서 미리 `await`해야 함(실습 버전은 15.x라 발생). 서버를 재실행해야 정상 동작할 수도 있음.
  - `async function`: 함수를 `async`로 선언해야 내부에서 `await`를 쓸 수 있음.
  - `const { slug } = await params;`: `await params`는 params가 가리키는 Promise를 해제(resolve)해서 실제 객체 `{ slug: "..." }`를 얻고, `const { slug } = ...`로 그 객체에서 `slug` 프로퍼티만 꺼내 오는 구조 분해 할당임(`const resolved = await params; const slug = resolved.slug;`와 동일).
  - `const post = posts.find((p) => p.slug === slug)`: `posts`는 배열(더미 데이터나 DB 조회 결과)이고, `.find()`는 조건에 맞는 첫 번째 요소를 반환, 못 찾으면 `undefined`를 반환함. 찾는 게 없는데 `post.title` 같은 접근을 하면 런타임 에러가 나므로 `if (!post)` 존재 검사가 필요함(문서에는 없어서 추가한 부분).
  - 데이터 소스가 크다면 `.find()`는 O(n)이므로 DB 쿼리로 바꿔야 함. O(n)은 입력 데이터 크기 n에 비례해 시간/메모리 사용량이 선형적으로 증가한다는 의미.
  - `Promise<{ slug: string }>` 타입을 명시하지 않아도 오류 없이 동작하지만, params가 비동기식이라는 것을 명확히 하고 코드 가독성을 높이며 `await`을 깜빡했을 때 TypeScript가 잡아줄 수 있으므로 Promise 명시를 권장함.
- `/blog/page.tsx`도 더미 데이터를 `map` 함수(구조 분해 할당)로 출력하도록 수정함. 보통 `/blog/page.tsx`는 포스팅 리스트를, `/blog/[slug]/page.tsx`는 상세 페이지를 출력하는 역할을 함.
### 5. Nesting layouts (중첩 레이아웃)
- 기본적으로 폴더 계층 구조의 레이아웃도 중첩되어 있음. 즉, 자식 prop을 통해 자식 레이아웃을 감싸게 됨. 특정 경로 세그먼트(폴더) 안에 레이아웃을 추가하여 레이아웃을 중첩할 수 있음.
- 예를 들어 `/blog` 경로에 대한 레이아웃을 만들려면 `blog` 폴더 안에 새 `layout` 파일을 추가함(`app/blog/layout.tsx`). `[slug]`에도 동일하게 레이아웃을 추가해볼 수 있음(`app/blog/[slug]/layout.tsx`).
- 위 레이아웃들을 결합하면 루트 레이아웃(`app/layout.tsx`)이 blog 레이아웃(`app/blog/layout.tsx`)을 감싸고, blog 레이아웃은 blog page와 블로그 게시물 페이지(`[slug]`)를 감쌈.
- 실습: RootLayout / BlogLayout / SlugLayout 각각에 구분되는 header·footer 문구를 추가한 뒤 렌더링 결과를 확인하면, `Root Layout Header` → `Blog Layout Header` → `Slug Layout Header` → 페이지 콘텐츠 → `Slug Layout Footer` → `Blog Layout Footer` → `Root Layout Footer` 순서로 계층적으로 중첩되어 출력됨을 확인함.
### 6. Rendering with search params (검색 매개변수를 사용한 렌더링)
- 서버 컴포넌트 `page`에서는 `searchParams` prop(타입: `Promise<{ [key: string]: string | string[] | undefined }>`)을 사용하여 검색 매개변수에 접근할 수 있음.
- `searchParams`를 사용하면 해당 페이지는 **동적 렌더링(dynamic rendering)**으로 처리됨. URL의 쿼리 파라미터를 읽기 위해서는 요청(request)이 필요하기 때문에, Next.js는 이 페이지를 정적으로 미리 생성할 수 없고 요청이 올 때마다 새로 렌더링함.
- 클라이언트 컴포넌트는 `useSearchParams` Hook을 사용하여 검색 매개변수를 읽을 수 있음.
- **언제 무엇을 사용하나**
  - 페이지 데이터 로드를 위해 검색 매개변수가 필요한 경우(페이지 매김, DB 필터링 등) → `searchParams` prop
  - 검색 매개변수가 클라이언트에서만 사용되는 경우(이미 로딩된 목록 필터링 등) → `useSearchParams`
  - 콜백이나 이벤트 핸들러에서 리랜더링 없이 읽고 싶은 경우 → `new URLSearchParams(window.location.search)`
- **params vs searchParams**: `params`는 동적 세그먼트(`[slug]`)에서 가져오는 값으로 URL의 path 부분 데이터, `searchParams`는 query string에서 가져오는 값으로 URL의 `?` 이후 key=value 데이터임.
- `searchParams`란 URL의 쿼리 문자열(Query String)을 읽는 방법임. 예: `/products?category=shoes&page=2` → `category=shoes`, `page=2`가 search parameters. `searchParams`는 컴포넌트의 props로 전달되며, 내부적으로는 `URLSearchParams`처럼 작동함.
- **정적 렌더링 vs 동적 렌더링 비교**
| 항목 | 정적 렌더링 (Static) | 동적 렌더링 (Dynamic) |
|---|---|---|
| 예시 | `/about`, `/blog` (미리 생성됨) | `/products?page=2` (요청 시 생성) |
| 장점 | 빠름, 캐시 가능 | 유연함, 쿼리나 요청 기반 응답 가능 |
| searchParams 사용 | 불가능 | 가능 |
 
- **실습**(`app/products/page.tsx`)
```tsx
export default async function ProductsPage({
  searchParams,
}: {
  searchParams: Promise<{ id?: string; name?: string }>
}) {
  const { id = "non id", name = "non name" } = await searchParams
  return (
    <div>
      <h1>Products Page</h1>
      <p>id : {id}</p>
      <p>name : {name}</p>
    </div>
  )
}
```
  - 6번 라인: `await searchParams`로 Promise가 끝나면 실제 값(객체)이 반환되고, 비구조화 할당으로 속성을 꺼내 옴. `{ id = "non id", name = "non name" }`처럼 왼쪽에 초기값을 지정해 두면 값이 없을 때 기본값이 사용됨.
  - 브라우저 확인: `/products` 접속 시 `id: non id`, `name: non name`(기본값)이 출력되고, `/products?id=foo&name=bar` 접속 시 `id: foo`, `name: bar`가 출력됨. 없는 속성(`&etc=1`)을 추가해도 오류는 없지만 의미는 없으며, 속성은 필요한 만큼 사용 가능함.
### 7. Linking between pages (페이지 간 연결) & Route 방식 비교
- `<Link>`는 Next.js에서 경로를 탐색하는 기본 방법이며, 보다 고급 탐색을 위해 `useRouter` Hook을 사용할 수도 있음.
- **React vs Next.js 라우팅 방식 차이**
| 항목 | React (기본) | Next.js |
|---|---|---|
| 라우팅 방식 | 수동 (사용자가 직접 설정) | 자동 (폴더/파일 기반) |
| 라우터 도구 | `react-router-dom` 같은 외부 라이브러리 필요 | 자체 내장된 파일 기반 라우팅 시스템 |
| 라우트 정의 방식 | 코드에서 직접 `<Route>`로 정의 | 파일/폴더 이름으로 라우트가 자동 매핑됨 |
| 예시 | `<Route path="/about" element={<About />} />` | `pages/about.js` 또는 `app/about/page.tsx` → `/about` 경로 자동 생성 |
 
- React는 기본적으로 라우팅 기능이 없기 때문에 직접 라우터 라이브러리를 설치해서 라우팅을 설정해야 하지만, Next.js는 자체적으로 라우팅 시스템을 내장하고 있음.


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
