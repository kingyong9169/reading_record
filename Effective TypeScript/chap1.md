# 1장 타입스크립트 알아보기

## 타입스크립트는 사용 방식 면에서 독특한 언어
- 인터프리터(파이썬, 루비)로 실행되는 것도 X
- 저수준 언어로 컴파일(Java, C) X
- JS로 컴파일, 실행 역시 JS로 이루어짐.

따라서 JS와 필연적 관계로 혼란스러운 일이 벌어지기도 한다.

## 아이템 1: TS와 JS의 관계 이해하기

### TS는 문법적으로 JS의 상위집합(superset)
JS 프로그램에 문법 오류가 없다면, 유효한 TS 프로그램이라고 할 수 있다. but JS 프로그램에 어떤 이슈가 존재한다면 **문법 오류가 아니라도 타입 체커에게 지적당할 가능성 O**

but 문법의 유효성과 동작의 이슈는 독립적인 문제! TS는 여전히 작성된 코드를 파싱하고 JS로 변환가능.<br>
JS 파일이 .js(.jsx) 확장자를 사용하는 반면, TS는 .ts(.tsx) 확장자를 사용하지만 `JS와 TS는 완전히 다른 언어라는 의미 X`. TS는 JS의 superset이기 때문에 .js 파일에 있는 코드는 이미 TS라고 할 수 있다.

이러한 특성은 기존에 존재하는 JS 코드를 TS코드로 마이그레이션하는 데 엄청난 이점! 기존 코드를 그대로 유지하면서 일부분에만 TS 적용이 가능하기 때문.

**모든 JS 프로그램이 TS라는 명제는 O but 반대는 X.**
TS프로그램이지만 JS가 아닌 프로그램이 존재. 이는 TS가 타입을 명시하는 추가적인 문법을 가지기 때문.

### 타입 시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것
정적 타입 시스템이라는 것은 바로 이런 특징! but 타입 체커가 모든 오류를 찾아내지는 않는다.

오류가 발생하지는 않지만 의도와 다르게 동작하는 코드도 존재. TS는 이러한 문제 중 몇 가지를 찾아내기도 한다.

``` js
const states = { name, capital };
console.log(states.capitol);
// undefined
```
JS코드이며 어떠한 오류도 없이 실행된다. but `states.capitol`은 의도한 코드가 아닌게 분명하다. 이런 경우에 TS 타입 체커는 **추가적인 타입 구문 없이도 오류를 찾아낸다.(또한, 훌륭한 해결책을 제시.)**

TS는 타입 구문 없이도 오류를 잡을 수 있지만, _**타입 구문을 추가한다면 훨씬 더 많은 오류를 찾아낼 수 있다.**_ 코드의 `의도`가 무엇인지 타입 구문을 통해 TS에게 알려줄 수 있기 때문에 코드의 동작과 의도가 다른 부분을 찾을 수 있다.

여기서 capital과 capitol을 바꾸면 `capital 속성이 ...없습니다. capitol을 사용하시겠습니까?`와 같은 해결책을 제시한다. 하지만 해결책은 잘못되었다. **이처럼 TS는 어느 쪽이 오타인지 판단하지 못한다.** 오류의 원인을 추측할 수는 있겠지만 항상 정확하지는 않다. **따라서 명시적으로 states를 선언하여 의도를 분명하게 하는 것이 좋다.**

``` ts
interface States {
  name: string;
  capital: string;
}
```
이제 오류가 어디에서 발생했는지 찾을 수 있고, 제시된 해결책도 올바르다. 의도를 명확히 해서 TS가 잠재적 문제점을 찾을 수 있게 했다.

지금까지 내용을 정리하여 `TS는 JS의 superset이다.`라는 문장이 잘못된 것처럼 느껴진다면 `타입 체커를 통과한 TS 프로그램` 영역 때문일 것이다. 평소 작성하는 TS코드가 바로 이 영역에 해당한다. 보통은 타입 체크에서 오류가 발생하지 않도록 신경을 쓰며 TS 코드를 작성하기 때문이다.

### TS 타입 시스템은 JS 런타임 동작을 모델링한다.
``` js
const x = 2 + '3';
const y = '2' + 3;
```

