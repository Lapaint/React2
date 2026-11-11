# 김민우 (202230305) - React2

## [2026-11-11] 11주차: Server and Client Components (이어서)

### 0. 오늘 한 일

- 지난주에 실습한 Context provider(`ThemeProvider`, `ThemeStatus`) 코드를 **한 줄씩 다시 설명**하며 복습함. (유니온 타입, `useEffect` 패턴, 속성 선택자 등)
- Context provider의 **실행 과정 리뷰, 동작 순서, 순서도 형식 정리**를 학습함.
- 3-6 외부(서드 파티) component의 이론을 학습하고, `acme-carousel`을 사용한 `Gallery` 실습을 진행함.

---

### 1. Context provider 실습 코드 설명 (복습)

#### 1-1. Context 생성 (`ThemeProvider.tsx`, client component)

- `createContext(...)`로 Context 객체를 생성하여 theme state를 공유함.
- `<{...}>` 부분은 **타입** 부분임.
  - `theme`: `'light'` 또는 `'dark'` 중 하나임.
  - `toggleTheme`: 아무 인자도 받지 않고, 반환값도 없는 함수라는 의미임.
- 초기값(default value)은 **provider가 없을 때 사용할 fallback value**임.
  - React에서는 `createContext()`를 호출할 때 기본값이 반드시 있어야 함.
  - 보통은 실제 동작하지 않는 빈 함수(`() => {}`)를 기본으로 넣어 둠.
  - 실제 동작은 `ThemeProvider` 컴포넌트에서 설정함.

```tsx
export const ThemeContext = createContext<{
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}>({
  theme: 'light',
  toggleTheme: () => {},
});
```

#### 1-2. `useState`와 유니온 타입

- `useState<'light' | 'dark'>('light')`에서 `<'light' | 'dark'>`는 TypeScript의 **유니온 타입(Union Type)** 이며, 초기값은 `'light'`라는 의미임.

```tsx
const [theme, setTheme] = useState<'light' | 'dark'>('light');
```

**TypeScript의 유니온 타입(Union Type)이란?**

- `|`(파이프)로 여러 타입을 연결해서 "이 값은 각각의 타입 중 하나가 될 수 있다"는 것을 지정함.
- 문자열 리터럴 유니온 타입의 경우 state 값으로 `'light'` 또는 `'dark'`만 설정할 수 있어, **코드 자동완성과 타입 안정성이 향상**됨.

#### 1-3. `useEffect`로 테마를 HTML 문서 전체에 적용하기

- 아주 전형적인 패턴이며, `useEffect` Hook은 컴포넌트가 렌더링된 **후** 부수 효과(side effect)를 실행하기 위한 함수임.

```tsx
useEffect(() => {
  if (typeof window !== 'undefined') {
    document.documentElement.dataset.theme = theme;
  }
}, [theme]);
```

**① if문의 조건절: "현재 실행 환경이 브라우저인지 확인"**

- 서버 사이드 렌더링(SSR) 단계에서는 `window` 객체가 없음.
- 만약 서버에서 `window`를 참조하면 오류가 발생함. (`ReferenceError: window is not defined`)
- 따라서 `typeof window !== 'undefined'`는 "클라이언트(브라우저) 환경일 때만 실행하라"는 뜻임.

**② `document.documentElement.dataset.theme = theme`의 의미**

- `document.documentElement`는 HTML 문서의 `<html>` 요소를 가리킴.
- `.dataset.theme = theme`은 `<html>` 태그에 `data-theme` 속성을 추가하는 코드임.
- 예를 들어 theme state 값이 `"dark"`라면 `<html data-theme="dark">`가 됨.

**③ 두 번째 인자 `[theme]` = 의존성 배열(dependency array)**

- `theme` 값이 변경될 때마다 `useEffect` 안의 코드가 **다시 실행**됨.
- 즉, 테마가 바뀔 때마다 HTML의 `data-theme` 속성도 업데이트됨.

#### 1-4. CSS 속성 선택자(Attribute Selector)

- 위 방법을 사용하면 CSS에서 **속성을 조건으로** 스타일을 다르게 지정할 수 있음.

```css
html[data-theme='light'] {
  background-color: white;
  color: black;
}

html[data-theme='dark'] {
  background-color: black;
  color: white;
}
```

- `html[data-theme='light']`는 **속성 선택자**로, CSS에서 클래스(`.class`)나 아이디(`#id`)처럼 요소를 선택하는 또 다른 방법임.
- 속성 선택자는 class를 여러 개 붙이는 경우보다 **스타일 충돌을 줄일 수 있음**.

#### 1-5. `toggleTheme`과 `ThemeContext.Provider`

```tsx
const toggleTheme = () => { setTheme((t) => (t === 'dark' ? 'light' : 'dark')) }
return (
  <ThemeContext.Provider value={{ theme, toggleTheme }}>
    {children}
  </ThemeContext.Provider>
)
```

- theme state를 **삼항 연산자**로 토글하여 `setTheme` 함수를 이용해 `toggleTheme`에 저장함.
- `ThemeContext.Provider`는 무엇일까?
  - `createContext` 함수를 호출하면 React가 Context 객체 하나를 만들어 줌.
  - 이 객체 안에는 여러 속성이 있는데, 대표적인 것이 `ThemeContext.Provider`, `ThemeContext.Consumer`임.
  - 즉, **Provider는 `createContext()`를 호출하면 자동으로 생성되는 React 컴포넌트**임.
- 따라서 `ThemeContext.Provider` 컴포넌트에 현재 theme state와 함께 `toggleTheme` 함수도 함께 props(`value`)로 전달함.
  - 하위 컴포넌트에서는 현재 theme state를 알 수 없기 때문에, 버튼 쪽으로 `toggleTheme` 함수와 theme state를 함께 전달하는 것임.

#### 1-6. 테마 토글 버튼 (`ThemeStatus.tsx`, client component)

- `ThemeContext`를 사용하기 위해 `ThemeProvider` 파일에서 import함.
- `useContext` 함수를 이용해서 `ThemeContext`에서 전달받은 `theme`와 `toggleTheme`를 추출함.
- 클릭 이벤트가 발생하면 추출된 `toggleTheme` 함수가 실행되고, 버튼 내의 삼항 연산자를 사용하여 버튼의 모양을 교체함.

```tsx
const { theme, toggleTheme } = useContext(ThemeContext)
// ...
<button onClick={toggleTheme} aria-label="toggle theme">
  Theme: {theme === 'dark' ? '🌑 Dark' : '☀️ Light'}
</button>
```

- `aria-label` 속성: 텍스트가 없거나 불분명할 때, 화면 낭독기 등 보조 기술이 버튼의 역할을 설명하도록 **접근성 이름**을 제공함.
- `aria`: Accessible Rich Internet Applications(접근 가능한 풍부한 인터넷 애플리케이션)의 약자임.

#### 1-7. RootLayout 수정 (server component)

- `ThemeProvider`와 `ThemeStatus`(버튼) 컴포넌트 사용을 위해 import를 추가함.
- `ThemeStatus`는 출력을 원하는 곳에 삽입함. (`<header>` 안)
- `ThemeProvider`는 `<header>`, `<main>`, `<footer>`를 감싸 줌.

**`{children}`만 감싸면 최적화된다는 문서 설명은 우리의 경우에 동작하지 않음**

- 문서에서는 `{children}`을 감싸면 최적화할 수 있다고 되어 있지만, 우리의 경우에는 동작하지 않음.
- 왜냐하면 우리는 **CSS의 속성 선택자로 `html`을 사용**했기 때문임. (`html[data-theme=...]`)

**그런데 `<body>`를 감싸도, `<html>`을 감싸도 동작하는 이유는?**

- `RootLayout` 컴포넌트의 `return` 내부를 보면 일반 HTML처럼 보이지만, 실제로 Next.js에서는 `<html>`과 `<body>`도 **React의 JSX 엘리먼트로 렌더링**됨.
- 즉, 렌더링 트리 상으로는 다르게 보여도, `useEffect`에서 **직접 DOM을 조작**하기 때문에 결과적으로 똑같이 보이는 것임.
- 코드가 완성되면 브라우저의 개발자 도구로 테마가 변할 때의 엘리먼트(`<html data-theme="...">`) 상태를 관찰해 볼 것.

---

### 2. Context provider의 실행 과정 리뷰

**① Context 생성 (`ThemeProvider.tsx`)**

- `createContext(...)`로 Context 객체를 만듦. 초기값(default value)은 provider가 없을 때 사용할 fallback값임. (여기서는 `theme: 'light'`, `toggleTheme: () => {}`)
- 이 파일 내부에서 `ThemeProvider` 컴포넌트를 정의함.
  - `useState`로 theme 상태를 관리함. (예: `'light' | 'dark'`)
  - `toggleTheme` 함수는 `setTheme`을 호출해 상태를 변경함.
  - `useEffect`로 상태 변경 시 `document.documentElement.dataset.theme`에 값을 기록함. (전역 스타일 적용 용도)
  - `ThemeContext.Provider`에 `value={{ theme, toggleTheme }}`를 넣고 `children`을 감싸 줌.
- **[요약]** 상태(state)와 상태를 바꾸는 함수(toggle)를 만든 뒤 Provider의 `value`로 내려 줌. 상태는 Provider 내부에 보관됨.

**② Provider 배치 (`RootLayout`)**

- `RootLayout`에서 `ThemeProvider`로 루트(또는 필요한 하위 트리)를 감싸 줌.
- 이렇게 하면 Provider 하위에 렌더링되는 모든 컴포넌트들이 `ThemeContext`에 접근할 수 있음.
- `children`은 `RootLayout`으로 전달된 자식 컴포넌트들을 의미하고, Provider가 그들을 감싸므로 자식들이 Context에 접근 가능해짐.

**③ Consumer 사용 (`ThemeStatus.tsx`)**

- `ThemeStatus`는 `'use client'`로 클라이언트 컴포넌트이며, `useContext(ThemeContext)`를 사용해 `value`를 읽어 들임.
- UI에서는 theme 값을 표시하고, 버튼 클릭 시 `toggleTheme()`을 호출함.

---

### 3. Context provider의 동작 순서 (버튼 클릭 시 흐름)

1. 사용자가 `ThemeStatus`의 버튼을 클릭함.
2. `toggleTheme()` 호출. (`ThemeStatus`가 Provider의 함수를 호출)
3. Provider 내부의 `setTheme`이 실행되어 theme 상태가 변경됨.
4. 상태 변경으로 Provider와 그 하위 컴포넌트들이 **리렌더링**되어 theme 값이 최신으로 반영됨.
5. `useEffect`가 실행되어 `document.documentElement.dataset.theme` 값도 갱신됨. (글로벌 스타일 반영)

---

### 4. Context provider 순서도 형식으로 정리

- Theme Context의 동작 흐름을 순서도 형식으로 정리한 것이며, 각 단계에 대응하는 파일/위치를 괄호 안에 표기함.

