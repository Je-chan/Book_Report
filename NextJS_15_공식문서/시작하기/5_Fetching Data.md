# 데이터 패칭

이 페이지에서는 서버 및 클라이언트 컴포넌트에서 데이터를 가져오는 방법과 데이터에 의존하는 컴포넌트를 **스트리밍**하는 방법을 안내합니다.

## 데이터 패칭

### 서버 컴포넌트

서버 컴포넌트에서는 다음을 사용하여 데이터를 가져올 수 있습니다:

1. `fetch` API
2. ORM 또는 데이터베이스

#### fetch API 사용하기

`fetch` API로 데이터를 가져오려면 컴포넌트를 비동기 함수로 만들고 `fetch` 호출을 await하세요. 예시:

```tsx
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

> **알아두면 좋은 점:**
>
> - `fetch` 응답은 기본적으로 캐시되지 않습니다. 그러나 Next.js는 라우트를 사전 렌더링하고 성능 향상을 위해 출력을 캐시합니다.
> - 동적 렌더링을 원한다면 `{ cache: 'no-store' }` 옵션을 사용하세요.
> - 개발 중에는 가시성과 디버깅을 위해 `fetch` 호출을 로그할 수 있습니다.

#### ORM 또는 데이터베이스 사용하기

서버 컴포넌트는 서버에서 렌더링되므로 ORM이나 데이터베이스 클라이언트를 사용하여 안전하게 데이터베이스 쿼리를 수행할 수 있습니다. 컴포넌트를 비동기 함수로 만들고 호출을 await하세요:

```tsx
import { db, posts } from '@/lib/db'

