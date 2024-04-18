클라이언트 컴포넌트를 사용하면 서버에서 사전에 렌더링된 UI 를 상호작용 가능하게 만들어주며 브라우저에서 클라이언트 JS 를 실행시킬 수 있다.

---

## **1. 클라이언트 렌더링의 장점**

클라이언트에서 렌더링 작업을 수행하면 몇 가지 이점이 존재한다.

1. 상호작용: 클라이언트 컴포넌트는 상태, 효과, 이벤트 리스너를 사용할 수 있다. 따라서 사용자에게 즉각적인 피드백을 제공하고 UI 를 업데이트 할 수 있다.
2. 브라우저 API: 클라이언트 컴포넌트는 브라우저 API 에 접근할 수 있다. 예를 들면, **geolocation** 이나 **localStorage** 등이 있다.

---

## **2. Next.js 에서 클라이언트 컴포넌트 사용하기**

클라이언트 컴포넌트를 사용하기 위해서는 React 의 **use client** 디렉티브를 import 문보다 더 위인 파일 최상단에 사용해야 한다.

**use client** 는 서버 컴포넌트, 클라이언트 컴포넌트 모듈 간의 경계를 선언한다. **use client** 를 파일에 정의하여 그 파일로 가져온 모든 다른 모듈들, 포함된 자식 컴포넌트들까지 클라이언트 번들의 일부로 간주한다.

```
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
}
```

아래의 다이어그램은 중첩된 컴포넌트(toggle.js) 에서 onClick, useState 를 사용할 때, **use client** 디렉티브가 정의돼 있지 않으면 오류를 발생시킨다는 것을 보여준다. 이는 기본적으로 **App Router** 의 모든 컴포넌트가 이런 API 를 사용할 수 없는 서버 컴포넌트기 때문이다. toggle.js 에서 "use client" 디렉티브를 정의함으로써, React 에게 이런 종류의 API 를 사용할 수 있는 클라이언트 바운더리임을 알려준다.

!https://blog.kakaocdn.net/dn/lOJ0l/btsGDA6re0W/KBpmcPTyCsK7f9qXtlkSH0/img.png

React 컴포넌트 트리에서 여러 "use client" 진입점(엔트리 포인트)를 설정할 수 있다. 이 방식은 애플리케이션을 여러 클라이언트 번들로 분할할 수 있게 해준다. 하지만 클라이언트에서 렌더링 해야 하는 모든 컴포넌트에 "use client" 를 정의할 필요가 없다. 경계를 한 번 정의하면, 그 안으로 가져온 모든 자식 컴포넌트와 모듈이 클라이언트 번들의 일부로 간주되기 때문이다.

---

## **3. 클라이언트 컴포넌트는 어떻게 렌더링하는가?**

Next.js 에서 클라이언트 컴포넌트는 요청이 전체 페이지를 로드(애플리케이션 첫 방문이나 브라우저 새로고침에 의한 페이지 리로드)하는 것인지, 아니면 후속 탐색의 일부인지에 따라 달라진다.

### **3-1) 전체 페이지 로드(Full Page Load)**

초기 페이지 로드를 최적화하기 위해, Next.js 는 React API 를 활용하여 클라이언트 및 서버 컴포넌트 모두에 대해 정적 HTML 미리보기를 서버에서 렌더링한다. 다시 말해, 사용자가 처음으로 애플리케이션을 방문하면 클라이언트 컴포넌트에서 사용될 자바스크립트 번들을 다운로드, 파싱, 실행할 필요 없이 페이지의 내용을 즉시 보게 된다.

서버에서는 아래의 과정이 진행된다.

1. React 는 클라이언트 컴포넌트에 대한 참조 데이터가 있는 특별 데이터 형식인 **RSC Payload** 로 서버 컴포넌트들을 렌더링한다.
2. Next.js 는 RSC Payload 와 클라이언트 JS 지시사항 을 사용해 서버에서 해당 경로에 대한 HTML 을 렌더링한다.

클라이언트에서는 다음의 과정이 진행된다.

1. HTML 파일은 해당 경로에서 상호작용은 안 되지만 가장 빠르게 화면을 보여주기 위해 사용된다.
2. *RSC Payload** 는 클라이언트 및 서버 컴포넌트 트리를 조정하고 DOM 을 업데이트하는데 사용된다.
3. JS 지시사항 은 클라이언트 컴포넌트를 **hydrate** 하고 UI 를 상호작용이 가능하게 만든다.

**hydrate**란 정적 HTML 을 상호작용 가능하게 만들기 위해서 DOM 에 이벤트 리스너를 연결하는 과정이다. 내부적으로는 **hydrateRoot** 라는 React API 를 활용한다.

### **3-2) 후속 탐색**

후속 탐색에서 클라이언트 컴포넌트는 서버에서 렌더링된 HTML 없이 전적으로 클라이언트에서 렌더링된다. 이는 클라이언트 컴포넌트 JS 번들이 다운로드되고 파싱된다. 번들이 준비되면 React 는 **RSC Payload** 를 활용해서 클라이언트 및 서버 컴포넌트 트리를 조정하고 DOM 을 업데이트한다.

---

## **4. 서버 환경으로 돌아가기**

**use client** 를 활용해 클라이언트 환경으로 바운더리를 설정했지만 서버 환경으로 돌아가고 싶을 때가 있을 수 있다. 예를 들어 클라이언트 번들 크기를 줄이거나, 서버에서 데이터를 가져오거나, 서버에서만 사용 가능한 API(* 예를 들면 cookies 같은 거) 를 사용하려는 경우가 있다.

클라이언트 컴포넌트, 서버 컴포넌트, **Server Actions** 가 섞인 중첩된 클라이언트 컴포넌트에서도 서버 코드를 유지시킬 수 있다. 이 방식은 다음 챕터의 Composition Patterns 에서 확인하면 된다.

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/rendering/client-components**

[Rendering: Client Components | Next.js
Learn how to use Client Components to render parts of your application on the client.
nextjs.org](https://nextjs.org/docs/app/building-your-application/rendering/client-components)