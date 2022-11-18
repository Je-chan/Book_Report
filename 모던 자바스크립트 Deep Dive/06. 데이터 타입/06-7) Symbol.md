# Symbol
- 심벌 타입은 ES6 에 추가된 7번 째 타입
- 변경 불가능한 원시 타입이다
  - 동시에 다른 값과 중복되지 않는다
- 이름이 충돌하지 않을 객체의 유일한 key 를 만들기 위해 사용한다
- 심벌은 함수를 호출해서 생성해야 하며 심벌 값은 외부에 노출되지 않고 절대 중복되지 않는다
```typescript
var key = Symbol('liebe');
console.log(typeof key) // symbol

var object = {}
object[key] = 'value'

console.log(object[key]) // 'value'
```

