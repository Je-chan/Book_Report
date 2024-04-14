미들웨어는 요청이 완료되기 전에 코드를 실행한다. 그 다음 들어오는 요청을 기반으로 응답을 수정하거나, 다시 쓰거나, 리다이렉션하거나 request 혹은 response 의 헤더를 수정하거나 직접 응답할 수 있다.

미들웨어는 캐시된 내용과 경로가 일치하기 전에 실행된다.

---

## **1. 사용 사례**

미들웨어를 어플리케이션에 통합하면 성능, 보안, 사용자 경험을 향상시킬 수 있다. 미들웨어가 특히 효과적인 시나리오는 다음과 같다.

1. 인증 및 권한 부여: 특정 페이지 혹은 API 경로에 액세스하기 전, 사용자를 식별하거 세션 쿠키를 체크한다
2. 서버 사이드 리다이렉션: 서버 수준에서 특정 조건에 따라 사용자를 리다이렉션 한다
3. 경로 바꾸기: 요청 속성에 따라서 API 경로 혹은 페이지에 대한 경로를 동적으로 다시 쓰도록 만든다. 이는 A/B 테스트, 기능 배포, 레거시 경로 등을 지원한다.
4. 봇 감지: 봇 트래픽을 감지하고 차단해서 자원을 보호한다
5. 로그 및 분석: 페이지 또는 API 에서 처리 되기 전에 요청 데이터를 캡쳐하고 분석한다.
6. 기능 플래깅: 실시간 기능 배포 또는 테스트를 위해 기능을 동적으로 활성화/비활성화 한다.

미들웨어가 최적의 접근 방법이 아닌 경우도 몇 가지 존재한다.

1. 복잡한 데이터 가져오기 및 조작
2. 무거운 계산 작업
3. 규모 확장성: 미들웨어가 서버의 부하를 초래하거나 규모 확장에 제한을 가할 수 있다.

---

## **2. 규칙**

미들웨어를 정의하는 방법은 프로젝트 최상단에 middleware.ts 파일을 사용하면 된다. 예를 들어서 pages 또는 app 과 동일한 수준에 있거나 src 내부에 있어야 한다.

프로젝트 당 하나의 middleware.ts 만 지원된다. 하지만 미들웨어 로직을 모듈식으로 구성할 수 있다. 미들웨어 기능을 별도의 .ts, .js 파일로 분리하고 해당 파일을 주요 middleware.ts 파일로 가져오는 방식이다. 이렇게 하면 경로별 미들웨어를 더 깔끔하게 관리할 수 있으며, 중앙 집중 제어를 위해서 middleware.ts 에서 집계된다. 단일 미들웨어 파일을 강제하는 것은 구성을 단순화하, 잠재적인 충돌 방지, 여러 미들웨어 레이어 기피를 통한 성능 최적화를 할 수 있다.

---

## **3. 예시**

```
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

// This function can be marked `async` if using `await` insideexport function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}

// See "Matching Paths" below to learn moreexport const config = {
  matcher: '/about/:path*',
}
```

---

## **4. 매칭 경로**

미들웨어는 프로젝트의 모든 경로에서 호출된다. 따라서 특정 경로를 정확하게 대상으로 지정하거나 제외하는 것이 매우 중요하다. 미들웨어의 실행 순서는 다음과 같다.

1. next.config.js 에서 **header**
2. next.config.js 에서 **redirects**
3. 미들웨어(rewirtes, redirects)
4. next.config.js 에서 **beforeFiles** (rewrites)
5. 파일 시스템 경로(public/, _next/static, pages/, app/ 등)
6. next.config.js 의 afterFiles (rewrites)
7. 동적 경로 (/blog/[slug])
8. next.config.js 의 fallback(rewrites)

미들웨어가 실행될 경로를 정의하는 방법에는 두 가지가 있다.

### **4-1) Matcher**

매처는 특정 경로에서 미들웨어를 실행할 수 있도록 필터링하게 해준다.

```
export const config = {
  matcher: '/about/:path*',
}
```

배열 구문을 사용해서 단일 경로 혹은 여러 경로를 일치시킬 수 있다.

```
export const config = {
  matcher: ['/about/:path*', '/dashboard/:path*'],
}
```

