# 6장 타입 선언과 @types

# Item 45

: **Put TypeScript and @types in devDependencies**

**Before you start…**

**dependencies ( a.k.a transitive dependencies)**
These are packages that are required **to run your JavaScript**. If you import lodash at runtime, then it should go in dependencies. When you publish your code on npm and another user installs it, it will also **install** these dependencies.

**devDependencies**

These packages are used **to develop and test your code but are not required at runtime**. Your test framework would be an example of a devDependency. Unlike dependencies, **these are _not_ installed transitively with your packages. —save-dev 를 통해 devDependencies 에 설치할 수 있습니다.**

**peerDependencies**

These are packages that you **require at runtime but don’t want to be responsible for tracking**. The canonical example is a **plug-in**. Your jQuery plug-in is compatible with a range of versions of jQuery itself, **but you’d prefer that the user select one, rather than you choosing for them**.

**Avoid installing TypeScript system-wide**

\*system-wide: -g

The first dependency to consider is **TypeScript itself**.

[How to set up TypeScript](https://www.typescriptlang.org/download)

**→ In your project or globally? Which one would be better choice?**

It is possible to install Type‐ Script system-wide,
but this is generally **a bad idea for two reasons**:

1. There’s no guarantee that you and your coworkers will always have the same version installed.
2. It adds a step to your project setup.

→ **project (devDependencies)**

1. That way you and your coworkers will always get the correct version when you
   **`run npm install`**

1. Also, no extra setup needed for your project
   `$ **npx tsc**`

**Put @types dependencies in devDependencies, not dependencies**

The next type of dependency to consider is **_type dependencies_ or @types.**
Your @types dependencies should also be **devDependencies**, even if the package itself is a direct dependency.

```jsx
// ex)
$ npm install react

$ npm install **--save-dev @types/react**
```

This is because…
The idea here is that you should publish JavaScript, not TypeScript, and your **JavaScript does not depend on the @types when you run it**.

<aside>
💡  Recoil 같이 ts 가 바탕이 된 라이브러리는 
따로 ***type dependencies* or @types 를 설치해줄 필요 없다.**

</aside>

# Item 46

: **Understand the Three Versions
Involved in Type Declarations**

타입스크립트는 dependency 관리를 더 어렵게 한다.

**This is because…**

<aside>
🐵 **1. The version of the package (ex. react)
2. The version of its type declarations (@types) (ex. @types/react)
3. The version of TypeScript (ex. typescript)**

</aside>

**위 세가지중 하나라도 싱크가 맞지 않으면 에러가 나기 때문이다.**

**ex)**

```jsx
$ **npm install react**
+ react@16.8.6
$ **npm install --save-dev @types/react**
+ @types/react@16.8.19
```

터미널에서 react 세팅을 할 때 흔히 볼 수 있는 화면이다.
16.8 까지는 맞지만 (major, minor 버전은 맞지만) 그 뒤 숫자(patch 버전)은 다르다는 걸 확인할 수 있다.
이럴 경우 public 하는데는 전혀 문제가 되지 않는다. 하지만 타입선언(개발) 부분에선 문제가 생길 수 있다.

**어떤 문제가 생길 수 있냐,**

- 첫째, 라이브러리 버전과 타입 선언의 버전이 맞지 않는다.
- 둘째, 타입스크립트 버전과 타입 선언의 버전이 맞지 않는다. (타스 업그레이드가 필요할 수도 있다.)
- 셋째, **타입 선언을 복제해 버릴 수도 있다.**
  ```jsx
  // 예)
  // @types/foo 와 @types/bar 를 devDependency로 가지고 있다고 해보자.
  // 만약 @types/bar 가 @types/foo 의 다른 버전을 원한다면, 기존 버전과 필요한 버전을 둘 다 설치해버릴 수 도 있습니다.
  node_modules/
        @types/
          foo/
            index.d.ts @1.2.3
          bar/
            index.d.ts
            node_modules/
  								@types/
  									foo/
  			               index.d.ts @2.3.4
  ```

**어떻게 해결할까? 두 가지 해결책: 번들링 타입과 완전한 타입**

