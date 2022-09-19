# 2장 타입스크립트의 타입 시스템

# 아이템 6 - 편집기를 사용하여 타입시스템 탐색하기

### 타임스크립트를 설치하면?

타입 스크립트 설치시 다음 두 가지를 실행 할 수 있습니다.

- 타입스크립트 컴파일러(tsc)
- 단독 실행이 가능한 타입스크립트 서버 (tsserver)

> 타입스크립트는 ‘**언어 서비스**’를 제공합니다.
>
> **언어서비스**는 코드 자동완성, 명세, 검새, 리팩터링을 포합합니다.
>
> 편집기를 사용하여 언어 서비스를 사용할 수 있고, 타입 스크립트 서버에서 언어 서비스를 제공하도록 설정하는 것을 추천합니다.
>
> (편집기는 코드 빌드 후 타입 시스템을 익히는 좋은 수단입니다.)

```jsx
let num = 10; // 타입을 추론해보세요.

function add(a: number, b: number) {
  return a + b;
}
```

→ 심벌 위에 마우스 커서를 올리면 추론되는 것을 확인 할 수 있습니다.

<aside>
ℹ️ ‘**추론’**은 타입 ‘넓히기’ 와 ‘좁히기’ 를 위해 꼭 필요한 과정입니다.

</aside>

아래 예시를 통해 message의 타입이 어떻게 좁혀지는지 확인해보세요.

```jsx
function logMessage(message: string | null) {
  if (message) {
    message;
  }
}
```

### Definite types

- `lib.dom.d.ts` 처럼 타입을 선언하여 사용할 수 있습니다.
- definite types 깃허브를 확인해보세요

https://github.com/axios/axios

---

# 아이템 7 - 타입이 값들의 집합이라고 생각하기

- 모든 변수는 런타임에 고유의 값을 가집니다.

  - 코드가 실행되기 전 ‘타입’ 기준으로 오류를 확인 할 수 있습니다.
    <aside>
    ℹ️ 여기서 ‘**타입**’ 은 ‘**할당 가능한 값들의 집합**’입니다.

    </aside>

    → 타입은 ‘범위’라고 불리기도 합니다.

## 집합으로 타입 알아보기

### 공집합

- 가장 작은 집합입니다.
- 아무것도 포함하지 않습니다. (아무 값도 할당 할 수 없습니다.)
- `never` 를 의미합니다.
  ```jsx
  const x: never = 3; // Type 'number' is no assignable
  ```

### 유닛, 리터럴 (unit, literal)

- 한가지 값만 포함합니다.

```jsx
type A = "A";
type B = "B";
type Ten = 10;
```

### 유니온 (Union)

- 타입을 두개 혹은 세개로 묶을 수 있습니다.

```jsx
type AB = 'A' | 'B';
type ABTen = AB |  10;
const ab : ABTen; // 할당한 가능한 타입들은?
```

- 유니온은 값들의 합집합을 의미합니다.
  (타입 스크립트 오류를 잘 살펴보세요. ‘할당 가능한’ 이라는 문구를 집합의 관점에서 ‘~의 집합에’ 해당하는지를 의미합니다.)

<aside>
ℹ️ **타입 체커**의 역할은 하나의 집합이 다른 집합의 부분 집합인지 검사하는 것입니다.

</aside>

### & 연산자 (인터섹션, intersection, 교집합)

- 교집합을 계산합니다.

```tsx
interface Person {
  name: string;
}

interface LifeSpan extends Person {
  birth: Date;
  death?: Date;
}

type PersonSpan = Person & LifeSpan;
```

- `PersonSpan` 타입을 예상해보세요
  공통으로 가지는 속성이 없으므로 `never` 가 예상 될 수 있습니다.
  하지만, 타입 스크립트 연산자는 인터페이스 속성이 아니라 값의 집합(타입 범위)에 적용됩니다.
  추가 적인 속성을 가지는 값도 타입에 속하게 됩니다.
  → 즉, 둘 값 모두를 가지는 인터섹션 타입에 속합니다.
  ```tsx
  const ps: PersonSpan = {
    name: "dahye",
    birth: new Date(),
    death: new Date(),
  };
  ```
    <aside>
    ℹ️ **인터센션 타입**은 타입 내 속성을 모두 포함하는 것이 일반적인 규칙입니다.
    
    </aside>

### 속성에 대한 인터섹션

- 각 타입에 속성을 모두 포함한다

### 인터페이스 유니온