| 단계 | 내용 | 파일/위치 |
| --- | --- | --- |
| 1. 앱 시작 / RootLayout 렌더 | `RootLayout`이 렌더되고 `ThemeProvider`로 `children`을 감쌈 | `layout.tsx` → `ThemeProvider` 사용 |
| 2. Context 생성(초기화) | `ThemeContext = createContext(...)`가 정의되어 있음 (기본값 제공) | `src/components/ThemeProvider.tsx` |
| 3. Provider 인스턴스 생성 | `ThemeProvider`가 실행되어 내부 state(`theme`, `setTheme`) 생성(`useState`), Provider의 `value = { theme, toggleTheme }`로 설정 | `src/components/ThemeProvider.tsx` |
| 4. 하위 트리 렌더링 | Provider로 감싼 `children`(페이지/컴포넌트)이 렌더됨. 이 하위 트리는 Context에 접근 가능 | `layout.tsx` → `children` |
| 5. Consumer 사용: `ThemeStatus` 렌더 | `useContext(ThemeContext)`로 `{ theme, toggleTheme }`를 가져옴 | `src/components/ThemeStatus.tsx` |
| 6. 사용자 상호작용: 버튼 클릭 | 버튼을 클릭하면 `toggleTheme()` 호출 | `ThemeStatus.tsx` 클릭 → `ThemeProvider`의 `toggleTheme` 실행 |
| 7. 상태 변경 내부 처리 | `toggleTheme`가 `setTheme`을 호출하여 theme 상태를 변경 (예: `'light'` → `'dark'`) | `ThemeProvider.tsx` |
| 8. 부수효과 실행(`useEffect`) | theme 변경으로 `useEffect`가 실행되어 `document.documentElement.dataset.theme = theme`로 설정 | `ThemeProvider.tsx`의 `useEffect` |
| 9. 리렌더링 전파 | state 변경으로 Provider와 그 하위 Consumer들이 최신 `value`를 받아 리렌더링됨. `ThemeStatus`는 새 theme 값을 받아 UI(아이콘/텍스트)를 갱신 | - |
| 10. 결과: 레이아웃/스타일 반영 | `data-theme` 속성 변경을 바탕으로 CSS(`[data-theme="dark"]` 등)가 적용되면 실제 시각적 테마 변경이 화면에 반영됨 | 전역 CSS(`globals.css`) |

**Flowchart**

```text
RootLayout 렌더
      ↓
ThemeProvider 생성 (useState: theme)
      ↓
Provider value 제공 -> children 렌더
      ↓
ThemeStatus(useContext) 읽음
      ↓
사용자 클릭 -> toggleTheme() 호출
      ↓
setTheme(newTheme) 실행 (state 변경)
      ↓
useEffect 실행 -> document.dataset.theme 업데이트
      ↓
Provider & Consumer 리렌더 -> UI 갱신
```

---

### 5. 3-6 외부(서드 파티) component

> 이 절은 **이론 위주**로 학습함. (실습은 6장 참고)

#### 5-1. 문서 내용

- client 전용 기능에 의존하는 외부 component를 사용하는 경우, 해당 component를 **client component에 래핑**하여 예상대로 작동하는지 확인할 수 있음.
- 예를 들어 `<Carousel />`은 `acme-carousel` 패키지에서 가져올 수 있음.
- 이 component는 `useState`를 사용하지만 `"use client"` 지시문은 없음.
- 그렇다면 `"use client"` 지시문 없이 어떻게 사용할 수 있을까?
  - **client component 내에서** `<Carousel />`을 사용하면 예상대로 작동함.

```tsx
// app/gallery.tsx
'use client'

import { useState } from 'react'
import { Carousel } from 'acme-carousel'

export default function Gallery() {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>View pictures</button>
      {/* Works, since Carousel is used within a Client Component */}
      {isOpen && <Carousel />}
    </div>
  )
}
```

- 그러나 **server component 내에서 직접 사용하려고 하면 오류가 발생**함.
  - Next.js가 `<Carousel />`이 client 전용 기능을 사용하고 있다는 것을 알지 못하기 때문임.
- 이 문제를 해결하려면 client 전용 기능에 의존하는 외부 component를 **자체 client component로 래핑**할 수 있음.

```tsx
// app/carousel.tsx
'use client'

import { Carousel } from 'acme-carousel'

export default Carousel
```

- 이제 server component 내에서 `<Carousel />`을 직접 사용할 수 있음.

```tsx
// app/page.tsx (Server Component)
import Carousel from './carousel'

export default function Page() {
  return (
    <div>
      <p>View pictures</p>
      {/* Works, since Carousel is a Client Component */}
      <Carousel />
    </div>
  )
}
```

| 사용 위치 | `"use client"` 없는 외부 component 직접 사용 | 해결 방법 |
| --- | --- | --- |
| client component 내부 | 가능함 | - |
| server component 내부 | **오류 발생** | `'use client'`를 선언한 파일에서 re-export하여 래핑 |

---

### 6. 외부(서드 파티) component 실습

#### 6-1. 문서대로 `gallery.tsx`를 작성하면 발생하는 오류

**① 모듈 미설치 오류**

```text
Cannot find module 'acme-carousel' or its corresponding type declarations. ts(2307)
```

- 먼저 모듈을 설치하면 오류를 해결할 수 있음.

**② 모듈 설치 후 `<Carousel />` 사용 시 오류**

```text
Property 'items' is missing in type '{}' but required in type 'CarouselProps'. ts(2741)
```

- `Carousel` 컴포넌트의 타입 정의(`CarouselProps`)에 **필수 prop인 `items`가 정의**되어 있어서, `<Carousel />`를 props 없이 렌더링해서 발생하는 타입 에러임.
- `items`에서 사용할 이미지는 `https://picsum.photos/`의 더미를 사용함.

#### 6-2. 문서의 코드를 수정한 `gallery.tsx`

- 문서의 코드를 다음과 같이 수정함. (`items` 배열을 추가하고 `<Carousel items={items} />`로 전달)

```tsx
// src/components/gallery.tsx
'use client'

import { Carousel } from 'acme-carousel'

export default function Gallery() {
  // 예제용 이미지 목록 (public/images에 파일을 두거나 외부 URL 사용)
  const items = [
    { id: '1', src: 'https://picsum.photos/id/1015/800/600', alt: 'Landscape 1' },
    { id: '2', src: 'https://picsum.photos/id/1016/800/600', alt: 'Landscape 2' },
    { id: '3', src: 'https://picsum.photos/id/1018/800/600', alt: 'Landscape 3' },
  ]

  return (
    <Carousel
      items={items}
    />
  )
}
```

#### 6-3. 스타일 적용 문제

- 오류 수정 후에도 동작은 하지만 **이미지가 첫 페이지에 모두 출력**되어 정상 동작이라고 할 수는 없음.
- 이유는 `acme-carousel`에서 제공하는 **style이 적용되지 않아서**임.
- style은 `node_modules/acme-carousel/dist/styles.css` 경로에 있지만, 이렇게 특정 모듈에 있는 스타일을 사용할 경우 `global.css`에 import해서 사용하는 것이 일반적임.

```css
@import 'acme-carousel/dist/styles.css';
```

- 하지만 이번 경우에는 `acme-carousel`의 특성 때문에 **오류가 발생**함.
- 이런 경우라면 스타일을 `components/`에 **복사해서** 사용함. (위치는 다른 곳이라도 상관 없음)
  - `gallery.tsx`에 `import './styles.css'`를 추가함.

> Carousel: 웹사이트나 앱에서 여러 개의 이미지, 비디오, 텍스트 등의 콘텐츠를 일정한 간격으로 순서대로 보여주는 UI 요소임.

#### 6-4. `acme-carousel`의 주요 옵션 및 기능

- 자동 전환(autoplay)
- 반응형(responsive)
- 지원 터치/스와이프 제어(touch/swipe)
- 가상화(virtualization) 및 지연 로딩(lazy loading)
- 접근성(accessibility) 기능
- 고급 애니메이션 및 3D 효과 등
- 자세한 설명은 https://www.npmjs.com/package/acme-carousel 에서 확인할 수 있음.

#### 6-5. Autoplay Carousel 적용

- 먼저 테스트를 위해서 `RootLayout`에 carousel 메뉴를 추가함.
- `gallery.tsx` 수정: 좌우 버튼을 누르지 않아도 슬라이드가 **자동으로 동작**하도록 `Slide` 컴포넌트를 추가함.
- 그런데 `Slide`는 named export가 아니라 **default export**이기 때문에 `{ Carousel, Slide }`처럼 import하면 오류가 발생함.

```text
Module '"acme-carousel"' has no exported member 'Slide'. Did you mean to use 'import Slide from "acme-carousel"' instead? ts(2614)
```

- 따라서 default export로 import함.

```tsx
import { useState } from 'react'
import { Carousel } from 'acme-carousel'
import Slide from 'acme-carousel'
```

- 사용하던 line 7의 state 구문(`useState`)을 삭제함.

<!-- TODO: 이후 Slide 적용 코드와 carousel 페이지 실습 내용은 스크린샷에 없어 작성하지 않음. 실습을 마치면 추가할 것 -->

---

### 7. 오늘 최종 폴더 구조

<!-- TODO: gallery.tsx, carousel 페이지는 실제 파일 생성 여부 확인 후 맞게 수정 -->

```text
src/
├─ app/                        # 라우팅 페이지만 관리
│  ├─ layout.tsx               # root layout (carousel 메뉴 추가)
│  ├─ globals.css              # data-theme 기반 light/dark 스타일
│  ├─ [id]/page.tsx            # slug page (server component)
│  ├─ counter/page.tsx         # counter page (server component)
│  ├─ interleaved/page.tsx     # interleaving 예제 (server component)
│  └─ carousel/page.tsx        # carousel 예제 (server component)
├─ components/
│  ├─ Counter.tsx              # client component
│  ├─ ClientLayout.tsx         # client component (children slot)
│  ├─ ServerContent.tsx        # server component (fetch)
│  ├─ ThemeProvider.tsx        # client component (Context 생성 + Provider)
│  ├─ ThemeStatus.tsx          # client component (useContext + toggle 버튼)
│  ├─ gallery.tsx              # client component (acme-carousel 래핑)
│  └─ styles.css               # acme-carousel 스타일 복사본
├─ ui/
│  └─ LikeButton.tsx           # client component
└─ lib/
   └─ data.tsx                 # 더미 데이터(posts)
```

---


## [2026-11-04] 10주차: Server and Client Components (이어서)

### 0. 오늘 한 일

- 3-3 server에서 client component로 데이터 전달의 나머지(`use` Hook, 직렬화)를 학습함.
- 3-4 server 및 client component 인터리빙(interleaving) 실습(`ClientLayout`, `ServerContent`, `interleaved` 페이지)을 완료함.
- Context의 개념과 Next.js에서 사용할 때의 고려사항을 학습함.
- 3-5 Context provider 실습(`ThemeProvider`, `ThemeStatus`로 light/dark 테마 토글)을 완료함.

