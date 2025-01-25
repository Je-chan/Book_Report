# 에러 처리

에러는 두 가지 범주로 나눌 수 있습니다: [예상 에러](#예상-에러-처리)와 [포착되지 않은 예외](#포착되지-않은-예외-처리)입니다. 이 페이지에서는 Next.js 애플리케이션에서 이러한 에러들을 어떻게 처리하는지 안내합니다.

## 예상 에러 처리

예상 에러는 서버 측 폼 검증이나 실패한 요청과 같이 애플리케이션의 정상적인 작동 중에 발생할 수 있는 에러입니다. 이러한 에러는 명시적으로 처리되어 클라이언트에 반환되어야 합니다.

### 서버 함수

서버 함수에서 예상 에러를 처리하기 위해 `useActionState` 훅을 사용할 수 있습니다.

이러한 에러의 경우, `try`/`catch` 블록을 사용하고 에러를 던지는 것을 피하세요. 대신 예상 에러를 반환값으로 모델링하세요.

```ts
'use server'

export async function createPost(prevState: any, formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')

  const res = await fetch('https://api.vercel.app/posts', {
    method: 'POST',
    body: { title, content },
  })
  const json = await res.json()

  if (!res.ok) {
    return { message: 'Failed to create post' }
  }
}
```

액션을 `useActionState` 훅에 전달하고 반환된 `state`를 사용하여 에러 메시지를 표시할 수 있습니다.

```tsx
'use client'

import { useActionState } from 'react'
import { createPost } from '@/app/actions'

const initialState = {
  message: '',
}

export function Form() {
  const [state, formAction, pending] = useActionState(createPost, initialState)

  return (
    <form action={formAction}>
      <label htmlFor="title">Title</label>
      <input type="text" id="title" name="title" required />
      <label htmlFor="content">Content</label>
      <textarea id="content" name="content" required />
      {state?.message && <p aria-live="polite">{state.message}</p>}
      <button disabled={pending}>Create Post</button>
    </form>
  )
}
```

> 💡 **실무 팁**
>
> **예상 에러 처리 베스트 프랙티스:**
> - **구조적 에러 모델링**: 에러 상태를 객체로 반환하여 다양한 에러 타입을 구분하세요
> - **사용자 친화적 메시지**: 기술적 에러보다는 사용자가 이해할 수 있는 메시지를 제공하세요
> - **접근성 고려**: `aria-live` 속성을 사용하여 스크린 리더 사용자도 에러를 인지할 수 있게 하세요

### 서버 컴포넌트

서버 컴포넌트 내에서 데이터를 가져올 때, 응답을 사용하여 조건부로 에러 메시지를 렌더링하거나 `redirect`할 수 있습니다.

```tsx
export default async function Page() {
  const res = await fetch(`https://...`)
  const data = await res.json()

  if (!res.ok) {
    return 'There was an error.'
  }

  return '...'
}
```

### Not found

라우트 세그먼트 내에서 `notFound` 함수를 호출하고 `not-found.js` 파일을 사용하여 404 UI를 표시할 수 있습니다.

```tsx
import { getPostBySlug } from '@/lib/posts'

export default async function Page({ params }: { params: { slug: string } }) {
  const { slug } = await params
  const post = getPostBySlug(slug)

  if (!post) {
    notFound()
  }

  return <div>{post.title}</div>
}
```

```tsx
// app/blog/[slug]/not-found.tsx
export default function NotFound() {
  return <div>404 - Page Not Found</div>
}
```

> **404 페이지 커스터마이징**
>
> `not-found.js` 파일을 통해 브랜드에 맞는 404 페이지를 만들 수 있습니다. 사용자가 길을 찾을 수 있도록 네비게이션 링크나 검색 기능을 포함하는 것이 좋습니다.

## 포착되지 않은 예외 처리

포착되지 않은 예외는 애플리케이션의 정상적인 흐름 중에 발생해서는 안 되는 버그나 문제를 나타내는 예상치 못한 에러입니다. 이러한 에러는 에러를 던져서 처리해야 하며, 에러 바운더리에 의해 포착됩니다.

### 중첩된 에러 바운더리

Next.js는 에러 바운더리를 사용하여 포착되지 않은 예외를 처리합니다. 에러 바운더리는 자식 컴포넌트의 에러를 포착하고 충돌한 컴포넌트 트리 대신 폴백 UI를 표시합니다.

라우트 세그먼트 내에 `error.js` 파일을 추가하고 React 컴포넌트를 내보내서 에러 바운더리를 만드세요:

```tsx
'use client' // 에러 바운더리는 클라이언트 컴포넌트여야 합니다