이 예제는 다른 언어였다면 런타임 오류가 될 만한 코드이다. 하지만 TS의 타입 체커는 정상으로 인식한다. 두 줄 모두 문자열 "23"이 되는 JS 런타임 동작으로 모델링된다.

반대로 정상 동작하는 JS 코드에 오류를 표시하기도 한다. 런타임 오류가 발생하지 않는 코드인데, 타입 체커는 문제점을 표시한다.
``` js
const a = null + 7; // JS에서는 7
const b = [] + 12; // JS에서는 '12'
alert('Hello', 'TypeScript') // JS에서는 "Hello"표시
```

JS이 런타임 동작을 모델링하는 것은 TS 타입 시스템의 기본 원칙이다. 그러나 앞에서 봤던 경우들처럼 단순히 런타임 동작을 모델링하는 것뿐만 아니라 의도치 않은 이상한 오류로 이어질 수도 있다는 점까지 고려해야 한다.`(capital, capitol 예제)`

언제 JS 런타임 동작을 그대로 모델링할지, 또는 추가적인 타입 체크를 할지 분명하지 않다면 과연 TS를 사용해도 되는지 의문이 들 수 있다. TS 채택 여부는 사용자의 선택에 달려있지만 TS의 도움을 받으면 오류가 적은 코드를 작성할 수 있다.


## 아이템 2: TS 설정 이해하기
다음 코드가 오류 없이 타입 체커를 통과할 수 있을지 생각해 보겠습니다.
``` ts
function add(a, b) {
  return a + b;
}
add(10, null);
```

설정이 어떻게 되어 있는지 모른다면 대답할 수 없는 질문입니다. TS컴파일러는 매우 많은 설정을 갖고 있습니다. 현재 시점에서는 설정이 거의 100개에 이릅니다.<br>
이 설정들은 커맨드 라인에서 사용할 수 있습니다.<br>
`$ tsc --noImplicitAny program.ts`

`tsconfig.json` 설정 파일을 통해서도 가능합니다.
``` json
{
  "compilerOptions": {
    "noImplicitAny": true
  }
}
```
**가급적 설정 파일을 사용하는 것이 좋다.** 그래야만 TS를 어떻게 사용할 계획인지 동료들이나 다른 도구들이 알 수 있다. 설정 파일은 `tsc --init`만 실행하며 간단히 생성된다.

TS의 설정들은 어디서 소스 파일을 찾을지, 어떤 종류의 출력을 생성할지 제어하는 내용이 대부분. **그런데 언어 자체의 핵심 요소들을 제어하는 설정도 있다. 대부분의 언어에서는 허용하지 않는 고수준 설계의 설정이다.** 설정을 제대로 사용하려면, `noImplicitAny`와 `strictNullChecks`를 이해해야 한다.<br>

### noImplicitAny
`noImplicitAny`는 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어한다. 위의 `add`코드는 `noImplicitAny`이 `false`일 때 유효하다.<br>
이 함수의 타입은 `function add(a: any, b: any): any`이다.<br>
any 타입을 매개변수에 사용하면 타입 체커는 속절없이 무력해진다. any는 유용하지만 매우 주의해서 사용해야 한다.

그런데 같은 코드임에도 `noImplicitAny`가 설정되었다면 오류가 된다. _이 오류들은 명시적으로 `: any`라고 선언해 주거나 더 분명한 타입을 사용하면 해결할 수 있다._
``` ts
function add(a: number, b: number) {
  return a + b;
}
```

TS는 타입 정보를 가질 때 가장 효과적이기 때문에, 되도록이면 `noImplicitAny`를 설정해야 한다. 그러면 TS가 문제를 발견하기 수월해지고, 코드의 가독성이 좋아지며, 개발자의 생산성이 향상된다. **참고로 `noImplicitAny` 해제는 JS로 되어 있는 기존 코드를 TS로 마이그레이션할 때에만 필요하다.**

### strictNullChecks
null과 undefined가 모든 타입에서 허용되는지 확인하는 설정이다.
`const x: number = null;`
- false일 때
  - 정상
- true일 때
  - null형식은 number형식에 할당할 수 없습니다.

null대신 undefined를 써도 같은 오류가 난다. 만약 null을 허용하려고 한다면 의도를 명시적으로 드러냄으로써 오류를 고칠 수 있다.<br>
`const x: number | null = null;`
만약 null을 허용하지 않으려면, 이 값이 어디서부터 왔는지 찾아야 하고, null을 체크하는 코드나 단언문을 추가해야 한다.

``` ts
const el = document.getElementById('status');
// el.textContent = 'Ready' => null인 것 같습니다.
if(el) el.textContent = 'Ready'; // null 제외
el!.textContent = 'Ready' // el이 null이 아님을 단언한다.
```

`strictNullChecks`는 nullrhk undefined 관련된 오류를 잡아 내는 데 많은 도움이 되지만, 코드 작성을 어렵게 한다. `strictNullChecks`를 설정하려면 `noImplicitAny`를 먼저 설정해야 한다.<br>
`strictNullChecks` 설정 없이 개발하기로 선택했다면 `undefined는 객체가 아닙니다.`라는 끔찍한 런타임 오류를 주의해야 한다. 프로젝트가 거대해질수록 설절 반경은 어려워질 것이므로, 가능한 한 초반에 설정하는 게 좋다.<br>
만약 언어에 의미적으로 영향을 미치는 설정들을(noImplicitThis, strictFunctionTypes 등) 체크하고 싶다면 `strict`설정을 하면 된다. 대부분의 오류를 잡아내줄 것이다.

## 아이템 3: 코드 생성과 타입이 관계없음을 이해하기
TS 컴파일러는 두 가지 역할을 수행한다.
- 최신 TS/JS를 브라우저에서 동작할 수 있도록 구버전의 JS로 트랜스파일한다.
- 코드의 타입 오류를 체크한다.

> **_트랜스파일?_**<br>
번역과 컴파일이 합쳐져 트랜스파일이라는 신조어가 탄생. 소스코드를 동일한 동작을 하는 다른 형태의 소스코드(다른 버전, 다른 언어 등)로 변환하는 행위를 의미한다. 결과물이 여전히 컴파일되어야 하는 소스코드이기 때문에 컴파일과는 구분해서 부른다.

여기서 놀라운 점은 `이 두 가지가 서로 완벽히 독립적`이라는 것이다. TS가 JS로 변환될 때 코드 내의 타입에는 영향을 주지 않는다. 또한 그 JS의 실행 시점에도 타입은 영향을 미치지 않는다.

TS컴파일러가 수행하는 두 가지 역할을 되짚어 보면, TS가 할 수 있는 일과 할 수 없는 일을 짐작할 수 있다.

### 타입 오류가 있는 코드도 컴파일 가능하다.
컴파일은 타입 체크와 독립적으로 동작하기 때문에, 타입 오류가 있는 코드도 컴파일 가능하다.

타입 체크와 컴파일이 동시에 이루어지는 C나 자바 같은 언어를 사용하던 사람이라면 이러한 상황이 매우 황당하게 느껴질 것이다. TS 오류는 C나 자바 같은 언어들의 경고와 비슷하다. 문제가 될 만한 부분을 알려주지만, 그렇다고 빌드를 멈추지는 않는다.

> **_컴파일과 타입 체크_**<br>
코드에 오류가 있을 때 "컴파일에 문제가 있다"고 말하는 경우를 보았을 것이다. 그러나 이는 기술적으로 틀린 말이다. 엄밀히 말하면 오직 코드 생성만이 "컴파일"이라고 할 수 있기 때문이다. 작성한 TS가 유효한 JS라면 TS 컴파일러는 컴파일을 해 낸다. 그러므로 코드에 오류가 있을 때 "타입 체크에 문제가 있다"고 말하는 것이 더 정확한 표현이다.

타입 오류가 있는 데도 컴파일된다는 사실 때문에 TS가 엉성한 언어처럼 보일 수 있다. 하지만 코드에 오류가 있더라도 컴파일된 산출물이 나오는 것이 실제로 도움이 된다. 웹 애플리케이션을 만들면서 어떤 부분에 문제가 발생했다고 가정한다. TS는 여전히 컴파일된 산출물을 생성하기 때문에, 문제가 된 오류를 수정하지 않더라도 애플리케이션의 다른 부분을 테스트할 수 있다.<br>
만약 오류가 있을 때 컴파일하지 않으려면, `tsconfig.json`에 `noEmitOnError`를 설정하거나 빌드 도구에 동일하게 적용하면 된다.

### 런타임에는 타입 체크가 불가능하다.
``` ts
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if(shape instanceof Rectangle) { // Rectangle은 형식만 참조하지만, 여기서는 값으로 사용되고 있다.
    return shape.width * shape.height; // Shape 형식에 height 속성이 없다.
  } else {
    return shape.width * shape.width;
  }
}
```

instanceof 체크는 런타임에 일어나지만, **Rectangle은 타입이기 때문에 런타임 시점에 아무런 역할을 할 수 없습니다.** TS의 타입은 "제거 가능"하다. 실제로 JS로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거되어 버린다.<br>
앞의 코드에서 다루고 있는 shape타입을 명확하게 하려면, 런타임에 타입 정보를 유지하는 방법이 필요하다. 하나의 방법은 height 속성이 존재하는지 체크해 보는 것이다.

``` ts
function calculateArea(shape: Shape) {
  if('height' in shape) {
    shape; // 타입이 Rectangle
    return shape.width * shape.height;
  } else {
    shape; // 타입이 Square
    return shape.width * shape.width;
  }
}
```

속성 체크는 런타임에 접근 가능한 값에만 관련되지만, 타입 체커 역시도 shape의 타입을 Rectangle로 보정해 주기 때문에 오류가 사라진다.<br>
타입 정보를 유지하는 또 다른 방법으로는 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 "태그"기법이 있다.

``` ts
interface Square {
  kind: 'square';
  width: number;
}
interface Rectangle extends Square {
  kind: 'rectangle';
  height: number;
  width: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if(shape.kind === 'rectangle') {
    shape; // 타입이 Rectangle
    return shape.width * shape.height;
  } else {
    shape; // 타입이 Square
    return shape.width * shape.width;
  }
}
```

여기서 Shape의 타입은 "태그된 유니온"의 한 예다. 이 기법은 런타임에 타입 정보를 손쉽게 유지할 수 있기 때문에, TS에서 흔하게 볼 수 있다.<br>
**타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법도 있다.** 타입을 클래스로 만들면 된다. Square와 Rectangle을 클래스로 만들면 오류를 해결할 수 있다.

``` ts
class Square {
  constructor(public width: number) {}
}
class Rectangle extends Square {
  constructor(public width: number, public height: number) {
    super(width);
  }
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if(shape instanceof Rectangle) {
    shape; // 타입이 Rectangle
    return shape.width * shape.height;
  } else {
    shape; // 타입이 Square
    return shape.width * shape.width; // 정상
  }
}
```

인터페이스는 타입으로만 사용 가능하지만, Rectangle을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없습니다.<br>
`type Shape = Square | Rectangle;`에서 Rectangle은 타입으로 참조되지만,<br>
`shape instanceof Rectangle`에서는 값으로 참조된다.

### 타입 연산은 런타임에 영향을 주지 않는다.
string 또는 number 타입인 값을 항상 Number로 정제하는 경우를 가정한다. 다음 코드는 타입 체커를 통과하지만 잘못된 방법을 썼다.
``` ts
function asNumber(val: number | string): number {
  return val as number;
}
```

변환된 JS를 보면 이 함수가 실제로 어떻게 동작하는지 알 수 있다.
``` ts
function asNumber(val) {
  return val;
}
```
코드에 아무런 정제 과정이 없습니다. as number는 타입 연산(타입 단언문)이고 런타임 동작에는 아무런 영향을 미치지 않는다. 값을 정제하기 위해서는 런타임의 타입을 체크해야 하고 JS 연산을 통해 변환을 수행해야 한다.

``` ts
function asNumber(val: number | string): number {
  return typeof(val) === 'string' ? Number(val) : val;
}
```

