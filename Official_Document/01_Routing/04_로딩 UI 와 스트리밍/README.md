특수 파일인 loading.js 는 React Suspense 와 함께 의미 있는 로딩 UI 를 생성하는데 동무을 준다. 이 규칙으로 루트 Segment 의 컨텐츠가 로드되는 동안, 서버에서 즉시 로딩 상태를 표시할 수 있다. 렌더링이 완성되면 새로운 컨텐츠로 자동 교체된다.

!https://blog.kakaocdn.net/dn/cY1mDF/btsGqNTwxCy/GhVsvDDK5RNvkOcFX3e9GK/img.png

---

## **1. 즉각적인 Loading 상태(Instant Loading States)**

즉각적인 Loading 상태는 경로를 이동했을 때 즉각적으로 보이는 fallback(* 번역하면 "대체"지만, React Suspense 에서 fallback prop 의미를 살리고자 fallback 을 그대로 사용함) UI 다. 스켈레톤이나 스피너와 같은 로딩 표시기를 미리 렌더링하거나, 후에 보여질 커버 사진, 제목 등의 작은 부분을 표시할 수도 있다.

로딩 상태를 만드는 방법은 loading.js 파일을 추가하는 것이다.

!https://blog.kakaocdn.net/dn/07G8N/btsGrUxFDYs/DwqTw47GA5ZPKJcR8wERM1/img.png

```
// app/dashboard/loading.tsxexport default function Loading() {
// 로딩 내에 스켈레톤과 같은 모든 UI를 추가할 수 있습니다.return <LoadingSkeleton />;
}
```

같은 폴더에서 loading.js 는 layout.js 내에 존재한다. 이 파일은 자동으로 page.js 파일과 그 하위 자식을 <Suspense> 로 감싼다.

!https://blog.kakaocdn.net/dn/cUTJAa/btsGrdYwvLb/tZWsXPKyXmRf4fr8MkgEq1/img.png

경로 이동은 즉시 이루어지고, 서버 중심의 경로 변경이더라도 동작한다.

경로 이동 중에는 중단이 가능하고, 경로의 콘텐츠가 완전히 로드될 때까지 다른 경로로 이동하기 위해 기다릴 필요가 없다.

새로운 루트 Segement 가 로드되는 동안 공유하고 있는 레이아웃은 상호 작용이 가능한 상태로 남아 있는다.

---

## **2. Suspense 를 활용한 스트리밍**

loading.js 이외에도 사용자가 만든 UI 컴포넌트에 **Suspense** 를 만들 수 있다. **App Router** 는 Node.js 및 Edge 런타임 모두에서 Suspense 를 사용한 스트리밍을 지원한다.

### **2-1) 스트리밍(Streaming) 이란?**

React 및 Next.js 에서 스트리밍이 동작하는 방법을 알기 전에 SSR 과 그 한계에 대한 이해를 하는 것이 좋다.

SSR 을 하게 되면, 사용자가 상호작용하기 위한 페이지를 보기 전에 몇 가지 단계를 거치게 된다.

1. 특정 페이지의 모든 데이터를 서버에서 가져온다.
2. 서버는 페이지의 HTML 을 렌더링한다.
3. 페이지의 HTML, CSS, JS 가 클라이언트로 전송된다.
4. 생성된 HTML 및 CSS 를 사용하여 상호작용이 불가능한 사용자 인터페이스를 보여준다.
5. React 가 사용자 인터페이스를 상호 작용 가능하게 만들기 위해 **hydrates** 를 한다.(* React hydration 은, 정적인 HTML 파일로 그려진 DOM Tree 에서 이벤트 리스너를 구독하기 위한 DOM 요소를 찾아 자바스크립트 속성만 넣는 것을 의미한다. React.hydrate(element, container, [callback])  hydrate 라는 용어는 수분감을 준다는 의미인데, 아무런 상호작용도 할 수 없는 메마른 DOM 에 생동감 있는 상호작용 코드를 불어 넣어 수분감 있게 DOM 을 꾸민다는 의미로 받아들이면 될 것 같다.)

!https://blog.kakaocdn.net/dn/bByQNQ/btsGqKJnlZz/gvKE2oem6LkTd9MF5O0tn1/img.png