export default async function Page() {
  const allPosts = await db.select().from(posts)
  return (
    <ul>
      {allPosts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

> 💡 **실무 팁**: 서버 컴포넌트에서 데이터 패칭 최적화를 위해 **연결 풀링**을 설정하여 연결 오버헤드를 줄이고, **쿼리 최적화**로 필요한 필드만 선택하며, **캐싱 전략**을 적절히 적용하고, **에러 처리**를 구현하세요.

### 클라이언트 컴포넌트

클라이언트 컴포넌트에서는 다음 두 가지 방법으로 데이터를 가져올 수 있습니다:

1. React의 `use` 훅
2. SWR이나 React Query 같은 커뮤니티 라이브러리

#### use 훅으로 데이터 스트리밍하기

React의 `use` 훅을 사용하여 서버에서 클라이언트로 데이터를 스트리밍할 수 있습니다. 서버 컴포넌트에서 데이터를 가져와서 프로미스를 클라이언트 컴포넌트에 prop으로 전달하는 것으로 시작하세요:

```tsx
import Posts from '@/app/ui/posts
import { Suspense } from 'react'

export default function Page() {
  // 데이터 패칭 함수를 await하지 마세요
  const posts = getPosts()

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Posts posts={posts} />
    </Suspense>
  )
}
```

그런 다음 클라이언트 컴포넌트에서 `use` 훅을 사용하여 프로미스를 읽으세요:

```tsx
'use client'
import { use } from 'react'

export default function Posts({ posts }) {
  const allPosts = use(posts)

  return (
    <ul>
      {allPosts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

위 예시에서 `<Posts>` 컴포넌트는 `<Suspense>` 경계로 감싸져 있습니다. 이는 프로미스가 해결되는 동안 폴백이 표시됨을 의미합니다.

#### 커뮤니티 라이브러리

클라이언트 컴포넌트에서 SWR이나 React Query 같은 커뮤니티 라이브러리를 사용하여 데이터를 가져올 수 있습니다. 이러한 라이브러리는 캐싱, 스트리밍 및 기타 기능에 대한 자체 의미론을 가지고 있습니다. SWR 예시:

```tsx
'use client'
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then((r) => r.json())

export default function BlogPage() {
  const { data, error, isLoading } = useSWR(
    'https://api.vercel.app/blog',
    fetcher
  )

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>

  return (
    <ul>
      {data.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

> 💡 **실무 팁**: 클라이언트 데이터 패칭 라이브러리 선택 시 **SWR**은 간단하고 가벼우며 소규모~중규모 프로젝트에 적합하고, **React Query (TanStack Query)**는 더 많은 기능과 세밀한 제어로 복잡한 요구사항이 있는 대규모 프로젝트에 적합합니다.

## 요청 중복 제거 및 데이터 캐시

`fetch` 요청을 중복 제거하는 한 가지 방법은 요청 메모이제이션을 사용하는 것입니다. 이 메커니즘을 통해 단일 렌더링 패스에서 동일한 URL과 옵션을 가진 `GET` 또는 `HEAD`를 사용하는 `fetch` 호출이 하나의 요청으로 결합됩니다.

요청 메모이제이션은 요청의 수명에 범위가 지정됩니다.

Next.js의 데이터 캐시를 사용하여 `fetch` 요청을 중복 제거할 수도 있습니다. 예를 들어 `fetch` 옵션에서 `cache: 'force-cache'`를 설정하는 것입니다.

데이터 캐시는 현재 렌더링 패스와 들어오는 요청에 걸쳐 데이터를 공유할 수 있게 합니다.

`fetch`를 사용하지 않고 ORM이나 데이터베이스를 직접 사용하는 경우, React `cache` 함수로 데이터 액세스를 감쌀 수 있습니다:

```tsx
import { cache } from 'react'
import { db, posts, eq } from '@/lib/db'

export const getPost = cache(async (id) => {
  const post = await db.query.posts.findFirst({
    where: eq(posts.id, parseInt(id)),
  })
})
```

> 💡 **실무 팁**: 데이터 캐싱 전략으로 **Request-level 캐싱** (React `cache`로 컴포넌트 트리 내 중복 제거), **Route-level 캐싱** (Next.js 데이터 캐시로 여러 요청에 걸쳐 데이터 공유), **Application-level 캐싱** (Redis/Memcached 활용), **CDN 캐싱** (정적 데이터/이미지)을 적절히 조합하세요.

## 스트리밍

서버 컴포넌트에서 `async/await`를 사용할 때 Next.js는 동적 렌더링을 선택합니다. 이는 모든 사용자 요청에 대해 데이터가 서버에서 가져와지고 렌더링됨을 의미합니다. 느린 데이터 요청이 있으면 전체 라우트가 렌더링에서 차단됩니다.

초기 로드 시간과 사용자 경험을 개선하기 위해 스트리밍을 사용하여 페이지의 HTML을 더 작은 청크로 나누고 서버에서 클라이언트로 점진적으로 전송할 수 있습니다.

애플리케이션에서 스트리밍을 구현하는 두 가지 방법이 있습니다:

1. `loading.js` 파일로 페이지 감싸기
2. `<Suspense>`로 컴포넌트 감싸기

### loading.js 사용하기

데이터를 가져오는 동안 **전체 페이지**를 스트리밍하기 위해 페이지와 같은 폴더에 `loading.js` 파일을 만들 수 있습니다.

```tsx
export default function Loading() {
  // 여기에 로딩 UI를 정의하세요
  return <div>Loading...</div>
}
```

네비게이션 시 사용자는 페이지가 렌더링되는 동안 즉시 레이아웃과 로딩 상태를 보게 됩니다. 렌더링이 완료되면 새 콘텐츠가 자동으로 교체됩니다.

내부적으로 `loading.js`는 `layout.js` 안에 중첩되고, `page.js` 파일과 그 아래의 모든 자식을 `<Suspense>` 경계로 자동으로 감쌉니다.

이 접근 방식은 라우트 세그먼트(레이아웃과 페이지)에 잘 작동하지만, 더 세밀한 스트리밍을 위해서는 `<Suspense>`를 사용할 수 있습니다.

### Suspense 사용하기

`<Suspense>`를 사용하면 페이지의 어떤 부분을 스트리밍할지 더 세밀하게 제어할 수 있습니다. 예를 들어, `<Suspense>` 경계 밖의 페이지 콘텐츠는 즉시 표시하고, 경계 안의 블로그 포스트 목록은 스트리밍할 수 있습니다:

```tsx
import { Suspense } from 'react'
import BlogList from '@/components/BlogList'
import BlogListSkeleton from '@/components/BlogListSkeleton'

export default function BlogPage() {
  return (
    <div>
      {/* 이 콘텐츠는 클라이언트에 즉시 전송됩니다 */}
      <header>
        <h1>블로그에 오신 것을 환영합니다</h1>
        <p>아래에서 최신 포스트를 읽어보세요.</p>
      </header>
      <main>
        {/* <Suspense> 경계로 감싼 콘텐츠는 스트리밍됩니다 */}
        <Suspense fallback={<BlogListSkeleton />}>
          <BlogList />
        </Suspense>
      </main>
    </div>
  )
}
```

### 의미 있는 로딩 상태 만들기

즉시 로딩 상태는 네비게이션 후 사용자에게 즉시 표시되는 폴백 UI입니다. 최상의 사용자 경험을 위해 의미 있고 앱이 응답하고 있음을 사용자가 이해할 수 있는 로딩 상태를 설계하는 것을 권장합니다. 예를 들어 스켈레톤, 스피너, 또는 커버 사진, 제목 등과 같은 미래 화면의 작지만 의미 있는 부분을 사용할 수 있습니다.

개발 중에는 React Devtools를 사용하여 컴포넌트의 로딩 상태를 미리보고 검사할 수 있습니다.

> 💡 **실무 팁**: 효과적인 로딩 상태 디자인을 위해 **스켈레톤 UI** (실제 콘텐츠 구조와 유사한 스켈레톤), **프로그레시브 로딩** (중요한 콘텐츠부터 순차 로드), **에러 상태** (재시도 버튼과 명확한 에러 메시지), **로딩 애니메이션** (적절한 시각적 피드백)을 고려하세요.

## 예시

### 순차적 데이터 패칭

순차적 데이터 패칭은 트리의 중첩된 컴포넌트가 각각 자체 데이터를 가져오고 요청이 중복 제거되지 않을 때 발생하여 응답 시간이 길어집니다.

한 패치가 다른 패치의 결과에 의존하기 때문에 이 패턴을 원하는 경우가 있을 수 있습니다.

예를 들어, `<Playlists>` 컴포넌트는 `<Artist>` 컴포넌트가 데이터 가져오기를 완료한 후에만 데이터 가져오기를 시작합니다. `<Playlists>`가 `artistID` prop에 의존하기 때문입니다:

```tsx
export default async function Page({ params }) {
  const { username } = await params
  // 아티스트 정보 가져오기
  const artist = await getArtist(username)

  return (
    <>
      <h1>{artist.name}</h1>
      {/* Playlists 컴포넌트가 로딩되는 동안 폴백 UI 표시 */}
      <Suspense fallback={<div>Loading...</div>}>
        {/* 아티스트 ID를 Playlists 컴포넌트에 전달 */}
        <Playlists artistID={artist.id} />
      </Suspense>
    </>
  )
}

async function Playlists({ artistID }) {
  // 아티스트 ID를 사용하여 플레이리스트 가져오기
  const playlists = await getArtistPlaylists(artistID)

  return (
    <ul>
      {playlists.map((playlist) => (
        <li key={playlist.id}>{playlist.name}</li>
      ))}
    </ul>
  )
}
```

사용자 경험을 개선하려면 데이터를 가져오는 동안 `fallback`을 표시하기 위해 React `<Suspense>`를 사용해야 합니다. 이렇게 하면 스트리밍이 활성화되고 순차적 데이터 요청에 의해 전체 라우트가 차단되는 것을 방지할 수 있습니다.

### 병렬 데이터 패칭

병렬 데이터 패칭은 라우트의 데이터 요청이 적극적으로 시작되고 동시에 시작될 때 발생합니다.

기본적으로 레이아웃과 페이지는 병렬로 렌더링됩니다. 따라서 각 세그먼트는 가능한 한 빨리 데이터 가져오기를 시작합니다.

그러나 *모든* 컴포넌트 내에서 여러 `async`/`await` 요청이 다른 것 뒤에 배치되면 여전히 순차적일 수 있습니다. 예를 들어, `getAlbums`는 `getArtist`가 해결될 때까지 차단됩니다:

```tsx
import { getArtist, getAlbums } from '@/app/lib/data'

export default async function Page({ params }) {
  // 이 요청들은 순차적입니다
  const { username } = await params
  const artist = await getArtist(username)
  const albums = await getAlbums(username)
  return <div>{artist.name}</div>
}
```

`Promise.all`을 사용하여 요청을 병렬로 시작할 수 있습니다:

```tsx
import Albums from './albums'

async function getArtist(username) {
  const res = await fetch(`https://api.example.com/artist/${username}`)
  return res.json()
}

async function getAlbums(username) {
  const res = await fetch(`https://api.example.com/artist/${username}/albums`)
  return res.json()
}

export default async function Page({ params }) {
  const { username } = await params
  const artistData = getArtist(username)
  const albumsData = getAlbums(username)

  // 두 요청을 병렬로 시작
  const [artist, albums] = await Promise.all([artistData, albumsData])

  return (
    <>
      <h1>{artist.name}</h1>
      <Albums list={albums} />
    </>
  )
}
```

> **알아두면 좋은 점**: `Promise.all`을 사용할 때 하나의 요청이 실패하면 전체 작업이 실패합니다. 이를 처리하려면 대신 `Promise.allSettled` 메서드를 사용할 수 있습니다.

### 데이터 프리로딩

차단 요청 위에서 적극적으로 호출하는 유틸리티 함수를 만들어 데이터를 프리로드할 수 있습니다. `<Item>`은 `checkIsAvailable()` 함수에 따라 조건부로 렌더링됩니다.

`checkIsAvailable()` 전에 `preload()`를 호출하여 `<Item/>` 데이터 의존성을 적극적으로 시작할 수 있습니다. `<Item/>`이 렌더링될 때까지 데이터가 이미 가져와진 상태입니다.

```tsx
import { getItem, checkIsAvailable } from '@/lib/data'

export default async function Page({ params }) {
  const { id } = await params
  // 아이템 데이터 로딩 시작
  preload(id)
  // 다른 비동기 작업 수행
  const isAvailable = await checkIsAvailable()

  return isAvailable ? <Item id={id} /> : null
}

export const preload = (id) => {
  // void는 주어진 표현식을 평가하고 undefined를 반환합니다
  void getItem(id)
}

export async function Item({ id }) {
  const result = await getItem(id)
  // ...
}
```

추가로 React의 `cache` 함수와 `server-only` 패키지를 사용하여 재사용 가능한 유틸리티 함수를 만들 수 있습니다. 이 접근 방식을 통해 데이터 가져오기 함수를 캐시하고 서버에서만 실행되도록 보장할 수 있습니다.

```ts
import { cache } from 'react'
import 'server-only'
import { getItem } from '@/lib/data'

export const preload = (id) => {
  void getItem(id)
}

export const getItem = cache(async (id) => {
  // ...
})
```

> 💡 **실무 팁**: 데이터 패칭 패턴 선택 시 **순차적 패칭** (데이터 간 의존성이 있을 때, Suspense로 워터폴 방지), **병렬 패칭** (독립적 데이터들을 Promise.all로 동시 가져오기), **프리로딩** (사용자 상호작용 전 미리 데이터 로드), **조건부 패칭** (권한/상태에 따라 필요한 데이터만 가져오기)을 적절히 선택하세요.

## API 참조

이 페이지에서 언급된 기능에 대해 API 참조를 읽어 자세히 알아보세요.

- [데이터 보안](/docs/app/guides/data-security.md) - Next.js의 내장 데이터 보안 기능과 애플리케이션 데이터 보호 모범 사례 학습
- [fetch](/docs/app/api-reference/functions/fetch.md) - 확장된 fetch 함수의 API 참조
- [loading.js](/docs/app/api-reference/file-conventions/loading.md) - loading.js 파일의 API 참조
- [logging](/docs/app/api-reference/config/next-config-js/logging.md) - 개발 모드에서 Next.js를 실행할 때 데이터 패치가 콘솔에 로그되는 방식 구성
- [taint](/docs/app/api-reference/config/next-config-js/taint.md) - 객체와 값 오염 활성화

---

**원문**: [Next.js 공식 문서 - Fetching Data](https://nextjs.org/docs/app/getting-started/fetching-data)