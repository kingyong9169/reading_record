# 함수형 기법과 라이브러리로 타입 흐름 유지하기
파이썬, C, 자바 등에서 볼 수 있는 표준 라이브러리가 js에는 포함되어 있지 않다. 수년간 많은 라이브러리들은 표준 라이브러리의 역할을 대신하기 위해 노력해 왔다. 제이쿼리는 DOM과의 상호작용뿐만 아니라 객체와 배열을 순회하고 매핑하는 기능을 제공했다. 언더스코어는 주로 일반적인 유틸리티 함수를 제공하는 데 초점을 맞추었고, 이러한 노력을 바탕으로 로대시가 만들어졌습니다. 람다 같은 최근의 라이브러리는 함수형 프로그래밍의 개념을 js 세계에 도입하고 있다.

이러한 라이브러리들의 일부 기능(map, flatMap, filter, reduce 등)은 순수 js로 구현되어 있다. 이러한 기법(그리고 로대시에서 제공되는 다른 것들)은 루프를 대체할 수 있기 때문에 js에서 유용하게 사용되는데, ts와 조합하여 사용하면 더욱 빛을 발한다. 그 이유는 타입 정보가 그대로 유지되면서 타입 흐름이 계속 전달되도록 하기 때문입니다. 반면에 직접 루프를 구현하면 타입 체크에 대한 관리도 직접 해야 한다.

어떤 csv 데이터를 파싱한다고 생각해 본다. 순수 js에서는 절차형 프로그래밍 형태로 구현할 수 있다.
```ts
const csvData = "...";
const rawRows = csvData.split("\n");
const headers = rawRows[0].split(",");

const rows = rawRows.slice(1).map(rosStr => {
  const row = {};
  rowStr.split(',').forEach((cell, idx) => {
    row[headers[idx]] = cell;
  });
  return row;
});
```
함수형 마인드를 조금이라도 가진 js 개발자라면 reduce를 사용해서 행 객체를 만드는 방법을 선호할 수도 있다.

```ts
const rows = rawRows.slice(1)
  .map(rowStr => rowStr.split(',').reduce(
    (row, val, i) => (row[headers[i]] = val, row),
    {}));
```

이 코드는 절차형 코드에 비해 세 줄을 절약했지만 보는 사람에 따라 더 복잡하게 느껴질 수도 있다. 키와 값 배열로 취합해서 객체로 만들어 주는 로대시의 zipObject ㅎ마수를 이용하면 더 짧게 만들 수 있다.

```ts
import _ from 'lodash';
const rows = rawRows.slice(1)
  .map(rowStr => _.zipObject(headers, rowStr.split(',')));
```

코드가 매우 짧아졌다. 그런데 js에서는 프로젝트에 서드파티 라이브러리 종속성을 추가할 때 신중해야 한다. 만약 서드파티 라이브러리 기반으로 코드를 짧게 줄이는 데 시간이 많이 든다면, 사용하지 않는 게 더 낫다.

but 같은 코드를 ts로 작성하면 서드파티 라이브러리를 사용하는 것이 무조건 유리하다. 타입 정보를 참고하며 작업할 수 있기 때문에 서드파티 라이브러리 기반으로 바꾸는 데 시간이 훨씬 단축된다.

한편, csv파서의 절차형, 함수형 버전 모두 같은 오류를 발생시킨다.

```ts
const rowsA = rawRows.slice(1).map(rowStr => {
  const row = {};
  rowStr.split(',').forEach((val, j) => {
    row[headers[j]] = val; // {} 형식에서 string 형식의 매개변수가 포함된 인덱스 시그니처를 찾을 수 없다.
  })
});

const rowsB = rawRows.slice(1)
  .map(rowStr => rowStr.split(',').reduce(
    (row, val, i) => (row[headers[i]] = val, row), // {} 형식에서 string 형식의 매개변수가 포함된 인덱스 시그니처를 찾을 수 없다.
    {}));
```

두 버전 모두 {}의 타입으로 `{ [column: string]: string }` 또는 `Record<string, string>`을 제공하면 오류가 해결된다. 반면 로대시 버전은 별도의 수정 없이도 타입 체커를 통과한다.

```ts
const rows = rawRows.slice(1)
  .map(rowStr => _.zipObject(headers, rowStr.split(','))); // 타입이 _.Dictionary<string>[]
```

Dictionary는 로대시의 타입 별칭이다. `Dictionary<string>`은  `{ [key: string]: string }` 또는 `Record<string, string>`과 동일하다. 여기서 중요한 점은 타입 구문이 없어도 rows의 타입이 정확하다는 것이다.

데이터의 가공이 정교해질수록 이러한 장점은 더욱 분명해진다.

```ts
interface BasketballPlayer {
  name: string;
  team: string;
  salary: number;
}
declare const rosters: { [team: string]: BasketballPlayer[] };
```

루프를 사용해 단순(flat) 목록을 만들려면 배열에 concat을 사용해야 한다. 다음 코드는 동작이 되지만 타입 체크는 되지 않는다.

