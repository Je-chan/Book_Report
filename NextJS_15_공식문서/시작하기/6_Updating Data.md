# 데이터 업데이트

React의 **서버 함수(Server Functions)**를 사용하여 Next.js에서 데이터를 업데이트할 수 있습니다. 이 페이지에서는 서버 함수를 [생성](#서버-함수-생성하기)하고 [호출](#서버-함수-호출하기)하는 방법을 설명합니다.

## 서버 함수 생성하기

`use server` 지시어를 사용하여 서버 함수를 정의할 수 있습니다. 비동기 함수의 상단에 지시어를 배치하여 해당 함수를 서버 함수로 표시하거나, 별도 파일의 상단에 배치하여 해당 파일의 모든 내보내기를 서버 함수로 표시할 수 있습니다.

```typescript
export async function createPost(formData: FormData) {
  'use server'
  const title = formData.get('title')
  const content = formData.get('content')
  // 데이터 업데이트
  // 캐시 재검증
}

export async function deletePost(formData: FormData) {
  'use server'
  const id = formData.get('id')
  // 데이터 업데이트
  // 캐시 재검증
}
```

### 서버 컴포넌트

함수 본문의 상단에 `"use server"` 지시어를 추가하여 서버 컴포넌트 내에서 서버 함수를 인라인으로 정의할 수 있습니다:

```typescript
export default function Page() {
  // 서버 액션
  async function createPost(formData: FormData) {
    'use server'
    // ...
  }

  return <></>
}
```

### 클라이언트 컴포넌트

클라이언트 컴포넌트에서는 서버 함수를 정의할 수 없습니다. 하지만 파일 상단에 `"use server"` 지시어가 있는 파일에서 가져와서 클라이언트 컴포넌트에서 호출할 수 있습니다:

```typescript
// actions.ts
'use server'

export async function createPost() {}
```

```typescript
// Button.tsx
'use client'

import { createPost } from '@/app/actions'

export function Button() {
  return <button formAction={createPost}>Create</button>
}
```

> 💡 **실무 팁**
>
> **서버 함수 구성 전략:**
> - **파일 분리**: 여러 서버 함수를 `actions.ts` 파일에 모아두고 상단에 `'use server'`를 선언하세요
> - **타입 안정성**: TypeScript를 사용한다면 FormData의 타입을 명시적으로 정의하여 런타임 오류를 방지하세요
> - **에러 처리**: 서버 함수에서는 try-catch를 활용한 에러 처리를 필수로 구현하세요

## 서버 함수 호출하기

서버 함수를 호출하는 두 가지 주요 방법이 있습니다:

- **폼(Forms)** - 서버 및 클라이언트 컴포넌트에서
- **이벤트 핸들러(Event Handlers)** - 클라이언트 컴포넌트에서

### 폼

React는 HTML `<form>` 요소를 확장하여 HTML `action` prop으로 서버 함수를 호출할 수 있게 합니다.

폼에서 호출되면 함수는 자동으로 `FormData` 객체를 받습니다. 네이티브 `FormData` 메서드를 사용하여 데이터를 추출할 수 있습니다:

```typescript
import { createPost } from '@/app/actions'

export function Form() {
  return (
    <form action={createPost}>
      <input type="text" name="title" />
      <input type="text" name="content" />
      <button type="submit">Create</button>
    </form>
  )
}
```

```typescript
'use server'

export async function createPost(formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')
  // 데이터 업데이트
  // 캐시 재검증
}
```

> **알아두면 좋은 점**: `action` prop에 전달될 때, 서버 함수는 서버 액션(Server Actions)이라고도 합니다.

### 이벤트 핸들러

`onClick`과 같은 이벤트 핸들러를 사용하여 클라이언트 컴포넌트에서 서버 함수를 호출할 수 있습니다.

```typescript
'use client'

import { incrementLike } from './actions'
import { useState } from 'react'

export default function LikeButton({ initialLikes }: { initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes)

  return (
    <>
      <p>Total Likes: {likes}</p>
      <button
        onClick={async () => {
          const updatedLikes = await incrementLike()
          setLikes(updatedLikes)
        }}
      >
        Like
      </button>
    </>
  )
}
```

> 💡 **실무 팁**
>
> **이벤트 핸들러에서 서버 함수 사용 시 주의사항:**
> - **비동기 처리**: 서버 함수 호출은 항상 `async/await`를 사용하세요
> - **로딩 상태**: 사용자 경험을 위해 로딩 인디케이터를 추가하세요
> - **에러 핸들링**: 네트워크 오류나 서버 오류에 대한 적절한 처리를 구현하세요

## 예시

### 대기 상태 표시하기

서버 함수를 실행하는 동안 React의 `useActionState` 훅을 사용하여 로딩 인디케이터를 표시할 수 있습니다. 이 훅은 `pending` 불린값을 반환합니다:

```typescript
'use client'

import { useActionState } from 'react'
import { createPost } from '@/app/actions'
import { LoadingSpinner } from '@/app/ui/loading-spinner'

export function Button() {
  const [state, action, pending] = useActionState(createPost, false)

  return (
    <button onClick={async () => action()}>
      {pending ? <LoadingSpinner /> : 'Create Post'}
    </button>
  )
}
```

### 캐시 재검증

업데이트를 수행한 후 서버 함수 내에서 `revalidatePath` 또는 `revalidateTag`를 호출하여 Next.js 캐시를 재검증하고 업데이트된 데이터를 표시할 수 있습니다:

```typescript
import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  'use server'
  // 데이터 업데이트
  // ...
  revalidatePath('/posts')
}
```

> **캐시 재검증 전략**
> - **revalidatePath**: 특정 경로의 캐시를 무효화합니다
> - **revalidateTag**: 특정 태그로 표시된 모든 캐시를 무효화합니다
> - 데이터 변경 후 관련된 페이지들이 최신 데이터를 표시할 수 있도록 적절한 재검증이 필요합니다

### 리디렉션

업데이트를 수행한 후 사용자를 다른 페이지로 리디렉션하고 싶을 수 있습니다. 서버 함수 내에서 `redirect`를 호출하여 이를 수행할 수 있습니다:

```typescript
'use server'

import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  // 데이터 업데이트
  // ...
  redirect('/posts')
}
```

> 💡 **실무 팁**
>
> **서버 함수 완성 패턴:**
> - **순서 준수**: 데이터 업데이트 → 캐시 재검증 → 리디렉션 순서로 진행하세요
> - **에러 케이스**: 데이터 업데이트 실패 시 사용자에게 적절한 피드백을 제공하세요
> - **낙관적 업데이트**: 빠른 사용자 경험을 위해 낙관적 업데이트 패턴을 고려해보세요

---

**원문**: [Next.js 공식 문서 - Updating Data](https://nextjs.org/docs/app/getting-started/updating-data)