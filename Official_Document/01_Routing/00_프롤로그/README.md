## **1. 용어**

!https://blog.kakaocdn.net/dn/beYL5h/btsGgDqatnl/X6cyopMtxxDVaZkABvfHV0/img.png

https://nextjs.org/docs/app/building-your-application/routing

### Tree

계층구조를 시각화하는 컨벤션이다. 예를 들면 부모와 자식간의 컴포넌트, 폴더 구조 등이 트리 구조로 돼있다.

### Subtree

트리의 한 부분이다. Root 트리가 새로 시작하여 마지막 Leaf 들로 끝난다.

### Root

Tree, 혹은 Subtree 의 시작이 되는 첫 번째 노드다.

### Leaf

Subtree 에서 더이상 자녀가 존재하지 않은 Subtree 의 노드들이다. 예를 들면 URL 에서 마지막 Segement 에 해당하는 것들이 있다.

!https://blog.kakaocdn.net/dn/cXeSVL/btsGgzuxzgK/ytX6S7ckeIVCFCJNXURNwK/img.png

### URL Segement

URL 의 한 부분으로 Slash 로 구분되는 것들이다.

### URL Path

URL 의 한 부분으로 Domain 뒤에 오는 모든 것을 의미한다.

---

## **2. app Router**

Next.js 13버전에서 React 서버 컴포넌트 기반의 새로운 App Router 를 만들었다. 이 router 를 사용하면 layout, 중첩 라우팅, loading 상태, 에러 핸들링 등 다양한 상황들을 지원한다.

App 라우터는 "app" 으로 불리는 새로운 디렉토리에서 작동한다. "app" 디렉토리를 채택했어도 이전에 사용되는 "pages" 라는 디렉토리도 사용할 수 있다.

다만, "app" 디렉토리르 사용한 앱 라우터가 페이지 라우터보다 더 우선된다. 디렉터리 간의 경로는 동일한 URL 경로를 사용하지 말아야 한다. 만일 동일한 경로를 사용하면 빌드할 때 문제가 생길 수 있다.

!https://blog.kakaocdn.net/dn/dVmntm/btsGi1cdQog/RTZ1q5MjjlzzGc4NUeSvS1/img.png

https://nextjs.org/docs/app/building-your-application/routing

기본적으로 "app" 내부 구성은 리액트의 서버 구성 요소다. 이 방식을 사용하면 성능이 최적화되고 클라이언트 컴포넌트도 사용할 수 있다.

---

## **3. 폴더와 파일 규칙**

****  Next.js 에서는 파일 시스템을 기반으로한 Route(경로) 를 사용한다.

폴더는 Route를 정의할 때 사용한다. Route 란 파일 시스템 계층을 따라서 Root 폴더에서 page.js 파일을 포함한 최종 Leaf 폴더로 이어진다.

파일은 UI 를 생성할 때 사용하며 Route Segement 에서 보여진다.

---

## **4. Route Segement**

****  각각의 폴더들은 Route Segement 다. 각각의 Route Segement 는 URL Path 의 Segement 와 동일하다.

!https://blog.kakaocdn.net/dn/bUZxjM/btsGiSNb77Q/LBbhkIN1NDLKOekiOrVVNk/img.png

---

## **5. Nested Routes**

****중첩 경로(Nested Routes) 를 생성하는 방법은 각각의 폴더 안에서 중첩된 폴더를 만드는 것이다. 예를 들어 /dashboard/settings 라는 Route(경로)를 만들고 싶다면 app 디렉토리 내에서 두 개의 폴더를 만들면 된다.

/ : (Root Segement)

dashboard : (Segement)

settings : (Segement)

---

## **5. File Conventions**

****Next.js 에서는 특별한 상황에 사용되는 UI 를 만들기 위해서 특별한 File 들이 존재한다.

| layout | 자식들이 공유하는 UI(* 항상 떠 있는 Sidebar 같은 거 만들 때) |
| --- | --- |
| page | route 와 매핑되는 특정한 UI |
| loading | 해당 segement 와 그의 자식들이 Loading 될 때 사용할 UI |
| not-found | 해당 segement 와 그의 자식들을 찾지 못했을 때 사용할 UI |
| error | 해당 segement 와 그의 자식들의 UI 에서 에러가 발생했을 때 사용할 UI |
| global-error | Global Error UI |
| route | 서버 사이드의 엔드포인트 |
| template | 리렌더링되는 Layout UI 에 특화 |
| default | 병렬 route 와 관련된 Fallback UI |

---

## **6. Component 계층 구조**

****Route 에서 위의 특수 파일로 명시된 React 컴포넌트는 특정 계층 구조를 갖고 리렌더링 된다.

!https://blog.kakaocdn.net/dn/NMaZ7/btsGiS0J0kx/mR8kTDQDpqS5WhZpyPAxUk/img.png

* Layout => template => error => loading => not-found => page 순으로 렌더링

중첩된 route 의 경우, 폴더 내의 컴포넌트들이 부모 폴더 내의 컴포넌트들 안으로 들어간다.

!https://blog.kakaocdn.net/dn/eAhmQ2/btsGhR9j0xb/NrVDE3wDK0M92624CPpRl0/img.png

---

## **7. Colocation(공동 배치)**

****특수 파일들 이외에도 app 디렉토리 내에 자신이 커스텀해서 만든 파일을 함께 넣을 수 있다. 컴포넌트, style, 테스트 코드 등 다양한 파일들을 넣을 수 있다.

이것이 가능한 이유는 해당 폴더에서 정의된 폴더는 page.js, route.js 에서

!https://blog.kakaocdn.net/dn/brx15h/btsGffJ9AJT/OzPmi8KHsdKBQ8i2qtMrXK/img.png

---

## **8. Advanced 라우팅 패턴**

****app 라우터는 더 진보된 형태의 라우팅 패턴을 제공한다.

### Parallel Routes (병행 경로)

하나의 화면에서 두 개 이상의 페이지를 동시에 표시할 수 있다. 대시보드와 같이 자체 navaigation 이 있는 대시보드에 사용하기 좋다

### Intercepting Routes (경로를 빼오기)

다른 경로의 UI 를 가로채 와서 다른 경로의 Context 에섭 보여줄 수 있다. 이런 패턴은 현재 페이지의 Context 를 유지하는 것이 중요할 때 사용할 수 있다. 예를 들어 하나의 작업을 편집하거나 피드에서 사진을 확장하는 동안에 다른 작업들의 상황을 모두 볼 수 있다.

## 

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing**

[Building Your Application: Routing | Next.js
Learn the fundamentals of routing for front-end applications.
nextjs.org](https://nextjs.org/docs/app/building-your-application/routing)