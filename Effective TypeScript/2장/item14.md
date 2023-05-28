# 타입 연산과 제너릭 사용으로 반복 줄이기
> 타입에서 같은 코드를 반복하지 말라는 DRY(Dont' Repeat Yourself) 원칙 지키기

```ts
interface Person {
  firstName: string;
  lastname: string;
}

interface PersonWithBirthDate {
  firstName: string;
  lastname: string;
  birth: Date;
}
```

타입 중복은 코드 중복만큼 많은 문제를 발생시킨다. 위 코드에서 middleName을 Person에 추가하면 Person과 PersonWIthBirthDate는 다른 타입이 된다.

타입에서 중복이 더 흔한 이유 중 하나: 공유된 패턴을 제거하는 메커니즘이 기존 코드에서 하던 것과 비교해 덜 익숙하기 때문.

## 타입에 이름 붙이기

**반복을 줄이는 가장 간단한 방법: 타입에 이름 붙이기**

```ts
interface Person {
  firstName: string;
  lastname: string;
}

interface PersonWithBirthDate extends Person {
  birth: Date;
}
```

두 인터페이스가 필드의 부분 집합을 공유한다면, 공통 필드만 골라서 기반 클래스로 분리하고 추가적인 필드만 작성하여 중복을 제거한다.

또한, 이미 존재하는 type을 확장하는 경우에, `&`(인터섹션) 연산자를 사용하여 타입을 확장할 수 있다.

```ts
type PersonWithBirthDate = Person & { birth: Date };
```

이 방법은 유니온 타입(확장할 수 없는)에 속성을 추가하려고 할 때 특히 유용하다.

## 매핑된 타입, 제너릭 타입 사용하기

```ts
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}
```
에서는 TopNavState를 확장하여 State를 구성하기보다, State의 부분 집합으로 TopNavState를 정의하는 것이 바람직해 보인다. 이 방법이 전체 앱의 상태를 하나의 인터페이스로 유지할 수 있게 해 준다.

```ts
interface TopNavState {
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
}
```

여기서 '매핑된 타입'을 사용하여 중복을 제거할 수 있다.

```ts
type TopNavState = {
  [k in 'userId' | 'pageTitle' | 'recentFiles']: State[k];
}
```

매핑된 타입은 배열의 필드를 루프 도는 것과 같은 방식이다. 이 패턴은 표준 라이브러리에서 Pick을 찾을 수 있다.

```ts
type Pick<T, K> = { [k in K]: T[k] };

type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

여기서 Pick은 제너릭 타입이다. Pick을 사용하는 것은 함수를 호출하는 것과 마찬가지이다. 함수에서 두 개의 매개변수 값을 받아서 결괏값을 반환하는 것처럼, Pick은 T, K 두 가지 타입을 받아서 결과 타입을 반환한다.

## 태그된 유니온

```ts
interface SaveAction {
  type: 'save';
  // ...
}
interface LoadAction {
  type: 'load';
  // ...
}
type Action = SaveAction | LoadAction;
type ActionType = 'save' | 'load';
```

```ts
type ActionType = Action['type'];
```
Action유니온을 인덱싱하면 타입 중복 없이 정의할 수 있다.

## 값의 형태에 해당하는 타입

```ts
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: '#00FF00',
  label: 'VGA',
}
```

이런 경우 typeof를 사용할 수 있다.

```ts
type Options = typeof INIT_OPTIONS;
```

js의 런타임 연산자 typeof를 사용한 것처럼 보이지만, 실제로는 ts 단계에서 연산되며 훨씬 더 정확하게 타입을 표현한다. 그런데 값으로부터 타입을 만들어 낼 때는 선언 순서를 주의해야 한다. **타입 정의를 먼저 하고 값이 그 타입에 할당 가능하다고 선언하는 것이 좋다.** 그렇게 해야 타입이 더 명확해지고, 예상하기 어려운 타입 변동을 방지할 수 있다.

## 함수, 메서드의 반환 값을 타입으로 만들기

```ts
function getUserInfo(userId: string) {
  // ...
  return {
    userId,
    name,
    age,
    height,
    weight,
    favoriteColor,
  }
}
```

표준 라이브러리에 ReturnType 제너릭이 있다.

```ts
type UserInfo = ReturnType<typeof getUserInfo>;
```

RetrunType은 함수의 '값'인 getUserInfo가 아니라 함수의 '타입'인 typeof getUserInfo에 적용된다. typeof와 마찬가지로 이런 기법은 신중하게 사용해야 한다. 적용 대상이 값인지 타입인지 정확히 알고, 구분해서 처리해야 한다.

## 타입을 만드는 함수

**제너릭 타입은 타입을 위한 함수와 같다.** 함수는 코드에 대한 DRY 원칙을 지킬 때 유용하게 사용된다. 타입에 대한 DRY 원칙의 핵심이 제너릭이라는 것은 어쩌면 당연한데 간과한 부분이 잇다.

함수에서 매개변수로 매핑할 수 있는 값을 제한하기 위해 타입 시스템을 사용하는 것처럼 제너릭 타입에서 매개변수를 제한할 수 있는 방법이 필요하다. 이 방법에는 extends가 있다. 제너릭 매개변수가 특정 타입을 확장한다고 선언할 수 있다.

```ts
interface Name {
  first: string;
  last: string;
}

