# 4장 타입 설계

### item28: 유효한 상태만 표현하는 타입을 지향하기

```tsx
interface State {
	loading: boolean
	error?: Error
}
```

위 interface의 아쉬운점

- 로딩과 에러상태 분리가 안되어있음 (타입상 로딩이면서 에러가 있을수도있음)
- 실제로 발생하지 않을 케이스임에 불구하고, 타입상으로 가능함

loading: true

error: Error

개선된 코드

```tsx
type State = RequestSuccess | RequestPending | RequestFailed

interface RequestSuccess {
	loading: false
}

interface RequestPending {
	loading: true
}

interface RequestFailed {
	loading: false
	error: Error
}
```

### item29: 사용할 때는 너그럽게, 생성할 때는 엄격하게

```tsx
interface NumberConstructor {
    (value?: any): number;
}

/** An object that represents a number of any kind. All JavaScript numbers are 64-bit floating-point numbers. */
declare var Number: NumberConstructor;

Number(1) // number 

Number({}) // number | null 

const valueAsNumber: number | null = Number({})
```

### item30: 문서에 타입 정보를 쓰지 않기

```tsx
/*
 * 문자열을 반환합니다.
 */
function getFgColor() {
	return { r: 0; g: 0; b: 0}
}

```

- 주석은 타입정보와 동기화 되지않는다.  (잘못 작성된 주석은 없는것만 못하다)
- 가급적 변수명과 타입을 활용하자

```tsx
declare function setTimeout(callback: (...args: any[]) => void, time?: number, ...args: any[]): NodeJS.Timeout;
```

- 개선
    
    ```tsx
    declare function setTimeout(callback: (...args: any[]) => void, ms?: number, ...args: any[]): NodeJS.Timeout;
    ```
    
    - time 이라는 변수명보단, ms라는 변수명을 통해 단위까지 표현할수있음 (주석이 없어도 표현가능하다.)

### item31: 타입 주변에 null 값 배치하기

### [#](https://www.typescriptlang.org/tsconfig#strictNullChecks)Strict Null Checks -`strictNullChecks`

When `strictNullChecks` is `false`, `null` and `undefined` are effectively ignored by the language. This can lead to unexpected errors at runtime.

When `strictNullChecks` is `true`, `null` and `undefined` have their own distinct types and you’ll get a type error if you try to use them where a concrete value is expected.

```tsx
declare const loggedInUsername: string;
 
const users = [
  { name: "Oby", age: 12 },
  { name: "Heera", age: 32 },
];
 
const loggedInUser = users.find((u) => u.name === loggedInUsername);
console.log(loggedInUser.age); // Object is possibly 'undefined'

User | undefined
```

- 런타임에 발생할 에러를 방지할 수 있기 때문에 기본적으로 사용하자

### item32: 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

```tsx
interface Animal {
	type: 'dog' | 'cat' | 'rabbit'
	name: 'dog' | 'cat' | 'rabbit'
}

function logName(animal: Animal){
	if(animal.type === 'dog'){
		animal.name // 'dog' | 'cat' | 'rabbit'
	}
} 
```

type과 name이 독립적이기 때문에 추론이안된다. 

```tsx
type Animal = Dog | Cat | Rabbit

interface Dog {
	type: 'dog'
	name: 'dog'
}

interface Cat {
	type: 'cat'
	name: 'cat'
}

interface Rabbit {
	type: 'rabbit'
	name: 'rabbit'
}

function logName(animal: Animal){
	if(animal.type === 'dog'){
		animal.name // 'dog'
	}
} 
```

두가지 프로퍼티가 같이 존재하거나 같이 존재하지 않는경우

```tsx
interface Person {
	name: string;
	placeOfBirth?: string;
	dateOfBirth?: Date;
}

placeOfBirth
dateOfBirth
```

타입으로 정보를 알려주자

```tsx
interface Name {
	name: string
}

interface PersonWithBirth extends Name {
	placeOfBirth: string;
	dateOfBirth: Date;
}

type Person = Name | PersonWithBirth
```

### item33: string 타입보다 더 구체적인 타입 사용하기

```tsx
interface Album {
	type: string
}

function getAlbumType(album: Album) {
	return album.type // string
}
```

- 어떤 타입값들이 사용될지 코드만 봐서 알수없다.

- 개선
    
    ```tsx
    interface Album {
    	type: 'live' | 'studio'
    }
    ```
    