- 유니온 타입에 속하는 값의 키를 찾는다.
- 유니온에 대한 keyof는 확인한다.

### 41page 인터페이스 유니온 잘 모르겠음 ㅠㅠ 다시 확인하기

```tsx
type K = keyof (Person | LifeSpan); //never

let ps2: Record<K, any> = {
  name: "",
};
```

### `extends` 키워드 사용하기

- 더 일반적인 타입을 확장하는 방법은 `extends` 키워드를 사용하는 것입니다.
  (42page)
- `extends` 는 ‘~에 할당 가능한’, ‘~의 부분의 집합’ 의 의미입니다.

> **서브타입**
>
> — 어떤 집합이 다른 집합의 부분 집합이라는 의미
>
> ```tsx
> interface Vector1D {
>   x: number;
> }
>
> interface Vector2D extends Vector1D {
>   y: number;
> }
>
> interface Vector3D extends Vector2D {
>   z: number;
> }
> ```
>
> 위의 타입은 아래와 같은 관계가 성립됩니다.
>
> `Vector3D` — Vector2D의 서브타입
>
> `Vector2D` - Vector1D의 서브타입
>
> → 이는 벤다이어그램으로 봤을때 집합의 개념을 이해하기 더 용이합니다.
>
> - 위의 인터페이스 코드를 재작성해서 할당 가능성의 관계를 확인할 수 있습니다.
>
>   ```tsx
>   interface Vector1D {
>     x: number;
>   }
>
>   interface Vector2D {
>     x: number;
>     y: number;
>   }
>
>   interface Vector3D {
>     x: number;
>     y: number;
>     z: number;
>   }
>   ```

<aside>
ℹ️ **제너릭 타입에서는 한정자로 사용됩니다.**

</aside>

- 이 문맥에서는 ‘~의 부분 집합’을 의미하기도 합니다
  (14아이템 참조)

  ```tsx
  /**
   * @description
   * string을 상속한다는 관점으로 이해하면 어렵습니다.
   * string 상속 의미를 집합의 개념으로 접근해보세요. string의 부분 집합 범위를 가지는 어떠한 타입이 됩니다.
   * @param val
   * @param key
   */

  function getKey<K extends string>(val: any, key: K) {}

  getKey({}, "x"); // OK, 'x' extends string
  getKey({}, Math.random() < 0.5 ? "a" : "b"); // OK, 'a'|'b' extends string
  getKey({}, 12);
  ```

  - 에러 ‘할당 할 수 없습니다’ 의 의미를 치환해서 생각해보세요.
    → ‘상속할 수 없습니다.’ 혹은 ‘~의 부분 집합’이 아닙니다. 로 받아들이면 문제가 없습니다.

```tsx
interface Point {
  x: number;
  y: number;
}
type PointKeys = keyof Point; // Type is "x" | "y"
function sortBy<K extends keyof T, T>(vals: T[], key: K): T[] {
  return vals.sort((a: T, b: T) => Number(a[key]) - Number(b[key]));
}
const pts: Point[] = [
  { x: 1, y: 1 },
  { x: 2, y: 0 },
];
sortBy(pts, "x"); // OK, 'x' extends 'x'|'y' (aka keyof T)
sortBy(pts, "y"); // OK, 'y' extends 'x'|'y'
sortBy(pts, Math.random() < 0.5 ? "x" : "y"); // OK, 'x'|'y' extends 'x'|'y'
sortBy(pts, "z");
```

→ K 가 상속하고 있는 것은 ‘x’, ‘y’가 가능하므로 집합 관계로 보면 자연스럽습니다.

(’z’ 는 집합에 속하지 않습니다.)

---

# 아이템 8 - 타입 공간과 값 공간의 심벌 구분하기

### 타입 스크립트의 심벌이 존재하는 공간

- 타입 공간
- 값 공간
  → 둘 중 한 공간에 존재합니다.

<aside>
ℹ️ 심벌의 이름은 같더라도 속하는 공간에 따라 다른 것은 나타낼 수 있습니다.

</aside>

```tsx
// 이름은 같지만 하나는 타입, 하나는 값입니다.
interface Cylinder {
  radius: number;
  height: number;
}
const Cylinder = (radius: number, height: number) => ({ radius, height });
```

> Playground에서 살펴보세요.
>
> - 자바스크립트 결과물을 보면 타입 정보가 제거 됩니다.
> - 심벌이 사라진다면 그것은 타입입니다.

## 값과 타입으로 사용 되는 `Class`, `Enum`

