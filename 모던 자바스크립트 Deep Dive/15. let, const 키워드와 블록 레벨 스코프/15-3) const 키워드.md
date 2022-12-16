# const 키워드
- const 는 상수를 선언하기 위해 사용한다
  - 하지만 그렇다고 해서 무조건 상수만을 위해 사용하지는 않는다
## 1) 선언과 초기화
- const 키워드로 선언한 변수는 반드시 선언과 동시에 초기화해야 한다
- let 키워드와 같이 블록 레벨 스코프를 가지며 호이스팅이 일어나지 않는 것처럼 동작한다

## 2) 재할당 금지
- var, let 키워드는 변수 재할당이 자유롭지만 
- const 키워드로 선언한 변수는 재할당이 금지된다

## 3) 상수
- const 키워드로 선언한 변수에 원시 값을 할당한 경우 변수 값을 변경할 수 없다
  - 원시 값이 변경 불가능한 값이기에 재할당 없이는 변경할 방법이 없기 때문
- 일반적으로 상수의 이름은 대문자로 선언해 상수임을 명확히 드러낸다
```typescript jsx
const TAX_RATE = 0.1;
let preTaxPrice = 100;

let afterTaxPrice = preTaxPrice + (preTaxPrice * TAX_RATE)
```

## 4) const 키워드와 객체
- const 키워드로 선어된 변수에 객체를 할당한 경우 값을 변경할 수 있다
- 원시 값은 재할당 없이 교체할 방법이 없지만 객체는 재할당 없이도 직접 변경이 가능하기 때문이다
```typescript jsx
const person = {
	name: "Liebe"
}

person.name = 'Bono'

console.log(person) // {name: 'Bono'}
```
- 즉, const 키워드는 재할당을 금지할 뿐 불변을 의미하지는 앟는다
  - 프로퍼티의 동적 생성, 삭제, 프로퍼티 값의 변경을 통해 객체 변경은 가능하다