```ts
let allPlayers = []; // allPlayers 변수는 형식을 확인할 수 없는 경우, 일부 위치에서 암시적으로 any[] 형식이다.
for (const roster of Object.values(rosters)) {
  allPlayers = allPlayers.concat(roster); // allPlayers 변수에는 암시적으로 any[] 형식이 포함된다.
}
```

이 오류를 고치려면 allPlayers에 타입 구문을 추가해야 한다.
    
```ts
let allPlayers: BasketballPlayer[] = [];
for(const players of Object.values(rosters)) {
  allPlayers = allPlayers.concat(players); // 정상
}
```

but 더 나은 해법은 `Array.prototype.flat`을 사용하는 것이다.

`const allPlayers = Obejct.values(rosters).flat()`

flat 메서드는 다차원 배열을 평탄화한다. 타입 시그니처는 `T[][] => T[]`같은 형태이다. 이 버전이 가장 간결하고 타입 구문도 필요 없다. 또한 allPlayers 변수가 향후에 변경되지 않도록 let 대신 const를 사용할 수 있다.

allPlayers를 가지고 각 팀별로 연봉 순으로 정렬해서 최고 연봉 선수의 명단을 만든다고 가정해 보겠다. 로대시 없는 방법은 다음과 같다. 함수형 기법을 쓰지 않은 부분은 타입 구문이 필요하다.

```ts
const teamToPlayers: { [team: string] } = {};
for (const player of allPlayers) {
  const { team } = players;
  teamToPlayers[team] = teamToPlayers[team] || [];
  teamToPlayers[team].push(player);
}

for(const players of Object.values(teamToPlayers)) {
  players.sort((a, b) => b.salary - a.salary);
}

const bestPaid = Object.values(teamToPlayers).map(players => players[0]);
bestPaid.sort((a, b) => b.salary - a.salary);
console.log(bestPaid);
```

로대시를 사용해서 동일한 작업을 하는 코드를 구현하면 다음과 같다.

```ts
const bestPaid = _(allPlayers)
  .groupBy(player => player.team)
  .mapValues(players => _.maxBy(players, p => p.salary)!);
  .values()
  .sortBy(p => -p.salary)
  .value(); // 타입이 BasketballPlayer[]
```
길이가 절반으로 줄었고, 보기에도 깔끔하며, null 아님 단언문(!, 타입 체커는 _.maxBy로 전달된 players 배열이 비어 있지 않은지 알 수 없다.)을 딱 한 번만 사용했다. 또한 로대시와 언더스코어의 개념인 **체인**을 사용했기 때문에, 더 자연스러운 순서로 일련의 연산을 작성할 수 있다. 만약 체인을 사용하지 않는다면 다음 예제처럼 뒤에서부터 연산이 수행된다.

```ts
_.c(_.b(_.a(v)))
```

체인을 사용하면 다음처럼 연산자의 등장 순서와 실행 순서가 동일하게 된다.

```ts
_(v).a().b().value()
```

_(v)는 값을 래핑하고, .value()는 언래핑한다.

래핑된 값의 타입을 보기 위해 체인의 각 함수 호출을 조사할 수 있고, 결과는 항상 정확하다.

로대시의 어떤 기발한 단축 기법이라도 ts로 정확하게 모델링될 수 있다. 그런데 내장된 Array.prototype.map 대신 _.map을 사용하려는 이유는 무엇일까? 콜백을 전달하는 대신 속성의 이름을 전달할 수 있기 때문이다.

```ts
const nameA = allPlayers.map(p => p.name);
const nameB = _.map(allPlayers, p => p.name);
const nameC = _.map(allPlayers, 'name');
```

**ts 타입 시스템이 정교하기 때문에 앞의 예제처럼 다양한 동작을 정확히 모델링할 수 있다. 사실 함수 내부적으로는 문자열 리터럴 타입과 인덱스 타입의 조합으로만 이루어져 있기 때문에 타입이 자연스럽게 도출된다.** 만약 C++ 또는 자바에 익숙하다면 이런 종류의 타입 추론이 마법처럼 느껴질 수 있다.

```ts
const salaries = _.map(allPlayers, 'salary');
const teams = _.map(allPlayers, 'team');
const mix = _.map(allPlayers, Math.random() < 0.5 ? 'name' | 'salary');
```

내장된 함수형 기법들과 로대시 같은 라이브러리에 타입 정보가 잘 유지되는 것은 우연이 아니다. **함수 호출 시 전달된 매개변수 값을 건드리지 않고 매번 새로운 값을 반환함으로써, 새로운 타입으로 안전하게 반환할 수 있다.** 넓게 보면, ts의 많은 부분이 js 라이브러리의 동작을 정확히 모델링하기 위해 개발되었다. 그러므로 라이브러리를 사용할 때 타입 정보가 잘 유지되는 점을 십분 활용해야 ts의 원래 목적을 달성할 수 있다.

## 요약
- 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기보다는 내장된 함수형 기법과 로대시 같은 유틸리티 라이브러리를 사용하는 것이 좋다.