1.  **번들링(Bundling Types) - TypeScript 를 이용한 프로젝트시(ex. axios)**

    - `d.ts` 파일을 따로 만들어 관련 타입끼리 묶어 관리하자(번들링)
      `tsc` 가 자동으로 타입 선언을 해주기 때문에 아래처럼 이용할 수 있다.
          ```jsx
          // package.json
          {
          "name": "left-pad",
          "version": "1.3.0",
          "description": "String left pad",
          	"main": "index.js",
          "types": "index.d.ts",
          // ...
          }
          ```
    - 장점: 위의 버전이 맞지 않는 오류를 해결할 수 있다.
    - 단점:

    1. First, what if there’s an error in the bundled types that can’t be fixed through augmentation?
    2. Second, what if your types depend on another library’s type declarations? Usually this would be a devDependency (Item 45).
    3. Third, what if you need to fix an issue with the type declarations of an old version of your library?
    4. Fourth, how committed to accepting patches for type declarations are you?

1.  **완전한 타입(DefinitelyTyped) - JavaScript 만 이용한 라이브러리 사용시**
    - 번들링 대신 **완전한 타입(DefinitelyTyped)** 을 쓰길 권한다.
    - `tsc` 가 자동으로 타입 선언을 해주지 않기 때문에 번들링 시 에러 발생 및 더 많은 작업이 필요할 수 있기 때문이다.

# Item 47

: **Export All Types That Appear in Public APIs**

1. Suppose you want to create some **secret, unexported types**:

```jsx

**interface** SecretName {
	first: **string**;
	last: **string**;
}

**interface** SecretSanta {
	name: SecretName;
	gift: **string**;

}

**export function** getGift(name: SecretName, gift: **string**): SecretSanta {
*// ...*

}
```

As a user of your module, I cannot directly import SecretName or SecretSanta, only getGift.
**But this is no barrier: because those types appear in an exported function signature, I can extract them.**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1b2f1d31-d668-4230-845d-8ebe83b27073/Untitled.png)

If your goal in not exporting these types was to preserve flexibility, then the jig is up! You’ve already committed to them by putting them in a public API. Do your users a favor and export them.

```jsx

**export** **interface** SecretName {
	first: **string**;
	last: **string**;
}

**export** **interface** SecretSanta {
	name: SecretName;
	gift: **string**;

}

**export function** getGift(name: SecretName, gift: **string**): SecretSanta {
*// ...*

}

```

→ **결론: 공개 메서드의 타입은 어차피 다 숨겨지지도 않으니, 유저들의 편의를 위해 타입도 export 하자 .**

```jsx
type MySanta = ReturnType<typeof getGift>;
// SecretSanta
type MyName = Parameters<typeof getGift>[0];
// SecretName
```

근데 Parameters, ReturnRtype으로 인덱스로 접근하는게 구려서 그냥 export하는게 나은것같네요(wlghks)

# \*\*Item 48

: Use TSDoc for API Comments\*\*

**Use JSDoc-/TSDoc-formatted comments**

As you know, there are 2 types of comments in coding:
**1) inline comment // 2) block comment( a.k.a JSDoc-style comment) /** \*/\*\*

**ex)**

```jsx
/** Generate a greeting. Result is formatted for display. */
function greetJSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`;
}

// Generate a greeting. Result is formatted for display.
function greet(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

**result)**

![*Figure 6-1. JSDoc-style comments are typically surfaced in tooltips in your editor.*](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/25dc7591-1054-4153-95da-dc111fd3c6d2/Untitled.png)

_Figure 6-1. JSDoc-style comments are typically surfaced in tooltips in your editor._

![_Figure 6-2. Inline comments are typically not shown in tooltips._

](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9b0368c8-3cd9-423b-b98f-f8fb4ff71506/Untitled.png)

_Figure 6-2. Inline comments are typically not shown in tooltips._

**Use @param, @returns, and Markdown for formatting**

함수가 아니라 타입(인터페이스)에 TSDoc 스타일 주석을 붙이면 어떻게 될까요?
The TypeScript language service supports this convention, and you should take advantage of it.

**ex)**

```
/** A measurement performed at a time and place. */

interface Measurement {
	/** Where was the measurement made? */
	position: Vector3D;
	/** When was the measurement made? In seconds since epoch. */
	time: number;
	/** Observed momentum */
	momentum: Vector3D;
}

```

**result)**

![_Figure 6-4. TSDoc for a field is shown when you mouse over that field in your editor._

](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a41b9e2e-9b6f-4136-9043-a4186a12acc1/Untitled.png)

_Figure 6-4. TSDoc for a field is shown when you mouse over that field in your editor._

**TSDoc comments are formatted using Markdown, so if you want to use bold, italic, or bulleted lists**

**ex)**

```
 /**
     * This _interface_ has **three** properties:
     * 1. x
     * 2. y
     * 3. z
     */

interface Vector3D {
	x: number;
	y: number;
	z: number;
}
```