import { useEffect } from 'react'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // 에러 리포팅 서비스에 에러 로그를 기록
    console.error(error)
  }, [error])

  return (
    <div>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // 세그먼트를 다시 렌더링하여 복구 시도
          () => reset()
        }
      >
        Try again
      </button>
    </div>
  )
}
```

에러는 가장 가까운 부모 에러 바운더리로 버블링됩니다. 이를 통해 라우트 계층 구조의 다른 레벨에 `error.tsx` 파일을 배치하여 세분화된 에러 처리가 가능합니다.

> **에러 바운더리의 한계**
>
> 에러 바운더리는 이벤트 핸들러 내부의 에러는 포착하지 않습니다. 렌더링 중 에러를 포착하여 전체 앱이 충돌하는 대신 **폴백 UI**를 표시하도록 설계되었습니다. 일반적으로 이벤트 핸들러나 비동기 코드의 에러는 렌더링 후에 실행되므로 에러 바운더리에서 처리되지 않습니다.

이러한 경우를 처리하려면 에러를 수동으로 포착하고 `useState` 또는 `useReducer`를 사용하여 저장한 다음, UI를 업데이트하여 사용자에게 알리세요.

```tsx
'use client'

import { useState } from 'react'

export function Button() {
  const [error, setError] = useState(null)

  const handleClick = () => {
    try {
      // 실패할 수 있는 작업 수행
      throw new Error('Exception')
    } catch (reason) {
      setError(reason)
    }
  }

  if (error) {
    /* 폴백 UI 렌더링 */
  }

  return (
    <button type="button" onClick={handleClick}>
      Click me
    </button>
  )
}
```

`useTransition`의 `startTransition` 내부에서 처리되지 않은 에러는 가장 가까운 에러 바운더리로 버블링됩니다.

```tsx
'use client'

import { useTransition } from 'react'

export function Button() {
  const [pending, startTransition] = useTransition()

  const handleClick = () =>
    startTransition(() => {
      throw new Error('Exception')
    })

  return (
    <button type="button" onClick={handleClick}>
      Click me
    </button>
  )
}
```

> 💡 **실무 팁**
>
> **에러 바운더리 구현 전략:**
> - **계층적 배치**: 페이지, 레이아웃, 컴포넌트 레벨에서 적절히 에러 바운더리를 배치하세요
> - **에러 리포팅**: Sentry, LogRocket 등의 서비스를 통해 프로덕션 에러를 모니터링하세요
> - **복구 메커니즘**: reset 함수를 활용하여 사용자가 에러 상태에서 쉽게 복구할 수 있게 하세요

### 전역 에러

흔하지는 않지만, 루트 앱 디렉토리에 위치한 `global-error.js` 파일을 사용하여 루트 레이아웃의 에러를 처리할 수 있습니다. 이는 국제화를 활용할 때에도 가능합니다. 전역 에러 UI는 활성화될 때 루트 레이아웃이나 템플릿을 대체하므로 자체 `<html>` 및 `<body>` 태그를 정의해야 합니다.

```tsx
'use client' // 에러 바운더리는 클라이언트 컴포넌트여야 합니다

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    // global-error는 html과 body 태그를 포함해야 합니다
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  )
}
```

> **전역 에러 처리 주의사항**
>
> 전역 에러는 가장 최후의 안전장치입니다. 가능하면 더 세분화된 에러 바운더리를 사용하여 사용자 경험을 향상시키고, 애플리케이션의 다른 부분이 정상적으로 작동할 수 있도록 하세요.

## API 참조

이 페이지에서 언급된 기능에 대해 API 참조를 읽어 자세히 알아보세요.

- **redirect** - redirect 함수에 대한 API 참조
- **error.js** - error.js 특수 파일에 대한 API 참조
- **notFound** - notFound 함수에 대한 API 참조
- **not-found.js** - not-found.js 파일에 대한 API 참조

> 📚 **종합 에러 처리 전략**
>
> **프로덕션 레벨 에러 처리 가이드:**
> - **예방적 접근**: TypeScript, 린팅, 테스트를 통해 에러를 사전에 방지하세요
> - **단계별 처리**: 예상 에러는 반환값으로, 예상치 못한 에러는 에러 바운더리로 처리하세요
> - **사용자 경험**: 에러 발생 시에도 사용자가 앱을 계속 사용할 수 있도록 적절한 폴백을 제공하세요
> - **모니터링과 개선**: 에러 발생 패턴을 분석하여 지속적으로 애플리케이션을 개선하세요

---

**원문**: [Next.js 공식 문서 - Error Handling](https://nextjs.org/docs/app/getting-started/error-handling)