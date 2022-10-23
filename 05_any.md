# 8. any 다루기

Date: October 5, 2022
Tags: Done🫥

---

## item 38. any 타입은 가능한 한 좁은 범위에서만 사용하기

문제: 함수의 리턴값이 any가 되는 상황

```tsx
function getAge() {
    return 30;
}

function convertInternationalAge(age) {
    return age + 1;
}

// function getPerson(): any
function getPerson() {
    **const age: any = getAge();**
    convertInternationalAge(age);
    return age;
}

function getAll() {
    const person = getPerson();
    **person.setAge(); // any <- error가 생기지 않는다, 런타임에러로 이어짐**
}
```

해결방법 

```tsx
function getAge() {
    return 30;
}

function convertInternationalAge(age) {
    return age + 1;
}

// function getPerson(): number
function getPerson() {
    ****const age = getAge();
    **convertInternationalAge(age as any);**
    return age;
}

function getAll() {
    const person = getPerson();
    **person.setAge(); // error Property 'setAge' does not exist on type 'number'**
}
```

문제: 객체가 any가 되는 상황

```tsx
type Person = {[key: string]: {name: string, age: number}}

const persons: Person = {
    soo: {
        name: 'soo',
        age: 30,
    },
    jin: 'jin_30'
} as any
```

해결방안

```tsx
const persons: Person = {
    soo: {
        name: 'soo',
        age: 30,
    },
    jin: 'jin_30' as any
}
```

## item 39. any를 구체적으로 변형해서 사용하기

```tsx
const personArray: **any** = [
    {name: 'soo', age: 30}, 
    'jin_30', 
    {name: 'solji', age: 30}, 
    'sunny_30'
    ];
const num = personArray.length; // any
```

```tsx
const personArray: **any[]** = [
    {name: 'soo', age: 30}, 
    'jin_30', 
    {name: 'solji', age: 30}, 
    'sunny_30'
    ];
const num = personArray.length; // number
```

- 배열: any → `any[]`
- 객체: any → `{[key: string]: any}`
- 함수: any
    - `type Fn0 = () ⇒ any;`
    - `type Fn1 = (arg: any) ⇒ any;`
    - `type Fn2 = (…args: any[]) ⇒ any;`
    

## item40. 함수 안으로 타입 단언문 감추기

```tsx
declare function cacheLast<T extends Function>(fn: T): T;
declare function shallowEqual(a: any, b: any): boolean;

// good
function cacheLast<T extends Function>(fn: T): T {
    let lastArgs: any[] | null = null;
    let lastResult: any;
    return function(...args: any[]) {
        if (!lastArgs || !shallowEqual(lastArgs, args)) {
            lastResult = fn(...args);
            lastArgs = args;
        }
        return lastResult;
    } as unknown as T;
}

declare function shallowObjectEqual<T extends object>(a: T, b: T): boolean;
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
    for (const [k, aVal] of Object.entries(a)) {
        if (!(k in b) || aVal !== (b as any)[k]) { // Element implicitly has an 'any' type because expression of type 'any' can't be used to index type 
            return false;
        }
    }
    return Object.keys(a).length === Object.keys(b).length;
};
```

## item41. any의 진화를 이해하기

변수의 타입은 변수를 선언할 때 결정 / 배열은 다양한 타입의 요소를 넣으면 배열의 타입이 확장되며 진화 

```tsx
// any의 진화 
function range(start: number, limit: number) {
    const out = []; // any[]
    for (let i=start; i<limit; i++) {
        out.push(i);
    }
    return out; // number[] 타입의 진화 (noImplicityAny 설정, 변수의 타입이 암시적 any)
}
// 에러 
function range(start: number, limit: number) {
    const out = []; // any[]
    if (start === limit) {
        return out; // <- 암시적 any상태인 변수에 어떠한 할당도 하지 않고 사용하려고 해서 에러남 
    }
    for (let i=start; i<limit; i++) {
        out.push(i);
    }
    return out; // number[]
}
```

```tsx
// any의 진화
let val; 
if (Math.random() < 0.5) {
    val = /hello/;
    val; // RegExp
} else {
    val = 12;
    val; // number
}
val; // number | RegExp

// any타입 명시
let val: any; // 변수 타입 명시
if (Math.random() < 0.5) {
    val = /hello/;
    val; // any
} else {
    val = 12;
    val; // any
}
val; // any
```

