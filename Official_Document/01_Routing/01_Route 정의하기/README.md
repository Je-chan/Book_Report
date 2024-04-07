## **1. 경로 만들기**

Next.js 는 파일 시스템 기반이다. 따라서 경로를 정의하는데 폴더를 사용한다.

각각의 폴더는 경로의 한 부분(Route Segment) 를 가리키고 이는 URL 의 Segement 와 매핑된다. 중첩된 경로를 사용하기 위해서는 중첩 폴더 구조를 사용하면 된다.

!https://blog.kakaocdn.net/dn/4jDIH/btsGhGz9ju3/TGdWAM1IokL4QJpTcnHzEK/img.png

특수 파일 중 하나인 page.js 는 해당 경로에 공개되는 UI 다.

!https://blog.kakaocdn.net/dn/do0amC/btsGqrQFXRu/lskySa87LuNb5k6gRj1gnk/img.png

위의 예시를 들어보자면 `/dashboard/analytics` URL 경로는 page.ts 파일이 없으므로 접근이 불가능하다. 보통 이런 폴더는 컴포넌트, 스타일, 이미지 등을 넣기 위한 것으로 사용된다

---

## **2. UI 만들기**

각 경로별 segement 를 위한 UI 를 만들기 위해 특수 파일을 활용하는 규칙이 존재한다. 예를 들어서 페이지를 만들고자 할 때 app 디렉토리 내부에 page.js 를 넣어야 하며, React 컴포넌트로 반환해야 한다.

```
export default function Page() {
  return <h1>Hello, Next.js!</h1>
}
```

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/defining-routes**

[Routing: Defining Routes | Next.js
Learn how to create your first route in Next.js.
nextjs.org](https://nextjs.org/docs/app/building-your-application/routing/defining-routes)