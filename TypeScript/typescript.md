# TypeScript

타입을 지정해 자바스크립트를 정적 언어로 사용하게 만드는 것. </br>
에러 발생시 에디터에서 에러 발생 ➡ 사용자가 아닌 개발자가 에러를 먼저 파악하게 됨.</br>
(JS의 경우 대부분의 에러를 무시하고 진행하여 사용자가 알게 되었음)

## 타입 시스템 - 기본

변수 생성시 타입을 지정해야 한다.

```ts
let a = 'hello'; // 타입 추론. 이 방법이 짧고 직관적
let b: boolean = true; // 타입 명시
let c = [1, 2, 3];
c.push('1'); // 타입이 달라서 불가능

let c: number[] = [];
c.push('1'); // 빈 문자열이지만 타입을 명시하여 에러 처리
```

## 타입 시스템 - optional

```ts
const obj: {
  name: string; // 기본 값
  age?: number;
} = {
  name: 'str',
};
```

age는 선택 값으로 값이 들어오지 않을 수 있음 </br>

```ts
if (obj.age < 10) {
}
// Object is possibly 'undefined'. 값이 없을 수 있음을 알려줌

if (obj.age && obj.age < 10) {
}
```

옵션 값을 사용하고 싶을 경우 있는지 먼저 확인해야 한다.

## 타입 시스템 - Alias

```ts
type Age = number; // * 일반 변수 선언처럼 사용할 수도 있음
type MainObj = {
  name: string;
  age?: Age; // * 적용 값
};

const objA: MainObj = {
  name: 'strA',
};
const objB: MainObj = {
  name: 'strB',
  age: 20,
};
```

Alias 타입으로 코드를 재사용할 수 있음.

```ts
type Name = string;
type Obj = {
  name: string,
  age?: number
}

funtion newPeople(name: string){ // 인수에 타입 지정
  return {
    name
  }
}

const peopleA = newPeople("manA") // success.
peopleA.age = 20 //error. return 값에 name만 들어옴

// function newPeople(name: string) : Obj {
  // 리턴값에 타입 지정 (Obj)

// 화살표 함수 버전
const newPeople = (name: string) : Obj => ({name})
```

### 정리

`이름 : 타입` 형식 </br>
변수, 인수, 함수 모두 적용.
</br></br>

타입을 여러개 지정할 경우 `|` 사용 </br>
`name: string | number`

## 타입 더보기

- readonly

```ts
type People = {
  readonly name: Name; // 변경시 에러 발생
  age?: number; // type: number | undefined
};
const addPeople = (name: string): People => ({ name });
const ManA = addPeople('A');
A.age = 20;
A.name('ManA');
// error. Cannot assign to 'name' because it is a read-only property
```

```ts
const numbers: readonly number[] = [1, 2, 3, 4];
numbers.push(1);
// error. Property 'push' does not exist on type 'readonly numbers[]'
```

배열을 직접적으로 변경할 수는 없지만 `filter`, `map` 처럼 원본 값을 바꾸지 않는다면 변경 가능 (불변성 유지)

### 튜플 (Tuple)

항상 정해진 개수의 인자가 들어오며, 타입 순서와 일치하게 작성

```ts
const tupleEx: [string, number, boolean] = ['A', 10, true]; // 순서가 달라도 에러 처리 (형식에 맞지 않음)
tupleEx[0] = 1; // error. Type 'number' is not assignable to type 'string'

const tupleRO: readonly [string, number, boolean] = ['A', 1, false];
tupleRO[0] = 'Hi'; // readonly 적용시 형식에 맞아도 변경할 수 없음.
// error. Cannot assign to '0' because it is a read-only property.
```

### others

```ts
let a: undefined = undefined;
let b: null = null;
let c: any; // 최대한 지양할 것.
```

- any 예시

```ts
const a = [1, 2, 3, 4];
const b = true;

a + b; // Operator '+' cannot be applied to types 'number[]' and 'boolean'

// --------- //

const a: any[] = [1, 2, 3, 4];
const b: any = true;
a + b; // ok. "1,2,3,4true"
```

- unknown (타입을 모를 때 사용. 변수의 타입을 먼저 확인해야 함)

```ts
let a: unknow;
let b = a + 1; // error. Object is of type 'unknown'

if (typeof a === 'number'){
  let b = a + 1;
}

if (typeof a === 'string'){
  let b = a.toUpperCase();
}
```
타입 확인 이후 형식에 맞는 결과를 진행

- void (return 하지 않는 함수를 대상으로 사용)
```ts
function hello() {
  console.log('x')
}
```
return할 것이 없다면 자동으로 void 타입임을 인지한다.

- never (절대 return하지 않을 때 발생) </br>
case1: 예외 실행
```ts
function hello():never{
  //return "X" 
  // error. Type 'string' is not assignable to type 'never'.
  
  throw new Error("xxx") // ok. return 없이 오류 발생
}
```
 case2: 인자에 대한 타입 변경
```ts
function hello(name: string|number){
  if(typeof name === 'string'){
    name // name: string
  } else if (typeof name === 'number'){
    name // name: number
  } else {
    name // name: never. 이 영역은 절대 실행되서는 안됨
  }
}
````