`class`, `enum`은 타입과 값 두가지 모두 가능한 예약어입니다.

### 타입, 값의 관점

- 타입 관점
  - typeof는 값을 읽어서 타입스크립트 타입 반환
  - typeof는 큰 타입의 일부분으로 사용 가능하다.
  - type 구문으로 이름을 붙이 ㄹ수 있다.
- 값의 관점
  - 자바스크립트 런타임의 typeof 값이 된다.
  - 런타임에 가르키는 심벌의 문자열을 반환

### `Class`

- 타입으로 사용될 때는 형태로 사용됩니다.
  - 타입 관점에서는 typeof로 값을 읽고 반환합니다.
- 값으로 사용될 때는 생성자가 사용됩니다.
  - 값의 관점에서는 런타입의 typeof 연산자로 사용됩니다.
  - 값 공간에서 런타임을 가르키는 문자열을 반환합니다.
    - 타입스크립트 타입과 다릅니다.
    - (string, number, boolean, undefined, object, function)

```tsx
class Cylinder {
  radius = 1;
  height = 1;
}

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape; // OK, 타입 Cylinder
    shape.radius; // OK, 타입 number
  }
}

const v = typeof Cylinder; // 값 Cylinder
type T = typeof Cylinder; // 타입 Cylinder
```

→ 클래스는 자바스크립트에서 실제 함수로 구현됩니다.

# 아이템 9 - 타입 단언보다 타입 선언 사용하기

## 타입 단언

- 강제로 타입을 지정
- 타입 체커에게 오류를 무시하라는 것
- 잉여 속성 체크가 되지 않는다.
    <aside>
    ℹ️ 꼭 필요한 경우는 오직 개발자가 판단하는 타입이 더 정확한 경우이다. (55page)
    
    </aside>
    
    — 타입스크립트가 런타임 환경을 알지 못해서인 경우가 대부분이다.
    
    (DOM에 접근할 수 없는 경우, 타입스크립트가 알지 못하는 정보를 개발자가 알고 있는 경우)

## 타입 선언

- 할당되는 값이 해당 인터페이스를 만족하는지 검사
- 안정성 체크가 가능하다.
- 잉여 속성 체크가 가능하다.
- 화살표 함수를 사용하는 경우, 직관적으로 사용할 수 있다.

## 요약

- 타입 단언보다는 타입 선언 `(:Type)` 을 사용하는 것이 좋다.
- 반환 타입을 명시하도록 하자.
- 타입스크립트보다 타입 정보를 더 잘 알고 있는 상황에서 단언문을 사용하자.

# 아이템 10 - 객체 래퍼 타입 피하기

> 자바스크립트는 객체 이외에 기본형의 타입을 가집니다.
>
> (string, number, boolean, null, undefined, symbol, bigint)
>
> 기본형은 불변이며, 메서드를 가지지 않습니다.

### 기본형 객체 래퍼 타입

- number ⇒ Number
- boolean ⇒ Boolean
- symbol ⇒ Symbol
- bigint ⇒ Bigint
- null, undefined에는 존재하지 않습니다.

이 래퍼 타입을 덕분에 기본형 값에 메서드 이용이 가능합니다.

<aside>
ℹ️ 타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링합니다.

</aside>

- 런타임 값을 객체가 아니고 기본형입니다.
  - 기본형 타입은 객체 래퍼에 할당 할 수 있습니다.
  - 타입 스크립트는 기본형 타입을 객체 래퍼에 할당 할 수 있습니다
    ⇒ 오해하기 쉽고, 그럴 필요가 없으므로 사용할 필요 없습니다. (아이템 19)

```tsx
const s1: String = "원시값";
const s2: string = "타입스크립트 모델링해주므로 대문자로 안해도 됩니다";
```

# 아이템 11 - 잉여 속성 체크의 한계 인지하기

### 타입이 명시된 변수에 객체 리터럴

1. 해당 타입 속성이 있는지 체크합니다.
2. 그 외 속성은 없는지 확인합니다.

> 잉여 속성 체크를 하면 타입 시스템 구조적 본질을 해치지 않습니다.
>
> 객체 리터럴에 알수 없는 속성을 허용하지 않습니다.

아래 예시 중 에러를 발생하는 코드는 어떤 것일까요?

```tsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
```

```tsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};

const r: Room = obj;
```

<aside>
ℹ️ 에러가 나는 코드 구조적 타입 시스템에서 발생할 수 있는 ‘잉여 속성 체크’ 과정이 수행

