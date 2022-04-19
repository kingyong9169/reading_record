# 아이템 12: 함수 표현식에 타입 적용하기

## 매개변수나 반환 값에 타입을 명시하는 것보다 함수 표현식 전체에 타입 구문을 적용하는 것이 좋습니다.

JS(그리고 TS)에서는 함수 문장(statement)과 함수 표현식(expression)을 다르게 인식합니다.
``` ts
function rollDice1(sides: number): number {} // 문장
const rollDice2 = function(sides: number): number {}; // 표현식
const rollDice3 = function(sides: number): number => {}; // 표현식
```

TS에서는 함수 표현식을 사용하는 것이 좋습니다. 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다는 장점이 있기 때문입니다.
``` ts
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = sides => {};
```
편집기에서 `sides`에 마우스를 올려 보면, TS에서는 이미 sides의 타입을 number로 인식하고 있다는 걸 알 수 있습니다.

함수 타입의 선언은 불필요한 코드의 반복을 줄입니다.
``` ts
function add(a: number, b: number) {}
function sub(a: number, b: number) {}
function mul(a: number, b: number) {}
function div(a: number, b: number) {}
```

반복되는 함수 시그니처를 하나의 함수 타입으로 통합할 수도 있습니다.
``` ts
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

이 예제는 함수 타입 선언을 이용했던 예제보다 타입 구문이 적습니다. 함수 구현부도 분리되어 있어 로직이 보다 분명해집니다.<br>
라이브러리는 공통 함수 시그니처를 타입으로 제공하기도 합니다. 예를 들어, react는 함수의 매개변수에 명시하는 `MouseEvent`타입 대신에 함수 전체에 적용할 수 있는 `MouseEventHandler`타입을 제공합니다. 만약 라이브러리를 직접 만들고 있다면, 공통 콜백 함수를 위한 타입 선언을 제공하는 것이 좋습니다.<br>
시그니처가 일치하는 다른 함수가 있을 때도 함수 표현식에 타입을 적용해볼 만합니다. 예를 들어, 웹브라우저에서 `fetch`함수는 특정 리소스에 `HTTP`요청을 보냅니다.

``` ts
const responseP = fetch('/quote?by=Mark+Twain'); // 타입이 Promise<Response>

async function getQuote() {
  const response = await fetch('/quote?by=Mark+Twain');
  const quote = await response.json();
  return quote
}
```
여기에 버그가 버그가 있습니다. `/quote`가 존재하지 않는 API라면, `404 Not Found`가 포함된 내용을 응답합니다. 응답은 JSON형식이 아닐 수 있습니다. `response.json()`은 JSON형식이 아니라는 새로운 오류 메시지를 담아 거절된 프로미스를 반환합니다. 호출한 곳에서는 새로운 오류 메시지가 전달되어 실제 오류인 404가 감춰집니다.<br>
또한, fetch가 실패하면 rejected Promise를 응답하지는 않는다는 걸 간과하기 쉽습니다.
``` ts
declare function fetch(
  input: RequestInfo, init? RequestInit
): Promise<Response>;
```

``` ts
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);
  if(!response.ok)
    throw new Error('Request failed: ' + response.status);
  return response;
}
```
이렇게 작성하여 상태 체크를 할 수 있습니다. 더 간결하게 작성해봅니다.

``` ts
const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if(!response.ok)
    throw new Error('Request failed: ' + response.status);
  return response;
}
```

함수 문장을 함수 표현식으로 바꾸고 함수 전체에 타입을 적용했습니다. 이는 TS가 input, init의 타입을 추론할 수 있게 해 줍니다. 타입 구문은 또한 `checkedFetch`의 반환 타입을 보장하며 fetch와 동일합니다. 예를 들어 `throw` 대신 `return`을 사용했다면, TS는 그 실수를 잡아냅니다.

이처럼 함수의 매개변수에 타입 선언을 하는 것보다 함수 표현식 전체 타입을 정의하는 것이 코드도 간결하고 안전합니다. 다른 함수의 시그니처와 동일한 타입을 가지는 새 함수를 작성하거나, 동일한 타입 시그니처를 가지는 여러 개의 함수를 작성할 때는 매개변수의 타입과 반환 타입을 반복해서 작성하지 말고 함수 전체의 타입 선언을 적용해야 합니다.

만약 같은 타입 시그니처를 반복적으로 작성한 코드가 있다면 함수 타입을 분리해 내거나 이미 존재하는 타입을 찾아보도록 합니다. 라이브러리를 직접 만든다면 공통 콜백에 타입을 제공해야 합니다.<br>
다른 함수의 시그니처를 참조하려면 typeof fn을 사용하면 됩니다.
