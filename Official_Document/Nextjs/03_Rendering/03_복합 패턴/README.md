React 애플리케이션을 빌드할 때 애플리케이션이 서버에서 렌더링돼야할지, 클라이언트에서 렌더링해야할지에 대한 고려를 해야 한다. 이 페이지에서는 서버 컴포넌트와 클라이언트 컴포넌트를 어느 때에 사용해야할지 몇가지 추천하는 복합 패턴들을 소개한다.

---

## **1. 서버 컴포넌트, 클라이언트 컴포넌트 사용 시기 요약**

| 필요한 작업 | 서버 컴포넌트 | 클라이언트 컴포넌트 |
| --- | --- | --- |
| 데이터 Fetching | O | X |
| 백엔드 자원에 직접적으로 접근 | O | X |
| 민감한 정보를 서버에서 유지(Access Toiken, API Key 등) | O | X |
| 서버에 의존성이 큰 경우와 클라이언트 사이드 JS 크기를 줄이고자 할 때 | O | X |
| 상호작용 및 이벤트 리스너 추가(onClick, onChange) | X | O |
| 상태 및 생명주기 이펙트 사용(useState, useReducer, useEffect) | X | O |
| 브라우저 전용 API 사용 | X | O |
| 상태(state), 이펙트(effect), 브라우저 전용 API 에 의존하는 커스텀 훅 사용 | X | O |
| React 클래스 컴포넌트 사용 | X | O |

---

## **2. 서버 컴포넌트 패턴**

클라이언트 사이드 렌더링을 선택하기 전에, 서버에서 데이터를 fetching 하거나, 데이터 베이스나 백엔드 서비스에 접근하는 작업을 수행할 수 있다.

### **2-1) 컴포넌트 간 데이터 공유**

서버에서 데이터를 fetching 할 때, 다른 컴포넌트 간에 가져온 데이터를 공유해야할 경우가 있을 수 있다. 예를 들어 같은 데이터에 의존하는 레이아웃과 페이지가 있을 수 있다.

React Context(* Context API)(서버에서 사용할 수 없다)를 사용하거나 데이터를 props 로 전달하는 대신, 여러 컴포넌트에서 필요로 하는 같은 데이터를 가져오기 위해 **fetch** 혹은 React 의 **cache** 함수를 사용할 수 있다. 이 방식은 중복된 요청에 대한 우려가 일절 없다. 왜냐하면 React 는 **fetch** 함수를 를 확장해서 데이터 요청을 자동적으로 메모리에 저장한다. 그리고 **cache** 함수는 **fetch** 함수를 사용할 수 없는 경우에 활용할 수 있다.

### **2-2) 클라이언트 환경에서 서버 코드 유지**

자바스크립트 모듈은 서버 및 클라이언트 컴포넌트 모듈 간에 공유될 수 있기에 오직 서버에서만 실행될 의도로 작성된 코드가 클라이언트로 유입될 수 있다.

예를 들면, 다음과 같은 데이터 fetching 함수가 있다고 가정해보자.

```
export async function getData() {
  const res = await fetch('https://external-service.com/data', {
    headers: {
      authorization: process.env.API_KEY,
    },
  })

  return res.json()
}
```

처음에는 getData 가 서버와 클라이언트 모두에서 동작될 것 같다. 하지만, 이 함수는 API_KEY 를 포함하고 있기 때문에 오직 서버에서만 실행이 될 것이란 의도로 작성됐다.

환경 변수 API_KEY 가 **NEXT_PUBLIC** 으로 시작하지 않기에 이는 서버에서만 접근할 수 있는 비공개 변수다. 환경 변수가 클라이언트에 누출되는 것을 막기 위해서, Next.js 는 비공개 환경 변수를 빈 문자열로 대체한다.

