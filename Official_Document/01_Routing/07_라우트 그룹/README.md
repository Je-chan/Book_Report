
app 디렉토리에서, 중첩 폴더는 기본적으로 URL 경로와 매핑된다. 하지만, 경로 그룹(Route Group) 을 활용하면, URL 경로와 매핑하는 것을 피할 수 있다.

이런 방법은 경로 Segement 를 구성하고 프로젝트 파일을 URL 경로 구조에는 영향을 주지 않은 채 로직들을 묶을 수 있도록 도와준다.

---

## **1. 규칙**

폴더의 이름을 괄호로 묶으면 경로 그룹을 만들 수 있다.

(폴더이름)

---

## **2. 예시**

### **2-1) URL 에 영향을 주지 않고 경로(Route)구성하기**

URL 에 영향을 주지 않고 경로를 구성하려면, 그룹을 만들어야 한다. 괄호 안의 폴더는 URL 에서 생략된다.

!https://blog.kakaocdn.net/dn/bZVZ21/btsGvXmZ9j7/hGFA5mjA1WcKKndktKMrrk/img.png

(marketing), (shop) 폴더가 URL 계층 구조에서 공유되고 있지만, 각 그룹에 대해 다른 레이아웃을 구성할 수도 있다.

!https://blog.kakaocdn.net/dn/bgWzBd/btsGvqCYnfh/x1xl9mIMYKVNbxpIr5Rgv0/img.png

### **2-2) 특정 Segement 들에만 layout 을 설정하기**

특정 경로들에만 레이아웃을 넣고 싶을 때는(* 예시를 보니, "/teat/a", "test/b", "test/c" 가 있을 때, "test/b", "test/c" 에만 레이아웃을 설정하고 싶을 때를 의미하는 것 같다. test 라는 폴더에서 layout 을 설정하면, "/a, /b, /c" 모두 레이아웃이 적용되기 때문) 새로운 경로 그룹을 생성하고 레이아웃을 적용하고 싶은 Segement 들을 경로 그룹 안으로 이동시킨다. 그룹 외부의 경로에는 경로 그룹에서 설정한 layout 이 적용되지 않는다.

!https://blog.kakaocdn.net/dn/bPuBWc/btsGtnURg9q/7Hz4hvgkKYKZKWzCAecHPK/img.png

(*이 예시에서는 (shop) 하위의 /account, /cart 에는 (shop) 의 layout.js 파일이 적용되지만 바깥의 /checkout 은 적용되지 않는다)

### **2-3) 여러 개의 루트 레이아웃 만들기**

여러 개의 루트 레이앗울 만들기 위해서는, 최상단의 layout.js 파일을 없애고 각각의 경로 그룹 안에 layout.js 파일을 넣으면 된다. 이 방식은 완전히 다른 UI 와 경험을 주고 싶을 때 유용하다. 각 루트 레이아웃은 <html> 과 <body> 태그를 필요로 한다.

!https://blog.kakaocdn.net/dn/buZjm7/btsGvq33dmA/lxLrmagpXOXpCDXRjAtgLK/img.png

---

## **3. 참고 사항**

- 경로 그룹의 이름은 구조를 짤 때 빼고는 특별한 의미가 없다. URL 에 아무런 영향을 주지 않는다.
- 경로 그룹을 갖는 경로 Segement 는 다른 경로와 동일한 URL 경로로 읽혀선 안 된다. 예를 들어서 "(marketing)/about/page.js" 와 "(shop)/about/page.js" 파일은 똑같이 "'/about" 경로를 갖기 때문.
- 만약 최상단의 layout.js 파일 없이 여러 개의 루트 레이아웃을 사용한다면, 클라이언트 사이드에서 탐색하는 방식이 아닌 모든 페이지를 한번에 로드하는 일이 발생한다. 예를 들어 "app/(shop)/layout.js" 루트 레이아웃을 사용하는 "/cart" 경로가 "app/(marketing)/layout.js" 를 사용하는 "/blog" 페이지로 이동하는 경우 전체 페이지를 새롭게 로드하는 현상이 발생한다. 물론, 이것은 여러 루트 레이아웃을 사용하는 경우에만 해당한다.

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/route-groups**

