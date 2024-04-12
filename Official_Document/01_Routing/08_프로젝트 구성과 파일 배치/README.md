
경로 폴더와 파일 규칙 을 제외하고는 Next.js 는 개발자가 프로젝트 파일을 어덯게 구성하고, 파일을 동일한 위치에 둘 것인지 간섭하지 않는다.

---

## **1. 안전한 동일 위치 파일 구성(기본)**

app 디렉토리에서는 중첩된 폴더 계층 구조가 실제 경로를 정의한다.

각각의 폴더는 경로의 Segement 를 표기하고 이는 URL path 의 Segement 를 표시한다.

하지만 이 경로 구조가 폴더로 정의된다 하더라도, 실제 해당 경로는 page.js 파일이나 route.js 파일이 없으면 사용자에게 공개되지 않는다.

!https://blog.kakaocdn.net/dn/d3pWTT/btsGsWb36jn/zPVEFGeQpUKp7B1dongJ20/img.png

그리고 해당 경로가 공개적으로 접근 가능하다 하더라도 page.js 혹은 route.js 에서 반환된 내용만을 클라이언트로 보낸다.

!https://blog.kakaocdn.net/dn/RmSB5/btsGtUdRWpr/G6DtnMoKi51CvaYnOyfPR1/img.png

이것이 의미하는 바는, app 디렉토리 내부에서 파일들을 개발자 임의로 생성더라도 실수로 그것이 화면에 보여질리가 없다는 것이다.

!https://blog.kakaocdn.net/dn/tCdzs/btsGtNTuJav/lvOVkyB16AaKzbxBeED4n0/img.png

단, 이런 케이스는 page 디렉토리에서 적용할 수 없다. page 디렉토리에서는 모든 파일을 경로로 간주하기 때문이다.

---

## **2. 프로젝트 구성 기능**

### **2-1) 개인 폴더(Private Folders)**

개인 폴더는 밑줄(_)을 접두사로 활용해서 만들어낼 수 있다 `_folderName`

이것이 의미하는 바는 경로 시스템에서 고려되지 않을 구현 세부정보임을 표시하는 것이다. 따라서, 해당 폴더 및 하위 폴더는 경로에서 제외된다.

!https://blog.kakaocdn.net/dn/pcovm/btsGvwDcrCn/nKbEkyS5FlaJqi5NtFTu51/img.png

'

app 디렉토리의 파일은 기본적으로 안전하게 특수 파일이 아닌 파일을 특수 파일과 함께 위치시킬 수 있으므로 개인 폴더가 필요하진 않다. 하지만 다음의 경우에는 유용할 수 있다.

1. UI 로직과 Routing 로직을 분리할 때
2. 프로젝트 나 Next.js 전반에 걸쳐서 내부 파일을 일관되게 관리하고 싶을 때
3. 코드 편집기에서 파일을 정렬 및 그룹화하는데 용이함
4. 미래의 Next.js 파일 규치과 잠재적인 이름 충돌 방지

### **2-2) src 디렉토리**

Next.js 는 src 디렉토리를 추가할 것인지 선택지로 제공한다. 이 방식을 사용하면, 구성 파일들(configuration)과 어플리케이션 관련 코드들을 분리할 수 있다.

!https://blog.kakaocdn.net/dn/uDK1l/btsGu2vBSr3/N3BQZ7MTLQHRGju0VT6Xuk/img.png

---

## **3. 프로젝트 구성 전략**

Next.js 프로젝트에서 파일과 폴더를 구성하는 방법에 옳고 그름은 없다. 다음에 따라오는 섹션은 일반적인 전략에 대한 높은 수준의 내용을 나열한다. 간단한 전략은 팀에서 활용하고 있는 전략을 선택하여 프로젝트 전반에 걸쳐 일관성을 유지하는 것이다.

아래 예제에서는 `components`, `lib` 은 placeholder 같은 것으로, 이름은 `ui`, `utils`, `hooks`, `styles` 와 같은 것으로 사용해도 된다.

### **3-1) 앱 외부에 프로젝트 파일을 저장**

이 전략은 어플리케이션 코드를 프로젝트 루트에 있는 공유 폴더에 저장하여 app 디렉토리를 순수하게 라우팅 용도로 유지하는 것이다.

!https://blog.kakaocdn.net/dn/b85Fd1/btsGvUYnlZO/0MVspq3rLMUnXpCyih4LuK/img.png

### **3-2) app 내부 최상위 폴더에 프로젝트 파일을 저장**

이 전략은 모든 애플리케이션 코드를 app 디렉토리의 루트에 있는 공유 폴더에 저장하는 방식이다.

!https://blog.kakaocdn.net/dn/cbomdo/btsGu05Hc1B/7c3Syv02jyRhWSMYCKktCK/img.png

### **3-3) 기능 또는 경로 별로 프로젝트 파일을 분리**

이 전략은 전역적으로 공유된 애플리케이션 코드를 app 디렉토리 루트에 저장하고, 구체적인 에플리케이션 코드는 사용하는 경로 Segement 에 분할하는 방식이다.

!https://blog.kakaocdn.net/dn/dqTc2m/btsGu14CuxQ/aCRjmQsUnQcGkYFdkVlQs0/img.png

---

## **Reference**

**https://nextjs.org/docs/app/building-your-application/routing/colocation#module-path-aliases**