결과적으로 getData() 는 클라이언트에서 가져오고 실행할 수 있지만, 예상대로 동작하지 않는다. 변수를 공개하면 함수가 클라이언트에서 작동하게 되지만, 개발자의 의도는 민감한 정보를 클라이언트에 노출시키고 싶지 않을 수 있다. 이런 서버 코드의 의도치 않은 클라이언트 사용 방지를 위해, 다른 개발자가 실수로라도 이런 모듈 중 하나를 클라이언트 컴포넌트에 가지고 오면 빌드 시간 오류를 발생시키는 "server-only" 라는 패키지를 사용할 수 있다.

```
npm install server-only
```

그리고, 서버 전용 코드를 포함하는 모든 모듈에 설치한 패키지를 가져온다.

```
import 'server-only'

export async function getData() {
  const res = await fetch('https://external-service.com/data', {
    headers: {
      authorization: process.env.API_KEY,
    },
  })

  return res.json()
}
```

이제 getData() 를 가져오는 클라이언트 컴포넌트는 이 모듈을 서버에서만 사용할 수 있다는 빌드 오류를 받게 된다.

이에 상응하는 패키지로 "client-only" 가 있다.

### **2-3) 서드파티 패키지 및 Provider 사용**

서버 컴포넌트는 React 의 새로운 기능으로, React 생태계 내의 서드 파티 패키지와 제공자들은 useState, useEffect, createContext 와 같은 클라이언트 전용 기능을 사용하는 컴포넌트에 **use client** 디렉티브를 추가하기 시작했다.

현재 많은 npm 패키지의 컴포넌트들이 클라이언트 전용 기능을 사용하고는 있지만 아직 **use client** 디렉티브가 없을 수 있다. 이런 서드파티 컴포넌트들은 클라이언트 컴포넌트 내에서는 예상대로 동작하나 서버 컴포넌트 내에서는 제대로 동작하지 않는다.

예를 들어, 가상의 acme-carousel 패키지를 설치했다고 가정해보자. 이 패키지에 useState 를 사용하는`<Carousel />` 컴포넌트가 있다. 아직 **use client** 디렉티브가 없는 이 컴포넌트를 클라이언트 컴포넌트 내에서 사용하면 예상대로 동작한다.

```
'use client'

import { useState } from 'react'
import { Carousel } from 'acme-carousel'

export default function Gallery() {
  let [isOpen, setIsOpen] = useState(false)

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>View pictures</button>

      {/* Works, since Carousel is used within a Client Component */}
      {isOpen && <Carousel />}
    </div>
  )
}
```

하지만, 서버 컴포넌트에서 직접 사용하려고 하면 오류가 발생한다.

```
import { Carousel } from 'acme-carousel'

export default function Page() {
  return (
    <div>
      <p>View pictures</p>

      {/* Error: `useState` can not be used within Server Components */}
      <Carousel />
    </div>
  )
}
```

이 문제를 해결하기 위해서는 클라이언트 전용 기능에 의존하는 서드파티 컴포넌트를 자신의 클라이언트 컴포넌트로 감싸서 사용하면 된다.

```
// app/carousel.tsx'use client'

import { Carousel } from 'acme-carousel';

export default Carousel;
```

이제 서버 컴포넌트에서도 `<Carousel />` 컴포넌트를 직접 사용할 수 있다.

```
import Carousel from './carousel'

export default function Page() {
  return (
    <div>
      <p>View pictures</p>

      {/*  Works, since Carousel is a Client Component */}
      <Carousel />
    </div>
  )
}
```

대부분의 경우, 서드파티 컴포넌트를 감싸서 사용할 필요는 없을 것이다. 하지만, Provider(* 예를 들면, Store 를 사용할 때 Root Layout 에 적용하는 것들. React Toastify 를 쓸 때도 마찬가지)  는 예외다. Provider 는 일반적으로 애플리케이션 루트에서 필요하다. 때문에 React 상태와 컨텍스트에 의존한다.

### (1) Context Provider 사용하기

**Context Provider** 는 일반적으로 애플리케이션의 루트 근처에서 전역적인 상태(현재 테마와 같은 것)를 공유하기 위해 렌더링된다. React Context 는 서버 컴포넌트에서 지원되지 않기에 애플리케이션의 루트에서 컨텍스트를 만들려고 하면 에러가 발생한다.

