# 레이아웃과 페이지

Next.js는 **파일 시스템 기반 라우팅**을 사용하므로, 폴더와 파일을 사용하여 라우트를 정의할 수 있습니다. 이 페이지는 레이아웃과 페이지를 만드는 방법과 그 사이를 연결하는 방법을 안내합니다.

## 페이지 만들기

**페이지**는 특정 라우트에서 렌더링되는 UI입니다. 페이지를 만들려면 `app` 디렉터리 안에 `page` 파일을 추가하고 React 컴포넌트를 기본 내보내기하면 됩니다. 예를 들어, 인덱스 페이지(`/`)를 만들려면:

```tsx
// app/page.tsx
export default function Page() {
  return <h1>Hello Next.js!</h1>
}
```

```jsx
// app/page.js
export default function Page() {
  return <h1>Hello Next.js!</h1>
}
```

> 💡 **실무 팁**: 페이지 컴포넌트 이름은 관례적으로 `Page`로 명명하지만, 실제로는 어떤 이름이든 상관없습니다. 중요한 것은 default export 되는 것입니다. 팀에서는 일관된 명명 규칙을 정하는 것이 좋습니다.

## 레이아웃 만들기

레이아웃은 여러 페이지 간에 **공유**되는 UI입니다. 네비게이션 시 레이아웃은 상태를 유지하고, 상호작용을 유지하며, 다시 렌더링되지 않습니다.

`layout` 파일에서 React 컴포넌트를 기본 내보내기하여 레이아웃을 정의할 수 있습니다. 컴포넌트는 페이지나 다른 레이아웃이 될 수 있는 `children` prop을 받아야 합니다.

예를 들어, 인덱스 페이지를 자식으로 받는 레이아웃을 만들려면 `app` 디렉터리 안에 `layout` 파일을 추가하세요:

```tsx
// app/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        {/* Layout UI */}
        {/* Place children where you want to render a page or nested layout */}
        <main>{children}</main>
      </body>
    </html>
  )
}
```

```jsx
// app/layout.js
export default function DashboardLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {/* Layout UI */}
        {/* Place children where you want to render a page or nested layout */}
        <main>{children}</main>
      </body>
    </html>
  )
}
```

위의 레이아웃은 `app` 디렉터리의 루트에 정의되어 있으므로 **루트 레이아웃**이라고 합니다. 루트 레이아웃은 **필수**이며 `html`과 `body` 태그를 포함해야 합니다.

> 📋 **알아두면 좋은 점**: 루트 레이아웃에서 `html` 태그에 `lang` 속성을 설정하는 것은 SEO와 접근성에 중요합니다. 한국어 사이트라면 `lang="ko"`로 설정하세요.

## 중첩 라우트 만들기

중첩 라우트는 여러 URL 세그먼트로 구성된 라우트입니다. 예를 들어, `/blog/[slug]` 라우트는 세 개의 세그먼트로 구성됩니다:

- `/` (루트 세그먼트)
- `blog` (세그먼트)
- `[slug]` (리프 세그먼트)

Next.js에서는:

- **폴더**는 URL 세그먼트에 매핑되는 라우트 세그먼트를 정의하는 데 사용됩니다.
- **파일**(`page` 및 `layout` 같은)은 세그먼트에 표시되는 UI를 만드는 데 사용됩니다.

중첩 라우트를 만들려면 폴더를 중첩하면 됩니다. 예를 들어, `/blog` 라우트를 추가하려면 `app` 디렉터리에 `blog`라는 폴더를 만드세요. 그런 다음 `/blog`를 공개적으로 접근 가능하게 만들려면 `page.tsx` 파일을 추가하세요:

```tsx
// app/blog/page.tsx
// Dummy imports
import { getPosts } from '@/lib/posts'
import { Post } from '@/ui/post'

export default async function Page() {
  const posts = await getPosts()

  return (
    <ul>
      {posts.map((post) => (
        <Post key={post.id} post={post} />
      ))}
    </ul>
  )
}
```

