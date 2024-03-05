# instance of 연산자
- 우변의 생성자 함수의 prototype 에 바인딩 된 객체가
  - 좌변의 객체의 프로토타입 체인 상에 존재하면 true
  - 그렇지 않으면 false 로 평가한다
```typescript jsx
function Person(name) {
	this.name = name
}

const me  = new Person('liebe')

console.log(me instanceof Person) // true
console.log(me instanceof Object) // true

//////////////////////////////////////////

const parent = {}

Object.setPrototypeOf(me, parent)

console.log(Person.prototype === parent) // false
console.log(parent.constructor === Person) // false
console.log(me instanceof Person) // false
console.log(me instanceof Object) // true

//////////////////////////////////////////

Person.prototype = parent

console.log(me instanceof Person) // true
console.log(me instanceof Object) // true

```
- me 객체는 프로토타입이 교체되어 프로토타입과 생성자 함수간의 연결이 파괴되었다
  - 이는 곧 me instanceof Person 값이 false 라는 것
- 하지만, Person 생성자 함수에 의해 생성된 인스턴스는 확실하다
- 여기에서 만약, parent 를 Person 생성자 함수의 prototype 프로퍼티에 바인딩하면 me instanceof Person 은 true 가 될 것이다
  - me 객체의 프로토타입 체인 상에 Person.prototype 이 존재하기 때문
- 생성자 함수의 prototype 에 바인딩된 객체가 프로토타입 체인 상에 존재하는지 확인하는 것이 instanceof 연산자다
```typescript jsx
function isInstanceOf(instance, constructor) {
	const prototype = Object.getPrototypeOf(instance)
  
  if(prototype === null) return false
  
  return prototype === constructor.prototype || isInstanceOf(prototype, constructor)
}
```
