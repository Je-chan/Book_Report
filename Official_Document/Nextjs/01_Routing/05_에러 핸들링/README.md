error.js 파일 규칙을 사용하면 중첩된 경로에서 예상치 못한 런타임 오류를 알맞게 처리할 수 있다.

- React 의 `<ErrorBoundary>` 를 자동 생성해 해당 경로와 중첩된 자식을 감싼다.
- 파일 시스템 계층 구조를 사용해서 특정 Segment 에 맞는 맞춤형 에러 UI 를 만들 수 있다.
- 영향을 받은 Segment 를 격리하면서 나머지 애플리케이션은 기능적으로 유지할 수 있다.
- 전체 페이지를 다시 로드하지 않고도 오류에서 복구를 시도할 수 있는 기능을 추가할 수 있다.

Error 관련 UI 를 만드는 방법은 Segment 내부에 error.js 파일을 추가하고 React 컴포넌트를 exporting 하는 것이다.

!https://blog.kakaocdn.net/dn/kHJ2s/btsGrcFp42Y/NLiKoYG5zEQP3mIeKnCtc0/img.png

```
'use client'// Error components must be Client Components

import { useEffect } from 'react'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
// Log the error to an error reporting serviceconsole.error(error)
  }, [error])

  return (
    <div>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </button>
    </div>
  )
}
```

---

## **1. error.js 동작 원리**

!https://blog.kakaocdn.net/dn/5nwlX/btsGsWV9vqB/D7pGdZ8LyK055c2ZWLdkRk/img.png

error.js 는 중첩된 하위 Segment 나 page.js 컴포넌트를 감싸는 React `<ErrorBoundary>` 를 자동으로 생성한다. error.js 파일에서 export 한 컴포넌트가 fallback 컴포넌트로 사용된다. `<ErrorBoundary>` 내에서 에러가 발생하면 에러가 포함되고 fallback 컴포넌트가 렌더링된다.

fallback 에러 컴포넌트가 활성화되면 `<ErrorBoundary>` 위의 레이아웃은 상태를 유지하고 상호작용이 가능하다. 에러 컴포넌트는 에러에서 복구하는 기능을 표시할 수 있다.

---

## **2. 에러에서 복구하기**

에러의 원인은 일시적일 수 있다. 이런 경우, 다시 시도하면 문제가 해결된다.

에러 컴포넌트는 **react()** 함수를 사용해서 사용자로부터 에러에서 복구를 시도하도록 요청할 수 있다. 함수를 실행하면 `<ErrorBoundary>` 에 감싸여 있던 컴포넌트를 렌더링하기 위해 다시 시도한다. 만약, 성공하면 fallback 에러 컴포넌트는 없어지고 렌더링 결과가 화면에 표시된다

---

## **3. 중첩 경로**

특수 파일을 통해 생성된 React 컴포넌트는 특정 중첩된 계층 구조에 렌더링된다. 예를 들어서 두 개의 Segment 가 모두 layout.js, error.js 파일을 포함하는 중첩 경로인 경우, 다음과 같은 간소화된 컴포넌트 계층 구조로 렌더링된다.

!https://blog.kakaocdn.net/dn/bzvkAn/btsGsjRABYz/faJ3Y6LJkww1BDbQX2g2N1/img.png

중첩된 컴포넌트 계층 구조는 중첩된 경로 전체에 걸쳐 error.js 파일의 동작에 영향을 준다.

에러는 가장 가까운 부모 `<ErrorBoundary>` 로 전달(* 버블링)된다. 다른 말로 바꾸자면, error.js 파일은 모든 중첩된 자식 Sgement 에 대한 에러를 처리한다. 경로의 다른 수준에 error.js 파일을 배치하여 더 정밀한 에러 UI 를 구현할 수 있다.

error.js 의 범위는 layout.js 에서 발생한 에러를 처리하지 못한다. 에러 경계가 해당 레이아웃의 컴포넌트 내부에 중첩되기 때문이다.

---

## **4. 레이아웃 내에서 발생한 Error 핸들링**

error.js 의 경계는 동일한 Segment 의 layout.js 또는 template.js 컴포넌트에서 발생한 에러를 핸들링하지 못한다. 이런 의도적인 계층 구조는 에러가 발생했을 때도 중요한 UI 가 기능적으로 유지할 수 있도록 한다.

특정 레이아웃이나 템플릿 내에서 에러를 처리하려면 레이아웃의 부모 Segment 에 error.js 파일을 배치해야 한다.

만약, 루트 레이아웃이나 템플릿 내에서 에러를 처리하려면 루트 디렉토리에 위치한 error.js 를 변형해서 global-error.js 파일로 만들어 사용하면 된다.

---

## **5. 루트 레이아웃에서 발생한 Error 핸들링**

`app/error.js` 는 루트 레이아웃(`app/layout.js`), 루트 템플릿(`app/template.js`) 컴포넌트에서 발생한 에러를 잡지 못한다.

이런 루트 컴포넌트의 에러를 명확하게 처리하기 위해선 루트 app 디렉토리에 위치한 error.js 의 변형인 `app/global-error.js` 를 사용하면 된다. 이 파일은 전체 애플리케이션을 감싸며, 활성화될 때 루트 레이아웃을 대체하는 fallback 컴포넌트로 작동한다. 따라서 global-error.js 파일을 만들 때는 <html>, <body> 태그를 정의해야 한다.

global-error.js 는 가장 세부적인 에러 UI 이며 전체 애플리케이션에 대한 모든 것을 잡는 에러 처리로 간주될 수 있다. 루트 컴포넌트는 일반적으로 동적이지 않으므로 발생할 가능성이 적으며 다른 `<ErrorBoundary>` 가 잡을 가능성이 크다.

global-error.js 가 정의돼 있더라도 전체 레이아웃 내에서 루트 error.js 를 정의하는 것이 좋다. 이는 전역적으로 공유되는 UI 와 브랜딩이 포함된 루트 레이아웃 내에서 fallback 컴포넌트가 렌더링 되기 때문이다.

### **5-1) 참고**

- global-error.js 는 프로덕션 환경에서만 활성화된다. 개발 중에는 오류창이 대신 표시된다.

---

## **5. 서버 에러 처리**

테서버 컴포넌트에서 에러가 발생하면 Next.js 는 가장 가까운 error.js 파일로 Error 객체를 넘긴다.

### **5-1) 민감한 에러 정보 보호**

Production 환경에서 클라이언트로 전달되는 Error 객체는 일반적인 메시지와 digest 속성을 지닌다. 이것은 클라이언트에 포함된 에러 안에 잠재적으로 민감한 세부 정보가 누출되는 것을 막기 위한 조치다.

메시지 속성에는 에러에 대한 일반적인 메시지가 포함된다. digest 속성에는 에러의 자동으로 생성된 해시가 포함되어 있어 해당 에러를 서버 측 로그에서 일치시킬 수 있다.

개발 중에 클라이언트로 전달되는 Error 객체는 직렬화돼 있어 원래 에러의 메시지가 포함돼 있으므로 디버깅에 용이하다.

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/error-handling#nested-routes**