type DancingDuo<T extends Name> = [T, T];

const couple1: DancingDuo<Name> = [
  { first: 'Fred', last: 'Astaire' },
  { first: 'Ginger', last: 'Rogers' },
]

const couple2: DancingDuo<{ first: string }> = [ // Name 타입에 필요한 last 속성이 { first: string }에 없습니다.
  { first: 'Sonny' },
  { first: 'Cher' },
]
```

Name을 확장하지 않기 때문에 오류가 발생한다.

> 현재 선언부에 항상 제너릭 매개변수를 작성하도록 되어 있다. 때문에 DancingDuo만 쓰면 동작하지 않는다.

앞에 나온 Pick의 정의는 extends를 사용해서 완성할 수 있다.

```ts
type Pick<T, K> = { [k in K]: T[k] }; // K 타입은 string | number | symbol타입에 할당할 수 없다.
```

K는 T 타입과 무관하고 범위가 너무 넓다. K는 인덱스로 사용될 수 있는 string | number | symbol이 되어야 하며 실제로는 범위를 조금 더 좁힐 수 있다.

```ts
type Pick<T, K keyof T> = { [k in K]: T[k] };
```

**타입이 값의 집합이라는 관점에서 생각하면 extends를 '확장'이 아니라 '부분 집합'이라는 걸 이해하는 데 도움이 된다.**

점점 더 추상적인 타입을 다루고 있지만, 원래의 목표를 잊지 말자. 원래 목표는 유효한 프로그램은 통과시키고 무효한 프로그램에는 오류를 발생시키는 것이다. 이 번 경우의 목표는 바로 Pick에 잘못된 키를 넣으면 오류가 발생해야 한다는 것이다.

값 공간에서와 마찬가지로 반복적인 코드는 타입 공간에서도 좋지 않다. 반복하지 않도록 주의하자.

## 요약
- DRY 원칙을 타입에도 최대한 적용해야 한다.
- 타입에 이름을 붙여 반복을 피하고 extends를 사용해 인터페이스 필드의 반복을 피해야 한다.
- 타입들 간 매핑을 위해 keyof, typeof, 인덱싱, 매핑된 타입을 사용하자.
- 제너릭 타입은 타입을 위한 함수다. 제너릭 타입을 제한하려면 extends를 사용하자.
- 표준 라이브러리에 정의된 Pick, Partial, ReturnType 같은 제너릭 타입에 익숙해져야 한다.
