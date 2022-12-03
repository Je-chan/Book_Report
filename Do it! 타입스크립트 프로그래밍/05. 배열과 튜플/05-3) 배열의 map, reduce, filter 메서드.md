# 1. filter
- filter 는 다음과 같은 타입의 형태로 설계돼 있다
```typescript
filter(callback: (value: T, index?: numbwer): boolean): T[]
```
- 이 filter 기능을 이용해서 홀수와 짝수를 구분하는 로직을 작성하면 다음과 같다
```typescript
const array: number[] = range(1, 10 + 1)
const half = array.length / 2

const odds: number[] = array.filter((value) => value % 2 !== 0)
const evens: number[] = array.filter((value) => value % 2 === 0)
```