```tsx
function makeSquares(start: number, limit: number) {
    const out = [];
    // 암시적 any타입은 함수 호출을 거쳐도 진화하지 않는다
    range(start, limit).forEach((i) => {
        out.push(i * i)
    });
    // range(start, limit).forEach(function(i){
    //     out.push(i * i);
    // });
    return out;
}

function makeSquares(start: number, limit: number) {
    const out = [];
    **out.push(...range(start, limit).map((i) => i * i));**
    return out;
}
```

타입을 안전하게 지키기 위해서는 암시적 any를 진화시키는 방식보다 명시적 타입 구문 사용

## item42. 모르는 타입의 값에는 any 대신 unknown을 사용하기

함수의 반환값과 관련된 형태 

```tsx
function parseYAML(yaml: string): any {

}

interface Book {
    name: string;
    author: string;
}

const book = parseYAML(`
    name: Wuthering heights
    author: Emily Bronte
`);

alert(book.title); // any <- 런타임에러
book('read'); // <- 런타임에러
// 어떤 타입이든 any타입에 할당 가능, any 타입은 어떠한 타입으로도 할당 가능
```

```tsx
function safeParseYAML(yaml: string): unknown {
    return parseYAML(yaml);
}

const book = safeParseYAML(`
    name: Wuthering heights
    author: Emily Bronte
`);

alert(book.name); // Object is of type 'unknown'.
book('read'); // Object is of type 'unknown'.

const book = safeParseYAML(`
    name: Wuthering heights
    author: Emily Bronte
`) as Book // 타입단언문 사용 
```

변수 선언과 관련된 형태

```tsx
interface Features {
    id?: string | number;
    geometry: Geometry;
    properties: unknown;
}

function processValue(val: unknown) {
    if (val instanceof Date) {
        val // (parameter) val: Date
    }
}
```

```tsx
interface Book {
    name: string;
    author: string;
}

function isBook(val: unknown): val is Book {
    return (
        typeof(val) === 'object' && val !== null && 
        'name' in val && 'author' in val // unknown 타입의 범위 줄이기
    );
}

function processValue(val: unknown) {
    if(isBook(val)) {
        val; // Book
    }
}
```

단언문과 관련된 형태

```tsx
 declare const foo:Foo;
let barAny = foo as any as Bar; //분리되는 순간 전염병
let barUnk = foo as unknown as Bar;
```

## item43. 몽키 패치보다 안전한 타입을 사용하기

```tsx
// Property 'monkey' does not exist on type 'RegExp'.
RegExp.prototype.**monkey** = 'Capuchin'; 
console.log(/123/.**monkey**); // Capuchin

// Property 'monkey' does not exist on type 'Document'
document.monkey ='Tamarin'; 
(document as any).monkey = 'Tamarin'; // <- 타입 안전성 상실, 언어 서비스 사용 불가
```

사이드이펙트 + 타입 체커는 추가한 속성에 대해서는 알지 못함

해결 방법 

1. DOM과 데이터 분리
2. 보강 사용

```tsx
// interface 보강 (타입 안전, 자동완성 사용, 몽키 패치가 어떤 부분에 적용되었는지 정확한 기록)
interface Document {
    monkey: string;
}
```

보강: 전역적으로 적용되므로 코드의 다른 부분이나 라이브러리로부터 분리 불가 / 애플리케이션이 실행되는 동안 속성을 할당하면 실행 시점에서 보강을 적용할 방법 없음(동적 할당 불가)

1. 타입 단언문 사용

```tsx
interface MonkeyDocument extends Document {
    monkey: string;
}
(document as MonkeyDocument).monkey = 'Macaque';
```

구체적인 타입 단언문 사용 

- Document 타입을 건드리지 않고 별도로 확장하는 새로운 타입 → 모듈 영역 문제 해결(import 하는 곳만 영역)

## item44. 타입 커버리지를 추적하여 타입 안전성 유지하기

- 명시적 any 타입
- 서드파티 타입 선언
- any의 갯수 추적: `type-coverage`

any가 등장하는 문제

```tsx
function getColumnInfo(name: string): any {
    return utils.buildColumnInfo(appState.dataSchema, name) // any 반환 
}

declare moudle 'my-module'; // 전체 모듈에 any타입 부여
import {someMethod, someSymbol} from 'my-module';
const pt1 = {
    x: 1,
    y: 2
};
const pt2 = someMethod(pt1, someSymbol);
```