</aside>

- 잉여 속성 체크를 원하지 않는다면, 인덱스 시그니처를 사용하세요.
  - 추가적인 속성을 예상할 수 있습니다.
  - 약한 타입은 공통 속성과 별도의 체크를 수행합니다.

> **공통 속성 체크 vs 잉여 속성 체크**
>
> ### 공통 속성 체크
>
> - 공통 속성이 있는지 확인하는 별도 체크를 수행합니다.
> - 잉여 속성 체크와 마찬가지로 오타를 잡는데 효과적입니다.
>   - 구조적으로 엄격하지 않습니다.
>
> ### 잉여 속성 체크
>
> - 객체 리터럴을 변수에 할당하거나 함수에 매개 변수로 전달 할 때 잉여 속성 체크가 수행됩니다.
>   - 적용 범위가 제한적입니다.
>   - 객체 리터럴에만 적용됩니다.
> - 임시 변수를 도입하면 잉여 속성 체크를 건너 뛸 수 있습니다.

# 아이템 12 - 함수 표현식에 타입 적용하기

> 자바스크립트와 타입스크립트는 함수 문장(statement)과 함수 표현식(expression)을 다르게 인식합니다.
>
> ```tsx
> function rollDice1(sides: number): number {
>   /* ... */
> } // Statement
> const rollDice2 = function (sides: number): number {
>   /* ... */
> }; // Expression
> const rollDice3 = (sides: number): number => {
>   /* ... */
> }; // Also expression
> ```

### 타입스크립트는 함수 표현식 사용을 권장합니다.

- 함수 타입으로 선언하면 재사용이 가능합니다.
  - 함수의 매개 변수부터 반환값까지 타입을 선언
  ```tsx
  type BinaryFn = (a: number, b: number) => number;
  const add: BinaryFn = (a, b) => a + b;
  const sub: BinaryFn = (a, b) => a - b;
  const mul: BinaryFn = (a, b) => a * b;
  const div: BinaryFn = (a, b) => a / b;
  ```
- 불필요한 코드의 반복을 줄여줍니다.
- 안정적입니다.
- 반복되는 함수 시그니처를 통합할 수 있습니다.

**예시 — declare 되어 있는 타입을 활용하여 추론 시킬 수 있습니다.**

```tsx
declare function fetch(
  input: RequestInfo | URL,
  init?: RequestInit
): Promise<Response>;

const checkedFetch: typeof fetch = async (input, init) => {
  //인자 타입이 추론됩니다.
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error("Request failed: " + response.status);
  }
  return response;
};
```

# 아이템 13 - 타입과 인터페이스 차이점 알기

명명된 타입(named type)을 정의하는 방법을 두 가지 입니다.

1. 타입

   ```tsx
   type TState = {
     name: string;
     capital: string;
   };
   ```

2. 인터페이스

   ```tsx
   interface IState {
     name: string;
     capital: string;
   }
   ```

<aside>
ℹ️ 대부분의 겨우 타입, 인터페이스를 사용해도 됩니다.

</aside>

### 공통점

- 정의 상태에는 차이가 없습니다.
- 인덱스 시그니처를 사용할 수 있습니다.

  ```tsx
  type TDict = { [key: string]: string };
  interface IDict {
    [key: string]: string;
  }

  type TFn = (x: number) => string;
  interface IFn {
    (x: number): string;
  }

  const toStrT: TFn = (x) => "" + x; // OK
  const toStrI: IFn = (x) => "" + x; // OK
  ```

  - 단순한 함수 타입은 타입 별칭이 더 나은 선택이지만, 추가적인 속성이 있다면 타입이나 인터페이스 어떤 것을 선택하든 상관없습니다.

- 모두 제너릭이 가능합니다.
  ```tsx
  type TPair<T> = {
    first: T;
    second: T;
  };
  interface IPair<T> {
    first: T;
    second: T;
  }
  ```
- 확장 할 수 있습니다.

  ```tsx
  type TState = {
    name: string;
    capital: string;
  };

  interface IState {
    name: string;
    capital: string;
  }

  interface IStateWithPop extends TState {
    population: number;
  }

  type TStateWithPop = IState & { population: number };
  ```

- 클래스를 구현 (implement) 시에는 타입과 인터페이스 모두 사용 가능합니다.

### 차이점

