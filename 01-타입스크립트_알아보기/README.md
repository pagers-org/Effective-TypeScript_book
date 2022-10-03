# 1 타입스크립트 알아보기

타입스크립트는 사용 방식 면에서 조금은 독특한 언어다. 

인터프리터로 실행되는 것도 아니고, 저수준 언어로 컴파일되는 것도 아니다. 또 다른 고수준 언어인 자바스크립트로 컴파일되며, 실행 역시 타입스크립트가 아닌 자바스크립트로 이루어진다.

# 아이템 1 - 타입스크립트와 자바스크립의 관계 이해하기

타입스크립트는 자바스크립트의 상위 집합(superset)이다.
![image](https://user-images.githubusercontent.com/77133565/193392552-07d6f1a1-6c02-478a-aff9-ac04bdffad09.png)

타입스크립트 컴파일러는 타입스크립트뿐만 아니라 일반 자바스크립트 프로그램에도 유용하다.

```tsx
let city = 'new york city';
console.log(city.toUppercase());

```
이 코드를 실행하면 다음과 같은 오류가 발생한다.

```
TypeError: city.toUppercase is not a function
//'toUppercase' 속성이 'string' 형식에 없습니다. 'toUpperCase'을(를) 사용하시겠습니까?
```

city 변수가 문자열이라는 것을 알려주지 않아도 타입스크립트는 초깃값으로부터 타입을 추론한다. 타입 시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것이다. 타입스크립트가 '정적' 타입 시스템이라는 것은 바로 이런 특징을 말하는 것이다!

그렇지만 타입 체커가 모든 오류를 찾아내지는 않는다.


---

# 아이템 2 - 타입스크립트 설정 이해하기

컴파일러 설정 파일 `tsconfig.json`

설정 파일은 tsc --init만 실행하면 간단히 생성된다. 또한 가급적 설정 파일을 사용하는 것이 좋다. 그래야 타입스크립트를 어떻게 사용할 계획인지 다른 동료 또는 도구가 알 수 있기 때문이다.


```jsx
function add(a,b) {
  // ~ 'a' 매개변수에는 암시적으로 'any' 형식이 포함됩니다.
  // ~ 'b' 매개변수에는 암시적으로 'any' 형식이 포함됩니다.
  return a+b;
}
```

any 코드를 넣지 않았지만, any 타입으로 간주되어 이를 '암시적 any'라고 부른다. 그런데 noImplicitAny가 설정되었다면 오류가 된다. 아래와 같다.

```jsx
{
	"compilerOptions": {
		"noImplicityAny": true
	}
}
```

### **noImplicitAny**

- 코드를 작성할 때마다 타입을 명시해야한다.
- 타입스크립트가 문제를 발견하기 수월해진다.
- 코드의 가독성이 좋아진다.
- 개발자의 생산성이 향상된다.


위와같은 장점을 가지니 되도록이면 noImplicitAny를 설정하자.


### **strictNullChecks**

null과 undefined가 모든 타입에서 허용되는지 확인하는 설정이다.

 ****

**strictNullChecks 해제 시 유효 코드**

```jsx
const x: number = null; // 정상, null은 유효한 값입니다.
```

**strictNullChecks 설정 시 오류 코드**

```jsx
const x: number = null; // ~ 오류 'null'형식은 'number' 형식에 할당할 수 없습니다.
```

null 대신 undefined를 써도 같은 오류가 난다. 만약에 null을 허용하려고 한다면 명시적으로 드러냄으로 오류를 고칠 수 있다.

```jsx
const x: number | null = null;
```

만약 null을 허용하지 않으려면, 이 값이 어디서부터 왔는지 찾아야하며, null을 체크하는 코드나 단언문(assertion)을 추가해야한다.

```jsx
const el = document.getElementById('status');
el.textContent = 'Ready';
// ~~ 오류: 개체가 null 인것 같습니다.

if (el) {
	el.textContent = 'Ready'; // 정상, null은 제외됩니다.
}
el!.textContent = 'Ready'; // 정상, el이 null이 아님을 단언합니다.
```

> **strictNullChecks는 null과 undefined 관련된 오류를 잡아 내는 데 많은 도움이 되지만, 코드 작성을 어렵게 한다.

새로운 프로젝트를 시작하는 경우에는 strictNullChecks 설정하는 것이 좋지만, 자바스크립트 코드를 마이그레이션 하는 중인 경우에는 설정하지 않아도 괜찮다.

strictNullChecks를 설정하려면 noImplicitAny를 먼저 설정해야한다.



# 아이템 3 - 코드 생성과 타입이 관계없음을 이해하기

크게 타입스크립트 컴파일러는 두 가지 역할을 수행한다.

- 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일한다.
- 코드의 타입 오류를 체크한다.

위의 두 가지는 완벽하게 독립적이다.


### 타입 오류가 있는 코드도 컴파일이 가능하다.

컴파일과 타입 체크와 독립적으로 동작하기 때문에, 타입 오류가 있는 코드도 컴파일이 가능하다. 

> 타입 오류가 있는 데도 컴파일 되는 것이 도움이 되는 부분도 있다.


### 런타임에는 타입 체크가 불가능하다.

```tsx
interface Square {
	width: number;
}

interface Rectangle extends Squre {
	height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
	if (shape instanceof Rectangle) {
				//  ~~~ Rectangle 형식만 참조하지만 여기서는 값으로 사용되고 있습니다.
		return shape.width * shape.height // ~~~ shape 형식에 height 속성이 없습니다.
	} else {
		return shape.width * shape.width;
	}
}
```

**유니온 타입을 통해 해결**

```tsx
interface Square {
	kind: 'square';
	width: number;
}

interface Rectangle {
	kind: 'rectangle';
	height: number;
	width: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
	if (shape.kind === 'rectangle') {
		shape; // 타입이 Rectangle
		return shape.width * shape.height;
	} else {
		shape;
		return shape.width * shape.width
	}
}
```


타입 (런타입 접근 불가)와 값(런타입 접근 가능)을 둘 다 사용하는 기법도 있다.



### 타입 연산은 런타임에 영향을 주지 않는다.



```tsx
function asNumber(val:number | string): number {
	return val as number;
}
```

위 코드는 타입 체커를 통과하지만 잘못된 방법을 썼다.


```tsx
function asNumber(val:number | string): number {
	return typeof(val) === 'string' ? Number(val) : val;
}
```

위 코드가 올바른 방법이다.


### 런타임 타입은 선언된 타입과 다를 수 있다.

```tsx
interface LightApiResponse {
	lightSwitchValue: boolean;
}

async function setLight() {
	const response = await fetch('/light');
	const result: LightApiResponse = await response.json();
	setLightSwitch(result.lightSwitchValue);
}
```

> 타입스크립트에서는 런타임 타임과 선언된 타입이 맞지 않을 수 있다.


### 타입스크립트 타입으로는 함수를 오버로드할 수 없다.

동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용하는 것을 '함수' 오버로딩이라 한다. 그러나 타입스크립트에서는 타입과 런타임의 동작이 무관하기 때문에, 함수 오버로딩은 불가능하다. 

```tsx
function add(a: number, b: number) { return a + b };
	// ~~ 중복된 함수 구현입니다.

function add(a: string, b: string) { return a + b };
	// ~~ 중복된 함수 구현입니다.
```

타입스크립트가 함수 오버로딩 기능을 지원하지만, 온전히 타입 수준에서만 동작한다.


### 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.

타입과 타입 연산자는 자바스크립트 변환 시점에 제거 되기 때문에, 런타임의 성능에 아무런 영향을 주지 않는다.

- 런타임 오버헤드가 없는 대신, 타입스크립트 컴파일러는 '빌드타임' 오버헤드가 있다.
- 코드 생성은 타입 시스템과 무관하다. 타입스크립트 타입은 런타임 동작이나 성능에 영향을 주지 않는다.
- 타입 오류가 존재하더라도 컴파일은 가능하다.
- 타입스크립트 타입은 런타임에 사용할 수 없다. 런타임에 타입을 지정하려면, 타입 정보 유지를 위한 별도의 방법이 필요하다.
    - 일반적으로 태그된 유니온 방법과 속성 체크 방법을 사용한다. 클래스 같이 타입스크립트 타입과 런타임 값, 둘 다 제공하는 방법이 있다.

## 아이템 4 - 구조적 타이핑에 익숙해지기

**덕 타이핑**: 객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주하는 방식이다.

- 구조적 타이핑을 사용하면 유닛 테스팅을 손쉽게 할 수 있다.

구조적 타이핑을 제대로 이해한다면 오류인 경우와 오류가 아닌 경우의 차이를 알 수 있고, 더욱 견고한 코드를 작성할 수 있다.

```tsx
interface Vector2D {
	x: number;
	y: number;
}

interface NamedVector {
	name: string;
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x+v.y * v.y);
}

const v: NameVector = { x:3, y:4, name: 'Zee' };
calculateLength(v); // 정상, 결과는 5

```

 
클래스 역시 구조적 타이핑 규칙을 따른다는 것을 명심해야한다. 클래스의 인스턴스가 예상과는 다를수 있다.


# 아이템 5 - any 타입 지양하기

타입스크립트의 타입 시스템은 점진적이고 선택적이다. 코드에 타입을 조금씩 추가할 수 있기 때문에 점진적이며, 언제든지 타입 체커를 해제할 수 있기 때문에 선택적이다. 이러한 기능들의 핵심은 any이다. 

```tsx
let age: number;
age = '12';
// ~~~ '12' 형식은 'number' 형식에 할당할 수 없습니다. 
age = '12' as any; // ok 
```

타입에 시간을 쏟고 싶지 않아 any 타입이나 타입 단언문(as any)를 추가하여 해결하고 싶을 것이다. 그러나 사용하는 것을 지양해야한다. 사용하더라도 그 위험성에 대해 알아야한다.

**any의 위험성**

### any 타입에는 타입 안정성이 없습니다.

any를 사용하면 number나 string 어떤 타입이든 할당할 수 있게 된다.

### any는 함수 시그니처를 무시하게 된다.

함수를 작성하는 경우에는 시그니처를 명시해야한다. 호출하는 쪽은 약속된 타입의 입력을 제공하고, 함수는 약속된 타입의 출력을 반환한다. 

### any 타입에는 언어 서비스가 적용되지 않는다.

어떤 심벌에 타입이 있다면 타입스크립트 언어 서비스는 자동완성 기능과 적절한 도움말을 제공한다. 그러나 any타입인 심벌을 사용하면 아무런 도움을 받을 수 없다.

### any 타입은 코드 리팩터링때 버그를 감춘다.

타입 체크를 통과했다고 끝난 것이 아니다. 놓친 타입 체크가 있을 수 있는데 이 때, any가 아니라 구체적인 타입을 사용한다면 타입 체커가 오류를 발견해줄 수도 있다.

### any는 타입 설계를 감춰버린다.

앱 상태 같은 객체를 정의하려면 꽤 복잡하지만 any 타입을 사용하면 간단히 끝내 버릴 수도 있다. 하지만 이때도 any를 사용하면 안된다. 객체를 정의할 때 특히 문제가 되는데, 상태 객체의 설계를 감춰버리기 때문이다. 

### any타입은 타입시스템의 신뢰도를 떨어트린다.

사람은 항상 실수를 하는데, 보통 타입 체커가 실수를 잡아주어 코드의 신뢰도가 높아진다. 그렇지만 any를 사용하면 타입 체커의 존재 이유가 없어진다.

