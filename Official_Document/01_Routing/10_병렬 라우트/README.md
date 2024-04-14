병렬 라우트(Parallel Routes)  는 동일한 레이아웃 안에서 동시에 혹은 조건부로 하나 이상의 페이지를 렌더링할 수 잇다. 이 방법은 대시보드, 소셜 사이트 피드와 같이 동적인 앱에서 활용도가 매우 높다.

!https://blog.kakaocdn.net/dn/ce1Jmr/btsGweWXdiO/XWrAKk5vSJcYYeiY3vwGW0/img.png

---

## **1. Slots**

병렬 라우트는 이름이 있는 slot에서 생성된다. slots 은 @ 기호를 사용해서 만든다. 예를 들어, 아래의 파일 구조는 두 개의 슬롯, @analytics 와 @team 으로 구성된 구조다.

!https://blog.kakaocdn.net/dn/cLuOMh/btsGt7Y5u99/WfH5W43rUJfEVd0M4QdhEk/img.png

슬롯은 부모 레이아웃에 props 로 전달된다. 위의 예시에서, `app/layout.js` 컴포넌트는 이제 @analytics 및 @team 슬롯을 props 로 넘겨 받아 자식 props 와 함께 병렬로 렌더링할 수 있다.

```
// TypeScript
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <>
      {children}
      {team}
      {analytics}
    </>
  )
}
```

여기서 주의할 점은 slots 은 Segement 가 아니라는 점이다. 즉, URL 구조에 영향을 주지 않는다. 예를 들어 `app/@analytics/views` 의 폴더 구조라면 URL 경로는 `/vies` 다.

참고로 **children** 은 Next.js 에서 default slots 으로 취급을 받는다. 이말인 즉슨 `app/views` 는 사실 `app/@children/views` 인 것이다.

---

## **2. 활성화 상태와 탐색**

기본적으로 Next.js 는 각 slots(또는 하위 페이지)에 대한 활성 상태를 추적한다. 하지만, slots 내에서 렌더링된 내용은 어떤 탐색을 하느냐에 따라 달라지게 된다.

1. Soft 탐색 : 클라이언트 사이드 탐색 중에 Next.js 는 부분 렌더링을 수행한다. 이 방식을 통해 slots 내에서 하위 페이지들을 변경하고, 다른 slots 의 활성화된 하위 페이지들은 유지한다. 심지어 현재 URL 과 일치하지 않더라도 다른 slots 의 활성화된 하위 페이지들은 유지된다.
2. Hard 탐색 : 전체 페이지를 로드한 후(새로고침 등) Next.js 는 현재 URL 과 일치하지 않는 슬롯에 대한 활성 상태를 결정하지 못한다. 대신, 일치하지 않는 슬롯에 대한 default.js 를 렌더링하거나 없는 경우 404 를 렌더링한다.

### **2-1) default.js**

일치하지 않는 슬롯에 대한 fallback 으로 default.js 파일을 정의할 수 있다. 초기 페이지를 로드하거나 전체 페이지를 새로고침 하는 중에 일치 하지 않는 슬롯에 대한 fallback 으로 렌더링된다.

아래의 폴더구조를 보면 @team slots 에는 `/settings` 페이지가 있지만 @analytics 는 없다.

!https://blog.kakaocdn.net/dn/7cgGF/btsGuLVw7Lv/KBwi19CuJ8z8MNVdYNBkXk/img.png

만약 `/settings` 페이지로 넘어가면 @team slots 은 `/settings` 페이지를 렌더링하고 @analytics slots 은 현재 활성 페이지를 유지한다.

여기서 새로고침하면 Next.js 는 @analytics 에 대한 default.js 파일을 렌더링하려고 할 것이다. 만약 default.js 파일이 존재하지 않으면 404를 렌더링한다.

추가적으로 **children** 이 암묵적으로 slot 이기에 따로  default.js 는 **children** 을 위한 fallback 으로 만들 수 있다.

### **2-2) useSelectedLayoutSegment(s)**

- *useSelectedLayoutSegment(s)** 는 병렬 라우트 내에서 활성화된 경로 Segment 를 읽을 수 있또록 하는 **parallelRoutesKey** 를 매개변수로 받는다.

```
// app/layout.tsx
import { useSelectedLayoutSegment } from 'next/navigation'

export default function Layout({ auth }: { auth: React.ReactNode }) {
  const loginSegment = useSelectedLayoutSegment('auth')
  // ...
}
```

사용자가 `app/@auth/login` 으로 이동할 때, 'loginSegment' 는 "login" 이라는 문자열이 될 것이다.

---

## **3. 예시**

### **3-1) 조건부 Route**

특정 조건에 따라 경로를 조건부로 렌더링하는 데 병렬 라우트를 사용할 수 있다. 예를 들어서 `/admin` 과 `/user` 는 역할에 따라 다른 대시보드 페이지를 렌더링할 수 있다.

!https://blog.kakaocdn.net/dn/6ncfE/btsGu4AAwy0/dlT9k4DBge7wnLclvMem50/img.png

```
import { checkUserRole } from '@/lib/auth'

export default function Layout({
  user,
  admin,
}: {
  user: React.ReactNode
  admin: React.ReactNode
}) {
  const role = checkUserRole()
  return <>{role === 'admin' ? admin : user}</>
}
```

### **3-2) 탭 그룹**

사용자가 slots 을 독립적으로 탐색할 수 있도록 slots 내에 레이아웃을 추가할 수 있다. 이런 방식은 탭을 만드는데 유용하다. 예를 들어 @analytics 슬롯에는 `/page-vies` 혹은 `/visitors` 두 개의 하위 페이지가 있다 해보자.

!https://blog.kakaocdn.net/dn/dKb7wp/btsGtnOHUWN/4VeQ11kk90QBR9GbTNQuqk/img.png

@analytics 를 안에서 layout.js 파일을 만들고 두 개의 페이지에 tab을 공유한다.

```
import Link from 'next/link'

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Link href="/page-views">Page Views</Link>
        <Link href="/visitors">Visitors</Link>
      </nav>
      <div>{children}</div>
    </>
  )
}
```

### **3-3) 모달**

병렬 라우트는 모달을 만들 때 인터셉트 라우트(Intercept Route) 와 함께 사용할 수 있다. 이 방식을 사용하면 모달을 만들 때 일반적으로 발생하는 몇 가지 문제들을 해결할 수 있다.

1. URL 을 통해 모달 콘텐츠를 공유 가능하게 만들기
2. 페이지를 새로 고침해도 모달이 닫히지 않은 상태 유지하기
3. 뒤로 가기를 할 때 모달을 닫고 이동하기
4. 앞으로 가기를 할 때 모달을 닫고 이동하기

이 UI 패턴을 구현하기 위해서 사용자는 클라이언트 사이드 탐색을 사용해 레이아웃에서 로그인 모달을 열거나 별도의 /login 페이지에서 엑세스할 수 있다.

!https://blog.kakaocdn.net/dn/G93at/btsGuNy8wUb/rkLsUD22zTwUClZyYjZfP0/img.png

이 패턴을 적용하기 위해서는 먼저 주력으로 사용할 로그인 페이지, `/login` 을 만드는 것으로 시작해야 한다.

!https://blog.kakaocdn.net/dn/cU92Ke/btsGtKXsacK/bwXrVOUeO82RfHxfHzyUPK/img.png

```
import { Login } from '@/app/ui/login'

export default function Page() {
  return <Login />
}
```

그 다음, @auth 하위에 slot 을 만들고, null 을 반환하는 default.js 파일을 만든다. null 을 리턴하는 방식은 활성화 되지 않았을 때 아무것도 렌더링되지 않을 것을 보장하기 위함이다.

```
export default function Default() {
  return null
}
```

@auth slot 안에 `/login` 경로를 `/(.)login` 폴더로 변경한 다음 intercept(가로채기)한다.(*Intercept Route) <Modal> 컴포넌트를 import 한다음, 그 자식으로 /(.)login/page.tsx 파일을 넣으면 된다.

```
import { Modal } from '@/app/ui/modal'
import { Login } from '@/app/ui/login'

export default function Page() {
  return (
    <Modal>
      <Login />
    </Modal>
  )
}
```

### (1) 모달 열기

이제, Next.js 를 활용해 모달을 열고 닫을 수 있다. 이런 방식을 사용하면 모달이 열릴 때 URL 이 올바르게 업데이트 되고, 뒤로 가기, 앞으로 가기 할 때도 모달이 열린 상태를 유지하게 된다.

모달을 열기 위해선, 부모 레이아웃에 @auth slot 을 props 로 전달하고 **children** props 과 함께 렌더링하면 된다.

```
// app/layout.tsximport Link from 'next/link'

export default function Layout({
  auth,
  children,
}: {
  auth: React.ReactNode
  children: React.ReactNode
}) {
  return (
    <>
      <nav>
        <Link href="/login">모달 열기</Link>
      </nav>
      <div>{auth}</div>
      <div>{children}</div>
    </>
  )
}
```

사용자가 <Link> 를 클릭하면 페이지가 `/login` 페이지로 이동하는 대신 모달이 열린다. 만일, 새로 고침이나 초기 로드할 때 `/login` 페이지로 이동하면 사용자가 주력 로그인 페이지로 이동하게 된다.

### (2) 모달 닫기

모달을 닫는 건 Modal 에서 **router.back()** 을 호출하거나 <Link>를 사용하면 된다.

```
// app/ui/modal.tsx'use client'

import { useRouter } from 'next/navigation'

export function Modal({ children }: { children: React.ReactNode }) {
  const router = useRouter()

  return (
    <>
      <button
        onClick={() => {
          router.back()
        }}
      >
        모달 닫기
      </button>
      <div>{children}</div>
    </>
  )
}
```

<Link> 컴포넌트를 사용해서 @auth slot 을 더 렌더링하지 않는 페이지로 이동하는 경우, null 을 반환하는 Dynamic Rotues 를 사용할 수 있다.

```
// app/ui/modal.tsximport Link from 'next/link'

export function Modal({ children }: { children: React.ReactNode }) {
  return (
    <>
      <Link href="/">모달 닫기</Link>
      <div>{children}</div>
    </>
  )
}

// app/@auth/[...catchAll]/page.tsx

export default function CatchAll() {
  return null
}
```

### 참고

1. @auth slot 을 닫기 위해서 ...slug 를 활용한다. 클라이언트 사이드에서 경로를 탐색할 때, slot 과 일치하지 않지만 여전히 보이는 경우가 있기 때문에 활성화되면서 null 을 반환해 모달을 닫아줄 slot 이 필요로 해진다.
2. 다른 예시로 갤러리의 사진 모달이

### **3-4) 로딩/에러 UI**

병행 라우트는 독립적으로 streaming 된다. 따라서 각 경로에 대한 독립적인 에러 및 로딩 상태를 정의할 수 있다.

!https://blog.kakaocdn.net/dn/oCT7O/btsGwQ9dH46/rc4B1K8Z54B1VGKTPAutrk/img.png

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/parallel-routes**

[Routing: Parallel Routes | Next.js
Simultaneously render one or more pages in the same view that can be navigated independently. A pattern for highly dynamic applications.
nextjs.org](https://nextjs.org/docs/app/building-your-application/routing/parallel-routes)