```
// app/layout.tsximport { createContext } from 'react';

// createContext is not supported in Server Componentsexport const ThemeContext = createContext({});

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
      </body>
    </html>
  );
}
```

이를 해결하기 위해선 컨텍스트와 그 제공자를 클라이언트 컴포넌트 내에서 생성하고 렌더링 해야 한다.

```
// app/theme-provider.tsx'use client'

import { createContext } from 'react';

export const ThemeContext = createContext({});

export default function ThemeProvider({
  children,
}: {
  children: React.ReactNode
}) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
}
```

이제 서버 컴포넌트는 클라이언트 컴포넌트로 표시된 Priovider 를 직접 렌더링할 수 있다.

```
// app/layout.tsximport ThemeProvider from './theme-provider';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

Providr 가 루트에 렌더링되면 앱 전체의 다른 클라이언트 컴포넌트들이 이 Context 를 사용할 수 있다.

Provider 는 가능한 트리 깊숙이 렌더링하는 것이 좋다. 위의 예제에서 ThemeProvider 가 전체 `<html>` 아닌 {children} 만 감싸는것에 주목해야 한다. 이는 Next.js 가 서버 컴포넌트의 정적 부분을 최적화하기 쉽게 만든다.

---

## **3. 클라이언트 컴포넌트**

### **3-1) 클라이언트 컴포넌트를 트리 하위로 이동하기**

클라이언트 자바스크립트 번들 크기를 줄이기 위해서, 클라이언트 컴포넌트를 컴포넌트 트리 하위로 이동하는 것이 좋다.

예를 들어 정적 요소(로고, 링크 등)와 상태를 사용하는 상호작용 검색 바를 포함한 레이아웃이 있을 수 있다.

전체 레이아웃을 클라이언트 컴포넌트로 만드는 대신 상호작용하도록 만드는 로직을 클라이언트 컴포넌트로 이동시키는 것이 좋다. (아래의 예씨에서 `<SearchBar />`) 그리고 레이아웃은 서버 컴포넌트로 유지하면 된다. 이렇게 하면 레이아웃의 모든 컴포넌트 자바스크립트를 클라이언트로 전송할 필요가 없게 된다.

```
// app/layout.tsx// SearchBar는 클라이언트 컴포넌트입니다.import SearchBar from './searchbar';
// Logo는 서버 컴포넌트입니다.import Logo from './logo';

