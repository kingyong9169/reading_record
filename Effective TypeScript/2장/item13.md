# 아이템 13: 타입과 인터페이스의 차이점 알기
TS에서 명명된 타입을 정의하는 방법은 두 가지가 있습니다.
``` ts
type TState = {
  name: string;
  capital: string;
}

interface IState {
  name: string;
  capital: string;
}
```
> 위와 같이 T, I를 붙이는 네이밍은 지양해야 할 스타일입니다. 표준 라이브러리에서도 일관성 있게 도입하지 않았기 때문에 유용하지도 않습니다.

명명된 타입을 정의할 때 인터페이스 대신 클래스를 사용할 수도 있지만, 클래스는 값으로도 쓰일 수 있는 JS런타임의 개념입니다.

대부분의 경우에는 타입을 상요해도 되고 인터페이스를 사용해도 됩니다. but 타입과 인터페이스 사이에 존재하는 차이를 분명하게 알고, 같은 상황에서는 동일한 방법으로 명명된 타입을 정의해 일관성을 유지해야 합니다.

먼저, 인터페이스 선언과 타입 선언의 비슷한 점입니다. 명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없습니다. 만약 IState, TState를 추가 속성과 함께 할당한다면 `잉여 속성 체크` 오류가 발생합니다.

인덱스 시그니처는 인터페이스, 타입 모두 사용할 수 있습니다.
``` ts
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}
```

함수 타입도 인터페이스, 타입으로 정의할 수 있습니다.
``` ts
type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}

const toStrT: TFn = x => '' + x; // 정상
const toStrT: IFn = x => '' + x; // 정상
```

이런 단순한 함수 타입에는 타입 별칭이 더 나은 선택이겠지만, 함수 타입에 추가적인 속성이 있다면 타입, 인터페이스 둘다 선택할 수 있습니다.

``` ts
type TFnWithProperties = {
  (x: number): number;
  prop: string;
}
interface IFnWithProperties = {
  (x: number): number;
  prop: string;
}
```

문법이 생소할 수도 있지만 JS에서 함수는 호출 가능한 객체라는 것을 떠올려 보면 납득할 수 있는 코드입니다.<br>
타입, 인터페이스 모두 제너릭이 가능합니다.

``` ts
type TPair<T> = {
  first: T;
  second: T;
}

interface IPair<T> {
  first: T;
  second: T;
}
```

인터페이스는 타입을 확장할 수 있으며(주의사항은 뒤에서 설명합니다.) 타입은 인터페이스를 확장할 수 있습니다.
``` ts
interface IStateWithPop extends TState {
  population: number;
}
type TStateWithPop = IState & { population: number; };
```

`IStateWithPop`, `TStateWithPop`은 동일합니다. 여기서 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지는 못한다는 것입니다. 복잡한 타입을 확장하고 싶다면 타입과 &를 사용해야 합니다.<br>
한편 클래스를 구현할 때는, 타입과 인터페이스 둘 다 사용할 수 있습니다.

``` ts
class StateT implements TState {
  name: string = '';
  capital: string = ''
}
class StateI implements IState {
  name: string = '';
  capital: string = ''
}
```

타입과 인터페이스의 다른 점입니다. 유니온 타입은 있지만 유니온 인터페이스라는 개념은 없습니다.
``` ts
type AorB = 'a' | 'b';
```

인터페이스는 타입을 확장할 수 있지만, 유니온은 할 수 없습니다. 그런데 유니온 타입을 확장하는 게 필요할 때가 있습니다.
``` ts
type Input = {};
type Output = {};
interface VariableMap {
  [name: string]: Input | Output;
}
```

또는 유니온 타입에 name 속성을 붙인 타입을 만들 수도 있습니다.
``` ts
type NamedVariable = (Input | Output) & { name: string };
```

이 타입은 인터페이스로 표현할 수 없습니다. `type`키워드는 일반적으로 `interface`보다 쓰임새가 많습니다. `type`키워드는 유니온이 될 수도 있고, 매핑된 타입 또는 조건부 타입 같은 고급 기능에 활용되기도 합니다.<br>
튜플과 배열 타입도 `type`키워드를 이용해 더 간결하게 표현할 수 있습니다.
``` ts
type Pair = [number, number];
type StringList = string[];
type NamedNums = [string, ...number[]];
```

`interface`로 튜플과 비슷하게 구현할 수는 있습니다.
``` ts
interface Tuple {
  0: number;
  1: number;
  length: 2;
}
const t: Tuple = [10, 20]; // 정상
```

but `interface`로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 `concat`같은 메서드들을 사용할 수 없습니다. 그러므로 튜플은 `type`키워드로 구현하는 것이 낫습니다.

반면 `interface`는 타입에 없는 몇 가지 기능이 있습니다. 그 중 하나는 바로 보강(augment)이 가능하다는 것입니다.
``` ts
interface IState {
  name: string;
  capital: string;
}
interface IState {
  population: number;
}
const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000
}; // 정상
```

이렇게 속성을 확장하는 것을 `선언 병합`이라고 합니다. 선언 병합은 주로 `타입 선언 파일`에서 사용됩니다. 따라서 타입 선언 파일을 작성할 때는 선언 병합을 지원하기 위해 반드시 `interface`를 사용해야 하며 표준을 따라야 합니다. 타입 선언에는 사용자가 채워야 하는 빈틈이 있을 수 있는데, 바로 이 선언 병합이 그렇습니다.<br>
TS는 여러 버전의 JS 표준 라이브러리에서 여러 타입을 모아 병합합니다. 예를 들어, Array 인터페이스는 `lib.es5.d.ts`에 정의되어 있고 기본적으로는 `lib.es5.d.ts`에 선언된 인터페이스가 사용됩니다. 그러나 `tsconfig.json`의 lib목록에 ES2015를 추가하면 TS는 `lib.es2015.d.ts`에 선언된 인터페이스를 병합합니다. 여기에는 ES2015에 추가된 또 다른 Array선언의 find같은 메소드가 포함됩니다. 이들은 병합을 통해 다른 Array 인터페이스에 추가됩니다. 결과적으로 각 선언이 병합되어 전체 메소드를 가지는 하나의 Array타입을 얻게 됩니다.<br>
병합은 선언과 마찬가지로 일반적인 코드에서도 지원되므로 언제 병합이 가능한지 알고 있어야 합니다. **타입은 기존 타입에 추가적인 보강이 없는 경우에만 사용해야 합니다.**

`type`과 `interface` 중 어느 것을 사용해야 할지 결론을 내려 보겠습니다. 복잡한 타입이라면 고민할 것도 없이 `type`별칭을 사용하면 됩니다. but `type`과 `interface`, 두 가지 방법으로 모두 표현할 수 있는 간단한 객체 타입이라면 일관성과 보강의 관점에서 고려해 봐야 합니다. 일관되게 `interface`를 사용하는 코드베이스에서 작업하고 있다면 `interface`를 사용하고, 일관되게 `type`을 사용 중이라면 `type`을 사용하면 됩니다.

아직 스타일이 확립되지 않은 프로젝트라면, 향후에 보강의 가능성이 있을지 생각해 봐야 합니다. 어떤 API에 대한 타입 선언을 작성해야 한다면 `interface`를 사용하는 게 좋습니다. API가 변경될 때 사용자가 `interface`를 통해 새로운 필드를 병합할 수 있어 유용하기 때문입니다. but 프로젝트 내부적으로 사용되는 타입에 선언 병합이 발생하는 것은 잘못된 설계입니다. 따라서 이럴 때는 `type`을 사용해야 합니다.