```jsx
// app/blog/page.js
// Dummy imports
import { getPosts } from '@/lib/posts'
import { Post } from '@/ui/post'

export default async function Page() {
  const posts = await getPosts()

  return (
    <ul>
      {posts.map((post) => (
        <Post key={post.id} post={post} />
      ))}
    </ul>
  )
}
```

중첩 라우트를 만들기 위해 폴더를 계속 중첩할 수 있습니다. 예를 들어, 특정 블로그 게시물에 대한 라우트를 만들려면 `blog` 안에 새로운 `[slug]` 폴더를 만들고 `page` 파일을 추가하세요:

```tsx
// app/blog/[slug]/page.tsx
function generateStaticParams() {}

export default function Page() {
  return <h1>Hello, Blog Post Page!</h1>
}
```

```jsx
// app/blog/[slug]/page.js
function generateStaticParams() {}

export default function Page() {
  return <h1>Hello, Blog Post Page!</h1>
}
```

폴더 이름을 대괄호로 감싸면(예: `[slug]`) 데이터로부터 여러 페이지를 생성하는 데 사용되는 **동적 라우트 세그먼트**가 생성됩니다. 예를 들어, 블로그 게시물, 제품 페이지 등이 있습니다.

> 💡 **실무 팁**: 동적 라우트를 사용할 때는 `generateStaticParams` 함수를 활용하여 빌드 시점에 정적 페이지를 미리 생성할 수 있습니다. 이는 SEO와 성능에 매우 유리합니다.

## 레이아웃 중첩

기본적으로 폴더 계층 구조의 레이아웃도 중첩되며, 이는 `children` prop을 통해 자식 레이아웃을 감싸는 것을 의미합니다. 특정 라우트 세그먼트(폴더) 안에 `layout`을 추가하여 레이아웃을 중첩할 수 있습니다.

예를 들어, `/blog` 라우트에 대한 레이아웃을 만들려면 `blog` 폴더 안에 새로운 `layout` 파일을 추가하세요.

```tsx
// app/blog/layout.tsx
export default function BlogLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return <section>{children}</section>
}
```

```jsx
// app/blog/layout.js
export default function BlogLayout({ children }) {
  return <section>{children}</section>
}
```

위의 두 레이아웃을 결합하면, 루트 레이아웃(`app/layout.js`)이 블로그 레이아웃(`app/blog/layout.js`)을 감싸고, 블로그 레이아웃이 블로그 페이지(`app/blog/page.js`)와 블로그 게시물 페이지(`app/blog/[slug]/page.js`)를 감쌉니다.

## 동적 세그먼트 만들기

동적 세그먼트를 사용하면 데이터로부터 생성되는 라우트를 만들 수 있습니다. 예를 들어, 각 개별 블로그 게시물에 대한 라우트를 수동으로 만드는 대신, 블로그 게시물 데이터를 기반으로 라우트를 생성하는 동적 세그먼트를 만들 수 있습니다.

동적 세그먼트를 만들려면 세그먼트(폴더) 이름을 대괄호로 감싸세요: `[segmentName]`. 예를 들어, `app/blog/[slug]/page.tsx` 라우트에서 `[slug]`가 동적 세그먼트입니다.

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPostPage({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const post = await getPost(slug)

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  )
}
```

```jsx
// app/blog/[slug]/page.js
export default async function BlogPostPage({ params }) {
  const { slug } = await params
  const post = await getPost(slug)

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  )
}
```

동적 세그먼트와 `params` props에 대해 더 자세히 알아보세요.

동적 세그먼트 내의 중첩된 레이아웃도 `params` props에 접근할 수 있습니다.

## 검색 매개변수로 렌더링

서버 컴포넌트 **페이지**에서는 `searchParams` prop을 사용하여 검색 매개변수에 접근할 수 있습니다:

```tsx
// app/page.tsx
export default async function Page({
  searchParams,
}: {
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}) {
  const filters = (await searchParams).filters
}
```

```jsx
// app/page.jsx
export default async function Page({ searchParams }) {
  const filters = (await searchParams).filters
}
```

`searchParams`를 사용하면 들어오는 요청에서 검색 매개변수를 읽어야 하므로 페이지가 **동적 렌더링**으로 선택됩니다.

클라이언트 컴포넌트는 `useSearchParams` 훅을 사용하여 검색 매개변수를 읽을 수 있습니다.

정적으로 렌더링된 라우트와 동적으로 렌더링된 라우트에서 `useSearchParams`에 대해 더 자세히 알아보세요.

### 언제 무엇을 사용할지

- **페이지 데이터를 로드**하기 위해 검색 매개변수가 필요할 때는 `searchParams` prop을 사용하세요 (예: 페이지네이션, 데이터베이스에서 필터링).
- 검색 매개변수가 **클라이언트에서만** 사용될 때는 `useSearchParams`를 사용하세요 (예: props를 통해 이미 로드된 목록 필터링).
- 작은 최적화로, 다시 렌더링을 트리거하지 않고 검색 매개변수를 읽기 위해 **콜백이나 이벤트 핸들러**에서 `new URLSearchParams(window.location.search)`를 사용할 수 있습니다.

> 💡 **실무 팁**: 검색 매개변수를 사용할 때는 사용자 경험을 고려하세요. 필터링이나 페이지네이션 시 URL이 변경되어 사용자가 북마크하거나 공유할 수 있도록 하는 것이 좋습니다.

## 페이지 간 링크

라우트 간 탐색을 위해 `<Link>` 컴포넌트를 사용할 수 있습니다. `<Link>`는 HTML `<a>` 태그를 확장하여 프리페칭과 클라이언트 사이드 내비게이션을 제공하는 내장 Next.js 컴포넌트입니다.

예를 들어, 블로그 게시물 목록을 생성하려면 `next/link`에서 `<Link>`를 가져와서 컴포넌트에 `href` prop을 전달하세요:

```tsx
// app/ui/post.tsx
import Link from 'next/link'

export default async function Post({ post }) {
  const posts = await getPosts()

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.slug}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```

```jsx
// app/ui/post.js
import Link from 'next/link'

export default async function Post({ post }) {
  const posts = await getPosts()

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.slug}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```

> 📋 **알아두면 좋은 점**: `<Link>`는 Next.js에서 라우트 간 탐색을 위한 주요 방법입니다. 더 고급 내비게이션을 위해 `useRouter` 훅을 사용할 수도 있습니다.

## API 참조

이 페이지에서 언급된 기능들에 대해 더 자세히 알아보려면 API 참조를 읽어보세요.

- **연결과 내비게이션** - 프리페칭, 프리렌더링, 클라이언트 사이드 내비게이션을 포함한 내장 내비게이션 최적화가 어떻게 작동하는지, 그리고 동적 라우트와 느린 네트워크를 위한 내비게이션 최적화 방법을 학습하세요.
- **layout.js** - layout.js 파일에 대한 API 참조입니다.
- **page.js** - page.js 파일에 대한 API 참조입니다.
- **Link 컴포넌트** - 내장 `next/link` 컴포넌트로 빠른 클라이언트 사이드 내비게이션을 활성화하세요.
- **동적 세그먼트** - 동적 라우트 세그먼트는 동적 데이터로부터 라우트 세그먼트를 프로그래밍적으로 생성하는 데 사용될 수 있습니다.

---

> 💡 **추가 팁**: 이 문서는 Next.js의 App Router의 기본 개념을 다룹니다. 실제 프로젝트에서는 메타데이터, 에러 처리, 로딩 상태 등도 함께 고려해야 합니다. 각 페이지와 레이아웃에서 적절한 메타데이터를 설정하는 것을 잊지 마세요!

**원문**: https://nextjs.org/docs/app/getting-started/layouts-and-pages