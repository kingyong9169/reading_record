# 아이템 8: 타입 공간과 값 공간의 심벌 구분하기

## TS 코드를 읽을 때 타입인지 값인지 구분하는 방법을 터득해야 한다.
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


## 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다.
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

## class와 enum 같은 키워드는 타입과 값 두 가지로 사용될 수 있다.
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

## 두 공간 사이에서 다른 의미를 가지는 코드 패턴들이 있다.
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