이런 단계는  순차적이면서 블로킹된다. 다시 말해, 서버는 페이지의 HTML 을 렌더링할 때 모든 데이터를 다 가져올 때까지 기다려야 한다. 그리고 클라이언트에서는 페이지의 모든 컴포넌트 코드가 다운로드될 때까지 UI 를 **hydrate** 할 수 없다.

React, Next.js 를 사용해서 구현하는 SSR 은 사용자에게 가능한 빨리 비상호작용적인 페이지를 최대한 빠르게 보여줌으로써 페이지 로드 성능을 개선한다.

!https://blog.kakaocdn.net/dn/LdIPj/btsGqyWDyJ9/IEQYgrVwNPgofPHDu2pHPk/img.png

하지만, 여전히 데이터를 모두 가져와야지만 사용자에게 보일 수 있으므로 느리다.

스트리밍을 사용하면, 페이지의 HTML 을 작은 조각으로 분할하고 이런 조각을 서버에서 클라이언트에 점진적으로 전송한다.

!https://blog.kakaocdn.net/dn/dujfFZ/btsGpXQeU01/Ds3ZupJKc7znNkF8kOKKG1/img.png

이런 방식을 사용하면 페이지의 일부를 빠르게 화면에 표시할 수 있고 어떤 UI 도 렌더링 되기 전에 모든 데이터를 기다릴 필요가 없게 된다.

React 의 컴포넌트 모델은 각 컴포넌트를 청크로 고려하기에 스트리밍 방식과 매우 잘 어울린다. 우선순위가 높은 컴포넌트나 데이터에 의존하지 않는 컴포넌트를 먼저 전송할 수도 있다. React 는 이런 컴포넌트 **hydration** 을 더 일찍 동작하게 할 수 있다. 우선순위가 낮은 컴포넌트들은 데이터를 가져온 후에 동일한 서버 요청에서 전송될 수 있다.

!https://blog.kakaocdn.net/dn/bqIH9b/btsGqNTw3wB/HK8ApG51k934cKPpm7Tpe0/img.png

스트리밍은 페이지가 빠르게 렌더링되는 것을 막는 방대한 양의 데이터 요청을 방지할 때 유용하다. 이 방식을 통해 TTFB(Time To First Byte), FCP(First Contentful Paint) 를 줄일 수 있다. 또, 느린 장치에서 특히 TTI(Time to Interactive) 를 개선하는데 도움이 된다.

### **2-2) 예시**

`<Suspense>` 는 비동기 작업을 수행하는 컴포넌트를 감싸고, 작업이 진행되는 동안 대체 UI 를 표시한다. 작업이 완료되면 컴포넌트를 교체하는 방식으로 동작한다.

```
import { Suspense } from 'react'
import { PostFeed, Weather } from './Components'

export default function Posts() {
  return (
    <section>
      <Suspense fallback={<p>Loading feed...</p>}>
        <PostFeed />
      </Suspense>
      <Suspense fallback={<p>Loading weather...</p>}>
        <Weather />
      </Suspense>
    </section>
  )
}
```

`<Suspense>` 를 사용하면 다음과 같은 이점을 얻을 수 있다.

- 서버에서 클라이언트로 HTML 을 점진적으로 렌더링하는 스트리밍 서버 렌더링
- 사용자 상호 작용에 따라 먼저 상호 작용 가능한 컴포넌트를 결정하는 선택적인 **hydration**

---

## **3. SEO**

Next.js 는 **generateMetadata** 내부의 데이터를 fetching 완료한 후에 클라이언트로 UI 를 스트리밍한다. 이건 스트리밍 응답의 첫 부분에 `<head>` 태그가 포함되도록 보장한다.

즉, 스트리밍 방식도 결국 서버에서 렌더링되기에 SEO 에 영향을 미치지 않는다.

---

## **4. 상태 코드**

스트리밍 중에는 요청이 성공했다는 신호로 200 상태 코드를 반환한다.

서버는 리다이렉션이나 Not Found 같은 문제나 오류를 스트리밍된 콘텐츠 내에서 클라이언트에 전달할 수 있다. 응답 헤더가 이미 클라이언트로 전송됐기에 응답의 상태 코드를 업데이트할 수 없다. 이는 SEO 에 영향을 미치지 않는다.

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming**