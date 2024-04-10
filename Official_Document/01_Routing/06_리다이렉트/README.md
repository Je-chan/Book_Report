## **1. redirect() 함수**

**redirect** 함수는 사용자를 다른 URL 로 리다이렉션 할 수 있게 한다. 리다이렉션은 **Server Components**, **Route Handlers**, **Server Actions** 에서 호출할 수 있다

**redirect** 는 일반적으로 mutation 또는 이벤트 후에 사용된다. 예를 들어서 게시물을 생성하는 경우, 다음과 같은 코드로 만들 수 있다.

```
'use server'

import { redirect } from 'next/navigation'
import { revalidatePath } from 'next/cache'

export async function createPost(id: string) {
  try {
// 데이터베이스 호출
  } catch (error) {
// 에러 처리
  }

  revalidatePath('/posts')// 캐시된 게시물 업데이트
  redirect(`/post/${id}`)// 새 게시물 페이지로 이동
}
```

### **1-1) 참고**

1. *redirect** 는 기본적으로 307(임시 리다이렉션) 코드를 반환한다. **Server Action** 에서 사용할 때는 POST 요청의 성공 페이지로 리다이렉션 하는데 일반적으로 사용되는 303 을 반환한다.
2. *redirect** 는 내부적으로 오류를 throw 하므로, try-catch 문을 밖에서 호출해야 한다.
3. *redirect** 는 렌더링 프로세스 중에 Client Component 에서 호출할 수 있지만 이벤트 핸들러에서는 사용할 수 없다. 이런 경우에는 **useRouter** 훅을 사용하면 된다.
4. *redirect** 는 절대 URL 도 허용하여 외부 링크로 리다이렉션할 수 있다.
5. 렌더링 중에 리다이렉션 하려면 next.config.js 또는 미들웨어를 사용하면 된다.

---

## **2. permanentRedirect 함수**

**permanentRedirect** 함수를 사용하면 사용자를 영구적으로 다른 URL 로 리다이렉션 한다. **permanentRedirect** 는 Server Component, Route Handlers, Server Actions 에서 호출할 수 있다.

**permanentRedirect** 는 사용자 이름을 변경한 후 사용자 프로필 URL 을 업데이트하는 등 엔티티의 근본 URL 을 변경하는 뮤테이션 또는 이벤트 후에 종종 사용된다.

```
'use server'

import { permanentRedirect } from 'next/navigation'
import { revalidateTag } from 'next/cache'

export async function updateUsername(username: string, formData: FormData) {
  try {
// 데이터베이스 호출
  } catch (error) {
// 에러 처리
  }

  revalidateTag('username')// 모든 username 참조 업데이트
  permanentRedirect(`/profile/${username}`)// 새 사용자 프로필로 이동
}
```

### **2-1) 참고**

- *permanentRedirect** 는 기본적으로 308 상태 코드를 반환한다.
- *permanentRedirect** 도 절대 URL 을 허용하여 외부 링크로 리다이렉션 할 수 있다.
- 렌더링 프로세스 전에 리다이렉션 하려면 next.config.js 나 미들웨어를 사용하면 된다.

---

## **3. useRouter Hook**

클라이언트 컴포넌트의 이벤트 핸들러 내에서 리다이렉션 할 경우, useRouter 훅의 'push' 메서드를 사용할 수 있다.

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

### **3-1) 참고**

만약 사용자를 프로그래밍적으로 이동시킬 필요가 없다면, <Link> 컴포넌트를 쓰는 것이 좋다.

---

## **4. next.config.js 안에 redirects**

'next.config.js' 파일의 'redirects' 옵션을 사용하면 들어오는 요청 경로를 다른 대상 경로로 리다이렉션 할 수 있다. 이는 페이지의 URL 구조를 변경하거나 미리 알려진 리다이렉션 목록이 있는 경우 유용하다.

'redirects' 옵션은 경로, 헤더, 쿠키, 및 쿼리 일치를 지원하여 들어오는 요청을 기반으로 사용자를 리다이렉션 할 수 있도록 도와준다.

```
module.exports = {
  async redirects() {
    return [
// Basic redirect
      {
        source: '/about',
        destination: '/',
        permanent: true,
      },
// Wildcard path matching
      {
        source: '/blog/:slug',
        destination: '/news/:slug',
        permanent: true,
      },
    ]
  },
}
```

### **4-1) 참고**

- 'redirects' 는 'permanent' 옵션을 사용해 307 또는 308 상태 코드를 반환할 수 있다.
- 'redirects' 는 플랫폼마다 제한이 있을 수 있다. Vercel 의 경우 1,024 개의 리다이렉션 제한이 있다. 많은 수의 리다이렉션을 관리하려면 미들웨어를 사용해 사용자 정의 솔루션을 만드는 것이 낫다ㅏ.

---

## **5. 미들웨어 내부에서 NextResponse.redirect**

미들웨어는 요청이 완료되기 전에 코드를 동작시킨다. 그리고 나서 이후에 들어온 요청을 기반으로 **NextResponse.redirect** 를 사용해 특정 조건이나 기준에 따라 리다이렉션을 수행할 수 있게 해준다. 이런 방법은 인증, 세션 관리 또는 많은 수의 리다이렉션을 관리하는 경우에 유용하다.

예를 들어, 사용자가 인증되지 않은 경우, '/login' 페이지로 리다이렉션 하려면 다음과 같이 할 수 있다.

```
import { NextResponse, NextRequest } from 'next/server'
import { authenticate } from 'auth-provider'

export function middleware(request: NextRequest) {
  const isAuthenticated = authenticate(request)

// If the user is authenticated, continue as normalif (isAuthenticated) {
    return NextResponse.next()
  }

// Redirect to login page if not authenticatedreturn NextResponse.redirect(new URL('/login', request.url))
}

export const config = {
  matcher: '/dashboard/:path*',
}
```

### **5-1) 참고**

미들웨어는 'next.config.js'  에서 리다이렉션이 이후, 렌더링 이전에 실행된다.

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/redirecting#redirect-function**

[Routing: Redirecting | Next.js
Learn the different ways to handle redirects in Next.js.
nextjs.org](https://nextjs.org/docs/app/building-your-application/routing/redirecting#redirect-function)