매처 구성은 정규식을 지원한다. "negative lookaheads" 나 "character matching" 등의 기능이 지원된다. 특정 경로를 제외한 모든 요청 경로를 일치시키는 "negative lookaheads" 방법은 다음과 같다.

```
export const config = {
  matcher: [
/*
     * Match all request paths except for the ones starting with:
     * - api (API routes)
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     */'/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
}
```

일부 요청을 건너뛰기 위해 누락 또는 포함하는 것과 같은 특정 요청에 대한 미들웨어 우회에는 **missing** 혹은 **has** 배열 또는 두 가지를 섞는 방법이 존재한다.

```
export const config = {
  matcher: [
/*
     * Match all request paths except for the ones starting with:
     * - api (API routes)
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     */
    {
      source: '/((?!api|_next/static|_next/image|favicon.ico).*)',
      missing: [
        { type: 'header', key: 'next-router-prefetch' },
        { type: 'header', key: 'purpose', value: 'prefetch' },
      ],
    },

    {
      source: '/((?!api|_next/static|_next/image|favicon.ico).*)',
      has: [
        { type: 'header', key: 'next-router-prefetch' },
        { type: 'header', key: 'purpose', value: 'prefetch' },
      ],
    },

    {
      source: '/((?!api|_next/static|_next/image|favicon.ico).*)',
      has: [{ type: 'header', key: 'x-present' }],
      missing: [{ type: 'header', key: 'x-missing', value: 'prefetch' }],
    },
  ],
}
```

matcher 의 값은 빌드 타임에 정적으로 분석돼야 하므로 상수여야 한다. 변수와 같이 동적인 값들은 무시된다.

### 구성된 matchers

1. 반드시 / 로 시작해야 한다.
2. 이름이 있는 매개변수를 포함할 수 있다: `/about/:path` 는 `/about/a` 혹은 `/about/b` 와 일치하지만 `/about/c/d` 와는 일치하지 않는다
3. 매개변수에 수정자를 사용할 수 있다: `/about/:path*` 는 *가 0개 이상이기에 `/about/a/b/c` 와도 일치한다. ? 는 0개 또는 1개이며, + 는 1개 이상을 뜻한다.
4. 괄호 안에 정규 표현식을 사용할 수 있다.: `/about/(.)` 는 `/about/:path` 와 동일하다

역 호환성을 위해 Next.js 는 항상 /public 을 /public/index 로 간주한다. 따라서 /public/:path 의 매처와 일치한다.

### **4-2) 조건문**

```
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/about')) {
    return NextResponse.rewrite(new URL('/about-2', request.url))
  }

  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.rewrite(new URL('/dashboard/user', request.url))
  }
}
```

---

## **5. NextResponse**

NextResponse API 를 사용하면 다음의 일들을 수행할 수 있다.

1. 들어오는 요청을 다른 URL 로 리다이렉션 한다.
2. 지정된 URL 을 표시하여 응답을 다시 작성할 수 있다.
3. API 라우트, getServerSideProps 및 리다이렉션 대상에 대한 요청 헤더를 설정한다
4. 응답 쿠키를 설정한다
5. 응답 헤더를 설정한다.

미들웨어에서 응답을 생성하려면 다음의 작업을 진행해야 한다.

1. 응답을 생성하는 경로로 리다이렉트 한다.
2. 직접 NextResponse 를 반환한다.

---

## **6. Cookie 사용하기**

쿠키는 일반적인 headrs 다. 요청에서 쿠키는 **Cookie** 헤더에 저장되고, 응답에서는 **Set-Cookie** 헤더를 사용한다. Next.js 는 **NextRequest**, **NextResponse** 의 **cookies** 확장을 통해 쿠키에 접근하고 조작할 수 있는 편리한 방법들을 제공한다.

들어오는 요청의 경우 **cookies** 는 다음의 메서드를 제공한다.  **get**, **getAll**, **set**, **delete**  쿠키가 있는지 확인하려면 has 를 사용하거나 모든 쿠키를 지우려면 clear 를 사용할 수 있다.

응답의 경우 **get**, **getAll**, **set**, **delete** 메서드들을 활용할 수 있다.

