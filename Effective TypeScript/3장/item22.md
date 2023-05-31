# 타입 좁히기
타입 넓히기의 반대는 타입 좁히기이다. ts가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말한다. 가장 일반적인 예시는 null 체크일 것이다.

**타입 체커는 일반적으로 조건문에서 타입 좁히기를 잘 해내지만, 타입 별칭이 존재한다면 그러지 못할 수도 있다. 타입 별칭은 아이템 24에서 다룬다.**

분기문에서 예외를 던지거나 함수를 반환하여(null check, instanceof, in, Array.isArray 등 사용) 블록의 나머지 부분에서 변수의 타입을 좁힐 수 있다.

ts는 일반적으로 조건문에서 타입을 좁히는 데 매우 능숙하다. but 타입을 섣불리 판단하는 실수를 저지르기 쉬으므로 다시 한 번 확인하자.

```ts
const el = document.getElementById('foo'); // type: HTMLElement | null
if(typeof el === 'object') el; // type: HTMLElement | null
```
- typeof null이 object이기 때문에 타입이 좁혀지지 않는다.

```ts
function foo(x?: number | string | null) {
  if(!x) x; // type: number | string | null | undefined
}
```
- '', 0 모두 false가 되기 때문에 타입은 좁혀지지 않았다.

타입을 좁히는 또 일반적인 방법은 명시적 **태그**를 붙이는 것이다.

```ts
function handleEvent(e: AppEvent) {
  switch(e.type) {
    case 'download':
      e; // type: DownloadEvent
      break;
    case 'play':
      e; // type: PlayEvent
      break;
  }
}
```

이 패턴은 **태그된 유니온** or **구별된 유니온**이라고 불리며, ts어디에서나 찾아볼 수 있다. 만약 ts가 타입을 식별하지 못한다면, 식별을 돕기 위해 커스텀 함수를 도입할 수 있다.

```ts
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el;
}

function getElementContent(el: HTMLElement) {
  if(isInputElement(el)) {
    return el.value;
  }
  return el.innerText;
}
```

이러한 기법을 **사용자 정의 타입 가드**라도 한다. 함수의 반환의 true인 경우, 타입 체커에게 매개변수의 타입을 좁힐 수 있다고 알려 준다.

어떤 함수들은 타입 가드를 사용해 배열과 객체의 타입 좁히기를 할 수 있다.

```ts
const members = ['Janet', 'Bob', 'Alice'].map(
  who => jackson5.find(n => n === who)
).filter(who => who !== undefined); // (string | undefined)[]
```

이 경우 타입 가드를 사용하자.

```ts
function isDefined<T>(x: T | undefined): x is T {
  return x !== undefined;
}
const members = ['Janet', 'Bob', 'Alice'].map(
  who => jackson5.find(n => n === who)
).filter(isDefined); // string[]
```

ts에서 타입이 어떻게 좁혀지는지 이해한다면 타입 추론에 대한 개념을 잡을 수 있고, 오류 발생의 원인을 알 수 있으며, 타입 체커를 더 효율적으로 이용할 수 있다.

# 요약
- 분기문 외에도 여러 종류의 제어 흐름을 살펴보며 ts가 타입을 좁히는 과정을 이해해야 한다.
- 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용하여 타입 좁히기 과정을 원활하게 할 수 있다.
