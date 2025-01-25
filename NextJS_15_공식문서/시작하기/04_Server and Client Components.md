# 서버 및 클라이언트 컴포넌트

기본적으로 레이아웃과 페이지는 **서버 컴포넌트**로, 서버에서 데이터를 가져와 UI의 일부를 렌더링하고, 선택적으로 결과를 캐시하며, 클라이언트로 스트리밍할 수 있습니다. 상호작용이나 브라우저 API가 필요한 경우 **클라이언트 컴포넌트**를 사용하여 기능을 레이어링할 수 있습니다.

이 페이지에서는 Next.js에서 서버 및 클라이언트 컴포넌트가 어떻게 작동하는지, 언제 사용해야 하는지, 그리고 애플리케이션에서 이들을 함께 구성하는 방법에 대한 예시를 설명합니다.

## 서버 및 클라이언트 컴포넌트는 언제 사용해야 할까요?

클라이언트와 서버 환경은 서로 다른 기능을 가지고 있습니다. 서버 및 클라이언트 컴포넌트를 사용하면 사용 사례에 따라 각 환경에서 로직을 실행할 수 있습니다.

다음과 같은 경우에 **클라이언트 컴포넌트**를 사용하세요:

- **상태(State)**와 **이벤트 핸들러**가 필요한 경우. 예: `onClick`, `onChange`
- **라이프사이클 로직**이 필요한 경우. 예: `useEffect`
- **브라우저 전용 API**가 필요한 경우. 예: `localStorage`, `window`, `Navigator.geolocation` 등
- **커스텀 훅**이 필요한 경우

다음과 같은 경우에 **서버 컴포넌트**를 사용하세요:

- 데이터베이스나 소스에 가까운 API에서 데이터를 가져오는 경우
- API 키, 토큰 및 기타 비밀을 클라이언트에 노출하지 않고 사용하는 경우
- 브라우저로 전송되는 JavaScript 양을 줄이는 경우
- 첫 번째 콘텐츠 페인트(FCP)를 개선하고 클라이언트로 콘텐츠를 점진적으로 스트리밍하는 경우

예를 들어, `<Page>` 컴포넌트는 포스트에 대한 데이터를 가져오는 서버 컴포넌트이며, 클라이언트 측 상호작용을 처리하는 `<LikeButton>`에 props로 전달합니다.

```tsx
import LikeButton from '@/app/ui/like-button'
import { getPost } from '@/lib/data'

export default async function Page({ params }) {
  const { id } = await params
  const post = await getPost(id)

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

```tsx
'use client'

import { useState } from 'react'

export default function LikeButton({ likes }) {
  // ...
}
```

> 💡 **실무 팁**: **서버 우선** - 기본적으로 서버 컴포넌트를 사용하고, 클라이언트 기능이 꼭 필요한 경우에만 클라이언트 컴포넌트로 전환하세요. **경계 최소화** - 전체 페이지를 클라이언트 컴포넌트로 만들지 말고, 상호작용이 필요한 부분만 분리하세요. **데이터 패칭은 서버에서** - 데이터베이스 접근, API 호출 등은 서버 컴포넌트에서 처리하세요.

## Next.js에서 서버 및 클라이언트 컴포넌트는 어떻게 작동할까요?

### 서버에서

서버에서 Next.js는 React의 API를 사용하여 렌더링을 조율합니다. 렌더링 작업은 개별 라우트 세그먼트(레이아웃과 페이지)별로 청크로 분할됩니다:

- **서버 컴포넌트**는 React Server Component Payload(RSC Payload)라는 특별한 데이터 형식으로 렌더링됩니다.
- **클라이언트 컴포넌트**와 RSC Payload는 HTML을 사전 렌더링하는 데 사용됩니다.

> **React Server Component Payload(RSC)란?**
>
> RSC Payload는 렌더링된 React Server Components 트리의 압축된 바이너리 표현입니다. 클라이언트의 React가 브라우저의 DOM을 업데이트하는 데 사용됩니다. RSC Payload에는 다음이 포함됩니다:
>
> - 서버 컴포넌트의 렌더링된 결과
> - 클라이언트 컴포넌트가 렌더링되어야 할 위치의 자리 표시자와 JavaScript 파일에 대한 참조
> - 서버 컴포넌트에서 클라이언트 컴포넌트로 전달되는 모든 props

### 클라이언트에서 (첫 번째 로드)

그런 다음 클라이언트에서:

1. **HTML**은 사용자에게 라우트의 빠른 비상호작용 미리보기를 즉시 표시하는 데 사용됩니다.
2. **RSC Payload**는 클라이언트와 서버 컴포넌트 트리를 조화시키는 데 사용됩니다.
3. **JavaScript**는 클라이언트 컴포넌트를 하이드레이션하고 애플리케이션을 상호작용 가능하게 만드는 데 사용됩니다.

> **하이드레이션이란?**
>
> 하이드레이션은 정적 HTML을 상호작용 가능하게 만들기 위해 DOM에 이벤트 핸들러를 연결하는 React의 프로세스입니다.

### 후속 네비게이션

후속 네비게이션에서:

- **RSC Payload**는 즉시 네비게이션을 위해 프리페치되고 캐시됩니다.
- **클라이언트 컴포넌트**는 서버 렌더링된 HTML 없이 클라이언트에서 완전히 렌더링됩니다.

## 예시

### 클라이언트 컴포넌트 사용하기

파일 상단, 임포트 위에 `"use client"` 지시어를 추가하여 클라이언트 컴포넌트를 만들 수 있습니다.

```tsx
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>{count} likes</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
}
```

`"use client"`는 서버와 클라이언트 모듈 그래프(트리) 간의 **경계**를 선언하는 데 사용됩니다.

파일이 `"use client"`로 표시되면 **모든 임포트와 자식 컴포넌트가 클라이언트 번들의 일부로 간주됩니다**. 이는 클라이언트용으로 의도된 모든 컴포넌트에 지시어를 추가할 필요가 없다는 것을 의미합니다.

### JS 번들 크기 줄이기

클라이언트 JavaScript 번들의 크기를 줄이려면 UI의 큰 부분을 클라이언트 컴포넌트로 표시하는 대신 특정 상호작용 컴포넌트에 `'use client'`를 추가하세요.

예를 들어, `<Layout>` 컴포넌트에는 로고와 네비게이션 링크 같은 대부분 정적인 요소들이 포함되어 있지만, 상호작용적인 검색 바도 포함되어 있습니다. `<Search />`는 상호작용적이며 클라이언트 컴포넌트여야 하지만, 레이아웃의 나머지 부분은 서버 컴포넌트로 유지될 수 있습니다.

```tsx
// 클라이언트 컴포넌트
import Search from './search'
// 서버 컴포넌트
import Logo from './logo'

// Layout은 기본적으로 서버 컴포넌트입니다
export default function Layout({ children }) {
  return (
    <>
      <nav>
        <Logo />
        <Search />
      </nav>
      <main>{children}</main>
    </>
  )
}
```

```tsx
'use client'

export default function Search() {
  // ...
}
```

> 💡 **실무 팁**: **컴포넌트 분리** - 상호작용이 필요한 작은 단위로 컴포넌트를 분리하여 클라이언트 번들 크기를 최소화하세요. **조건부 렌더링** - 사용자 상호작용에 따라 동적으로 로드되는 컴포넌트는 `dynamic import`를 고려하세요. **서버 우선 설계** - 가능한 한 많은 로직을 서버에서 처리하고, 클라이언트는 UI 상호작용에만 집중하세요.

### 서버에서 클라이언트 컴포넌트로 데이터 전달

props를 사용하여 서버 컴포넌트에서 클라이언트 컴포넌트로 데이터를 전달할 수 있습니다.

```tsx
import LikeButton from '@/app/ui/like-button'
import { getPost } from '@/lib/data'

export default async function Page({ params }) {
  const { id } = await params
  const post = await getPost(id)

  return <LikeButton likes={post.likes} />
}
```

```tsx
'use client'

export default function LikeButton({ likes }) {
  // ...
}
```

또는 `use` 훅을 사용하여 서버 컴포넌트에서 클라이언트 컴포넌트로 데이터를 스트리밍할 수 있습니다.

> **알아두면 좋은 점**: 클라이언트 컴포넌트에 전달되는 props는 React에 의해 직렬화 가능해야 합니다.

### 서버 및 클라이언트 컴포넌트 인터리빙

서버 컴포넌트를 클라이언트 컴포넌트에 prop으로 전달할 수 있습니다. 이를 통해 클라이언트 컴포넌트 내에서 서버 렌더링된 UI를 시각적으로 중첩할 수 있습니다.

일반적인 패턴은 `children`을 사용하여 `<ClientComponent>`에 *슬롯*을 만드는 것입니다. 예를 들어, 서버에서 데이터를 가져오는 `<Cart>` 컴포넌트를 클라이언트 상태를 사용하여 가시성을 토글하는 `<Modal>` 컴포넌트 내부에 배치하는 것입니다.

```tsx
'use client'

export default function Modal({ children }) {
  return <div>{children}</div>
}
```

그런 다음 부모 서버 컴포넌트(예: `<Page>`)에서 `<Cart>`를 `<Modal>`의 자식으로 전달할 수 있습니다:

```tsx
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

이 패턴에서 모든 서버 컴포넌트는 props로 전달된 것들을 포함하여 미리 서버에서 렌더링됩니다. 결과 RSC 페이로드에는 컴포넌트 트리 내에서 클라이언트 컴포넌트가 렌더링되어야 할 위치의 참조가 포함됩니다.

### 컨텍스트 프로바이더

React 컨텍스트는 현재 테마와 같은 전역 상태를 공유하는 데 일반적으로 사용됩니다. 그러나 React 컨텍스트는 서버 컴포넌트에서 지원되지 않습니다.

컨텍스트를 사용하려면 `children`을 받는 클라이언트 컴포넌트를 만드세요:

```tsx
'use client'

import { createContext } from 'react'

export const ThemeContext = createContext({})

export default function ThemeProvider({ children }) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
}
```

그런 다음 서버 컴포넌트(예: `layout`)로 가져오세요:

```tsx
import ThemeProvider from './theme-provider'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  )
}
```

서버 컴포넌트가 이제 프로바이더를 직접 렌더링할 수 있으며, 앱 전체의 다른 모든 클라이언트 컴포넌트가 이 컨텍스트를 사용할 수 있습니다.

> **알아두면 좋은 점**: 프로바이더를 트리에서 가능한 한 깊게 렌더링해야 합니다. `ThemeProvider`가 전체 `<html>` 문서 대신 `{children}`만 감싸는 것을 주목하세요. 이렇게 하면 Next.js가 서버 컴포넌트의 정적 부분을 최적화하기 쉬워집니다.

### 서드파티 컴포넌트

클라이언트 전용 기능에 의존하는 서드파티 컴포넌트를 사용할 때는 클라이언트 컴포넌트로 감싸서 예상대로 작동하도록 할 수 있습니다.

예를 들어, `<Carousel />`을 `acme-carousel` 패키지에서 가져올 수 있습니다. 이 컴포넌트는 `useState`를 사용하지만 아직 `"use client"` 지시어가 없습니다.

클라이언트 컴포넌트 내에서 `<Carousel />`을 사용하면 예상대로 작동합니다:

```tsx
'use client'

import { useState } from 'react'
import { Carousel } from 'acme-carousel'

export default function Gallery() {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>View pictures</button>
      {/* 클라이언트 컴포넌트 내에서 사용되므로 작동함 */}
      {isOpen && <Carousel />}
    </div>
  )
}
```

그러나 서버 컴포넌트 내에서 직접 사용하려고 하면 오류가 발생합니다. 이를 해결하려면 클라이언트 전용 기능에 의존하는 서드파티 컴포넌트를 자체 클라이언트 컴포넌트로 감쌀 수 있습니다:

```tsx
'use client'

import { Carousel } from 'acme-carousel'

export default Carousel
```

이제 서버 컴포넌트 내에서 직접 `<Carousel />`을 사용할 수 있습니다:

```tsx
import Carousel from './carousel'

export default function Page() {
  return (
    <div>
      <p>View pictures</p>
      {/* Carousel이 클라이언트 컴포넌트이므로 작동함 */}
      <Carousel />
    </div>
  )
}
```

> **라이브러리 작성자를 위한 조언**
>
> 컴포넌트 라이브러리를 구축하는 경우, 클라이언트 전용 기능에 의존하는 진입점에 `"use client"` 지시어를 추가하세요. 이렇게 하면 사용자가 래퍼를 만들 필요 없이 서버 컴포넌트로 컴포넌트를 가져올 수 있습니다.
>
> 일부 번들러는 `"use client"` 지시어를 제거할 수 있다는 점을 주목하세요. React Wrap Balancer와 Vercel Analytics 저장소에서 `"use client"` 지시어를 포함하도록 esbuild를 구성하는 방법의 예시를 찾을 수 있습니다.

### 환경 오염 방지

JavaScript 모듈은 서버와 클라이언트 컴포넌트 모듈 간에 공유될 수 있습니다. 이는 실수로 서버 전용 코드를 클라이언트에 가져올 수 있음을 의미합니다. 예를 들어, 다음 함수를 고려해보세요:

```ts
export async function getData() {
  const res = await fetch('https://external-service.com/data', {
    headers: {
      authorization: process.env.API_KEY,
    },
  })

  return res.json()
}
```

이 함수는 클라이언트에 절대 노출되어서는 안 되는 `API_KEY`를 포함합니다.

Next.js에서는 `NEXT_PUBLIC_` 접두사가 있는 환경 변수만 클라이언트 번들에 포함됩니다. 변수에 접두사가 없으면 Next.js는 이를 빈 문자열로 바꿉니다.

결과적으로 `getData()`가 클라이언트에서 가져와 실행될 수 있더라도 예상대로 작동하지 않습니다.

클라이언트 컴포넌트에서 실수로 사용하는 것을 방지하려면 `server-only` 패키지를 사용할 수 있습니다.

그런 다음 서버 전용 코드가 포함된 파일에 패키지를 가져오세요:

```js
import 'server-only'

export async function getData() {
  const res = await fetch('https://external-service.com/data', {
    headers: {
      authorization: process.env.API_KEY,
    },
  })

  return res.json()
}
```

이제 클라이언트 컴포넌트에서 모듈을 가져오려고 하면 빌드 시간 오류가 발생합니다.

해당하는 `client-only` 패키지는 `window` 객체에 액세스하는 코드와 같은 클라이언트 전용 로직을 포함하는 모듈을 표시하는 데 사용할 수 있습니다.

Next.js에서 `server-only` 또는 `client-only` 설치는 **선택사항**입니다. 그러나 린팅 규칙이 불필요한 의존성에 플래그를 지정하는 경우 문제를 피하기 위해 설치할 수 있습니다.

```bash
npm install server-only
```

Next.js는 모듈이 잘못된 환경에서 사용될 때 더 명확한 오류 메시지를 제공하기 위해 `server-only` 및 `client-only` 가져오기를 내부적으로 처리합니다. NPM의 이러한 패키지 내용은 Next.js에서 사용되지 않습니다.

Next.js는 또한 `noUncheckedSideEffectImports`가 활성화된 TypeScript 구성을 위해 `server-only` 및 `client-only`에 대한 자체 타입 선언을 제공합니다.

> 💡 **실무 팁**: **패키지 활용** - `server-only`와 `client-only` 패키지를 프로젝트 초기부터 설정하여 실수를 방지하세요. **환경 변수 관리** - 클라이언트에 노출될 변수는 반드시 `NEXT_PUBLIC_` 접두사를 사용하세요. **API 호출 분리** - 서버 전용 API 호출은 별도 유틸리티 파일로 분리하고 `server-only`를 적용하세요.

## 다음 단계

이 페이지에서 언급된 API에 대해 자세히 알아보세요.

- [use client](/docs/app/api-reference/directives/use-client.md) - 클라이언트에서 컴포넌트를 렌더링하기 위해 use client 지시어를 사용하는 방법 학습

---

**원문**: [Next.js 공식 문서 - Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)