- **인터페이스는 유니온과 같은 복잡한 타입은 확장할 수 없습니다.**

  - 타입을 확장하고 싶다면, 타입과 & 를 사용해야 합니다.
  - 아래와 같은 타입은 확장 할 수 없습니다.

  ```tsx
  type Input = {
    /* ... */
  };
  type Output = {
    /* ... */
  };

  type NamedVariable = (Input | Output) & { name: string };
  ```

  > 일반적으로 `type` 키워드가 쓰임새가 많습니다.
  >
  > 1. `type` 은 유니온이 될 수도 있습니다.
  > 2. 매핑된 타입과 조건부 타입과 같은 고급 기능 활용이 가능합니다.

- **인터페이스는 선언 병합(declaration merging)이 가능합니다.**
  - **‘보강’**이 가능하다.
  ```tsx
  interface IState {
    name: string;
    capital: string;
  }
  interface IState {
    population: number;
  }
  const hanam: IState = {
    name: "hanam",
    capital: "hanam",
    population: 500_000,
  };
  ```

# 아이템 14 - 연산과 제너릭 사용으로 반복 줄이기

1. DRY(Don’t repeat yourself) 원칙을 적용해야합니다.

- 타입에 이름을 붙여 반복을 피하고, 인터페이스에 필드 반복을 피합시다.
- typeof, keyof, 인덱싱, 매핑 타입을 학습하여 줄일 수 있습니다.
- 제너릭 타입을 함수와 같습니다.
  - extends 를 사용하여 타입을 지정하여 좁힐 수 있습니다.

# 아이템 15 - 동적 데이터에 인덱스 시그니처 사용하기

- 자바스크립트 객체는 문자열 키를 타입 값에 관계 없이 매핑합니다.
  - 인덱스 시그니처로 유연하게 매핑 할 수 있습니다.

```
type Rocket = {[property: string]: string}; // 인덱스 시그니처
const rocket: Rocket = {
 name: 'Falcon 9',
 variant: 'v1.0',
 thrust: '4,940 kN',
}; // OK
```

> 키의 이름 : 키의 위치를 표시한다. (타입 체커에서 사용하지 않습니다.)
>
> 키의 타입 : string, number, symbol 조합
>
> 값의 타입 : 어떤 것이든 될 수 있습니다.

### 단점

- 모든 키를 허용합니다.
- 특정 키가 필요하지 않습니다.
- ‘키’마다 다른 타입이 됩니다.
- 키는 엇이든 가능하므로 자동 완성 기능이 동작하지 않습니다.
  ⇒ 즉, 부정확합니다.

### 대안 0 - 인터페이스 활용하기

```tsx
interface Row1 {
  [column: string]: number;
} // Too broad
interface Row2 {
  a: number;
  b?: number;
  c?: number;
  d?: number;
} // Better
type Row3 =
  | { a: number }
  | { a: number; b: number }
  | { a: number; b: number; c: number }
  | { a: number; b: number; c: number; d: number };
```

### 대안1 - `Record` 사용하기

```tsx
type Vec3D = Record<"x" | "y" | "z", number>;
// Type Vec3D = {
// x: number;
// y: number;
// z: number;
// }
```

### 대안2 - 매핑된 타입 사용하기

```tsx
type Vec3D = { [k in "x" | "y" | "z"]: number };
// Same as above
type ABC = { [k in "a" | "b" | "c"]: k extends "b" ? string : number };
// Type ABC = {
// a: number;
// b: string;
// c: number;
// }
```

# 아이템 16 - number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

> 자바스크립트는 **암시적 타입 강제**로 악명 높습니다.
>
> → 대부분 `===` , `≠=` 를 사용해서 해결 가능합니다.

### 배열의 인덱스

- 배열은 `string` 타입의 인덱스로 접근 가능합니다.
  → Object.keys를 이용하면 키자 문자열로 출력됩니다!
- Array 타입 선언
  ```tsx
  interface Array<T> {
    // ...
    [n: number]: T;
  }
  ```

> 인덱스 시그니처가 number 표현된다면 입력 값이 number 여야함을 의미합니다.
>
> 하지만 실제 런타임에서 사용되는 키는 `string` 입니다.

### 요약

- 배열은 객체이므로 키는 숫자가 아니라 문자열
- 인덱스 시그니처는 number보다 `Array`, `튜플`, `ArrayLike` 타입 사용하기

# 아이템 17 - 변경 관련된 오류 방지를 위해 readonly 사용하기

### `readonly number[]`

- 배열의 요소를 읽을 수 있지만, 쓸 수 없습니다.
- length를 읽을 수 있지만 바꿀 수 없습니다.
- 배열을 변경하는 메서드를 사용할 수 없습니다.
  - 예시
    ```tsx
    function arraySum(arr: readonly number[]) {
      let sum = 0,
        num;
      while ((num = arr.pop()) !== undefined) {
        // ~~~ 'pop' does not exist on type 'readonly number[]'
        sum += num;
      }
      return sum;
    }
    ```
- `readonly number[]` 는 `number[]` 보다 기능이 많습니다.
  - `number[]` 은 `readonly number[]` 보다 기능이 많으므로 서브셋입니다.

```tsx
const a: number[] = [1, 2, 3];
const b: readonly number[] = a;
const c: number[] = b; //b가 변경가능하므로 error
```

### Readonly 의 특징

<aside>
ℹ️ 매개변수를 수정하지 않는 경우 `readonly` 로 선언합시다.

</aside>

- 지역 변수와 관련된 모든 종류의 변경 오류를 방지 할 수 있습니다.
- 의도치 않은 변경을 방지 할 수 있습니다.
- 인덱스 시그니처에도 사용할 수 있습니다.
  - 읽기를 허용하고 쓰기를 방지할 수 있습니다.

<aside>
ℹ️ `readonly` 는 얉게(shallow) 동작합니다.

</aside>

- 객체 자체가 `readonly` 는 아닙니다.
  ```tsx
  const dates: readonly Date[] = [new Date()];
  dates.push(new Date());
  // ~~~~ Property 'push' does not exist on type 'readonly Date[]'
  dates[0].setFullYear(2037);
  ```
- `Readonly` 제너릭에도 해당됩니다.
  ```tsx
  interface Outer {
    inner: {
      x: number;
    };
  }
  const o: Readonly<Outer> = { inner: { x: 0 } };
  o.inner = { x: 1 };
  // ~~~~ Cannot assign to 'inner' because it is a read-only property
  o.inner.x = 1; // OK
  ```

# 아이템 18 - 매핑된 타입을 사용하여 값을 동기화하기

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b3e9863c-7f6b-49bf-a33f-3fe1024846d7/Untitled.png)

만약 위와 같은 산점도(scatter plot)을 그리기 위한 **UI 컴포넌트를 작성**한다고 가정하겠습니다.

```tsx
interface ScatterProps {
  // The data
  xs: number[];
  ys: number[];

  // Display
  xRange: [number, number];
  yRange: [number, number];
  color: string;

  // Events
  onClick: (x: number, y: number, index: number) => void;
}
```

데이터나 디스플레이 속성이 변경되면 다시 그려야 하지만, **이벤트 핸들러가 변경되면 다시 그릴 필요가 없습니다**.
(리액트의 useCallback 훅은 렌더링할 때마다 새 함수를 생성하지 않도록 하는 또 다른 기법입니다.)

최적화를 **두 가지 방법**으로 구현해보겠습니다.

## 실패에 닫힌 접근법 (= 보수적 접근법)

오류가 발생했을 때 적극적으로 대처하는 방향을 말합니다.

```tsx
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== "onClick") return true;
    }
  }
  // oldProps와 newProps를 받아 onClick 제외 새로운 속성이 추가되면 true 그 외는 false
  return false;
}
```

### 장점

- 새로운 속성이 생길 때마다 차트가 변경되기 때문에 차트가 정확합니다.

### 단점

- 너무 자주 그려질 가능성이 있습니다.

## 실패에 열린 접근법

오류 발생 시 소극적으로 대처하는 방향입니다.

```tsx
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // (no check for onClick)
  );
}
```

### 장점

- 차트를 불필요하게 다시 그리는 단점을 해결했습니다.

### 단점

- 실제로 차트를 다시 그려야 할 경우에 누락되는 일이 생길 수 있습니다.

---

## **매핑된 타입으로 문제 해결하기**

```tsx
const **REQUIRES_UPDATE**: {[k in keyof ScatterProps]: boolean} = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  **onClick: false**,
};

function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && **REQUIRES_UPDATE[k]**) {
      return true;
    }
  }
  return false;
}

// 만약 ScatterProps에 onDoubleClick: () => void가 들어간다면?!
// REQUIRES_UPDATE['onDoubleCLick'] > 'onDoubleClick' 속성이 타입에 없다고 오류 발생!
```

### 장점

- 한 객체가 또 다른 객체와 정확히 같은 속성을 가졌는지 확인할 때 이상적입니다.