```
// middleware.tsimport { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
// Assume a "Cookie:nextjs=fast" header to be present on the incoming request// Getting cookies from the request using the `RequestCookies` APIlet cookie = request.cookies.get('nextjs')
  console.log(cookie)// => { name: 'nextjs', value: 'fast', Path: '/' }const allCookies = request.cookies.getAll()
  console.log(allCookies)// => [{ name: 'nextjs', value: 'fast' }]

  request.cookies.has('nextjs')// => true
  request.cookies.delete('nextjs')
  request.cookies.has('nextjs')// => false

// Setting cookies on the response using the `ResponseCookies` APIconst response = NextResponse.next()
  response.cookies.set('vercel', 'fast')
  response.cookies.set({
    name: 'vercel',
    value: 'fast',
    path: '/',
  })
  cookie = response.cookies.get('vercel')
  console.log(cookie)// => { name: 'vercel', value: 'fast', Path: '/' }// The outgoing response will have a `Set-Cookie:vercel=fast;path=/` header.

  return response
}
```

---

## **7. 헤더 설정**

NextResponse API 를 사용해서 요청 및 응답 헤더를 설정할 수 있다.

```
// middleware.tsimport { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
// Clone the request headers and set a new header `x-hello-from-middleware1`const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-hello-from-middleware1', 'hello')

// You can also set request headers in NextResponse.rewriteconst response = NextResponse.next({
    request: {
// New request headers
      headers: requestHeaders,
    },
  })

// Set a new response header `x-hello-from-middleware2`
  response.headers.set('x-hello-from-middleware2', 'hello')
  return response
}
```

---

## **8. CORS**

미들웨어에서 CORS 헤더를 설정해 프리플라이트 요청이나 간단한 요청들을 포함한 CORS 요청을 허용할 수 있다.

```
// middleware.tsimport { NextRequest, NextResponse } from 'next/server'

const allowedOrigins = ['https://acme.com', 'https://my-app.org']

const corsOptions = {
  'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
}

export function middleware(request: NextRequest) {
// Check the origin from the requestconst origin = request.headers.get('origin') ?? ''
  const isAllowedOrigin = allowedOrigins.includes(origin)

// Handle preflighted requestsconst isPreflight = request.method === 'OPTIONS'

  if (isPreflight) {
    const preflightHeaders = {
      ...(isAllowedOrigin && { 'Access-Control-Allow-Origin': origin }),
      ...corsOptions,
    }
    return NextResponse.json({}, { headers: preflightHeaders })
  }

// Handle simple requestsconst response = NextResponse.next()

  if (isAllowedOrigin) {
    response.headers.set('Access-Control-Allow-Origin', origin)
  }

  Object.entries(corsOptions).forEach(([key, value]) => {
    response.headers.set(key, value)
  })

  return response
}

export const config = {
  matcher: '/api/:path*',
}
```

---

## **9. 응답 생성**

미들웨어에서 직접 응답할 수 있다. 이를 위해서는 **Response** 나 **NextResponse** 인스턴스를 반환하면 된다.

```
// middleware.tsimport { NextRequest } from 'next/server'
import { isAuthenticated } from '@lib/auth'

// Limit the middleware to paths starting with `/api/`export const config = {
  matcher: '/api/:function*',
}

export function middleware(request: NextRequest) {
// Call our authentication function to check the requestif (!isAuthenticated(request)) {
// Respond with JSON indicating an error messagereturn Response.json(
      { success: false, message: 'authentication failed' },
      { status: 401 }
    )
  }
}
```

### **9-1) waitUntil 과 NextFetchEvent**

**NextFetchEvent** 객체는 기존 **FetchEvent** 객체에서 확장된 것으로 **waitUnitl** 메서드를 갖고 있다.

**waitUntil** 메서드는 Promise 를 인수로 사용한다. Promise 가 해결될 때까지 미들웨어의 수명을 연장하는데 백그라운드 작업을 할 때 유용하다.

```
// middleware.tsimport { NextResponse } from 'next/server'
import type { NextFetchEvent, NextRequest } from 'next/server'

export function middleware(req: NextRequest, event: NextFetchEvent) {
  event.waitUntil(
    fetch('https://my-analytics-platform.com', {
      method: 'POST',
      body: JSON.stringify({ pathname: req.nextUrl.pathname }),
    })
  )

  return NextResponse.next()
}
```

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/middleware**

[Routing: Middleware | Next.js
Learn how to use Middleware to run code before a request is completed.
nextjs.org](https://nextjs.org/docs/app/building-your-application/routing/middleware)