### 런타임 타입은 선언된 타입과 다를 수 있다.
다음 함수를 보고 마지막의 console.log까지 실행될 수 있을지 생각해보자.
``` ts
function setLightSwitch(value: boolean) {
  switch(value) {
    case true:
      turnLightOn();
      break;
    case false:
      turnLightOff();
      break;
    default:
      console.log("실행되지 않을까 봐 걱정됩니다.");
  }
}
```
TS는 일반적으로 실행되지 못하는 죽은(dead) 코드를 찾아내지만, 여기서는 strict를 설정하더라도 찾아내지 못한다. 그러면 마지막 부분을 실행할 수 있는 경우는 무엇일까?<br>
`: boolean`이 타입 선언문이라는 것에 주목해보자. TS의 타입이기 때문에 런타임에 제거된다. JS였다면 실수로 함수를 "ON"으로 호출할 수도 있었다.<br>
순수 TS에서도 마지막 코드를 실행하는 방법이 존재한다. 예를 들어, 네트워크 호출로부터 받아온 값으로 함수를 실행하는 경우가 있다.

``` ts
interface LightApiResponse {
  lightSwitchValue: boolean;
}
async function setLight() {
  const response = await fetch("/light");
  const result: LightApiResponse = await response.json();
  setLightSwitch(result.lightSwitchValue);
}
```

`/light`를 요청하면 그 결과로 반환하라고 선언했지만, 실제로 그렇게 되리라는 보장은 없다. `API`를 잘못 파악해서 `lightSwitchValue`가 실제로 문자열이었다면, 런타임에는 `setLightSwitch`함수까지 전달될 것이다.<br>
TS에서는 런타임 타입과 선언된 타입이 맞지 않을 수 있다. 타입이 달라지는 혼란스러운 상황을 가능한 한 피해야 한다. 선언되 타입이 언제든지 달라질 수 있다는 것을 명심해야 한다.

### TS타입으로는 함수를 오버로드할 수 없다.
C++같은 언어는 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용한다. 이를 "함수 오버로딩"이라고 한다. 그러나 TS에서는 **타입과 런타임의 동작이 무관하기 때문에, 함수 오버로딩은 불가능하다.**<br>
TS가 함수 오버로딩 기능을 지원하기는 하지만, 온전히 타입 수준에서만 동작한다. 하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만, 구현체는 오직 하나다.
``` ts
function add(a: number, b: number): number;
function add(a: string, b: string): string;
function add(a, b) {
  return a + b;
}

const three = add(1, 2); // number
const twelve = add('1', '2'); // string
```

add에 대한 처음 두 개의 선언문은 타입 정보를 제공할 뿐이다. 이 두 선언문은 TS가 JS로 변환되면서 제거되며, 구현체만 남게 된다.

### TS 타입은 런타임 성능에 영향을 주지 않는다.
타입과 타입 연산자는 JS 변환 시점에 제거되기 때문에, 런타임의 성능에 아무런 영향을 주지 않는다. TS의 정적 타입은 실제로 비용이 전혀 들지 않는다. TS를 쓰는 대신 런타임 오버헤드를 감수하며 타입 체크를 해 본다면, TS 팀이 다음 주의사항들을 얼마나 잘 테스트해 왔는지 몸소 느낄 수 있다.

- 런타임 오버헤드가 없는 대신, `빌드타임 오버헤드`가 있다. 컴파일러 성능을 매우 중요하게 생각하여 컴파일은 일반적으로 상당히 빠른 편이며 특히 증분 빌드 시에 더욱 체감된다. 오버헤드가 커지면 빌드 도구에서 `트랜스타일만(transpile only)`을 설정하여 타입 체크를 건너뛸 수 있다.

- TS가 컴파일하는 코드는 오래된 런타임 환경을 지원하기 위해 호환성을 높이고 성능 오버헤드를 감안할지, 호환성을 포기하고 성능 중심의 네이티브 구현테를 선택할지의 문제에 맞닥뜨릴 수도 있다. 예를 들어 제너레이터 함수가 ES5 타깃으로 컴파일되려면, TS 컴파일러는 호환성을 위한 특정 헬퍼 코드를 추가할 것이다. 이런 경우가 제너레이터의 호환성을 위한 오버헤드 또는 성능을 위한 네이티브 구현체 선택의 문제이다. 어떤 경우든지 호환성과 성능 사이의 선택은 컴파일 타깃과 언어 레벨의 문제이며 여전히 타입과는 무관하다.