**result)**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/24438cb3-ec5a-4b9c-88df-a39e9ace4ea2/Untitled.png)

**Avoid including type information in documentation**

- 주석은 가능하면 짧고 간단하게. 필요한 정보만.
- 주석에 type 명시하지 마라. 쓸데 없음. 이미 type signiture 만으로도 충분히 명시적이다.

# \*\*Item 49

: Provide a Type for _this_ in Callbacks 🚨\*\*

**Understand How _this_ Binding Works**

아래 ResetButton 클래스에는 2가지 메서드가 정의되어 있습니다.
각각 메서드에는 this 가 있습니다.

```jsx
class ResetButton {
  render() {
    return makeButton({ text: "Reset", onClick: this.onClick });
  }

  onClick() {
    alert(`Reset ${this}`);
  }
}
```

Button 을 눌러 onClick 을 실행 시키면 결과는 어떻게 될까요?
**“Reset undefined”** 이라는 알림창이 뜹니다! **( this = undefined)**
this 바인딩 문제 때문인데요,
이럴 경우 가장 간단한 해결책은 constructor 를 아래와 같이 지정해주는 것입니다.

```jsx
class ResetButton {
  constructor() {
    this.onClick = this.onClick.bind(this); // this = ResetButton
  }

  render() {
    return makeButton({
      text: "Reset",
      onClick: this.onClick,
    });
  }

  onClick() {
    alert(`Reset ${this}`); // "this" always refers to the ResetButton instance.
  }
}
```

더 간단한 방법으로는 화살표 함수로 만들어주는 방식이 있습니다.

```jsx
// a shorthand: arrow function

class ResetButton {
  render() {
    return makeButton({ text: "Reset", onClick: this.onClick });
  }

  onClick = () => {
    alert(`Reset ${this}`); // "this" always refers to the ResetButton instance.
  };
}
```

**Provide a type for _this_ in callbacks when it’s part of your API**

**그래서 이게 TypeScript 와 무슨 상관인가요?**
TypeScript 는 JavaScript 의 상위모델이기 때문에 TypeScript 역시 JavaScript 의 this binding 규칙을 따라게 됩니다. 더불어 **TypeScript 는 올바른 this context 를 넣도록 도와줍니다**.

**예)**

```jsx
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener("keydown", (e) => {
    fn.call(el, e);
  });
}
```

콜백함수(인자fn)의 인자 this 는 특별하게 처리됩니다.

call 을 붙여 하나의 인자로 처리해주지 않는다면?
→ 에러: 인자를 하나만 보내라고 처리가 됩니다.

```jsx
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener("keydown", (e) => {
    fn(el, e);
    // ~ Expected 1 arguments, but got 2
  });
}
```

그렇다고 e 하나만 넣으면?
→ 에러: 올바른 this context 를 넣도록 도와주는 TypeScript

```jsx
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener("keydown", (e) => {
    fn(e);
    // ~~~~~ The 'this' context of type 'void' is not assignable
    // to method's 'this' of type 'HTMLElement'
  });
}
```

**해결책 2가지)**

**1)** **declare _this_ type** for better type safety

```jsx
declare let el: HTMLElement;

addKeyListener(el, function(e) {
	this.innerHTML;
	// OK, "this" has type of HTMLElement
});
```

**2) arrow function**

```jsx
class Foo {
  registerHandler(el: HTMLElement) {
    addKeyListener(el, (e) => {
      this.innerHTML;
      // ~~~~~~~~~ Property 'innerHTML' does not exist on type 'Foo'
    });
  }
}
```

# \*\*Item 50

: Prefer Conditional Types
to Overloaded Declarations\*\*

<aside>
💡 **Review for function overloading (Item3)**
TypeScript provides the concept of function overloading. You can have multiple functions with the same name but different parameter types and return type. However, the number of parameters should be the same.

```jsx
function double(a:string):string;

function double(a:number): number;

function double(a:any): any {
    return a + a;
}

double("Hello "); // returns "Hello Hello "
double(10); // returns 20
```

</aside>

**solution 1: Union Type**

```jsx
function double(x: number|string): number|string;
function double(x: any) { return x + x; }
```

**However…**

```jsx
typeof double("Hello "); // number | string
```

**solution 2: Generics**

```jsx
function double<T extends number|string>(x: T): T;
function double(x: any) {
	return x + x;
}
```

**However…**
The types are now a little *too p*recise.

```jsx
const num = double(12);
// Type is 12
const str = double("x");
// Type is "x"
```

**solution 3: to provide multiple type declarations**

```jsx
function double(x: number): number;
function double(x: string): string;
function double(x: any) { return x + x; }
```

→ However… still lil bug

```jsx
const num = double(12);
// Type is number
const str = double("x");
// Type is string

function f(x: number | string) {
  return double(x);
  // ~ Argument of type 'string | number' is not assignable
  //   to parameter of type 'string'
}
```

**solution 4(결론): to use a _conditional type_**

```jsx

function add<T extends number | string>(
x:T

): T extends string ? string : number;

function add(x: any) { return x + x; }

```

This is similar to the first attempt to type double using a generic, but with a more
elaborate return type. You read the conditional type like you’d read a ternary (?:)
operator in JavaScript:

# \*\*Item 51

: Mirror Types to Sever Dependencies 🚨\*\*

**불필요한 의존성을 피하기 위해 그냥 새로운 타입을 프로젝트를 설정해라(= 미러링 기법)**

**예)**
Rather than using the **declaration of Buffer from @types/node**,
**you can write your own with just the methods and properties you need**.

```jsx
function parseCSV(contents: string | Buffer): { [column: string]: string }[] {
  if (typeof contents === "object") {
    // It's a buffer
    return parseCSV(contents.toString("utf8"));
  }
  // ...
}
```

In this case that’s just a toString method that accepts an encoding:

```jsx
interface CsvBuffer {
  toString(encoding: string): string;
}

function parseCSV(
  contents: string | CsvBuffer
): { [column: string]: string }[] {
  // ...
}
```

이렇게 설정된 function 은 아래처럼 사용할 수 있다.

```jsx
parseCSV(new Buffer("column1,column2\nval1,val2", "utf-8")); // OK
```

# \*\*Item 52

: Be Aware of the Pitfalls of Testing Types 🚨\*\*

결론: **how do you test types? → use dtslint or a similar tool that inspects types**

```jsx
declare function map<U, V>(array: U[], fn: (u: U) => V): V[];
```

How can you check that this type declaration results in the expected types?

```jsx
map(["2017", "2018", "2019"], (v) => Number(v));
```

```jsx
test("square a number", () => {
  square(1);
  square(2);
});
```

Sure, this tests that the square function doesn’t throw an error. But it’s missing any checks on the return value, so there’s no real test of the behavior. An incorrect implementation of square would still pass this test.

```jsx
function assertType<T>(x: T) {}

assertType<number[]>(map(['john', 'paul'], name => name.length));
```

```jsx
const n = 12;

assertType < number > n; // OK
```

```jsx
const beatles = ['john', 'paul', 'george', 'ringo'];
assertType<{name: string}[]>(
      map(beatles, name => ({ name, inYellowSubmarine: name === 'ringo' })
			)
);
// OK
```

```jsx
const add = (a: number, b: number) => a + b;
assertType<(a: number, b: number) => number>(add); // OK

const double = (x: number) => 2 * x;
assertType<(a: number, b: number) => number>(double); // OK!?
```

```jsx
const g: (x: string) => any = () => 12; // OK
```

```jsx
map(array, (name, index, array) => {
  /* ... */
});
```

```jsx
const double = (x: number) => 2 * x;
let p: Parameters<typeof double> = null!;
assertType<[number, number]>(p);
// ~ Argument of type '[number]' is not
// assignable to parameter of type [number, number]
let r: ReturnType<typeof double> = null!;
assertType<number>(r); // OK
```

```jsx
const beatles = ['john', 'paul', 'george', 'ringo']; assertType<number[]>(map(
beatles,
function(name, i, array) {
// ~~~~~~~ Argument of type '(name: any, i: any, array: any) => any' is
//
         not assignable to parameter of type '(u: string) => any'
assertType<string>(name); assertType<number>(i); assertType<string[]>(array); assertType<string[]>(this);
                    // ~~~~ 'this' implicitly has type 'any'
return name.length; }
));
```

```jsx
declare function map<U, V>(
  array: U[],
  fn: (this: U[], u: U, i: number, array: U[]) => V
): V[];
```

```jsx
declare module 'overbar';
```

```jsx
const beatles = ["john", "paul", "george", "ringo"];
map(
  beatles,
  function (
    name, // $ExpectType string
    i, // $ExpectType number
    array // $ExpectType string[]
  ) {
    this; // $ExpectType string[]
    return name.length;
  }
); // $ExpectType number[]
```

It’s tempting to make assertions about types inside the type system using the tools that TypeScript provides.

→ But there are several pitfalls with this approach.

---