```tsx
const color = {
	grey100: '#efefef',
	blue100: '#341234'
} as const

function getColor(key: string){
	return color[key] // Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{ readonly grey100: "#efefef"; readonly blue100: "#341234"; }'.
  // No index signature with a parameter of type 'string' was found on type '{ readonly grey100: "#efefef"; readonly blue100: "#341234"; }'.
}
```

```tsx
const color = {
	grey100: '#efefef',
	blue100: '#341234'
} as const

function getColor(key: keyof typeof color): typeof color[keyof typeof color]{
	return color[key] 
}
```

교재 예시

```tsx
interface Album {
	artist: string;
	title: string;
	releaseDate: Date;
	recordingType: 'studio' | 'live'
}

function pluck<T>(records: T[], key: keyof T): T[keyof T][] {
	return records.map(r => r[key]);
}

pluck(albums, 'releaseDate') // (string | Date)[]
albums // Album[] 
Date[]
key: artist | title | ..
```

- 개선
    
    ```tsx
    
    function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
    	return records.map(r => r[key]);
    }
    
    pluck(albums, 'releaseDate') // Date[]
    ```
    

### item34: 부정확한 타입보다는 미완성 타입을 사용하기

```tsx
type Expression4 = number | string | CallExpression

type CallExpression = MathCall | RGBCall | CaseCall;

interface MathCall {
  0: '+' | '-'
  1: Expression4
  2: Expression4
  length: 3
}

interface RGBCall {
  0: 'rgb'
  1: Expression4
  2: Expression4
  3: Expression4
  length: 4
}

interface CaseCall {
  0: 'case'
  1: Expression4
  2: Expression4
  3: Expression4
  length: 6
}

const ex: Expression4[] = [
  1,
  ['+',121,2],
  ['-'], // Type '"-"' is not assignable to type '"case"'.(2322)
  ['+', 12, 2, 3], // Type '"-"' is not assignable to type '"case"'.(2322)
]
```

- 타입이 복잡할수록 어떤 에러상황인지 나타내기가 어려움 (여러가지 상황이존재)
- v4.8.4 에서는 유니온 타입중 마지막 타입까지 대응되지 않으면, 마지막 타입에 대한 에러메시지가 떨어짐
[playground](https://www.typescriptlang.org/play?ts=4.8.4#code/C4TwDgpgBAogHmAThAziglgewHYBYoC8U2ArgLYBGEiUAPlCsIutgOZ1QDCAhgDa-wkqDDgCwAKAmhIXPgITI0WbISgBZbsAAWPfhygAlAOIAhXbw48UEcwG4JElsGoAzbgGNoG7eagBvCSgoAAYALigAcgBqCI4IgFoIwKgARnDBRRE8ZIAmdIVhZVxk3gg2bXCAZgkAXwdxJ1cPaGMzOX9ksMjEVgok8SC02AKlHGKBqDzhoVHsicr8mazxoNLyrXDxuskG7GdEN09Za18Aia6I925rfsHFzKLc+8Kx5IXph9eJtdYKqAA2Wr1dw4RhQCBwZ6zXAAbQAuqoYckUgAaZIw6IRFEpHKonJwtETDGJAnozHYnIoymVUniOFAA)
- 너무 구체적인 타입보다는, 모호한 타입이 좋은 경우도 있음

### item35: 데이터가 아닌, API와 명세를 보고 타입 만들기

데이터

```tsx

const user = {
	name: '서지환',
	nickname: 'hwan'
}

```

실제 서버 데이터 인터페이스

```tsx

interface User {
	name: string;
	nickname?: string;
}
```

- API 명세를 확인해서 타입을 만들자
- 자동화하자 (extra mile 👍 백로그)

### item36: 해당 분야의 용어로 타입 이름 짓기

```tsx
interface Animal {
	name: string;
}
```

- 실제 이름인지, 동물의 학명인지 알기 어려움

```tsx
interface Animal {
	commonName: string;
}
```

### item37: 공식 명칭에는 상표를 붙이기

```tsx
interface Vector2d {
  x: number
  y: number
}

function calculateNorm(vector: Vector2d){
  return {x: vector.x, y: vector.y}
}

const position = {x:1, y:2, z:3}
calculateNorm(position) 
```

- 잉여 속성 체크를 하지 않는 경우,  의도하지 않은 값이 반환될 수 있다.

```tsx
interface Vector2d {
  x: number
  y: number
  _brand: '2d'
}

function generateVector2d(x: number, y: number): Vector2d{
  return { x, y, _brand: '2d'}
}

function calculate2d(vector: Vector2d){
  return {x: vector.x, y: vector.y}
}

const position = {x:1, y:2, z:3} 

calculate2d(generateVector2d(3,5))
calculate2d(position) // error 
```