// Layout는 기본적으로 서버 컴포넌트입니다.export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Logo />
        <SearchBar />
      </nav>
      <main>{children}</main>
    </>
  );
}
```

### **3-2) 서버에서 클라이언트 컴포넌트로 props 전달하기(직렬화)**

서버 컴포너트에서 데이터를 가져오고, 클라이언트 컴포넌트로 props 를 전달하고 싶을 수 있다. 서버에서 클라이언트 컴포넌트로 전달되는 속성은 React 에 의해 직렬화(Serilaze)가 가능해야 한다. (* 직렬화가 가능한 데이터는 원시 타입은 모두 가능하고, Array, Map, Set, TypedArray, ArrayBuffer, Date, FormData, Plain Object, Functions, Promise 등이다. 반면 지원하지 않는 것은 React Element 나 JSX과 이것을 반환하는 함수, Class, 전역적으로 등록하지 않은 Symbols)

클라이언트 컴포넌트가 직렬화할 수 없는 데이터에 의존하는 경우, 서드 파티 라이브러리를 사용해 클라이언트에서 데이터를 가져오거나 **Route Handler** 를 활용해서 데이터를 가져올 수 있다.

---

## **4. 서버 컴포넌트, 클라이언트 컴포넌트 교차 사용**

서버 컴포넌트와 클라이언트 컴포넌트를 교차해서 사용할 때, UI 를 컴포넌트 트리로 시각화하는 것이 도움될 수 있다. 서버 컴포넌트인 루트 레이아웃에서 시작해 **use client** 디렉티브를 추가하여 클라이언트에서 특정 서브 트리를 렌더링할 수 있다.

이런 클라이언트 서브트리 내에서도 서버 컴포넌트를 중첩하거나 서버 액션을 호출할 수 있지만, 몇 가지 사항을 염두에 두어야 한다.

1. 요청-응답의 생명주기 동안, 코드는 서버에서 클라이언트로 이동한다. 클라이언트 상태에서 서버의 데이터나 자원에 접근해야 하는 경우, 서버로 새로운 요청을 하게 되는 것이지 앞뒤로 스위칭되는 것이 아니다.
2. 서버로 새 요청이 이루어지면 모든 서버 컴포넌트가 먼저 렌더링된다. 이는 클라이언트 컴포넌트 내에 중첩된 컴포넌트들을 포함한다. 렌더링 결과(**RSC Payload**) 는 클라이언트 컴포넌트 위치에 대한 참조를 포함한다. 그런 다음 클라이언트에서 React 는 **RSC Payload** 를 사용해 서버와 클라이언트 컴포넌트를 하나의 트리로 조정한다.
3. 클라이언트 컴포넌트는 서버 컴포넌트 뒤에 렌더링된다. 서버 컴포넌트를 클라이언트 컴포넌트 모듈로 직접 가져올 수 없다(서버로 다시 요청해야 한다) 대신, 서버 컴포넌트를 클라이언트 컴포넌트의 props 로 전달할 수 있다.

### **4-1) 지원하지 않는 패턴 : 클라이언트 컴포넌트에서 서버 컴포넌트 import**

다음 패턴은 지원하지 않는다. 서버 컴포넌트를 클라이언트 컴포넌트로 직접 import 할 수 없다.

```
'use client'

// You cannot import a Server Component into a Client Component.import ServerComponent from './Server-Component'

export default function ClientComponent({
  children,
}: {
  children: React.ReactNode
}) {
  const [count, setCount] = useState(0)

  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>

      <ServerComponent />
    </>
  )
}
```

### **4-2) 지원하는 패턴 : 클라이언트 컴포넌트에 서버 컴포넌트를 Props 로 전달**

다음 패턴은 지원한다. 서버 컴포넌트를 클라이언트 컴포넌트의 props 로 전달할 수 있다.

일반적인 패턴은 React **children** props 를 사용하여 클라이언트 컴포넌트에서 "slot" 을 형성하는 것이다. 아래의 예제에서 `<ClientComponent>` 는 **children** prop 을 받는다.

```
'use client'

import { useState } from 'react'

export default function ClientComponent({
  children,
}: {
  children: React.ReactNode
}) {
  const [count, setCount] = useState(0)

  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      {children}
    </>
  )
}
```

`<ClientComponent>` 는 **children** 이 결국 서버 컴포넌트의 결과로 채워질 것인지 알지 못한다. `<ClientComponent>` 의 유일한 책임은 **children** 이 최종적으로 배치될 위치를 결정하는 것이다.

부모 서버 컴포넌트에서, `<ClientComponent>` 와 `<ServerComponent>` 를 모두 가져와서 `<ServerComponent>` 를 `<ClientComponent>` 의 자식으로 전달할 수 있다.

```
// app/page.tsx// 이 패턴은 작동합니다:// 서버 컴포넌트를 클라이언트 컴포넌트의 자식이나 prop으로 전달할 수 있습니다.import ClientComponent from './client-component'
import ServerComponent from './server-component'

// Next.js의 페이지는 기본적으로 서버 컴포넌트입니다export default function Page() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  )
}
```

이런 접근 방식으로 `<ClientComponent>` 와 `<ServerComponent>` 는 독립적으로 렌더링될 수 있으며, 이 경우 `<ServerComponent>` 는 클라이언트가 렌더링되기 훨씬 전에 서버에서 렌더링될 수 있다.

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns**

[Rendering: Composition Patterns | Next.js
Recommended patterns for using Server and Client Components.
nextjs.org](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns)