# 2장 타입스크립트의 타입 시스템
타입 시스템이란 무엇인지, 어떻게 사용해야 하는지, 무엇을 결정해야 하는지, 가급적 사용하지 말아야 할 기능은 무엇인지 알아본다.

## 아이템 6: 편집기를 사용하여 타입 시스템 탐색하기
TS를 설치하면,<br>
- TS 컴파일러(tsc)
- 단독으로 실행할 수 있는 TS 서버(tsserver)
를 실행할 수 있다.

보통은 tsc를 실행하는 것이 주된 목적이지만, TS서버 또한 `언어 서비스`를 제공한다는 점에서 중요하다. 언어 서비스에는 코드 자동완성, 명세(사양) 검사, 검색, 리팩터링이 포함된다. 보통은 편집기를 통해서 언어 서비스를 사용하는데, TS서버에서 언어 서비스를 제공하도록 설정하는 게 좋다.<br>

### 편집기를 사용하면 어떻게 타입 시스템이 동작하는지, TS가 어떻게 타입을 추론하는지 개념을 잡을 수 있다.
이러한 서비스들을 차치하더라도, 편집기는 코드를 빌드하고 타입 시스템을 익힐 수 있는 최고의 수단이다. 그리고 편집기는 TS가 언제 타입 추론을 실행할 수 있는지에 대한 개념을 잡게 해 주는데, 이 개념을 확실히 잡아야 간결하고 읽기 쉬운 코드를 작성할 수 있다.

편집기마다 조금씩 다르지만 보통의 경우에는 심벌 위에 마우스 커서를 대면 TS가 그 타입을 어떻게 판단하고 있는지 확인할 수 있다. 조건문의 분기에서 값의 타입이 어떻게 변하는지 살펴보는 것은 타입 시스템을 연마하는 매우 좋은 방법이다. 또한, 객체에서는 개별 속성을 살펴봄으로써 TS가 어떻게 각각의 속성을 추론하는지 살펴볼 수 있다.<br>
만약 연산자 체인(chain) 중간의 추론된 제너릭 타입을 알고 싶다면, 메서드 이름을 조사하면 된다.
``` ts
path.split('/').slice(1).join('/')
```
에서 slice에 마우스를 대면 `(method) Array<string>.slice(start?: number, end?: number): string[]`이라고 나온다. 여기서 split결과의 타입이 string의 배열이라고 추론되었음을 의미한다.

편집기상의 타입 오류를 살펴보는 것도 타입 시스템의 성향을 파악하는 데 좋은 방법이다.
``` ts
function getElement(elOrId: string | HTMLElement | null): HTMLElement {
  if(typeof elOrId === 'object') {
    return erOrId; // HTMLElement | null 형식은 HTMLElement 형식에 할당할 수 없습니다.
  } else if(elOrId === null) {
    return document.body;
  } else {
    const el = document.getElementById(elOrId);
    return el; // HTMLElement | null 형식은 HTMLElement 형식에 할당할 수 없습니다.
  }
}
```
if문의 의도는 단지 HTMLElement라는 객체를 골라내는 것이었다. but JS에서 `typeof null`은 `object`이므로, `erOrId`는 여전히 분기문 내에서 `null`일 가능성이 있다. 그러므로 처음에 `null`체크를 추가해서 바로잡는다.<br>
else문은 `document.getElementById`가 `null`을 반환할 가능성이 있어서 발생했고, 첫 번째 오류와 동일하게 `null`체크를 추가하고 예외를 던져야 한다.

### TS가 동작을 어떻게 모델링하는지 알기 위해 타입 선언 파일을 찾아보는 방법을 터득해야 한다.
또한, 언어 서비스는 라이브러리, 라이브러리의 타입 선언을 탐색할 때 도움이 된다.
``` ts
const response = fetch('https://example.com');
```
코드 내에서 `fetch`함수가 호출되고, 이 함수를 더 알아보길 원한다고 가정할 때, 편집기의 `Go to Definition`옵션을 제공한다. 이 옵션을 선택하면 TS에 포함되어 있는 DOM 타입 선언인 `lib.dom.d.ts`로 이동한다. 이를 통해 해당 함수가 어떤 매개변수를 받고 어떤 값을 반환하는지 파악할 수 있다.<br>
결론적으로 타입 선언은 TS가 무엇을 하는지, 어떻게 라이브러리가 모델링되었는지, 어떻게 오류를 찾아낼지 살펴볼 수 있는 훌륭한 수단이라는 것을 알 수 있다.

## 아이템 7: 타입이 값들의 집합이라고 생각하기

### 타입을 값의 집합으로 생각하면 이해하기 편하다. 이 집합은 유한(boolean, 리터럴 타입)하거나 무한(number, string)하다.
런타임에 모든 변수는 JS 세상의 값으로부터 정해지는 각자의 고유한 값을 가진다. 그러나 코드가 실행되기 전, 즉 TS가 오류가 체크하는 순간에는 타입을 가지고 있다. `할당 가능한 값들의 집합`이 타입이라고 생각하면 된다. 이 집합은 타입의 `범위`라고 부르기도 한다.<br>
가장 작은 집합은 아무 값도 포함하지 않는 공집합이며, TS에서는 never타입이다. never타입으로 선언된 변수의 범위는 공집합이기 때문에 아무런 값도 할당할 수 없다.<br>
그 다음으로 작은 집합은 한 가지 값만 포함하는 타입이다. 이들은 TS에서 유닛타입이라고도 불리는 리터럴 타입이다.
``` ts
type A = 'A';
type B = 'B';
type Twelve = 12;
```

두 개 혹은 세 개로 묶으려면 유니온 타입을 사용한다.
``` ts
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;
```
유니온 타입은 값 집합들의 합집합을 일컫는다.

다양한 TS 오류에서 `할당 가능한`이라는 문구를 볼 수 있다. 이 문구는 집합의 관점에서 `~의 원소(값과 타입의 관계)` 또는 `~의 부분 집합(두 타입의 관계)`를 의미한다.
``` ts
const a: AB = 'A'; // 정상
const c: AB = 'C'; // '"C"' 형식은 'AB'형식에 할당할 수 없습니다.
```

C는 유닛 타입이다. 범위는 단일 값 "C"로 구성되며 AB의 부분 집합이 아니므로 오류다. 집합의 관점에서 `타입 체커의 주요 역할`은 `하나의 집합이 다른 집합의 부분 집합인지 검사하는 것`이라고 볼 수 있다.

또는 다음처럼 원소를 서술하는 방법도 있다.
``` ts
interface Identified {
    id: string;
}
```
인터페이스가 타입 범위 내의 값들에 대한 설명이라고 생각해 본다. 어떤 객체가 string으로 할당 가능한 id속성을 가지고 있다면 그 객체는 Identified이다.

### 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있다.
`구조적 타이핑 규칙들은 어떠한 값이 다른 속성도 가질 수 있음`을 의미한다. 심지어 함수 호출의 매개변수에서도 다른 속성을 가질 수 있다. 이러한 사실은 특정 상황에서만 추가 속성을 허용하지 않는 잉여 속성 체크(express property checking)만 생각하다 보면 간과하기 쉽다.

### 객체 타입에서는 A & B인 값이 A와 B의 속성을 모두 가짐을 의미한다.
타입 연산은 집합의 범위에 적용된다. A와 B의 인터섹션은 A의 범위와 B의 범위의 인터섹션이다.
``` ts
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & Lifespan;
```

& 연산자는 두 타입의 인터섹션(교집합)을 계산한다. 언뜻 보면 두 인터페이스는 공통으로 가지는 속성이 없기 때문에 공집합(never)으로 예상하기 쉽다. 그러나 `타입 연산자는 인터페이스의 속성이 아닌, 값의 집합(타입의 범위)에 적용`된다. 그리고 추가적인 속성을 가지는 값도 여전히 그 타입에 속한다. 그래서 두 인터페이스 모두 가지는 값은 인터섹션 타입에 속하게 된다.

``` ts
const ps: PersonSpan = {
  name: 'Alan Turing',
  birth: new Date('1912/06/23'),
  death: new Date('1954/06/07'),
}
```

당연히 앞의 세 가지보다 더 많은 속성을 가지는 값도 PersonSpan타입에 속한다. 인터섹션 타입의 값은 각 타입 내의 속성을 모두 포함하는 것이 일반적인 규칙이다. 규칙이 속성에 대한 인터섹션에 관해서는 맞지만, 두 인터페이스의 유니온에서는 그렇지 않다.
``` ts
type K = keyof (Person | Lifespan); // 타입이 never
```
앞의 유니온 타입에 속하는 값은 어떠한 키도 없기 때문에, 유니온에 대한 keyof는 공집합(never)이어야만 한다. 조금 더 명확히 써 보자면 다음과 같다.
``` ts
keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```
이 등식은 TS의 타입 시스템을 이해하는 데 큰 도움을 될 것이다.

### A는 B를 상속, A는 B에 할당 가능, A는 B의 서브타입, A는 B의 부분 집합
TS타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합(벤 다이어그램)으로 표현된다. 두 타입은 서로 서브타입이 아니면서도 겹쳐질 수 있다.

조금 더 일반적으로 타입을 선언하는 방법은 extends 키워드를 쓰는 것이다.
``` ts
interface person {
  name: string;
}
interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```
타입이 집합이라는 관점에서 extends의 의미는 `~에 할당 가능한`과 비슷하게 `~의 부분 집합`이라는 의미로 받아들일 수 있다. PersonSpan 타입의 모든 값은 name, birth를 가져야 제대로 된 부분 집합이 된다.<br>
> **_서브타입_**<br>
어떤 집합이 다른 집합의 부분 집합.

1차원, 2차원, 3차원 벡터의 관점에서 생각해 보면 다음과 같은 코드를 작성할 수 있다.
``` ts
interface Vertor1D { x: number; }
interface Vertor2D extends Vertor1D { y: number; }
interface Vertor3D extends Vertor2D { z: number; }
```

Vector3D는 Vector2D의 서브타입이고 Vector2D는 Vector1D의 서브타입이다. 보통 이 관계는 상속 관계로 그려지지만, 집합의 관점에서는 벤 다이어그램으로 그리는 게 더욱 적절하다.
``` ts
function getKey<K extends string>(val: any, key: K) {
  // ...
}
```
string을 상속한다는 의미를 객체 상속의 관점으로 생각한다면 이해하기가 어렵다. 반면 string을 상속한다는 의미를 집합의 관점에서 생각해보면 쉽게 이해할 수 있다. string의 부분 집합 범위를 가지는 어떠한 타입이 된다. 이 타입은 string리터럴 타입, string리터럴 타입의 유니온, string자신을 포함한다.

타입들이 엄격한 상속 관계가 아닐 때는 집합 스타일이 더욱 바람직하다. 예를 들어, `string|number`와 `string|Date` 사이의 인터섹션은 공집합이 아니며(string이다.) 서로의 부분 집합도 아니다.<br>
타입이 집합이라는 관점은 배열과 튜플의 관계 역시 명확하게 만든다.
``` ts
const list = [1, 2]; // number[]
const tuple: [number, number] = list; // number[] 타입은 [number, number] 타입의 0, 1 속성에 없습니다.
```

이 코드에서 숫자 배열을 숫자들의 쌍이라고 할 수는 없다. 빈 리스트와 [1]이 그 반례이다. `number[]`는 `[number, number]`의 부분 집합이 아니기 때문에 할당할 수 없다.(반대는 동작한다.)

트리플(세 숫자를 가지는 타입)은 구조적 타이핑의 관점으로 생각하면 쌍으로 할당 가능할 것으로 생각된다. 그렇다면 쌍은 0, 1번 키를 가지므로, 2번 같은 다른 키를 가질 수 있을지 확인해 본다.
``` ts
const triple: [number, number, number] = [1, 2, 3];
const double: [number, number] = triple;
// [number, number, number] 형식은 [number, number] 형식에 할당할 수 없습니다.
// length 속성의 형식이 호환되지 않습니다.
// 3 형식은 2 형식에 할당할 수 없습니다.
```
오류가 발생했는데 그 이유가 매우 흥미롭다. TS는 숫자의 쌍을 {0: number, 1: number} 로 모델링하지 않고 {0: number, 1: number, length: 2}로 모델링했다. 그래서 length의 값이 맞지 않기 때문에 할당문에 오류가 발생했다. 쌍에서 길이를 체크하는 것은 합리적이다.<br>
타입이 값의 집합이라는 건, 동일한 값의 집합을 가지는 두 타입은 같다는 의미가 된다. 두 타입이 의미적으로 다르고 우연히 같은 범위를 가진다고 하더라도, 같은 타입을 두 번 정의할 이유는 없다.

한편 TS 타입이 되지 못하는 값의 집합들이 있다는 것을 기억해야 한다. 정수에 대한 타입, 또는 x 와 y 속성 외에 다른 속성이 없는 객체는 TS 타입에 존재하지 않는다. 가끔 `Exclude`를 사용해서 일부 타입을 제외할 수 이는 있지만, 그 결과가 적절한 TS타입일 때만 유효하다.

``` ts
type T = Exclude<string|Date, string|number>; // 타입은 Date
type NonZeroNums = Exclude<number, 0>; // 타입은 여전히 number
```

### 타입스크립트 용어 및 집합 용어
- `never` : 공집합
- `리터럴 타입` : 원소가 1개인 집합
- `값이 T에 할당 가능` : 값이 T의 원소
- `T1이 T2에 할당 가능` : T1이 T2의 부분 집합
- `T1이 T2를 상속` : T1이 T2의 부분 집합
- `T1 | T2(유니온)` : T1과 T2의 합집합
- `T1 & T2(인터섹션)` : T1과 T2의 교집합
- `unknown` : 전체(universal)집합

## 아이템 8: 타입 공간과 값 공간의 심벌 구분하기

### TS 코드를 읽을 때 타입인지 값인지 구분하는 방법을 터득해야 한다.
TS의 심벌(Symbol)은 타입 공간이나 값 공간 중의 한 곳에 존재한다. 심벌은 이름이 같더라도 속하는 공간에 따라 다른 값을 나타낼 수 있기 때문에 혼란스러울 수 있다.
``` ts
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });
```

`interface Cylinder`에서 `Cylinder`는 타입으로 쓰인다. `const Cylinder`에서 `Cylinder`와 이름은 같지만 값으로 쓰이며, 서로 아무런 관련도 없다. 상황에 따라서 `Cylinder`는 타입으로 쓰일 수도 있고, 값으로 쓰일 수도 있다. 이런 점이 가끔 오류를 야기한다.
``` ts
function calculateVolume(shape: unknown) {
  if(shape instanceof Cylinder) {
    shape.raidus // {}형식에 radius 속성이 없습니다.
  }
}
```
오류를 살펴보면, 아마도 `instanceof`를 이용해 `shape`가 `Cylinder`타입인지 체크하려고 했을 것이다. 그러나 `instanceof`는 JS의 런타임 연산자이고, 값에 대해서 연산을 한다. 그래서 `instanceof Cylinder`는 타입이 아니라 함수를 참조한다.<br>
이처럼 한 심벌이 타입인지 값인지는 언뜻 봐서 알 수 없다. 어떤 형태로 쓰이는지 문맥을 살펴 알아내야 한다.


### 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다.
type과 interface같은 키워드는 타입 공간에만 존재한다.

``` ts
type T1 = 'string literal';
type T2 = 123;
const v1 = 'string literal';
const v2 = 123;
```
일반적으로 type이나 interface다음에 나오는 심벌은 타입인 반면, const나 let 선언에 쓰이는 것은 값이다. **_컴파일 과정에서 타입 정보는 제거되기 때문에, 심벌이 사라진다면 그것은 타입에 해당된다._**

TS 코드에서 타입과 값은 번갈아 나올 수 있다. 타입 선언(:) 또는 단언문(as) 다음에 나오는 심벌은 타입인 반면, =다음에 나오는 모든 것은 값이다.
``` ts
interface Person {
  first: string;
  last: string;
}

cosnt p: Person = { first: 'Jane', last: 'Jacobs' }; 
//    -           --------------------------------- 값
//       ------                                     타입

function email(p: Person, subject: string, body: string): Response {
//       ----- -          -------          ----                      값
//                ------           ------        ------   --------   타입 
}
```

### class와 enum 같은 키워드는 타입과 값 두 가지로 사용될 수 있다.
`class`와 `enum`은 상황에 따라 타입과 값 두 가지 모두 가능한 예약어이다. 아래 예제에서 Cylinder클래스는 타입으로 쓰였다.

``` ts
class Cylinder {
  radius=1;
  height=1;
}

function calcuateVOlume(shape: unknown) {
  if(shape instanceof Cylinder) {
    shape // 정상, 타입은 Cylinder
    shape.radius // 정상, 타입은 number
  }
}
```
클래스가 타입으로 쓰일 때는 `형태(속성과 메소드)`가 사용되는 반면, 값으로 쓰일 때는 `생성자`가 사용된다. 한편, 연산자 중에서도 타입에서 쓰일 때와 값에서 쓰일 때 다른 기능을 하는 것들이 있다. 그 예 중 하나로 `typeof`를 들 수 있다.
``` ts
type T1 = typeof p; // 타입은 Person
type T2 = typeof email;
// 타입은 (p: Person, subject: string, body: string) => Response
const v1 = typeof p; // 값은 'object'
const v2 = typeof email; // 값은 'function'
```
`타입의 관점`에서, `typeof`는 값을 읽어서 TS타입을 반환한다. `타입 공간`의 `typeof`는 보다 큰 타입의 일부분으로 사용할 수 있고, `type구문`으로 이름을 붙이는 용도로도 사용할 수 있다.<br>
반면 `값의 관점`에서 `typeof`는 `JS런타임의 typeof연산자`가 된다. `값 공간의 typeof`는 대상 심벌의 런타임 타입을 가리키는 문자열을 반환하며, TS 타입과는 다르다. JS의 런타임 타입 시스템은 TS의 정적 타입 시스템보다 훨씬 간단하다. TS타입의 종류가 무수히 많은 반면, JS에는 과거부터 지금까지 단 6개`(string, number, boolean, undefined, object, function)`의 런타임 타입만이 존재한다.

`Cylinder`예제에서 본 것처럼 `class` 키워드는 값과 타입 두 가지로 모두 사용된다. 따라서 클래스에 대한 `typeof`는 상황에 따라 다르게 동작한다.
``` ts
const v = typeof Cylinder; // 값이 'function'
type T = typeof Cylinder; // 타입이 typeof Cylinder
```
클래스가 JS에서는 실제 함수로 구현되기 때문에 첫 번째 줄의 값은 'function'이 된다. 두 번째 줄에서 중요한 것은 `Cylinder`가 인스턴스의 타입이 아니라는 점이다. 실제로는 new키워드를 사용할 때 볼 수 있는 생성자 함수이다.
``` ts
declare let fn: T;
const c = new fn(); // 타입이 Cylinder
```

`InstanceType`제네릭을 사용해 생성자 타입과 인스턴스 타입을 전환할 수 있다.
``` ts
type C = InstanceType<typeof Cylinder>; // 타입이 Cylinder
```

속성 접근자 []는 타입으로 쓰일 때에도 동일하게 동작한다. 그러나 `obj['field']`와 `obj.field`는 값이 동일하더라도 타입은 다를 수 있다. 따라서 타입의 속성을 얻을 때에는 반드시 첫 번째 방법(`obj['field']`)을 사용해야 한다.

인덱스 위치에는 유니온 타입과 기본형 타입을 포함한 어떠한 타입이든 사용할 수 있다.
``` ts
type PersonEl = Person['first' | 'last']; // 타입은 string
type Tuple = [string, number, Date];
type TupleEl = Tuple[number]; // 타입은 string | number | Date
```

### 두 공간 사이에서 다른 의미를 가지는 코드 패턴들이 있다.
- 값으로 쓰이는 `this`는 JS의 `this키워드`이다. 타입으로 쓰이는 `this`는, 일명 `다형성(polymorphic) this`라고 불리는 `this의 TS타입`이다. 서브클래스의 메서드 체인을 구현할 때 유용하다.
- 값에서 &와 |는 AND와 OR 비트연산이다. 타입에서는 인터섹션과 유니온이다.
- `const`는 새 변수를 선언하지만, `as const`는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꾼다.
- `extends`는 `서브클래스(class A extends B)`또는 `서브타입(interface A extends B)`또는 제네릭 타입의 한정자`(Generic<T extends number>)`를 정의할 수 있다.
- `in`은 `루프(for(key in object))`또는 `매핑된(mapped)타입`에 등장한다.

TS코드가 잘 동작하지 않는다면 타입 공간과 값 공간을 혼동해서 잘못 작성했을 가능성이 크다. 예를 들어, 단익 객체 매개변수를 받도록 email함수를 변경했다고 생각해 본다.

``` ts
function email(options: {person: Person, subject: string, body: string}) {
  //...
}

// JS에서는 구조분해 할당을 사용할 수 있다.
function email({person, subject, body}) {
  //...
}

// 그런데 TS에서 구조분해 할당을 하면 오류가 발생한다.
function email({
  person: Person, // 바인딩 요소 'Person'에 암시적으로 'any'형식이 있습니다.
  subject: string, // string식별자가 중복되었습니다. 바인딩 요소 'string'에 암시적으로 'any'형식이 있습니다.
  body: string // string식별자가 중복되었습니다. 바인딩 요소 'string'에 암시적으로 'any'형식이 있습니다.
})
```

값의 관점에서 Person과 string이 해석되었기 때문에 오류가 발생했다. Person이라는 변수명과 string이라는 이름을 가지는 두 개의 변수를 생성하려 한 것이다.
``` ts
function email(
  {person, object, body}: {person: Person, subject: string, body: string}
) {
  // ...
}
```

## 아이템 9: 타입 단언보다는 타입 선언을 사용하기
TS에서 변수에 값을 할당하고 타입을 부여하는 방법은 두 가지이다.
``` ts
interface Person { name: string };

const alice: Person = { name: 'Alice' }; // 타입은 Person
const bob = { name: 'Bob' } as Person; // 타입은 Person
```

이 두 가지 방법은 결과가 같아 보이지만 그렇지 않다.<br>
첫 번째 `alice: Person`은 변수에 `타입 선언`을 붙여서 그 값이 선언된 타입임을 명시<br>
두 번째 `as Person`은 `타입 단언`을 수행. 그러면 TS가 추론한 타입이 있더라도 `Person`타입으로 간주.

### 타입 단언보다 타입 선언을 사용하는 게 좋은 이유
``` ts
const alice: Person = {}; // Person 유형에 필요한 name속성이 {} 유형에 없습니다.
const bob = {} as Person; // 오류 없음
```
타입 선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사한다. 앞의 예제에서는 그러지 못했기 때문에 TS가 오류를 표시했다. 타입 단언은 강제로 타입을 지정햇으니 타입 체커에게 오류를 무시하라고 하는 것이다.

타입 선언과 단언의 차이는 속성을 추가할 때도 마찬가지이다.
``` ts
const alice: Person = {
  name: 'Alice',
  occupation: 'TypeScript developer' // 개체 리터럴은 알려진 속성만 지정할 수 있으며 Person 형식에 occupation이 없습니다.
}
const bob = {
  name: 'Bob',
  occupation: 'JavaScript developer'
} as Person; // 오류 없음
```
타입 선언문에서는 잉여 속성 체크가 동작했지만, 단언문에서는 적용되지 않는다. 따라서 타입 단언이 꼭 필요한 경우가 아니라면, 안전성 체크도 되는 타입 선언을 사용하는 것이 좋다.

### 화살표 함수의 반환 타입을 명시하는 방법을 터득해야 한다.
화살표 함수의 타입 선언은 추론된 타입이 모호할 때가 있다. 예를 들어, 아래 코드에서 `Person` 인터페이스를 사용하고 싶다고 가정해 본다.
``` ts
const people = ['alice', 'bob', 'jan'].map(name => ({ name }));
// Person[]을 원했지만 결과는 { name: string; }[]...
```

`{ name }`에 타입 단언을 쓰면 문제가 해결되는 것처럼 보인다.
``` ts
const people = ['alice', 'bob', 'jan'].map(
  name => ({ name }) as Person);
// Person[]
```

그러나 타입 단언을 사용하면 런타임에 문제가 발생하게 된다.
``` ts
const people = ['alice', 'bob', 'jan'].map(name => ({} as Person)); // 오류 없음
```

단언문을 쓰지 않고, 다음과 같이 화살표 함수 안에서 타입과 함께 변수를 선언하는 것이 가장 직관적이다.
``` ts
const people = ['alice', 'bob', 'jan'].map(name => {
  const person: Person = { name };
  return person
}); // Person[]
```

코드 리팩토링
``` ts
const people = ['alice', 'bob', 'jan'].map(
  (name): Person => ({ name })
); // Person[]
```
이 코드는 바로 앞의 코드와 동일한 체크를 수행한다. **여기서 소괄호는 매우 중요한 의미를 지닌다.** `(name): Person`은 `name`의 타입이 없고, 반환 타입이 `Person`이라고 명시한다. 그러나 `(name: Person)`은 `name`의 타입이 `Person`임을 명시하고 반환 타입은 없기 때문에 오류가 발생한다.

``` ts
const people: Person[] = ['alice', 'bob', 'jan'].map(
  (name): Person => ({ name })
); // Person[]
```
이 코드는 최종적으로 원하는 타입을 직접 명시하고, TS가 할당문의 유효성을 검사하게 한다.

but 함수 호출 체이닝이 연속되는 곳에서는 `체이닝 시작에서부터 명명된 타입을 가져야` 한다. 그래야 정확한 곳에 오류가 표시된다.

### TS보다 타입 정보를 더 잘 알고 있는 상황에서 타입 단언문과 null 아님 단언문을 사용하면 된다.
`타입 단언이 꼭 필요한 경우`를 살펴본다. 타입 단언은 타입 체커가 추론한 타입보다 우리가 판단하는 타입이 더 정확할 때 의미가 있다. 예를 들어, DOM 엘리먼트에 대해서는 TS보다 우리가 더 정확히 알고 있을 것이다.
``` ts
document.querySelector('#myButton').addEventListener('click', e => {
  e.currentTarget // 타입은 EventTarget
  const button = e.currentTarget as HTMLButtonElement;
  button // 타입은 HTMLButtonElement
})
```
TS는 DOM에 접근할 수 없기 때문에 `#myButton`이 버튼 엘리먼트인지 알지 못한다. 그리고 이벤트의 `currentTarget`이 같은 버튼이어야 하는 것도 알지 못한다. 우리는 TS가 알지 못하는 정보를 가지고 있기 때문에 여기서는 타입 단언문을 쓰는 것이 타당하다.

또한, 자주 쓰이는 특별한 문법(!)을 사용해서 `null이 아님을 단언`하는 경우도 있다.
``` ts
const elNull = document.getElementById('foo'); // 타입은 HTMLElement | null
const el = document.getElementById('foo')!; // 타입은 HTMLElement
```

변수의 접두사로 쓰인 !는 boolean의 부정문이다. 그러나 접미사로 쓰인 !는 그 값이 null이 아니라는 단언문으로 해석된다. 우리는 !를 일반적인 단언문처럼 생각해야 한다.<br>
단언문은 컴파일 과정 중에 제거되므로, 타입 체커는 알지 못하지만 그 값이 null이 아니라고 확신할 수 있을 때 사용해야 한다. 만약 그렇지 않다면, null인 경우를 체크하는 조건문을 사용해야 한다.

타입 단언문으로 임의의 타입 간에 변환을 할 수는 없다. A가 B의 부분집합인 경우에 타입 단언문을 사용해 변환할 수 있다.<br>
`HTMLElement`는 `HTMLElement | null`의 서브타입이기 때문에 이러한 타입 단언은 동작한다. `HTMLButtonElement`는 `EventTarget`의 서브타입이기 때문에 역시 동작한다. 그리고 `Person`은 `{}의 서브타입`이므로 동작한다.<br>
그러나 `Person`과 `HTMLElement`는 서로의 서브타입이 아니기 때문에 변환이 불가능하다.
``` ts
interface Person { name: string; }
const body = document.body;
const el = body as Person;
           // HTMLElement 형식을 Person형식으로 변환하는 것은
           // 형식이 다른 형식과 충분히 겹치지 않기 때문에
           // 실수일 수 있습니다. 이것이 의도적인 경우에는
           // 먼저 식을 unknown으로 변환하십시오.
```

이 오류를 해결하려면 `unknown`타입을 사용해야 한다. 모든 타입은 `unknown`의 서브타입이기 때문에 `unknown`이 포함된 단언문은 항상 동작한다.<br>
`unknown`단언은 임의의 타입 간에 변환을 가능케 하지만, `unknown`을 사용한 이상 적어도 무언가 위험한 동작을 하고 있다는 걸 알 수 있다.
``` ts
const body = document.body as unknown as Person; // 정상
```