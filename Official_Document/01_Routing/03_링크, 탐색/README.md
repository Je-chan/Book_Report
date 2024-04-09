## **1. `<Link>` 컴포넌트**

`<Link>` 는 <a> 태그를 확장해서 경로 간의 prefetching(사전 로드), 클라이언트 쪽 네비게이션을 제공한다. Next.js 의 경로 간 이동에 권장하고 또 주된 방법이다.

`<Link>` 컴포넌트를 사용하기 위해서는 `next/link` 에서 import 해오고, 컴포넌트에 href prop 을 전달한다.

```
import Link from 'next/link'

export default function Page() {
  return <Link href="/dashboard">Dashboard</Link>
}
```

### 

### **1-1) Examples**

### 1-1-1) 동적 Segement 로 Link 달기

동적 Segement 에 링크를 설정할 경우, 템플릿 리터럴을 사용할 수 있다.

```
import Link from 'next/link'

export default function PostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```

### 1-1-2) 활성화된 Link 인지 확인하기

**usePathname()** 을 활용해서 링크가 활성화됐는지 확인할 수 있다. 예를 들어서, 활성 링크에 클래스를 추가하려면 현재 경로가 링크의 href 와 일치하는지 확인할 수 있다.

```
'use client'

import { usePathname } from 'next/navigation'
import Link from 'next/link'

export function Links() {
  const pathname = usePathname()

  return (
    <nav>
      <ul>
        <li>
          <Link className={`link ${pathname === '/' ? 'active' : ''}`} href="/">
            Home
          </Link>
        </li>
        <li>
          <Link
            className={`link ${pathname === '/about' ? 'active' : ''}`}
            href="/about"
          >
            About
          </Link>
        </li>
      </ul>
    </nav>
  )
}
```

### ****

### 1-1-3) id 로 Scroll 하기

Next.js  **App Router** 의 기본적인 동작은 새로운 경로로 스크롤하거나, "뒤로 가기", "앞으로 가기" 등으로 탐색했을 때 스크롤 위치를 유지하는 것이다.

만약, 특정 id 로 스크롤하길 원한다면 URL 에 "#" 해시 링크를 추가하거나 href props 로 해시 링크를 전달할 수 있다. /`<Link>`/ 가 <a> 요소로 렌더링되기에 가능한 일이다.

```
<Link href="/dashboard#settings">Settings</Link>

// 출력
<a href="/dashboard#settings">Settings</a>
```

### 1-1-4) 스크롤 복원 비활성화하기

Next.js **App Router** 의 기본 동작은 새로운 경로로 스크롤하거나 "뒤로 가기", "앞으로 가기" 를 할 때 스크롤의 위치를 유지하거나 페이지 상단으로 스크롤 하는 것이다. 이 기본 동작을 비활성화하기 위해서는 `<Link>` 에 "scroll={false}" 를 전달하거나, **router.push()**, **router.relplace()** 에 scroll: false 를 전달할 수 있다.

```
// next/link
<Link href="/dashboard" scroll={false}>
  Dashboard
</Link>

// 혹은

// useRouter
import { useRouter } from 'next/navigation'

const router = useRouter()

router.push('/dashboard', { scroll: false })
```

---

## **2. useRouter()**

**useRouter()** 훅을 사용하면 클라이언트 컴포넌트에서 경로를 프로그래밍적으로 변경할 수 있다.

```
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button type="button" onClick={() => router.push('/dashboard')}>
      Dashboard
    </button>
  )
}
```

그러나 특별한 사유가 아니라면 `<Link>` 태그를 사용할 것을 권장한다.

---

## **3. redicrect 함수**

서버 컴포넌트의 경우, **redirect** 함수를 사용해서 경로를 이동할 수 있다.

```
import { redirect } from 'next/navigation'

async function fetchTeam(id: string) {
  const res = await fetch('https://...')
  if (!res.ok) return undefined
  return res.json()
}

export default async function Profile({ params }: { params: { id: string } }) {
  const team = await fetchTeam(params.id)
  if (!team) {
    redirect('/login')
  }

// ...
}
```

### **3-1) 참고**

- *redirect** 는 기본적으로 307(임시 리다이렉션) 상태 코드를 반환한다. 서버에서 동작할 때는 POST 요청의 성공 결과로 리다이렉션하는 303 상태코드를 반환한다.
- *redirect** 는 내부적으로 오류를 throw 하기에 try/catch 를 외부에서 호출해야 한다.
- *redirect** 는 렌더링 과정 중 클라이언트 컴포넌트에서 호출할 수는 있지만, 이벤트 핸들러에서는 호출할 수 없다. 클라이언트 컴포넌트에서 호출하고 싶다면 **useRouter()** 를 사용하면 된다.
- *redirect** 는 절대 URL 경로를 허용하여 외부 링크로도 연결할 수 있다.
- 만약, 렌더링 전에 리다이렉션 하고 싶다면 next.config.js 혹은 Middleware 를 사용하는 것이 좋다.

---

## **4. Native History API 사용하기**

Next.js 는 웹 브라우저의 window.history.pushState, window.history.replaceState 메서드를 사용한다. 이 메서드들을 활용하면 페이지를 다시 로드하지 않고 브라우저의 히스토리 스택을 업데이트 한다.

pushState, replaceState 호출은 Next.js 라우터에 통합돼서 **usePathname()** , **useSearchParams()** 와 동기화될 수 있다.

### **4-1) window.history.pushState**

브라우저의 히스토리 스택에 항목을 추가할 때 사용한다. 사용자는 이전 상태로 다시 되돌릴 수 있다.

```
'use client'

import { useSearchParams } from 'next/navigation'

export default function SortProducts() {
  const searchParams = useSearchParams()

  function updateSorting(sortOrder: string) {
    const params = new URLSearchParams(searchParams.toString())
    params.set('sort', sortOrder)
    window.history.pushState(null, '', `?${params.toString()}`)
  }

  return (
    <>
      <button onClick={() => updateSorting('asc')}>Sort Ascending</button>
      <button onClick={() => updateSorting('desc')}>Sort Descending</button>
    </>
  )
}
```

### **4-2) window.history.replaceState**

브라우저의 히스토리 스택에서 현재 항목을 대체할 때 사용한다. 이렇게 하면 사용자는 이전 상태로 다시 이동할 수 없다.

```
'use client'

import { usePathname } from 'next/navigation'

export function LocaleSwitcher() {
  const pathname = usePathname()

  function switchLocale(locale: string) {
// 예: '/en/about' 또는 '/fr/contact'const newPath = `/${locale}${pathname}`
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

---

## **5. Routing, Navigation 은 어떻게 동작하는가**

**App Router** 는 경로를 이동하기 위해 혼합된 접근 방식을(* 서버와 클라이언트 사이드 마다 각자 다르게) 사용한다. 서버에서는 응용 프로그램 코드가 자동으로 경로 Segement 에 의해서 코드 분할(Code-Split)을 한다. 클라이언트에서는 Next.js 가 경로 Segement 를 prefetching 하고 캐시한다. 이것은 사용자가 새로운 경로로 이동할 때 브라우저가 페이지를 다시 로드하지 않고 변경된 경로만 다시 렌더링되도록 해서 경로 이동간의 경험과 성능을 개선한다.

### **5-1) 코드 분할(Code-Split)**

코드 분할은 응용 프로그램의 코드를 더 작은 사이즈로 분할하고 브라우저에서 다운로드 받고 실행이 될 수 있도록 한다. 이런 방식으로 각 요청에 대한 전송되는 데이터 양과 실행 시간이 감소돼 성능이 향상된다.

서버 컴포넌트를 사용하면 응용 프로그램 코드가 자동으로 경로 Segement 로 분할된다. 이 방식을 사용하면 현재 경로에서 필요한 코드만 탐색될 때 로드한다.

### **5-2) Prefetching(* 의미로는 사전로드지만, 프리패칭이란 표현이 고유언어처럼 사용된다.)**

Prefetching 은 사용자가 해당 경로에 방문하기 전에 백그라운드에서 경로를 Prefetching 한다

Next.js 에서는 두 가지 방법으로 Prefetching 할 수 있다.

1. `<Link>` 컴포넌트: 해당 경로가 사용자의 뷰포트에 나타날 때 자동으로 Prefetching 이 이뤄진다. 페이지가 처음으로 로드될 때 스크롤을 통해 뷰에 나타날 때,prefetching 이 이뤄진다.
2. router.prefetch() : **useRouter()** 훅에서 prefetching 을 할 수 있다

prefetch={false} 를 사용해서 Prefetching 을 비활성화할 수 있다. 혹은, prefetch={true} 로 로딩 경계를 넘어서 전체 페이지 데이터를 사전 로드 할 수도 있다. 참고로, Pretching 은 Production 모드에서만 가능하다.

### **5-3) Caching**

Next.js dptjsms **Router Cache** 라는 메모리 내에 클라이언트 사이드 캐시가 있다. 사용자가 앱을 탐색하는 동안, Prefetching 된 경로 Segement 혹은, 방문한 경로의 React Server Component Palyoad 가 캐시에 저장된다.

이는 경로를 이동할 때 가능한 캐시된 것을 사용하여 서버에 새롭게 요청하는 것을 방지할 수 있다. 이렇게 하면 많은 양의 요청을 감소시키고 전송된 데이터의 양을 줄임으로써 성능을 향상시킨다.

### **5-4) 부분 렌더링(Partial Rendering)**

부분 렌더링은 경로를 이동할 때 변경되는 경로 Segement 만 클라이언트에서 다시 렌더링되고 공유되는 Segement 는 보존하는 방법이다.

예를 들어서, `/dashboard/settings`, `dashboard/analytics` 두 라우트를 탐색할 때 설정 및 분석 page.js 는 렌더링되겠지만 공유되는 dashboard 의 layout 은 보존된다.

!https://blog.kakaocdn.net/dn/QWCKM/btsGrUK9STK/fxwRG7p7ScbOsc34o3SAFk/img.png

부분 렌더링 없이, 각각의 네비게이션은 클라이언트에서 완전히 새로운 페이지를 리렌더링해야할 수도 있다. 오직, 변경된 부분만 새롭게 렌더링함으로써 전달돼야 하는 데이터의 양과 실행 시간을 감축시켜 성능이 향상된다.

### **5-5) Soft Naviagtion**

브라우저는 페이지 간에 "Hard Navigation" 을 수행한다. Next.js 의 **App Router** 는 "Soft Naviagtion" 을 수행한다. 이는 변경된 경로 Segement 만 리렌더링되도록(부분 렌더링) 한다. 이렇게 하면 네비게이션 중에 클라이언트 React 상태가 보존될 수 있다.

### **5-6) "뒤로 가기", "앞으로 가기"**

기본적으로 Next.js 는 "뒤로 가기", "앞으로 가기" 의 탐색에 대해서 스크롤 위치를 유지하고 **Router Cache** 에서 저장된 경로 Segement 를 재사용한다.

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating**