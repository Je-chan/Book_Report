# Object 생성자 함수
- new 연산자와 함께 Object 생성자 함수를 호출하면 빈 객체를 생성해서 반환한다
- 빈 객체 생성한 이후 프로퍼티, 메서드를 추가할 수 있다

```typescript jsx
const person = new Object()

person.name = "liebe"

console.log(person) // {name: liebe}
```
- 생성자 함수란 new 연산자를 호출해 객체(인스턴스)를 생성하는 함수를 의미한다
- String, Number, Boolean, Function, Array, Date, RegExp, Promise 등도 인스턴스 생성할 수 있다