---

### 1. 3-3 server에서 client component로 데이터 전달 (이어서)

- 다른 방법으로는 `use` Hook을 사용해 server component에서 client component로 데이터를 **스트리밍**할 수도 있음. (Fetching Data 장에서 소개함)
- client component에 전달되는 props는 React로 **직렬화(serialization)가 가능**해야 함.

**직렬화(serialization)란?**

- 메모리에 있는 복잡한 데이터를 **바이트의 연속 형태로 변환**하는 과정임.
- 자바스크립트의 객체나 배열처럼 구조가 있는 데이터를 파일로 저장하거나 네트워크로 전송하기 쉽게 만드는 과정임.
- React나 Next.js 같은 프레임워크는 컴포넌트의 상태나 트리 구조를 **서버에서 직렬화**하여 클라이언트로 전송하고, **클라이언트에서 역직렬화**하는 과정을 자주 수행함.

---

### 2. 3-4 server 및 client component 인터리빙

#### 2-1. 개념

- 인터리빙(Interleaving)은 여러 데이터 블록이나 비트를 섞어서 전송/처리하여 오류 발생 시 영향을 최소화하는 기술임.
- 문서에서는 server component와 client component가 **섞여서(interleaved)** 동작할 때 자주 등장하는 패턴을 말함.
- server component의 props를 client component에 전달할 수 있고, 이를 통해 client component 안에서 server에서 렌더링된 UI를 시각적으로 중첩할 수 있음.
- `<ClientComponent>`에 **공간(slot)** 을 만들고 `children`을 끼워 넣는 패턴이 일반적임.

```tsx
// app/ui/modal.tsx (문서 예시)
'use client'

export default function Modal({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>
}
```

- 예를 들어 client의 state로 표시 여부를 toggle하는 `<Modal>` 안에, server에서 데이터를 가져오는 `<Cart>`가 있는 경우임.
- 부모 server component(`<Page>`)가 `<Modal>`의 자식으로 `<Cart>`를 전달함. (Modal을 불러오는 곳이 Page이므로 Page가 parent가 됨)

```tsx
// app/page.tsx (문서 예시)
import Modal from './ui/modal'
import Cart from './ui/cart'

export default function Page() {
  return (
    <Modal>
      <Cart />
    </Modal>
  )
}
```

- 이 패턴에서는 모든 server component(props 포함)가 server에서 **미리 렌더링**됨.
- 생성된 RSC 페이로드에는 component 트리 안에서 client component가 렌더링되어야 하는 위치에 대한 **참조**가 포함됨.

**실습 전 정리**

- 클라이언트 컴포넌트가 **껍데기** 역할을 하고, 서버 컴포넌트가 그 안의 내용(`children`)으로 들어오는 구조임.
- Next.js 기본 동작
  - Server Component → 서버에서 렌더링됨. (데이터 패칭 가능)
  - Client Component → 브라우저에서 렌더링됨. (상호작용 가능)
- 즉 **서버 컴포넌트 안에 클라이언트 컴포넌트를 넣을 수는 있지만, 반대는 직접적으로는 불가능**함.
- 그래서 클라이언트 컴포넌트 안에 만든 `children` 슬롯에 서버 컴포넌트를 **끼워 넣는** 방식으로 둘을 섞어 쓰자는 아이디어가 interleaving임.

#### 2-2. 실습

**[실습1] ClientLayout 컴포넌트 (client)**

- `children`을 받는 껍데기 역할이며, `{children}` 자리가 slot임.

```tsx
// src/components/ClientLayout.tsx (Client Component)
'use client';

export default function ClientLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="layout">
      <h1>클라이언트 레이아웃</h1>
      <div className="content">
        {children} {/* <-- 이 부분이 "slot" */}
      </div>
    </div>
  );
}
```

**[실습2] ServerContent 컴포넌트 (server)**

- `'use client'`가 없으므로 server component이고, `async` 함수라서 컴포넌트 안에서 바로 `await fetch`를 사용할 수 있음.
- 데이터는 `jsonplaceholder.typicode.com`(테스트용 가짜 API 사이트)에서 가져옴.
- 슬라이드는 `https://`를 사용하지만 우리 코드는 `http://`로 작성되어 있음.

```tsx
// src/components/ServerContent.tsx (Server Component)
export default async function ServerContent() {
  const data = await fetch('http://jsonplaceholder.typicode.com/posts/1').then((r) => r.json());
  return <p>서버에서 가져온 제목: {data.title}</p>;
}
```

**[실습3] interleaved 라우팅 페이지 (server)**

- 일반적인 라우팅 페이지와 동일하며, 작성한 두 컴포넌트를 섞어서 렌더링함.
- `/interleaved` 주소로 접근함. (현재 `layout.tsx`의 nav에는 이 페이지의 링크를 추가하지 않았고, 주소창에 직접 입력해서 확인함)

```tsx
// src/app/interleaved/page.tsx (Server Component)
import ClientLayout from '@/components/ClientLayout';
import ServerContent from '@/components/ServerContent';

export default function Page() {
  return (
    <ClientLayout>
      <ServerContent />
    </ClientLayout>
  );
}
```

**동작 과정**

1. Next.js는 먼저 `ServerContent`를 서버에서 렌더링 → HTML로 변환함.
2. 이 HTML을 `ClientLayout`의 `{children}` 자리에 **끼워 넣음**.
3. 그 다음 클라이언트에서는 `ClientLayout`만 hydration(JS 연결)함.
4. 결국 서버 데이터는 이미 들어와 있고, 버튼이나 이벤트 등은 클라이언트 컴포넌트에서 처리가 가능해짐.
   → 이렇게 둘이 섞여(interleaved) 있는 패턴이 되는 것임.

---

### 3. Context란 무엇인가?

- Next.js에서 Context는 **React의 Context API**를 사용하여 컴포넌트 사이에 데이터를 공유하는 메커니즘임.
- 부모 컴포넌트에서 자식 컴포넌트로 직접 props를 전달하지 않고도, 특정 데이터를 필요한 컴포넌트에서 쉽게 접근하고 사용할 수 있게 해 줌.

#### 3-1. Context의 주요 특징

| 특징 | 설명 |
| --- | --- |
| 전역 상태 관리 | 애플리케이션 전체에서 공유해야 하는 데이터를 중앙 집중적으로 관리함. (예: 사용자 정보, 테마 설정 등) |
| props drilling 문제 해결 | 컴포넌트 트리가 깊어질수록 props를 계속 전달해야 하는 번거로움을 줄여 주고, 코드의 가독성과 유지 보수성을 높임 |
| React 컴포넌트에서 사용 | Context는 React에서 제공하는 기능이므로 Next.js에서도 React 컴포넌트를 사용하여 구현함 |

#### 3-2. Next.js에서 Context 사용 시 고려사항

| 구분 | 내용 |
| --- | --- |
| 클라이언트 컴포넌트 | Next.js 13부터 client component 안에서 Context를 자유롭게 사용할 수 있음 |
| 서버 컴포넌트 | server component에서는 직접 Context를 생성하거나 사용할 수 없고, client component로 데이터를 전달하여 **간접적으로** 사용함 |
| 성능 | Context를 과도하게 사용하면 렌더링 성능에 영향을 줄 수 있으므로 필요한 경우에만 신중하게 사용해야 함 |

#### 3-3. 설명을 위한 예시 (React)

- `MyContext`는 Context 객체를 나타내고, `MyContext.Provider`는 `MyComponent`에 데이터를 제공함.
- `useContext(MyContext)`를 통해 `MyComponent`는 `"Hello from Context"` 값을 가져와서 렌더링함.

```tsx
// Context 생성
const MyContext = React.createContext();

function MyComponent() {
  const value = useContext(MyContext);
  return <div>{value}</div>;
}

function App() {
  return (
    <MyContext.Provider value="Hello from Context">
      <MyComponent />
    </MyContext.Provider>
  );
}
```

- 추가 정보: React Context API 공식 문서, Next.js 공식 문서를 참고함. Next.js에서도 제공하지만 React에서 제공하는 기능이므로 React 문서에 더 자세한 설명이 있음.

---

### 4. 3-5 Context provider (컨텍스트 제공자)

#### 4-1. 문서 내용

- React Context는 일반적으로 테마처럼 **전역 상태를 공유**하는 데 사용됨.
- 그러나 **server component에서는 React Context가 지원되지 않음.**
- 따라서 Context를 사용하려면 `children`을 허용하는 **client component**로 만들어야 함.

```tsx
// app/theme-provider.tsx (문서 예시)
'use client'

import { createContext } from 'react'

export const ThemeContext = createContext({})

export default function ThemeProvider({
  children,
}: {
  children: React.ReactNode
}) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
}
```

- 문서 예시는 오류는 없으나 육안으로 변화를 느낄 수 없기 때문에, 별도의 예제로 확인함.

**알아두면 좋은 정보**

- Provider component는 트리에서 **가능한 한 깊숙이** 렌더링해야 함.
- `ThemeProvider`가 전체 `<html>` 문서 대신 `{children}`만 래핑하는 방식에 주목함.
- 이렇게 하면 Next.js가 server component의 정적 부분을 더 쉽게 최적화할 수 있음.

#### 4-2. 실습: 테마(light/dark) 토글

**변경 사항 요약**

- `ThemeProvider`가 theme state와 toggle 함수를 제공하고, `document`에 `data-theme`을 설정함. (`ThemeProvider.tsx`)
- `ThemeStatus`에 toggle 버튼을 구현함. (새로 추가, `ThemeStatus.tsx`)
- `RootLayout`이 `ThemeProvider`로 body 안쪽을 감싸고, `header`에 `ThemeStatus`(client component)를 추가함. (`layout.tsx`)
- `globals.css`에 `data-theme` 값에 따른 light/dark 스타일을 작성함.



**[실습1] ThemeProvider (Context 생성 + Provider)**

```tsx
// src/components/ThemeProvider.tsx
'use client';

import { createContext, useEffect, useState } from 'react';

export const ThemeContext = createContext<{
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}>({
  theme: 'light',
  toggleTheme: () => {},
});

export default function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  useEffect(() => {
    if (typeof window !== 'undefined') {
      document.documentElement.dataset.theme = theme;
    }
  }, [theme]);

  const toggleTheme = () => {
    setTheme((t) => (t === 'dark' ? 'light' : 'dark'));
  };
  return <ThemeContext.Provider value={{ theme, toggleTheme }}>{children}</ThemeContext.Provider>;
}
```

**[실습2] ThemeStatus (Context 사용 + toggle 버튼)**

- `useContext(ThemeContext)`로 `theme`과 `toggleTheme`을 가져와 버튼으로 표시함.

```tsx
// src/components/ThemeStatus.tsx
'use client';

import { useContext } from 'react';
import { ThemeContext } from './ThemeProvider';

export default function ThemeStatus() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  return (
    <div style={{ display: 'inline-block', marginLeft: 12 }}>
      <button onClick={toggleTheme} aria-label="toggle theme">
        Theme: {theme === 'dark' ? '🌑 Dark' : '☀️ Light'}
      </button>
    </div>
  );
}
```

**[실습3] RootLayout 수정**

- `ThemeProvider`로 `<header>`, `<main>`, `<footer>`를 감싸고, `<header>` 안에 `<ThemeStatus />`를 추가함.
- `layout.tsx` 자체는 server component를 유지하고, client component(`ThemeProvider`, `ThemeStatus`)만 가져다 씀. (3-2 JS bundle 줄이기 원칙)

```tsx
// src/app/layout.tsx
import Link from 'next/link';
import './globals.css';
import ThemeStatus from '@/components/ThemeStatus';
import ThemeProvider from '@/components/ThemeProvider';

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="ko">
      <body>
        <ThemeProvider>
          <header>
            <p>=== Root Layout Header ===</p>
            <nav>
              <Link href="/">Home</Link> | <Link href="/counter">Counter</Link>
              {/* ... Slug Page Menu 링크 ... */}
            </nav>
            <ThemeStatus />
          </header>
          <main>{children}</main>
          <footer>=== Root Layout Footer ===</footer>
        </ThemeProvider>
      </body>
    </html>
  );
}
```

**[실습4] globals.css**

- 기존 tailwind import와 기본 변수를 지우고, `html`의 `data-theme` 값에 따라 배경/글자색이 바뀌도록 작성함.
- `transition`으로 색 변화가 0.3초 동안 부드럽게 일어남.

```css
/* src/app/globals.css */
html[data-theme='light'] {
  background-color: white;
  color: black;
  transition: background-color 0.3s, color 0.3s;
}

html[data-theme='dark'] {
  background-color: #121212;
  color: white;
  transition: background-color 0.3s, color 0.3s;
}
```

#### 4-3. 코드 설명

**① Context 생성 (ThemeProvider.tsx, client component)**

- context를 사용하면 props를 사용하지 않고도 전역적으로 사용할 theme, 언어 설정, 로그인 정보 등을 하위 컴포넌트에 전달할 수 있음.
- `createContext()`는 React 컴포넌트 트리 전체에 값을 공유할 수 있도록 하는 역할을 함.
- `createContext<{...}>(...)`로 Context 객체를 생성하여 theme state를 공유함.
- `<{...}>` 부분은 **타입** 부분임.
  - `theme`: `'light'` 또는 `'dark'` 중 하나임.
  - `toggleTheme`: 아무 인자도 받지 않고, 반환값도 없는 함수라는 의미임.
- 초기값(default value)은 **provider가 없을 때 사용할 fallback value**임.
  - React에서는 `createContext()`를 호출할 때 기본값이 반드시 있어야 함.
  - 보통은 실제 동작하지 않는 빈 함수(`() => {}`)를 기본으로 넣어 둠.
  - 실제 동작은 `ThemeProvider` 컴포넌트에서 설정함. (`theme: 'light'`, `toggleTheme: () => {}`)

**② ThemeProvider 컴포넌트**

- `useState<'light' | 'dark'>('light')`로 현재 theme state를 만들고, 초기값은 `'light'`임.
- `useEffect`는 `theme`이 바뀔 때마다 `document.documentElement.dataset.theme = theme`을 실행해 `<html data-theme="...">` 속성을 바꿈. 이 속성을 `globals.css`의 선택자(`html[data-theme='dark']`)가 읽어서 색이 바뀜.
  - `typeof window !== 'undefined'`는 브라우저 환경인지 확인하는 안전장치임. (`document`는 브라우저에만 있음)
- `toggleTheme`은 `setTheme`에 함수형 업데이트를 사용해 `dark`이면 `light`로, 아니면 `dark`로 바꿈.
- `ThemeContext.Provider value={{ theme, toggleTheme }}`으로 하위 컴포넌트 전체에 theme 값과 toggle 함수를 제공함.

**③ ThemeStatus 컴포넌트**

- `useContext(ThemeContext)`로 Provider가 제공한 `theme`, `toggleTheme`을 **props 없이** 바로 꺼내 씀.
- 버튼을 클릭하면 `toggleTheme`이 실행되어 state가 바뀌고, 버튼 문구와 페이지 색이 함께 바뀜.
- `ThemeStatus`는 `useContext`와 `onClick`을 사용하므로 `'use client'`가 필요함.

**④ 동작 흐름**

1. 버튼 클릭 → `toggleTheme()` 실행 → `theme` state 변경
2. `theme`이 바뀌어 `ThemeProvider`가 리렌더링되고, `useEffect`가 실행됨
3. `<html>`의 `data-theme` 속성 변경 → `globals.css`의 light/dark 스타일이 적용됨
4. Context를 사용하는 `ThemeStatus`도 새 `theme` 값으로 리렌더링되어 버튼 문구가 바뀜

---

### 5. 오늘 최종 폴더 구조

```text
src/
├─ app/                        # 라우팅 페이지만 관리
│  ├─ layout.tsx               # root layout (ThemeProvider로 감쌈, ThemeStatus 추가)
│  ├─ globals.css              # data-theme 기반 light/dark 스타일
│  ├─ [id]/page.tsx            # slug page (server component)
│  ├─ counter/page.tsx         # counter page (server component)
│  └─ interleaved/page.tsx     # interleaving 예제 (server component)
├─ components/
│  ├─ Counter.tsx              # client component
│  ├─ ClientLayout.tsx         # client component (children slot)
│  ├─ ServerContent.tsx        # server component (fetch)
│  ├─ ThemeProvider.tsx        # client component (Context 생성 + Provider)
│  └─ ThemeStatus.tsx          # client component (useContext + toggle 버튼)
├─ ui/
│  └─ LikeButton.tsx           # client component
└─ lib/
   └─ data.tsx                 # 더미 데이터(posts)
```

---

## [2026-10-28] 9주차: Server and Client Components (이어서)

### 0. 오늘 한 일

- 3-1 client component 실습(Counter 컴포넌트, counter 페이지, layout 메뉴)을 완료함.
- 3-2 JS bundle 크기 줄이기, 3-3 server에서 client component로 데이터 전달을 학습함.
- 사용자 정의 component/library는 `src/app` 밖(`src/components`, `src/ui`, `src/lib`)에서 관리함.

---

### 1. 3-1 client component 사용 (실습)

- 파일 맨 위에 `'use client'`를 선언하면 해당 파일이 server와 client 모듈 트리의 **경계**가 됨.
- Counter 컴포넌트를 작성하는 이유는 **사용할 목적**이 있기 때문임. 대표적인 목적은 두 가지임.
  - 다른 component의 완성을 위해 사용함.
  - 라우팅 페이지의 렌더링을 위해 사용함.

**[실습1] Counter 컴포넌트**

- `src/components` 디렉토리를 만들고 `Counter` 컴포넌트를 작성함.
- `useState`를 사용하므로 `'use client'`가 필요함.

```tsx
// src/components/Counter.tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>{count} likes</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

**[실습2] counter 라우팅 페이지**

- 라우팅 페이지를 만들고 `Counter` 컴포넌트를 호출함.
- 이 page는 `'use client'`가 없으므로 **server component**이고, 그 안에서 client component인 `Counter`를 불러 쓰는 구조임.

```tsx
// src/app/counter/page.tsx
import Counter from '@/components/Counter';

export default function CounterPage() {
  return (
    <div>
      <h1>Counter Page</h1>
      <Counter />
    </div>
  );
}
```

**[실습3] layout에 링크 메뉴 추가**

- slug page(like-button)와 counter page를 **모든 라우팅 페이지에서** 확인하기 위해 메뉴를 작성함.
- 모든 페이지에서 공통으로 보여야 하므로 `src/app/layout.tsx`(root layout)의 `<header>` > `<nav>` 안에 `Link`로 작성함.

```tsx
// src/app/layout.tsx
import Link from 'next/link';

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="ko">
      <body>
        <header>
          <p>=== Root Layout Header ===</p>
          <nav>
            <Link href="/">Home</Link> | <Link href="/counter">Counter</Link>
            <br />
            <br />
            Slug Page Menu :&nbsp;
            <Link href="/nextjs">nextjs</Link>&nbsp;|&nbsp;
            <Link href="/routing">routing</Link>&nbsp;|&nbsp;
            <Link href="/ssr-ssg">ssr-ssg</Link>&nbsp;|&nbsp;
            <Link href="/dynamic-routes">dynamic-routes</Link>
            <br />
          </nav>
          <br />
          <br />
        </header>
        <main>{children}</main>
        <footer>=== Root Layout Footer ===</footer>
      </body>
    </html>
  );
}
```

---

### 2. 3-2 JS bundle 크기 줄이기

- client JavaScript 번들 크기를 줄이려면 UI의 큰 부분을 client component로 표시하는 대신, **특정 대화형 component에만** `'use client'`를 추가함.
- 예를 들어 `<Layout>`은 로고, 탐색 링크 같은 **정적 요소**가 대부분이지만 **대화형 검색창**이 포함되어 있음.
- `<Search />`는 대화형이므로 client component가 되어야 하지만, **나머지 layout은 server component로 유지**해야 함.

| component | 종류 | 이유 |
| --- | --- | --- |
| `<Search />` | client component | 사용자와의 상호작용이 바로 이루어질 가능성이 있음 |
| `<Logo />` | server component | 상대적으로 중요하지 않고, 이미지 등 용량이 클 수 있음 |
| `Layout` | server component (기본값) | 정적 요소가 대부분이므로 server로 유지함 |

```tsx
// app/layout.tsx (Layout은 기본적으로 Server Component)
import Search from './search'; // Client Component
import Logo from './logo'; // Server Component

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Logo />
        <Search />
      </nav>
      <main>{children}</main>
    </>
  );
}
```

```tsx
// app/ui/search.tsx
'use client';

export default function Search() {
  // ...
}
```

---

### 3. 3-3 server에서 client component로 데이터 전달

- **props**를 사용해 server component에서 client component로 데이터를 전달할 수 있음.
- 앞에서 작성한 `PostPage`(`src/app/[id]/page.tsx`, server component)가 client component인 `LikeButton`으로 `likes` props를 전달함.
- 슬라이드는 `getPost(params.id)`를 사용하지만, 우리 코드는 `posts.find()`로 `id`를 직접 비교하고 없으면 `notFound()`를 호출함.

```tsx
// src/app/[id]/page.tsx (server component)
import LikeButton from '@/ui/LikeButton';
import { posts } from '@/lib/data';
import { notFound } from 'next/navigation';

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const post = posts.find((p) => p.id === id);

  if (!post) {
    notFound();
  }

  return (
    <div>
      <main>
        <h1>{post.title}</h1>
        {/* ... */}
        <LikeButton likes={post.likes} />
      </main>
    </div>
  );
}
```

```tsx
// src/ui/LikeButton.tsx (client component)
'use client';

import { useState } from 'react';

export default function LikeButton({ likes }: { likes: number }) {
  // Optimistic Updates(낙관적 업데이트)
  // 서버에 저장하지 않고 단순히 UI 상에서 좋아요 수를 증가시키는 역할을 함.
  const [count, setCount] = useState<number>(likes ?? 0);
  const [isLiking, setIsLiking] = useState(false);

  const handleLike = async () => {
    setIsLiking(true);
    setCount((c) => c + 1);
    // 예제에서는 짧은 지연 후 버튼 상태만 해제함.
    setTimeout(() => setIsLiking(false), 300);
  };

  return (
    <button onClick={handleLike} disabled={isLiking} aria-pressed={false}>
      따봉 {count}
    </button>
  );
}
```

- 전달된 props는 RSC Payload에 포함되어 client로 넘어가고, client component가 이를 초기값(`useState(likes ?? 0)`)으로 사용함.

---

### 4. 오늘 최종 폴더 구조

```text
src/
├─ app/                    # 라우팅 페이지만 관리
│  ├─ layout.tsx           # root layout (링크 메뉴)
│  ├─ [id]/page.tsx        # slug page (server component)
│  └─ counter/page.tsx     # counter page (server component)
├─ components/
│  └─ Counter.tsx          # client component
├─ ui/
│  └─ LikeButton.tsx       # client component
└─ lib/
   └─ data.tsx             # 더미 데이터(posts)
```

---

## [2026-10-14] 7주차: Server and Client Components

### 0. Introduction

- 기본적으로 `layout`과 `page`는 **server component**임.
- server에서 데이터를 가져와 UI 일부를 렌더링하고, 선택적으로 결과를 cache한 뒤 client로 스트리밍할 수 있음.
- 상호작용이나 브라우저 API가 필요하면 **client component**를 사용해 기능을 계층화함.
- 이번 장에서는 server/client component가 작동하는 방식, 사용하는 시기, 적용 예제를 다룸.

---

### 1. server 및 client component를 언제 사용해야 하나요?

#### 1-1. 사용 기준

| client component를 사용하는 경우 | server component를 사용하는 경우 |
| --- | --- |
| state, event handler (`onClick`, `onChange`) | 서버의 DB 또는 API에서 data를 가져올 때 |
| Lifecycle logic (`useEffect`) | API key, token 등 보안 데이터를 client에 노출하지 않고 사용할 때 |
| 브라우저 전용 API (`localStorage`, `window`, `Navigator.geolocation` 등) | 브라우저로 전송되는 JavaScript 양을 줄이고 싶을 때 |
| 사용자 정의 Hook | FCP(First Contentful Paint)를 개선하고 콘텐츠를 점진적으로 스트리밍할 때 |

#### 1-2. 예시: Page(server) + LikeButton(client)

- `<Page>`는 게시물 데이터를 가져와 `<LikeButton>`에 props로 전달하는 **server component**임.
- `@/ui/LikeButton`은 상호작용(state)이 필요하므로 파일 맨 위에 `'use client'`를 선언함.

#### 1-3. 문서의 코드를 완성해 보기

**① data.tsx (더미 데이터)**
- 4장에서 사용한 더미 데이터를 사용하되, slug로 쓸 필드명을 `id`로 변경하고 `likes` 필드를 추가함.

```tsx
// src/lib/data.tsx
export const posts = [
  { id: "nextjs", title: "Next.js 소개", content: "Next.js는 React 기반의 풀스택 프레임워크입니다.", likes: 5 },
  { id: "routing", title: "App Router 알아보기", content: "Next.js 13부터는 App Router가 도입되었습니다.", likes: 3 },
  { id: "ssr-ssg", title: "SSR vs SSG", content: "서버사이드 렌더링과 정적 사이트 생성의 차이를 알아봅니다.", likes: 0 },
  { id: "dynamic-routes", title: "동적 라우팅", content: "Next.js에서 [slug]를 활용한 라우팅 방식입니다.", likes: 0 },
];
```

**② slug page**
- 문서의 `getPost`는 별도로 구현하지 않고, slug에 포함된 `id` 값을 `posts`에서 직접 비교해 가져옴.
- 해당 post가 없으면 `notFound()`를 호출함.

```tsx
// src/app/[id]/page.tsx
import LikeButton from '@/ui/LikeButton'
import { posts } from '@/lib/data'
import { notFound } from 'next/navigation'

export default async function PostPage({
  params
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params;
  const post = posts.find((p) => p.id === id);

  if (!post) {
    notFound();
  }

  return (
    <div>
      <main>
        <h1>{post.title}</h1>
        {/* ... */}
        <LikeButton likes={post.likes} />
      </main>
    </div>
  )
}
```

**③ (참고) getPost를 문서처럼 쓰려면**
- 좋은 방법은 아니지만, `data.tsx`에 더미 데이터와 함께 `getPosts()`(data 반환) → `getPost(id)`(id 비교 후 post 반환) 함수를 정의해야 함.

**④ LikeButton 컴포넌트**

```tsx
// src/ui/LikeButton.tsx
'use client'

import { useState } from 'react'

export default function LikeButton({ likes }: { likes: number }) {
  const [count, setCount] = useState<number>(likes ?? 0)
  const [isLiking, setIsLiking] = useState(false)

  const handleClick = async () => {
    // 낙관적 업데이트
    setIsLiking(true)
    setCount((c) => c + 1)

    // 실제 저장 로직(API 호출 등)이 있다면 이곳에서 호출
    setTimeout(() => setIsLiking(false), 300)
  }

  return (
    <button onClick={handleClick} disabled={isLiking} aria-pressed={false}>
      👍 {count}
    </button>
  )
}
```

- `count`: like 버튼을 클릭한 횟수 (초기값: data의 `likes` 필드)
- `isLiking`: "서버에 요청이 진행 중인지"를 나타내는 state (초기값: `false`)
- `handleClick`: `isLiking`을 `true`로, `count`를 +1 함. `setTimeout`은 **네트워크 지연 시뮬레이션**이며, 테스트할 때는 300ms보다 3000ms 정도로 늘려야 확인하기 쉬움 (3초 동안 클릭해도 반응 없음).

**⑤ Optimistic Update vs Pessimistic Update**

| 구분 | 낙관적 업데이트 (Optimistic) | 비관적 업데이트 (Pessimistic) |
| --- | --- | --- |
| UI 업데이트 시점 | 요청 직후 즉시 | 서버 응답을 받은 후 |
| 응답 실패 시 | 원래 상태로 롤백 | UI 변경 없음 |
| 속도 및 반응성 | 빠름 | 상대적으로 느림 |
| 장점 | 즉각적인 피드백, 네트워크가 느려도 체감 속도가 빠름 | 서버 응답 기반이라 데이터 일관성 보장, 잘못된 정보 표시 염려 없음 |
| 단점 | 오류 발생 시 잠시 잘못된 정보가 표시될 수 있고, 복구 로직이 필요함 | 응답이 늦어지면 사용자 경험 저하 |
| 적용 예시 | 좋아요 버튼, 댓글 작성 | 데이터 조회 후 렌더링 |

- LikeButton은 **낙관적 업데이트** 방식임 (서버 응답을 기다리지 않고 UI를 먼저 변경).

**⑥ `isLiking` state의 역할**
- **중복 클릭 방지**: `true`인 동안 버튼을 `disabled`로 만들어 중복 요청(중복 낙관적 업데이트)을 막음.
- **UI 피드백**: 로딩 상태 표시(스피너나 문구)에 사용 가능.
- **상태 안정화**: 요청이 끝날 때까지 추가 상태 변경을 잠시 멈추게 해 일관된 동작을 보장함.

**⑦ `likes ?? 0` (Null 병합 연산자) vs `||` (or 연산자)**
- `??`: 왼쪽이 `null` 또는 `undefined`일 때만 오른쪽 값을 반환함. 값이 있으면 그대로 반환함.
- `||`: **falsy 값** 전부를 오른쪽 값으로 대체함.

| Falsy 값 | 설명 |
| --- | --- |
| `false` | 불리언 false 값 그 자체 |
| `0` | 숫자 0은 false로 취급 |
| `''` | 빈 문자열 |
| `null` | 값이 없음을 의도적으로 명시 (개발자가 할당) |
| `undefined` | 값이 할당되지 않았을 때 (시스템에 의해 자동 부여) |
| `NaN` | Not-a-Number, 숫자가 아님 |

- **테스트 결과**
  - `data.tsx`의 `likes` 값 중 하나를 `false`로 바꾸고 `??`로 테스트 → `{count}` 부분에 아무것도 표시되지 않음 (정상).
  - `??`를 `||`로 바꿔서 테스트 → `false`, 빈 문자열(`''`) 모두 `0`이 반환됨. `||`는 falsy 값이 적용되기 때문임.

---

### 2. Next.js에서 server와 client component는 어떻게 작동합니까?

#### 2-1. server component의 작동

- Next.js는 server에서 React의 API를 사용해 렌더링을 조정함.
- 렌더링 작업은 개별 라우팅 세그먼트 별 묶음(Chunk)으로 나뉨. (`layout` 및 `page`)
- server component는 **RSC Payload**(React Server Component Payload)라는 특수한 데이터 형식으로 렌더링됨.
- client component와 RSC Payload는 HTML을 **미리 렌더링(prerender)** 하는 데 사용됨.

**RSC Payload란?**
- 렌더링된 React Server Components 트리를 직렬화하여 표현한 컴팩트한 형태이며, client에서 React가 브라우저의 DOM을 업데이트하는 데 사용됨.
- 포함 내용: server component의 렌더링 결과 / client component가 렌더링될 위치 및 JavaScript 파일 참조를 위한 자리 표시자(Placeholder) / server → client component로 전달되는 모든 props

**RSC Payload는 JSON인가, 바이너리인가?**
- 과거에는 JSON 문자열로 전달했음. (예: `{ type: "component", props: { title: "Hello" } }`)
- 현재 최신 React, 특히 Next.js App Router는 **compact binary format**(React 전용 이진 포맷)을 스트림으로 전달함.

| 비교 항목 | JSON | Binary (RSC) |
| --- | --- | --- |
| 크기 | 크다 | 작다 (compact) |
| 파싱 속도 | 느리다 (문자열 파싱) | 빠르다 (스트림 + 이진 파싱) |
| 구조 표현력 | 제한적 (문자만) | 더 복잡한 참조 및 내부 구조 표현 가능 |
| RSC 최적화 | - | 최적화됨 |

- 동작 흐름: React가 server에서 만든 UI 트리를 바이너리 스트림으로 client에 전달 → client 측 React가 해석해 UI를 재구성함. Next.js가 이 작업을 자동으로 처리하므로 개발자는 신경 쓰지 않아도 됨.

#### 2-2. client component의 작동 (첫 번째 load)

1. **HTML**: 사용자에게 경로(라우팅 페이지)의 비대화형(정적) 미리보기를 즉시 보여주는 데 사용됨.
2. **RSC 페이로드**: client와 server component 트리를 조정하는 데 사용됨.
3. **JavaScript**: client component를 hydration하고, 애플리케이션을 대화형으로 만드는 데 사용됨.

**Hydration이란?**
- 이벤트 핸들러를 DOM에 연결하여 정적 HTML을 인터랙티브하게 만드는 React의 프로세스임.

#### 2-3. 후속 네비게이션

- RSC 페이로드는 즉시 탐색할 수 있도록 **prefetch 및 cache**됨.
- client component는 server에서 렌더링된 HTML 없이 **전적으로 client에서 렌더링**됨.

---

### 3. Example

#### 3-1. client component 사용

- 파일의 맨 위(import문 위)에 `"use client"` 지시문을 추가해 client component를 생성함.
- `"use client"`는 server와 client 모듈 트리 사이의 **경계**를 선언하는 데 사용됨.
- 파일에 `"use client"`가 표시되면 해당 파일의 모든 import와 자식 component는 client 번들의 일부로 간주됨.
- 따라서 client를 대상으로 하는 모든 component에 이 지시문을 추가할 필요가 없음.

> 3-1 실습(Counter 컴포넌트, counter 페이지, layout 메뉴)은 [2026-10-28] 8주차에서 진행함.


---

## [2026-10-07] 6주차: Linking and Navigating (이어서) & Server and Client Components

### 1. 일반 함수와 객체 구조 분해 할당 매개변수 처리

- 일반 함수는 `props` 전체를 받고, 구조 분해 할당은 필요한 값(`params`)만 바로 꺼내 씀.
- `function Page({ params })`는 `const params = props.params`를 축약한 것 → **Next.js가 넘겨준 객체 중에서 `params`를 받아 사용**한다는 의미.

```tsx
export default async function Page({
                                     params,
                                   }: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  // ...
}
```

| 부분 | 의미 |
| --- | --- |
| `{ params, }` | 실제로 받을 값을 구조 분해 할당하는 부분 |
| `{ params: Promise<{ slug: string }> }` | TypeScript에서 `params`의 타입을 설명하는 부분 |
| `slug: string` | 객체 중 `slug`의 값이 문자열이라는 의미 |
| `Promise<...>` | 나중에 이런 형태로 prop을 전달하겠다는 의미 |
| `await params` | `params`가 Promise이기 때문에 이렇게 받을 수 있음 |

---

### 2. generateStaticParams 실습 (blog3)

- `blog2` 디렉토리를 복사해서 사용하면 빠르게 실습 가능, 테스트 편의를 위해 메뉴에 `Blog3` 추가.

```
src/app/(marketing)/
└─ blog3/
   ├─ page.tsx     // 블로그 목록
   ├─ posts.tsx    // 더미 데이터
   └─ [slug]/
      └─ page.tsx  // 개별 포스트
```

```tsx
// src/app/(marketing)/blog3/[slug]/page.tsx
import { notFound } from 'next/navigation';
import { posts } from '../posts';

// 빌드 시점에 미리 생성할 slug 목록 반환 → 각 slug마다 정적 페이지 생성
export async function generateStaticParams() {
  return posts.map((post) => ({
    slug: post.slug,
  }));
}

// params는 Promise이므로 await로 값을 꺼낸 뒤 사용
export default async function PostPage({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;

  // posts에서 slug에 해당하는 포스트 찾기
  const post = posts.find((p) => p.slug === slug);

  // 일치하는 포스트가 없으면 404 페이지 렌더링
  if (!post) {
    notFound();
  }

  return (
          <article>
            <h1>{post.title}</h1>
            <p>{post.content}</p>
          </article>
  );
}
```

- 목록 페이지는 `<Link href={`/blog3/${post.slug}`}>`로 각 포스트에 연결 (`posts.map()` 사용).
- 리스트의 링크를 통해 슬러그에 접근하면 오류가 나지 않지만, **직접 링크로 접근하면 오류가 발생**함 (슬라이드 설명).

#### 빌드 시 동작 흐름

1. 빌드 시점에 Next.js가 `src/app/(marketing)/blog3/[slug]/page.tsx` 같은 **동적 라우트**를 찾으면 `generateStaticParams()`를 실행.
2. 반환값은 `[{ slug: "hello" }, { slug: "world" }, { slug: "nextjs" }]` 형태.
3. 각 `params`에 대해 `page.tsx`를 실행하여 **정적 HTML** 생성 (예: `/blog/hello/index.html`).

- `generateStaticParams()` 자체는 **slug 배열만 반환**함.
- Next.js 빌드 프로세스가 배열을 순회 → 각 slug에 대해 `page.tsx` 실행 → 정적 HTML 생성.
- `map` 함수는 HTML을 작성해야 할 리스트를 Next.js에게 전달하는 역할.

#### generateStaticParams 없음 vs 있음

| 항목 | generateStaticParams 없음 | generateStaticParams 있음 |
| --- | --- | --- |
| 페이지 생성 시점 | 요청 시 서버에서 생성 (SSR/ISR) | 빌드 타임에 생성 (SSG) |
| 초기 로딩 속도 | 서버 렌더링 필요 → 상대적으로 느림 | 정적 HTML 제공 → 매우 빠름 |
| SEO | 가능하긴 함, 하지만 요청 시 생성 | 매우 유리 (검색엔진 즉시 HTML 크롤링 가능) |
| 유연성 | slug를 무한정 지원 가능 (DB 조회 등) | slug를 미리 알아야 함 (동적 slug는 제한적) |

#### await이 없어도 async를 붙여 두는 이유

- Next.js 13+ App Router의 `page.tsx` 같은 Server Component는 **비동기 렌더링을 전제**로 함 (데이터를 fetch하는 경우가 많음).

| 이유 | 설명 |
| --- | --- |
| 일관성 유지 | 어떤 페이지는 async, 어떤 페이지는 일반 function이면 혼란스러움 (공식 문서도 대부분 async function으로 예시 작성) |
| 확장성 | 지금은 더미 데이터지만 나중에 DB/API에서 `await fetch(...)`가 들어갈 수 있으므로 미리 붙여 두면 수정 불필요 |
| React Server Component 호환성 | Server Component는 Promise를 반환할 수 있어야 하고, Next.js는 async 패턴에 맞춰 최적화된 렌더링 파이프라인을 가짐 → 오버헤드 거의 없음 |

---

### 3. Linking and Navigating (이어서)

#### 2-3. 느린 네트워크

- 네트워크가 느리거나 불안정하면 링크 클릭 전에 **프리페칭이 완료되지 않을 수 있음** (정적/동적 경로 모두 영향).
- 이 경우 `loading.tsx`가 아직 프리페칭되지 않아 즉시 표시되지 않을 수 있음.
- 체감 성능 개선: **`useLinkStatus` Hook**으로 전환 중 인라인 시각적 피드백(스피너, 텍스트 글리머 등) 표시.

```tsx
'use client'
import { useLinkStatus } from 'next/link'

export default function LoadingIndicator() {
  const { pending } = useLinkStatus()
  return pending ? (
          <div role="status" aria-label="Loading" className="spinner" />
  ) : null
}
```

- 초기 애니메이션 지연(예: 100ms) + `opacity: 0`으로 시작하면 로딩 표시기를 **디바운스**할 수 있음 → 내비게이션이 지정된 지연 시간보다 오래 걸릴 때만 표시.
- ※ debounce: 연속적으로 발생하는 이벤트를 그룹화하여 특정 시간 간격 이후에 한 번만 처리하는 기술 (과도한 이벤트 방지, 성능 최적화).

#### 2-4. 프리페칭 비활성화

- `<Link prefetch={false} href="/blog">`로 프리페치를 끌 수 있음 → 대량의 링크 목록(예: 무한 스크롤 테이블) 렌더링 시 불필요한 리소스 사용 방지.

| 라우팅 | 프리페칭 비활성화 시 |
| --- | --- |
| 정적 라우팅 | 사용자가 링크를 클릭할 때만 가져옴 |
| 동적 라우팅 | 클라이언트가 이동하기 전에 서버에서 먼저 렌더링되어야 함 |

- 완전히 끄지 않고 리소스를 줄이려면 **마우스 호버 시에만 프리페치**하면 됨 → 방문 가능성이 높은 경로로만 제한.

```tsx
// app/ui/hover-prefetch-link.tsx
'use client'
import Link from 'next/link'
import { useState } from 'react'

function HoverPrefetchLink({ href, children }: {
  href: string
  children: React.ReactNode
}) {
  const [active, setActive] = useState(false)

  return (
          <Link
                  href={href}
                  prefetch={active ? null : false}
                  onMouseEnter={() => setActive(true)}
          >
            {children}
          </Link>
  )
}
```

- 실습 (`layout.tsx`): `Home`, `Blog`, `Blog2`, `Blog3`는 `<Link>` (링크가 호버되거나 뷰포트에 들어오면 프리페치), `Contact`는 일반 `<a>` 태그라서 프리페치 없음.

#### 2-5. Hydration이 완료되지 않음

- `<Link>`는 클라이언트 컴포넌트이므로 라우팅 페이지를 프리페치하기 전에 **하이드레이션이 먼저 되어야** 함.
- 초기 방문 시 대용량 JavaScript 번들 때문에 하이드레이션이 지연되어 프리페칭이 바로 시작되지 않을 수 있음.
- React는 선택적 Hydration으로 완화하며, 추가 개선 방법:
  - `@next/bundle-analyzer` 플러그인으로 대규모 종속성을 제거하고 번들 크기를 식별/축소.
  - 가능하면 클라이언트에서 서버로 로직을 이동 (서버 및 클라이언트 컴포넌트 문서 참조).

```bash
pnpm add @next/bundle-analyzer
```

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {}

const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
})

module.exports = withBundleAnalyzer(nextConfig)
```

```bash
ANALYZE=true npm run build   # 또는 yarn / pnpm
```

- 실행하면 브라우저에 **리포트 탭 3개**가 새로 열려 번들 크기를 확인할 수 있음.

#### Hydration이란?

- 서버에서 생성된 HTML에 JavaScript 로직을 추가하여 **동적으로 상호작용이 가능하도록 만드는 과정**.
- SSR은 서버에서 미리 HTML을 만들어 전달 → 초기 로딩은 빠르지만 HTML이 정적인 상태라 상호작용에는 추가 작업이 필요함.
- Hydration은 SSR로 생성된 정적 HTML에 클라이언트 측 JavaScript(이벤트 리스너, 상태 관리 등)를 연결하여 로드 후에도 사용자와 상호작용이 가능하게 만듦.

#### 3. Examples - 네이티브 히스토리 API

- Next.js에서는 `window.history.pushState`, `window.history.replaceState`로 **페이지를 다시 로드하지 않고** 브라우저 기록 스택을 업데이트할 수 있음.
- 두 호출은 Next.js 라우터와 통합되어 `usePathname`, `useSearchParams`와 동기화 가능.

| 메서드 | 동작 | 이전 상태로 | 예시 |
| --- | --- | --- | --- |
| `pushState` | 기록 스택에 새 항목 **추가** | 돌아갈 수 있음 | 제품 목록 정렬 |
| `replaceState` | 기록 스택의 현재 항목을 **교체** | 돌아갈 수 없음 | 애플리케이션 로케일(Locale) 전환 |

- ※ Locale: 사용자의 언어, 지역, 날짜/시간 형식, 숫자 표기법 등 UI에서 사용되는 다양한 설정을 정의하는 문자열.

#### 복습: How navigation works

- 알아야 할 4가지 개념: **Server Rendering / Prefetching / Streaming / Client-side transitions**
- Server Rendering: 레이아웃(layout)과 페이지(page)는 기본적으로 React 서버 컴포넌트이며, 초기/후속 내비게이션 모두 서버 컴포넌트 페이로드(Server Component Payload)가 클라이언트로 전송되기 전에 서버에서 생성됨.
- Client-side transitions: 일반적인 서버 렌더링 페이지 이동은 전체 페이지를 로드해서 state 삭제, 스크롤 위치 초기화, 상호작용 차단이 발생함 → `<Link>`를 통한 클라이언트 측 전환으로 방지.
  - 공유 레이아웃과 UI 유지
  - 현재 페이지를 미리 가져온(prefetch) 로딩 상태 또는 새 페이지로 교체
  - 프리페칭 및 스트리밍과 함께 사용하면 동적 경로에서도 빠른 전환 가능

---

### 4. 05. Server and Client Components

#### Introduction

- 기본적으로 `layout`과 `page`는 **server component**.
- 서버에서 데이터를 가져와 UI 일부를 렌더링하고, 선택적으로 결과를 cache한 후 client로 스트리밍 가능.
- 상호작용이나 브라우저 API가 필요한 경우 **client component**로 기능을 계층화.

----


#### 1. server 및 client component를 언제 사용해야 하나요?

| 구분 | 이럴 때 사용 |
| --- | --- |
| **client component** | state 및 event handler (`onClick`, `onChange`) / Lifecycle logic (`useEffect`) / 브라우저 전용 API (`localStorage`, `window`, `Navigator.geolocation` 등) / 사용자 정의 Hook |
| **server component** | 서버의 DB 또는 API에서 data를 가져올 때 / API key, token 등 보안 데이터를 client에 노출하지 않고 사용할 때 / 브라우저로 전송되는 JavaScript 양을 줄이고 싶을 때 / 첫 번째 콘텐츠 페인트(FCP)를 개선하고 콘텐츠를 client에 점진적으로 스트리밍할 때 |


## [2026-09-30] 5주차: Route 방식 비교 & Linking and Navigating

### Route 방식 비교 (React vs Next.js)

- React는 기본적으로 라우팅 기능이 없어 **라우터 라이브러리를 직접 설치**해서 설정해야 함.
- Next.js는 **라우팅 시스템을 자체적으로 내장**하고 있음.

| 항목 | React (기본) | Next.js |
| --- | --- | --- |
| 라우팅 방식 | **수동** (사용자가 직접 설정) | **자동** (폴더/파일 기반) |
| 라우터 도구 | react-router-dom 같은 외부 라이브러리 필요 | 자체 내장된 파일 기반 라우팅 시스템 |
| 라우트 정의 방식 | 코드에서 직접 `<Route>`로 정의 | 파일/폴더 이름으로 라우트가 자동 매핑됨 |
| 예시 | `<Route path="/about" element={<About />} />` | `pages/about.js` 또는 `app/about/page.tsx` → `/about` 경로 자동 생성 |

#### App Router의 강력한 기능들

| 기능 | 설명 |
| --- | --- |
| 중첩 레이아웃 | 여러 레벨의 `layout.js` 파일을 통해 레이아웃을 계층적으로 구성 가능 |
| 서버 컴포넌트 (RSC) | 서버에서만 렌더링되는 컴포넌트로 성능 최적화 가능 (React Server Component) |
| 로딩 UI | 페이지 전환 중 보여줄 `loading.js` 파일 제공 |
| 에러 UI | 특정 경로에서만 발생하는 에러를 처리할 `error.js` 제공 |
| 병렬 라우팅 | 하나의 경로 안에서 탭 같은 독립적인 뷰를 병렬로 렌더링 가능 |

#### 프로젝트별 추천 방식

| 상황 | 추천 방식 |
| --- | --- |
| 새 프로젝트 시작 | **App Router** (app 디렉토리 기반) |
| 기존 프로젝트 유지보수 | `pages/` 계속 사용 가능하지만, 점차 마이그레이션 필요 |
| React처럼 수동 라우팅이 필요한 경우 | React + react-router-dom 사용 가능 (Next.js는 자동 라우팅이 기본) |

---

### 05. Linking and Navigating

#### Introduction

- Next.js에서 경로(route)는 기본적으로 **서버에서 렌더링**되므로, 클라이언트는 새 경로를 표시하기 전에 서버의 응답을 기다려야 하는 경우가 많음.
- Next.js는 **prefetching, streaming, client-side transitions(클라이언트 사이드 전환)** 기능을 기본 제공하여 네비게이션 속도가 빠르고 반응성이 뛰어남.
- 이번 장에서는 네비게이션이 작동하는 방식, 동적 라우트와 느린 네트워크에 맞게 네비게이션을 최적화하는 방법을 다룸.

---

### 1. How navigation works (네비게이션 작동 방식)

- 다음 4가지 개념에 익숙해지는 것이 좋음: **Server Rendering / Prefetching / Streaming / Client-side transitions**

#### 1-1. Server Rendering (서버 렌더링)

- Next.js에서 **레이아웃(layout)과 페이지(page)는 기본적으로 React 서버 컴포넌트**임.
- 초기 네비게이션 및 후속 네비게이션 시, **서버 컴포넌트 페이로드(Server Component Payload)** 는 클라이언트로 전송되기 전에 서버에서 생성됨.
- 서버 렌더링은 발생 시점에 따라 두 가지 유형이 있음.

| 유형 | 발생 시점 | 특징 |
| --- | --- | --- |
| 정적 렌더링 (사전 렌더링) | 빌드 시점 또는 재검증(revalidation) 중 | 결과가 **캐시**됨. 재검증을 사용하면 전체 앱을 다시 빌드하지 않고 캐시 항목을 업데이트 가능 |
| 동적 렌더링 | 클라이언트 요청 시점 | 요청에 대한 응답으로 렌더링됨 |

- 서버 렌더링의 단점은 클라이언트가 새 경로를 표시하기 전에 **서버의 응답을 기다려야 한다는 것**임.
- Next.js는 방문 가능성이 높은 경로를 **미리 가져오고(prefetching)**, **클라이언트 측 전환(client-side transitions)** 을 수행하여 이 지연 문제를 해결함.

**알아두면 좋습니다: 최초 방문을 위해서 HTML이 생성됩니다.**

- 일반적인 React 앱(CSR만 사용)은 처음 방문 시 **빈 HTML + JavaScript 파일**만 내려주고, 브라우저가 JS를 실행해야 화면이 렌더링됨.
- Next.js는 특정 URL을 처음 방문하면 서버가 **해당 페이지의 HTML을 미리 생성**해서 전달함.
  - 브라우저는 JS 실행 전에도 즉시 HTML 뼈대와 콘텐츠를 표시할 수 있음.
  - 이후 React가 **하이드레이션(hydration)** 과정을 거쳐 상호작용이 가능해짐.
- 즉, 초기 방문 시에도 HTML을 생성해 내려주기 때문에 **UX가 좋아지고 SEO에도 유리**함.

#### 1-2. Prefetching (프리페칭: 미리 가져오기)

- 사용자가 해당 경로로 이동하기 **전에 백그라운드에서 해당 경로를 로드**하는 프로세스.
- 링크를 클릭하기 전에 다음 경로 렌더링에 필요한 데이터가 준비되어 있어 경로 간 이동이 즉각적으로 느껴짐.
- Next.js는 **`<Link>` 컴포넌트**와 연결된 경로를 자동으로 사용자 뷰포트에 미리 가져옴.
- `<a>` 태그를 사용하면 **프리페칭을 하지 않음**.

```tsx
// app/layout.tsx
import Link from 'next/link'

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
          <html>
          <body>
          <nav>
            {/* Prefetched when the link is hovered or enters the viewport */}
            <Link href="/blog">Blog</Link>
            {/* No prefetching */}
            <a href="/contact">Contact</a>
          </nav>
          {children}
          </body>
          </html>
  )
}
```

- 경로의 어느 정도를 프리페칭할지는 정적 경로인지 동적 경로인지에 따라 달라짐.

| 경로 유형 | 프리페칭 범위 |
| --- | --- |
| 정적 경로 | **전체 경로**가 프리페치됨 |
| 동적 경로 | 프리페치를 건너뛰거나, `loading.tsx`가 있는 경우 **부분적으로** 프리페칭됨 |

- 동적 라우팅을 건너뛰거나 부분적으로 프리페칭하여 사용자가 방문하지 않을 수도 있는 경로에 대한 **서버의 불필요한 작업을 방지**함.
- 다만 네비게이션 전에 서버 응답을 기다리면 앱이 응답하지 않는다는 인상을 줄 수 있으며, 동적 경로의 네비게이션 환경을 개선하려면 **스트리밍**을 사용할 수 있음.

#### 1-3. Streaming (스트리밍)

- 서버가 전체 경로가 렌더링될 때까지 기다리지 않고, **동적 경로의 일부가 준비되는 즉시 클라이언트에 전송**할 수 있음.
- 페이지의 일부가 아직 로드 중이더라도 사용자는 더 빨리 콘텐츠를 볼 수 있음. (공유 레이아웃과 로딩 스켈레톤을 미리 요청 가능)
- Next.js는 내부적으로 `page.tsx` 콘텐츠를 **`<Suspense>` 경계로 자동 래핑**함.
  - 미리 가져온 대체 UI는 경로가 로드되는 동안 표시되고, 준비가 되면 실제 콘텐츠로 대체됨.
  - `<Suspense>`를 사용하여 중첩된 컴포넌트에 대한 로딩 UI를 만들 수도 있음.
- **loading skeletons**: 웹/앱에서 콘텐츠가 로드되는 동안 사용자에게 보여지는 빈 화면의 일종.

**`loading.tsx`의 이점**

- 사용자에게 즉각적인 네비게이션과 시각적 피드백을 제공함.
- 공유 레이아웃은 상호작용이 가능하며, 네비게이션은 중단할 수 있음.
- 핵심 웹 지표(TTFB, FCP, TTI)가 개선됨.

**Core Web Vitals (웹 성능 지표)**

- Next.js 공식 문서에서 이야기하는 TTFB, FCP, TTI는 과거에 주로 사용하던 **레거시 지표**이며, 웹페이지가 기술적으로 로드되는 순서대로 시간을 측정함.

| 지표 | 의미 |
| --- | --- |
| TTFB (Time to First Byte) | 네트워크와 서버의 성능. 시간이 길면 서버가 느리거나 네트워크 연결에 문제가 있는 것 |
| FCP (First Contentful Paint) | 하얀 화면에서 무언가 처음 뜰 때까지의 시간 (페이지가 로딩되기 시작했다고 인지하는 순간) |
| TTI (Time to Interactive) | 페이지가 완전히 똑똑해진 시점. 버튼을 눌렀을 때 정상 작동할 수 있는 준비가 완료된 시간 |

- 최신 성능 측정에서는 TTI의 중요도가 낮아지고 **TBT(Total Blocking Time)나 INP**로 대체되는 추세.

**Shared layouts remain interactive and navigation is interruptible**

- **Shared layouts remain interactive**
  - App Router에서는 `layout.tsx`가 여러 페이지 간에 공유됨. (예: `/blog/page.tsx`와 `/blog/[slug]/page.tsx` 모두 `blog/layout.tsx`를 공유)
  - 페이지 이동 시 `layout.tsx`는 다시 리렌더링되지 않고 유지되므로, 사이드바·네비게이션 메뉴·음악 플레이어 같은 UI가 새 페이지 로딩 중에도 계속 동작함.
- **navigation is interruptible**
  - 페이지 이동 중 사용자가 다른 네비게이션 동작을 하면 **이전 로딩을 취소(cancel)** 해 줌.
  - 네트워크 요청이나 렌더링이 진행 중이라도 다시 클릭하면 이전 요청은 중단되고 새 요청만 실행됨.

#### 1-4. Client-side transitions (클라이언트 측 전환)

- 일반적으로 서버 렌더링 페이지로 이동하면 전체 페이지가 로드되어 **state가 삭제되고, 스크롤 위치가 재설정되며, 상호작용이 차단**됨.
- Next.js는 `<Link>` 컴포넌트를 사용하는 **클라이언트 측 전환**으로 이를 방지함. 페이지를 다시 로딩하는 대신 콘텐츠를 동적으로 업데이트함.
  - 공유 레이아웃과 UI를 유지함.
  - 현재 페이지를 미리 가져온(prefetching) 로딩 상태 또는 사용 가능한 경우 새 페이지로 바꿈.
- 서버에서 렌더링된 앱을 클라이언트에서 렌더링된 앱처럼 느껴지게 하며, 프리페칭 및 스트리밍과 함께 사용하면 동적 경로에서도 빠른 전환이 가능함.

#### 1절 실습: 네비게이션 작동 방식

- 디렉토리 구조 (디렉토리 이름 `blog`는 다른 것으로 해도 됨)

```
app/
  ├── page.tsx        // Root Page
  ├── layout.tsx      // RootLayout
  └── blog/
      ├── page.tsx    // 블로그 목록
      └── loading.tsx // 로딩 스켈레톤
```

- Root Page를 간단히 작성하고, blog 디렉토리에 간단한 page와 로딩 스켈레톤을 만듦.
- RootLayout에 `Link` 컴포넌트를 이용해서 blog 네비게이션을 만듦.
- 로딩 스켈레톤의 동작을 확인하기 위해 blog page에 **time delay**를 줌.
- 참고: 공식 문서에는 RootLayout에 `<a>` 태그로 blog 네비게이션을 만드는 예제가 있음.

---

### 2. 전환을 느리게 만드는 요인

- Next.js는 최적화를 통해 네비게이션 속도가 빠르지만, 특정 조건에서는 전환 속도가 여전히 느릴 수 있음.

#### 2-1. 동적 경로 없는 `loading.tsx`

- 동적 경로로 이동할 때 클라이언트는 서버 응답을 기다려야 해서 **앱이 응답하지 않는다는 인상**을 받을 수 있음.
- **부분 프리페칭을 활성화**하고, 즉시 네비게이션을 트리거하고, 경로가 렌더링되는 동안 로딩 UI를 표시하려면 **동적 경로에 `loading.tsx`를 추가**하는 것이 좋음.

```tsx
// app/blog/[slug]/loading.tsx
export default function Loading() {
  return <LoadingSkeleton />
}
```

**알아두면 좋은 정보: devIndicators**

- 개발 모드에서 Next.js 개발자 도구(Devtools)를 사용하여 경로가 정적인지 동적인지 확인할 수 있음. (보통 좌측 하단에 N자 아이콘으로 표시)
- Next.js 15.2.0부터 `position` 옵션이 새롭게 추가되었고, `appIsrStatus`, `buildActivity`, `buildActivityPosition` 옵션은 더 이상 사용되지 않음.
- 아이콘이 보이지 않으면 `next.config.ts`에 `devIndicators`를 추가하고, 위치를 바꾸고 싶다면 인디케이터 설정에서 변경함. (아직은 라우팅 결과 정도만 확인 가능)

```ts
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  devIndicators: { position: 'bottom-left' },
};

export default nextConfig;
```

#### 2-2. 동적 세그먼트 없는 `generateStaticParams`

- 동적 세그먼트는 사전 렌더링될 수 있지만, `generateStaticParams`가 누락되어 사전 렌더링되지 않으면 해당 경로는 **요청 시점에 동적 렌더링**으로 대체됨.
- `generateStaticParams`를 추가하여 **빌드 시점에 경로가 정적으로 생성**되도록 함.

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())

  return posts.map((post) => ({
    slug: post.slug,
  }))
}

export default async function Page({
                                     params,
                                   }: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  // ...
}
```

#### 실습 1: `generateStaticParams`가 없는 경우 (blog2)

- 디렉토리 구조 (더미 데이터는 3장에서 사용했던 것을 사용, 테스트가 편하게 blog2 메뉴를 만듦)

```
src/app/(marketing)/
  └── blog2/
      ├── page.tsx    // 블로그 목록
      ├── posts.tsx   // 더미 데이터
      └── [slug]/
          └── page.tsx // 개별 포스트
```

```tsx
// src/app/(marketing)/blog2/[slug]/page.tsx
import { posts } from "../posts";

export default async function PostPage({
                                         params,
                                       }: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params; // generateStaticParams가 없으므로 런타임에서 처리
  const post = posts.find((p) => p.slug === slug);

  if (!post) {
    return <h1>포스트를 찾을 수 없습니다.</h1>;
  }

  return (
          <article>
            <h1>{post.title}</h1>
            <p>{post.content}</p>
          </article>
  );
}
```

- `generateStaticParams`를 안 쓰면 **요청할 때마다 서버에서 동적으로 처리**함.
- 자주 변하지 않는 페이지는 `generateStaticParams` 사용을 권장함. (정적 사이트처럼 빠르기 때문)
- 사용자 입력, DB 조회 등이 필요한 경우는 `generateStaticParams` 없이 **런타임 처리**를 하는 것이 좋음.

#### 실습 2: `generateStaticParams`를 사용하는 경우 (blog3)

- blog2 디렉토리를 복사해서 blog3로 만들면 실습을 빠르게 진행할 수 있음.
- 아래처럼 `params`를 `await` 없이 바로 사용하면, 리스트에서 링크를 통해 슬러그에 접근할 때는 오류가 나지 않지만 **직접 링크로 접근하면 오류가 발생**함.

```tsx
// (오류 발생) params를 바로 사용
export default async function PostPage({ params }: { params: { slug: string } }) {
  const post = posts.find((p) => p.slug === params.slug);
  // ...
}
```

- 오류를 수정하기 위해서는 **`async`, `await`를 사용**해야 함.

```tsx
// src/app/(marketing)/blog3/[slug]/page.tsx
import { notFound } from "next/navigation";
import { posts } from "../posts";

