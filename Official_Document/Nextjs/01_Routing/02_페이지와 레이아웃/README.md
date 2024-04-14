layout.js, page.js, template.js. 이 세 개의 특수 파일은 경로에서 UI 를 만드는데 사용된다. 이 페이지에서는 이 특수 파일을 어떻게 사용할 수 있는지에 대해 설명한다.

# **1. Pages**

page 는 해당 경로의 하나뿐인 UI 다. page.js 파일을 통해서 컴포넌트를 export 해 페이지를 정의할 수 있다. 예 를 들어서 index 페이지를 만들고 싶다면, app 디렉토리 안에 page.js 파일을 넣으면 된다. (* 그 해당 경로의 고유 UI 를 index.html 이 아닌 page.js 로 정의하면 된다는 것 같다)

!https://blog.kakaocdn.net/dn/cD6byx/btsGqkxtCAs/mJy9pHQDqrGKcchQqBjgSK/img.png

```tsx
// `app/page.tsx` is the UI for the `/` URLexportdefaultfunctionPage() {
return <h1>Hello, Home page!</h1>
}
```

그 다음 더 많은 페이지들을 만들기 위해서 새로운 폴더를 ㅁ나들고 page.js 안에 넣으면 된다. 예를 들어, `/dashboard` 라는 경로를 ㅁ나들고자 할 때는 `dashboard` 라는 이름의 폴더를 만들고 그 안에 page.js 를 넣으면 된다.

---

# **2. Layouts**

레이아웃 은 다수의 경로에서 공유되는 UI 다. 이곳, 저곳 다른 페이지들을 방문할 때 layout 상태는 유지되고 상호 작용되는 상태로 유지되며 리렌더링 하지 않는다. 또, 레이아웃도 중첨이 가능하다.

레이아웃을 정의하는 방법은 layout.js 파일에서 React 컴포넌트를 export 하는 것이다. export 된 컴포넌트는 **children** prop 을 받아야 하며, 자식 컴포넌트, 자식 레이아웃 컴포넌트 등이 렌더링되는 동안에 채워진다. (* 이는 모든 하위 컴포넌트들이 해당 레이아웃을 포함한다는 얘기다)

!https://blog.kakaocdn.net/dn/cvalii/btsGqJKlod0/QeS6NTBC46f5ks89hIJ2dk/img.png

따라서 `/dashboard` 경로의 page.js 나, `dashboard/settings` 경로의 page.js 는 동일한 레이아웃을 공유한다.

```tsx
exportdefaultfunctionDashboardLayout({
  children, // will be a page or nested layout
}: {
  children: React.ReactNode
}) {
return (
    <section>
      {/* Include shared UI here e.g. a header or sidebar */}
      <nav></nav>

      {children}
    </section>
  )
}
```

### **2-1) Root Layout (Required)**

루트 레이아웃은 app 디렉토리의 최상위 레벨에서 정의되고 모든 경로에 다 적용된다. 이 레이아웃은 필수조건이며 반드시 html, body 태그를 포함해야 한다. 그리고 이 루트 레이아웃을 통해서 최초에 서버에서 보내줄 HTML 을 수정할 수 있다.

```tsx
exportdefaultfunctionRootLayout({
  children,
}: {
  children: React.ReactNode
}) {
return (
    <html lang="en">
      <body>
        {/* Layout UI */}
        <main>{children}</main>
      </body>
    </html>
  )
}
```

### **2-2) 중첩 레이아웃**

기본적으로 폴더 계층 구조에서 레이아웃은 중첩이 된다. 이는 **children** prop 을 통해서 자식 레이아웃을 감쌀 수 있다. 특정 경로의 segment(폴더) 내에 layout.js 를 추가하여 레이아웃을 중첩할 수 있다.

예를 들어서 /dashboard 경로에 대한 레이아웃을 만들려면 대시보드 폴더 안에 새 layout.js 파일을 추가하면 된다.

!https://blog.kakaocdn.net/dn/HebzF/btsGqOx5rjN/YakxplQ3k8sWR9Ff2HkhQK/img.png

```tsx
exportdefaultfunctionDashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
return <section>{children}</section>
}
```

위의 예시처럼 두개의 레이아웃이 결합되려면 루트 레이아웃은 dashboard 레이아웃을 감싸는 형태가 된다.

### **2-3) 참고**

- Root 레이아웃에서는 <html> 태그와 <body> 태그를 모두 갖고 있어야 한다.
- layout.js 와 page.js 가 한 폴더 내에 같이 있으면 layout.js 가 page.js 를 감싸는 형태가 된다
- layout.js 는 기본적으로 서버 컴포넌트다. (클라이언트 컴포넌트로는 변경 가능하다)
- layout.js 에서 데이터를 fetching 할 수 있다.
- 레이아웃에서 **children** 으로 데이터를 전달할 수 없다. 하지만 동일한 데이터를 여러 번 fetching 할 수 있으며 React 는 자동으로 성능에 영향을 주지 않고 요청을 중복 제거한다.
- 레이아웃은 경로에 직접적인 접근이 불가능하다. 모든 경로에 접근하기 위해서는 클라이언트 컴포넌트에서 **useSelectedLayoutSegement(s)** 를 사용해야 한다.

---

# **3.  Templates**

templates 는 각각의 레이아웃이나 페이지를 감싼다는 점에서 layout.js 와 비슷해보인다. 하지만, 레이아웃이 경로를 이동하더라도 상태를 유지하는 것과는 달리 template 은 경로간 탐색 시 새로운 인스턴스를 매번 생성한다. 따라서 사용자가 템플릿을 공유하는 경로를 탐색할 때마다 컴포넌트의 새로운 인스턴스를 매번 mount 하고 DOM 엘리먼트가 계속 재생산되며 상태는 유지되지 않는다.

몇몇 특정한 상황에서는 이런 기능을 사용해야할 때가 있다.

- **useEffect** 나 **useState** 에 의존하는 기능.
- 기본적인 프레임워크 동작을 변경하는 경우: 예를 들면, 레이아웃 내부의 **Suspense** 는 레이아웃이 처음 로드될 때만 fallback 을 보여주고 페이지를 전환하는 동안에는 보여주지 않는다. template 을 사용하면 각 탐색마다 fallback 이 표시된다.

template 은 template.js 파일에서 React 컴포넌트를 export 하여 정의할 수 있다. 이 컴포넌트도 마찬가지로 **children** props 를 받아야 한다.

!https://blog.kakaocdn.net/dn/QdO6O/btsGsjRvy30/gz1dYT1YcWTKGcfc5yFhlk/img.png

```tsx
exportdefaultfunctionTemplate({ children }: { children: React.ReactNode }) {
return <div>{children}</div>
}
```

---

# **4. Metadata**

app 디렉토리에서 Metadata API 를 활용해 title, meta 와 같은 <head> HTML 요소를 수정할 수 있다.

Metadata 는 layout.js,page.js 파일에서 metadata 객체, 또는 generateMetadata 함수를 export 하여 정의한다.

```tsx
import {Metadata }from 'next'

exportconst metadata:Metadata = {
  title: 'Next.js',
}

exportdefaultfunctionPage() {
return '...'
}
```

### **4-1) 참고**

루트 레이아웃에 대한 <title>, <meta> 와 같은 <head> 태그를 수동으로 추가하면 안 된다. 반드시 Metadata API 를 활용해야 하며, 이는 <head> 요소를 스트리밍하고 중복을 제거하는 등의 요구 사항을 자동으로 처리한다.