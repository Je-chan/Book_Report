# 링킹과 네비게이션

Next.js에서는 기본적으로 라우트가 서버에서 렌더링됩니다. 이는 클라이언트가 새로운 라우트를 보기 전에 서버 응답을 기다려야 한다는 것을 의미합니다. Next.js는 내장된 **prefetching**, **streaming**, **client-side transitions** 기능을 제공하여 네비게이션이 빠르고 반응적으로 유지되도록 합니다.

이 가이드는 Next.js에서 네비게이션이 어떻게 작동하는지, 그리고 **동적 라우트**와 **느린 네트워크**에 대해 어떻게 최적화할 수 있는지 설명합니다.

## 네비게이션 작동 방식

Next.js에서 네비게이션이 어떻게 작동하는지 이해하려면 다음 개념들을 알아두는 것이 도움이 됩니다:

- [서버 렌더링](#서버-렌더링)
- [프리페칭](#프리페칭)
- [스트리밍](#스트리밍)
- [클라이언트 사이드 전환](#클라이언트-사이드-전환)

### 서버 렌더링

Next.js에서 레이아웃과 페이지는 기본적으로 **React Server Components**입니다. 초기 네비게이션과 후속 네비게이션에서 서버 컴포넌트 페이로드가 클라이언트로 전송되기 전에 서버에서 생성됩니다.

서버 렌더링에는 *언제* 일어나는지에 따라 두 가지 유형이 있습니다:

- **정적 렌더링(Static Rendering) 또는 사전 렌더링(Prerendering)**: 빌드 시간 또는 재검증 중에 발생하며 결과가 캐시됩니다.
- **동적 렌더링(Dynamic Rendering)**: 클라이언트 요청에 응답하여 요청 시간에 발생합니다.

서버 렌더링의 단점은 클라이언트가 새로운 라우트를 볼 수 있기 전에 서버의 응답을 기다려야 한다는 것입니다. Next.js는 사용자가 방문할 가능성이 있는 라우트를 **프리페칭**하고 **클라이언트 사이드 전환**을 수행하여 이러한 지연을 해결합니다.

> **알아두면 좋은 점**: 초기 방문 시에는 HTML도 생성됩니다.

### 프리페칭

프리페칭은 사용자가 네비게이션하기 전에 백그라운드에서 라우트를 로드하는 과정입니다. 사용자가 링크를 클릭할 때 다음 라우트를 렌더링할 데이터가 이미 클라이언트 측에서 사용 가능하기 때문에 애플리케이션의 라우트 간 네비게이션이 즉시 느껴집니다.

Next.js는 `<Link>` 컴포넌트로 연결된 라우트가 사용자의 뷰포트에 들어올 때 자동으로 프리페치합니다.

```tsx
import Link from 'next/link'

export default function Layout({ children }) {
  return (
    <html>
      <body>
        <nav>
          {/* 링크가 호버되거나 뷰포트에 들어올 때 프리페치됨 */}
          <Link href="/blog">Blog</Link>
          {/* 프리페치 없음 */}
          <a href="/contact">Contact</a>
        </nav>
        {children}
      </body>
    </html>
  )
}
```

얼마나 많은 라우트가 프리페치되는지는 정적인지 동적인지에 따라 달라집니다:

- **정적 라우트**: 전체 라우트가 프리페치됩니다.
- **동적 라우트**: 프리페칭이 건너뛰어지거나, `loading.tsx`가 있는 경우 부분적으로 프리페치됩니다.

동적 라우트를 건너뛰거나 부분적으로 프리페치함으로써 Next.js는 사용자가 절대 방문하지 않을 수도 있는 라우트에 대한 서버의 불필요한 작업을 피합니다. 그러나 네비게이션 전에 서버 응답을 기다리는 것은 사용자에게 앱이 응답하지 않는다는 인상을 줄 수 있습니다.

동적 라우트의 네비게이션 경험을 개선하려면 **스트리밍**을 사용할 수 있습니다.

> 💡 **실무 팁**: 프리페칭을 효과적으로 활용하려면 중요한 네비게이션에는 반드시 `<Link>` 컴포넌트를 사용하고, 동적 라우트에는 `loading.tsx`를 추가하여 부분 프리페칭을 활성화하세요. 무한 스크롤이나 대량의 링크 목록에서는 `prefetch={false}` 옵션을 고려해보세요.

### 스트리밍

스트리밍을 사용하면 서버가 전체 라우트가 렌더링될 때까지 기다리지 않고 준비되는 대로 동적 라우트의 일부를 클라이언트에 보낼 수 있습니다. 이는 페이지의 일부가 여전히 로딩 중이더라도 사용자가 더 빨리 무언가를 볼 수 있다는 것을 의미합니다.

동적 라우트의 경우 **부분적으로 프리페치**될 수 있습니다. 즉, 공유 레이아웃과 로딩 스켈레톤을 미리 요청할 수 있습니다.

스트리밍을 사용하려면 라우트 폴더에 `loading.tsx`를 생성하세요:

```tsx
export default function Loading() {
  // 라우트가 로딩되는 동안 표시될 대체 UI를 추가합니다.
  return <LoadingSkeleton />
}
```

내부적으로 Next.js는 자동으로 `page.tsx` 내용을 `<Suspense>` 경계로 감쌉니다. 프리페치된 대체 UI가 라우트가 로딩되는 동안 표시되고, 준비되면 실제 내용으로 교체됩니다.

> **알아두면 좋은 점**: 중첩된 컴포넌트에 대한 로딩 UI를 만들기 위해 `<Suspense>`를 직접 사용할 수도 있습니다.

`loading.tsx`의 장점:

- 사용자를 위한 즉시 네비게이션과 시각적 피드백
- 공유 레이아웃이 인터랙티브하게 유지되고 네비게이션이 중단 가능함
- 개선된 Core Web Vitals: TTFB, FCP, TTI

### 클라이언트 사이드 전환

전통적으로 서버 렌더링된 페이지로의 네비게이션은 전체 페이지 로드를 트리거합니다. 이는 상태를 지우고, 스크롤 위치를 재설정하며, 상호작용을 차단합니다.

Next.js는 `<Link>` 컴포넌트를 사용한 클라이언트 사이드 전환으로 이를 방지합니다. 페이지를 다시 로드하는 대신 다음과 같이 동적으로 내용을 업데이트합니다:

- 공유 레이아웃과 UI를 유지
- 현재 페이지를 프리페치된 로딩 상태 또는 사용 가능한 경우 새 페이지로 교체

클라이언트 사이드 전환은 서버 렌더링된 앱이 클라이언트 렌더링된 앱처럼 *느껴지게* 만드는 것입니다. 그리고 **프리페칭**과 **스트리밍**과 결합되면 동적 라우트에서도 빠른 전환을 가능하게 합니다.

## 전환을 느리게 만드는 요인들

이러한 Next.js 최적화는 네비게이션을 빠르고 반응적으로 만듭니다. 그러나 특정 조건에서는 전환이 여전히 *느리게* 느껴질 수 있습니다. 다음은 일반적인 원인들과 사용자 경험을 개선하는 방법입니다:

### loading.tsx가 없는 동적 라우트

동적 라우트로 네비게이션할 때 클라이언트는 결과를 표시하기 전에 서버 응답을 기다려야 합니다. 이는 사용자에게 앱이 응답하지 않는다는 인상을 줄 수 있습니다.

동적 라우트에 `loading.tsx`를 추가하여 부분 프리페칭을 활성화하고, 즉시 네비게이션을 트리거하며, 라우트가 렌더링되는 동안 로딩 UI를 표시하는 것을 권장합니다.

```tsx
export default function Loading() {
  return <LoadingSkeleton />
}
```

> **알아두면 좋은 점**: 개발 모드에서는 Next.js Devtools를 사용하여 라우트가 정적인지 동적인지 식별할 수 있습니다.

### generateStaticParams가 없는 동적 세그먼트

동적 세그먼트가 사전 렌더링될 수 있지만 `generateStaticParams`가 누락되어 그렇지 않은 경우, 라우트는 요청 시간에 동적 렌더링으로 폴백됩니다.

`generateStaticParams`를 추가하여 빌드 시간에 라우트가 정적으로 생성되도록 보장하세요:

```tsx
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())

  return posts.map((post) => ({
    slug: post.slug,
  }))
}

export default async function Page({ params }) {
  const { slug } = await params
  // ...
}
```

> 💡 **실무 팁**: `generateStaticParams` 사용 시 블로그 포스트나 상품 페이지 같은 미리 알 수 있는 동적 라우트에 적용하되, 너무 많은 정적 페이지 생성은 빌드 시간을 늘릴 수 있으니 핵심 페이지에만 적용하세요. ISR(Incremental Static Regeneration)과 함께 사용하면 더욱 효과적입니다.

### 느린 네트워크

느리거나 불안정한 네트워크에서는 사용자가 링크를 클릭하기 전에 프리페칭이 완료되지 않을 수 있습니다. 이는 정적 및 동적 라우트 모두에 영향을 줄 수 있습니다.

인식되는 성능을 개선하기 위해 `useLinkStatus` 훅을 사용하여 전환이 진행 중일 때 사용자에게 인라인 시각적 피드백을 표시할 수 있습니다:

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

초기 애니메이션 지연을 추가하여 로딩 인디케이터를 "디바운스"할 수 있습니다:

```css
.spinner {
  opacity: 0;
  animation: fadeIn 500ms 100ms forwards, rotate 1s linear infinite;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes rotate {
  to { transform: rotate(360deg); }
}
```

### 프리페칭 비활성화

`<Link>` 컴포넌트의 `prefetch` prop을 `false`로 설정하여 프리페칭을 선택 해제할 수 있습니다:

```tsx
<Link prefetch={false} href="/blog">
  Blog
</Link>
```

그러나 프리페칭을 비활성화하면 트레이드오프가 있습니다:

- **정적 라우트**는 사용자가 링크를 클릭할 때만 가져옵니다
- **동적 라우트**는 클라이언트가 네비게이션하기 전에 먼저 서버에서 렌더링되어야 합니다

호버 시에만 프리페치하는 대안적 접근법:

```tsx
'use client'

import Link from 'next/link'
import { useState } from 'react'

function HoverPrefetchLink({ href, children }) {
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

### 하이드레이션이 완료되지 않음

`<Link>`는 클라이언트 컴포넌트이며 라우트를 프리페치하기 전에 하이드레이션되어야 합니다. 초기 방문에서 큰 JavaScript 번들은 하이드레이션을 지연시킬 수 있습니다.

개선 방법:

- `@next/bundle-analyzer` 플러그인을 사용하여 번들 크기 식별 및 축소
- 가능한 곳에서 클라이언트에서 서버로 로직 이동

> 💡 **실무 팁**: Dynamic import를 사용하여 큰 라이브러리를 지연 로딩하고, Tree shaking이 제대로 작동하는지 확인하며, 서버 컴포넌트를 최대한 활용하여 클라이언트 번들 크기를 줄이세요.

## 예시

### 네이티브 History API

Next.js를 사용하면 페이지를 다시 로드하지 않고 브라우저의 히스토리 스택을 업데이트하기 위해 네이티브 `window.history.pushState`와 `window.history.replaceState` 메서드를 사용할 수 있습니다.

#### window.history.pushState

브라우저의 히스토리 스택에 새 항목을 추가합니다. 상품 목록을 정렬하는 예시:

```tsx
'use client'

import { useSearchParams } from 'next/navigation'

export default function SortProducts() {
  const searchParams = useSearchParams()

  function updateSorting(sortOrder) {
    const params = new URLSearchParams(searchParams.toString())
    params.set('sort', sortOrder)
    window.history.pushState(null, '', `?${params.toString()}`)
  }

  return (
    <>
      <button onClick={() => updateSorting('asc')}>오름차순 정렬</button>
      <button onClick={() => updateSorting('desc')}>내림차순 정렬</button>
    </>
  )
}
```

#### window.history.replaceState

브라우저의 히스토리 스택에서 현재 항목을 교체합니다. 애플리케이션의 로케일을 전환하는 예시:

```tsx
'use client'

import { usePathname } from 'next/navigation'

export function LocaleSwitcher() {
  const pathname = usePathname()

  function switchLocale(locale) {
    const newPath = `/${locale}${pathname}`
    window.history.replaceState(null, '', newPath)
  }

  return (
    <>
      <button onClick={() => switchLocale('en')}>English</button>
      <button onClick={() => switchLocale('fr')}>French</button>
    </>
  )
}
```

> 💡 **실무 팁**: History API는 필터링이나 정렬 상태를 URL에 반영하거나, 모달/탭 상태를 URL로 관리하여 뒤로가기 버튼이 자연스럽게 작동하도록 할 때 매우 유용합니다.

---

## 관련 문서

- [Link 컴포넌트](/docs/app/api-reference/components/link.md) - 내장 `next/link` 컴포넌트로 빠른 클라이언트 사이드 네비게이션 활성화
- [loading.js](/docs/app/api-reference/file-conventions/loading.md) - loading.js 파일의 API 참조
- [프리페칭](/docs/app/guides/prefetching.md) - Next.js에서 프리페칭을 구성하는 방법 학습

---

**원문**: [Next.js 공식 문서 - Linking and Navigating](https://nextjs.org/docs/app/getting-started/linking-and-navigating)