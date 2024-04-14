
인터셉트 라우트(Intercept Route) 는 현재 레이아웃 내에서 애플리케이션의 다른 부분에서 경로를 보여줄 수 있게 만든다. 이 패러다임은 사용자가 다른 컨텍스트를 전환하지 않고도 경로의 내용을 표시하고 싶을 때 유용하다.

예를 들어, 피드에서 사진을 클릭할 때 사진을 모달로 표시해 피드 위에 겹쳐서 보이게 할 수 있다. 이 경우, Next.js 는 `/photo/123` 의 경로를 가로채서 URL 을 가리고 /feed 위에 덮어 올린다.

!https://blog.kakaocdn.net/dn/dOitc3/btsGvsas4lp/iAXmkHpnRqBoR3FPXhElq0/img.png

하지만, 공유 가능한 URL 을 클릭하거나 페이지를 새로 고쳐서 사진으로 이동할 때는 모달 대신 전체 사진 페이지가 렌더링돼야 한다.(* /photo/123 으로 이동) 이런 경우에는 경로 가로채기가 일어나선 안 된다.

!https://blog.kakaocdn.net/dn/cAp8IB/btsGt51nTdq/z9kwzCNegQpqJkOWs7eRC0/img.png

---

## **1. 규칙**

경로를 가로채는 것은 (..) 으로 정의할 수 있다. 이 방식은 마치 상대 경로를 찾으러 갈 대와 유사하다.

1. (.) 동일한 레벨의 Segment 를 매칭한다
2. (..) 한 단계 위의 Segment 를 매칭한다
3. (..)(..) 두 단계 위의 Segment 를 매칭한다
4. (...) 루트, app 디렉터리에서 Segment 를 매칭한다

예를 들어, 피드 Segment 내에서 사진 Segment 를 가로채기 위해선 `(..)photo` 디렉터리를 생성해야 한다.

!https://blog.kakaocdn.net/dn/Rjes6/btsGvW3rWPx/yydC8JyvTD4QvaVR7WxsSK/img.png

---

## **2. 예시**

### **2-1) 모달**

인터셉트 라우트는 모달을 만들 때 병렬 라우트와 함께 사용할 수 있다. 다음의 예시들은 갤러리에서 클라이언트 사이드 탐색을 활용해 사진 모달을 띄우거나 공유된 URL 을 통해 바로 `/photo` 로 넘어가는 UI다.

!https://blog.kakaocdn.net/dn/W8MJQ/btsGwMFNAUW/WoE5hA0jOM5CKohLCn0NIK/img.png

위의 예시에서 보면 모달에 보여줘야할 photo 는 앞에 (..) 이 사용됐다.(* 위에 @modal, feed 가 있고 이 photo 가 있어 (..)(..) 을 사용해야할 거 같지만) 왜냐하면 @modal 은 slot 이지 segment 가 아니기 때문.

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/route-handlers**

[Routing: Route Handlers | Next.js
Create custom request handlers for a given route using the Web's Request and Response APIs.
nextjs.org](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)