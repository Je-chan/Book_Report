# 캐싱과 재검증

캐싱은 데이터 패칭 및 기타 계산 결과를 저장하여 동일한 데이터에 대한 향후 요청을 다시 작업을 수행하지 않고도 더 빠르게 처리할 수 있는 기술입니다. 재검증을 통해서는 전체 애플리케이션을 다시 빌드하지 않고도 캐시 항목을 업데이트할 수 있습니다.

Next.js는 캐싱과 재검증을 처리하기 위한 몇 가지 API를 제공합니다. 이 가이드에서는 언제, 어떻게 사용하는지를 안내합니다.

- [`fetch`](#fetch)
- [`unstable_cache`](#unstable_cache)
- [`revalidatePath`](#revalidatepath)
- [`revalidateTag`](#revalidatetag)

## `fetch`

기본적으로 `fetch` 요청은 캐시되지 않습니다. `cache` 옵션을 `'force-cache'`로 설정하여 개별 요청을 캐시할 수 있습니다.

```tsx
export default async function Page() {
  const data = await fetch('https://...', { cache: 'force-cache' })
}
```

> **알아두면 좋은 점**: 기본적으로 `fetch` 요청은 캐시되지 않지만, Next.js는 `fetch` 요청이 있는 라우트를 사전 렌더링하고 HTML을 캐시합니다. 라우트가 동적임을 보장하려면 `connection` API를 사용하세요.

`fetch` 요청에서 반환된 데이터를 재검증하려면 `next.revalidate` 옵션을 사용할 수 있습니다.

```tsx
export default async function Page() {
  const data = await fetch('https://...', { next: { revalidate: 3600 } })
}
```

이렇게 하면 지정된 시간(초) 후에 데이터가 재검증됩니다.

> 💡 **실무 팁**
>
> **fetch 캐싱 전략:**
> - **정적 데이터**: 자주 변경되지 않는 데이터는 `force-cache`를 사용하세요
> - **시간 기반 재검증**: 정기적으로 업데이트되는 데이터는 적절한 `revalidate` 시간을 설정하세요
> - **캐시 제어**: API 응답의 성격에 따라 캐시 전략을 다르게 적용하세요

## `unstable_cache`

`unstable_cache`를 사용하면 데이터베이스 쿼리와 기타 비동기 함수의 결과를 캐시할 수 있습니다. 사용하려면 함수를 `unstable_cache`로 감싸세요. 예를 들어:

```tsx
// app/lib/data.ts
import { db } from '@/lib/db'

export async function getUserById(id: string) {
  return db
    .select()
    .from(users)
    .where(eq(users.id, id))
    .then((res) => res[0])
}
```

```tsx
// app/page.tsx
import { unstable_cache } from 'next/cache'
import { getUserById } from '@/app/lib/data'

export default async function Page({
  params,
}: {
  params: Promise<{ userId: string }>
}) {
  const { userId } = await params

  const getCachedUser = unstable_cache(
    async () => {
      return getUserById(userId)
    },
    [userId] // 캐시 키에 사용자 ID 추가
  )
}
```

이 함수는 캐시가 어떻게 재검증되어야 하는지 정의하는 세 번째 선택적 객체를 받습니다. 다음을 포함합니다:

- `tags`: Next.js가 캐시를 재검증하는 데 사용하는 태그 배열
- `revalidate`: 캐시가 재검증되어야 하는 시간(초)

```tsx
const getCachedUser = unstable_cache(
  async () => {
    return getUserById(userId)
  },
  [userId],
  {
    tags: ['user'],
    revalidate: 3600,
  }
)
```

> 💡 **실무 팁**
>
> **unstable_cache 활용법:**
> - **데이터베이스 쿼리**: 복잡하고 비용이 많이 드는 데이터베이스 쿼리에 적용하세요
> - **외부 API 호출**: 서드파티 API 호출 결과를 캐시하여 응답 속도를 개선하세요
> - **태그 전략**: 관련된 데이터끼리 같은 태그를 사용하여 일괄 재검증하세요

## `revalidateTag`

`revalidateTag`는 태그를 기반으로 하여 이벤트에 따라 캐시 항목을 재검증하는 데 사용됩니다. `fetch`와 함께 사용하려면 먼저 `next.tags` 옵션으로 함수에 태그를 지정하세요:

```tsx
export async function getUserById(id: string) {
  const data = await fetch(`https://...`, {
    next: {
      tags: ['user'],
    },
  })
}
```

또는 `tags` 옵션으로 `unstable_cache` 함수를 표시할 수 있습니다:

```tsx
export const getUserById = unstable_cache(
  async (id: string) => {
    return db.query.users.findFirst({ where: eq(users.id, id) })
  },
  ['user'], // 변수가 매개변수로 전달되지 않는 경우 필요
  {
    tags: ['user'],
  }
)
```

그런 다음 Route Handler 또는 Server Action에서 `revalidateTag`를 호출하세요:

```tsx
import { revalidateTag } from 'next/cache'

export async function updateUser(id: string) {
  // 데이터 변경
  revalidateTag('user')
}
```

여러 함수에서 동일한 태그를 재사용하여 모든 함수를 한 번에 재검증할 수 있습니다.

> **태그 기반 재검증의 장점**
> - **세밀한 제어**: 특정 데이터 타입과 관련된 캐시만 선택적으로 무효화
> - **효율성**: 변경된 데이터와 관련된 캐시만 재검증하여 성능 최적화
> - **일관성**: 관련된 모든 캐시를 동시에 업데이트하여 데이터 일관성 보장

## `revalidatePath`

`revalidatePath`는 이벤트에 따라 라우트를 재검증하는 데 사용됩니다. 사용하려면 Route Handler 또는 Server Action에서 호출하세요:

```tsx
import { revalidatePath } from 'next/cache'

export async function updateUser(id: string) {
  // 데이터 변경
  revalidatePath('/profile')
}
```

> 💡 **실무 팁**
>
> **재검증 전략 선택 가이드:**
> - **revalidateTag**: 여러 페이지에 걸쳐 동일한 데이터가 사용될 때
> - **revalidatePath**: 특정 페이지나 레이아웃의 모든 데이터를 새로고침할 때
> - **조합 사용**: 복잡한 애플리케이션에서는 두 방법을 조합하여 사용하세요

## API 참조

이 페이지에서 언급된 기능에 대해 API 참조를 읽어 자세히 알아보세요.

- **fetch** - 확장된 fetch 함수에 대한 API 참조
- **unstable_cache** - unstable_cache 함수에 대한 API 참조
- **revalidatePath** - revalidatePath 함수에 대한 API 참조
- **revalidateTag** - revalidateTag 함수에 대한 API 참조

> 📚 **캐싱 전략 종합 가이드**
>
> **효과적인 캐싱을 위한 종합 전략:**
> - **계층별 캐싱**: CDN, Next.js 캐시, 데이터베이스 캐시를 계층적으로 활용하세요
> - **캐시 무효화 패턴**: 데이터 변경 시점을 명확히 파악하고 적절한 재검증 전략을 수립하세요
> - **성능 모니터링**: 캐시 히트율과 응답 시간을 지속적으로 모니터링하세요
> - **사용자 경험**: 캐시로 인한 데이터 지연을 최소화하고 일관된 UX를 제공하세요

---

**원문**: [Next.js 공식 문서 - Caching and Revalidating](https://nextjs.org/docs/app/getting-started/caching-and-revalidating)