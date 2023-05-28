# 매핑된 타입을 사용하여 값을 동기화하기

산점도를 그리는 UI를 만든다고 가정한다.

```ts
interface ScatterProps {
  // The Data
  xs: number[];
  ys: number[];

  // display
  xRange: [number, number];
  yRange: [number, number];

  // Events
  onClick: (x: number, y: number, index: number) => void;
}
```

불필요한 작업을 피하기 위해, 필요할 때만 차트를 다시 그릴 수 있다. 데이터나 디스플레이 속성이 변경되면 다시 그려야 하지만, 이벤트 핸들러가 변경되면 다시 그릴 필요가 없다. 이런 종류의 최적화는 리액트 컴포넌트에서는 일반적인 일인데, 렌더링할 때마다 이벤트 핸들러 Prop이 새 화살표 함수로 설정된다.

최적화를 두 가지 방법으로 구현해 본다.

```ts
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && k !== 'onClick') return true;
  }
  return false;
}
```

만약 새로운 속성이 추가되면 shouldUpdate 함수는 값이 변경될 때마다 차트를 다시 그릴 것이다. 이렇게 처리하는 것을 **보수적 접근법**또는 **실패에 닫힌 접근법**이라고 한다. 이 접근법을 이용하면 차트가 정확하지만 너무 자주 그려질 가능성이 있다.

> 실패에 닫힌 방법은 오류 발생에 적극적으로 대처하는 방향을 말한다. 말 그대로 방어적, 보수적 접근법이다. 반대로 실패에 열린 방법은 오류 발생 시에 소극적으로 대처하는 방향이다. 만약 보안과 관련된 곳이면 실패에 닫힌 방법을 써야 할 것이고, 기능에 무리가 없고 사용성이 중요한 곳이라면 실패에 열린 방법을 써야 할 것이다.

두 번째 방법은 **실패에 열린 접근법**이다.

```ts
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // no check for onClick
  )
}
```

위 코드는 차트를 불필요하게 다시 그리는 단점을 해결했다. but 실제로 차트를 다시 그려야 할 경우에 누락되는 일이 생길 수 있다. 이는 히포크라테스 전집에 나오는 원칙 중 하나인 **우선, 망치지 말 것**을 어기기 때문에 일반적인 경우에 쓰이는 방법은 아니다.

앞선 두 가지 최적화 방법 모두 이상적이지 않다. 새로운 속성이 추가될 때 직접 shouldUpdate를 고치도록 주석을 작성할 수 있다.

```ts
interface ScatterProps {
  // The Data
  xs: number[];
  ys: number[];
  ...
  // 참고: 여기에 속성을 추가하려면, shouldUpdate를 고치세요!
}
```

그러나 이 방법 역시 최선이 아니며, 타입 체커가 대신할 수 있게 하는 것이 좋다. 다음은 타입 체커가 동작하도록 개선한 코드이다. **핵심은 매핑된 타입과 객체를 사용하는 것이다.**

```ts
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
}

function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) return true;
  }
  return false;
}
```

`[k in keyof ScatterProps]`은 타입 체커에게 `REQUIRES_UPDATE`가 ScatterProps와 동일한 속성을 가져야 한다는 정보를 제공한다.

이 방식은 오류를 정확히 잡아 낸다. 속성을 삭제하거나 이름을 바꾸어도 비슷한 오류가 발생한다. 여기서 boolean 값을 가진 객체를 사용했다는 점이 중요하다.

여기서 우리는 실패에 열린 방법 or 닫힌 방법을 선택할지 정해야 한다.

**매핑된 타입은 한 객체가 또 다른 객체와 정확히 같은 속성을 가지게 할 때 이상적이다.** 타입스크립트가 코드에 제약을 강제하도록 할 수 있다.

## 요약
- 매핑된 타입을 사용해서 관련된 값과 타입을 동기화하도록 한다.
- 인터페이스에 새로운 속성을 추가할 때, 선택을 강제하도록 매핑된 타입을 고려해야 한다.