// 빌드 시점에 미리 생성할 slug 목록을 반환
export async function generateStaticParams() {
  return posts.map((post) => ({
    slug: post.slug,
  }));
}

// params는 Promise일 수 있으므로 await params로 값을 해제(unwrap)한 후 접근
export default async function PostPage({
                                         params,
                                       }: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;

  const post = posts.find((p) => p.slug === slug);

  // 일치하는 포스트가 없으면 404 처리 (실무에서는 notFound() 호출이나 커스텀 404 컴포넌트 사용)
  if (!post) {
    notFound();
  }

  return (
          <article>
            <h1>{post.title}</h1>
            <p>{post.content}</p>
          </article>
  );
}
```

**`generateStaticParams` 동작 흐름**

- 빌드 시점에 Next.js가 `src/app/(marketing)/blog3/[slug]/page.tsx` 같은 동적 라우트를 찾으면 `generateStaticParams()`를 실행함.
- 반환값은 `[{ slug: "hello" }, { slug: "world" }, { slug: "nextjs" }]` 같은 **slug 객체 배열** 형태임.
- 각 params에 대해 `page.tsx`를 실행하여 **정적 HTML을 생성**함.

| params | 빌드 후 생성되는 파일 |
| --- | --- |
| `{ slug: "hello" }` | `/blog/hello/index.html` |
| `{ slug: "world" }` | `/blog/world/index.html` |
| `{ slug: "nextjs" }` | `/blog/nextjs/index.html` |

- 정리
  - `generateStaticParams()` 자체는 **slug 배열만 반환**함.
  - Next.js 빌드 프로세스가 이 배열을 순회하며 → 각 slug에 대해 `page.tsx` 실행 → 정적 HTML 생성.
  - `map` 함수는 **HTML을 작성해야 할 리스트를 Next.js에게 전달**하는 역할을 함.



---



## [2026-09-23] 4주차: 동적 라우팅 심화 & 페이지 연결

### 1. Link Component (API Reference 복습)
- `<Link>`는 HTML `<a>` 요소를 확장하여 프리페칭(prefetching)과 라우트 간 클라이언트 사이드 내비게이션 기능을 제공하는 React 컴포넌트임. Next.js에서 라우트 간 이동을 위해 주로 사용됨.
- 주요 prop

| Prop | 예시 | 타입 |
| --- | --- | --- |
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
| --- | --- | --- |
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
| --- | --- | --- |
| 라우팅 방식 | 수동 (사용자가 직접 설정) | 자동 (폴더/파일 기반) |
| 라우터 도구 | `react-router-dom` 같은 외부 라이브러리 필요 | 자체 내장된 파일 기반 라우팅 시스템 |
| 라우트 정의 방식 | 코드에서 직접 `<Route>`로 정의 | 파일/폴더 이름으로 라우트가 자동 매핑됨 |
| 예시 | `<Route path="/about" element={<About />} />` | `pages/about.js` 또는 `app/about/page.tsx` → `/about` 경로 자동 생성 |

- React는 기본적으로 라우팅 기능이 없기 때문에 직접 라우터 라이브러리를 설치해서 라우팅을 설정해야 하지만, Next.js는 자체적으로 라우팅 시스템을 내장하고 있음.

---

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
