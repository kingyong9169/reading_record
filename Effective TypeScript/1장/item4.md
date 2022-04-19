# 아이템 4: 구조적 타이핑에 익숙해지기

## JS는 본질적으로 `덕 타이핑` 기반이다.
만약 어떤 함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용한다. TS는 이런 동작, 즉 매개변수 값이 요구사항을 만족한다면 타입이 무엇인지 신경 쓰지 않는 동작을 `그대로 모델링하기 위해 구조적 타이핑을 사용한다.` 그런데 타입 체커의 타입에 대한 이해도가 사람과 조금 다르기 때문에 가끔 예상치 못한 결과가 나오기도 한다. `구조적 타이핑을 제대로 이해한다면 오류인 경우와 오류가 아닌 경우의 차이를 알 수 있고, 더욱 견고한 코드를 작성할 수 있다.`

> `덕 타이핑?`<br>
객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주하는 방식.

``` ts
interface Vector2D {
  x: number;
  y: number:
}
interface NameVector {
  name: string;
  x: number;
  y: number:
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}
```

`NameVector`는 number타입의 x와 y속성이 있기 때문에 `calculateLength` 함수로 호출 가능하다. TS는 다음 코드를 이해할 수 있을 정도로 충분히 영리하다.

``` ts
const v: NameVector = { x: 3, y: 4, name: 'Zee' };
calculateLength(v); // 정상, 5
```

흥미로운 점은 `Vector2D`와 `NameVector`의 관계를 전혀 선언하지 않았다는 것이다. 그리고 `NameVector`를 위한 별도의 `calculateLength`를 구현할 필요도 없다. `TS 타입 시스템은 JS의 런타임 동작을 모델링한다.` `NameVector`의 구조가 `Vector2D`와 호환되기 때문에 `calculateLength` 호출이 가능하다. 여기서 구조적 타이핑이라는 용어가 사용된다.

하지만 구조적 타이핑 때문에 문제가 발생하기도 한다.
``` ts
interface Vector3D {
  x: number;
  y: number:
  z: number:
}

function nomalize(v: Vector3D) {
  const length = calculateLength(v);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}
```

TS가 오류를 잡지 못한 이유이다. `calculateLength`는 2D 벡터를 기반으로 연산하는데, 버그로 인해 `normalize`가 3D로 연산되었다. z가 정규화에서 무시된 것이다.<br>
그런데 타입 체커가 이 문제를 잡아내지 못했다.<br>
`Vector3D`와 호환되는 `{x, y, z}` 객체로 `calculateLength`를 호출하면, 구조적 타이핑 관점에서 x와 y가 있어서 `Vector2D`와 호환된다. 따라서 오류가 발생하지 않았고, 타입 체커가 문제로 인식하지 않았다.(이런 경우를 오류로 처리하기 위한 설정이 존재한다. 아이템 37에서..)<br>
함수를 작성할 때, 호출에 사용되는 매개변수의 속성들이 매개변수의 타입에 선언된 속성만을 가질 거라 생각하기 쉽다. 이러한 타입은 "봉인된" 또는 "정확한" 타입이라고 불리며, TS 타입 시스템에서는 표현할 수 없다. `좋든 싫든 "열려"있다.` 이런 특성 때문에 가끔 당황스러운 결과가 발생한다.

> `열려있다.`<br>
타입의 확장에 열려 있다는 의미이다. 즉, 타입에 선언된 속성 외에 임의의 속성을 추가하더라도 오류가 발생하지 않는다.

``` ts
function calculateLengthL1(v: Vector3D) {
  let length = 0;
  for(const axis of Object.keys(v)) {
    const coord = v[axis]; // string은 Vector3D의 인덱스로 사용할 수 없기에 암시적으로 any타입이다.
    length += Math.abs(coord);
  }
  return length;
}
```
오류를 납득하기 어렵다. axis는 Vector3D 타입인 v의 키 중 하나이기 때문에 x, y, z 중 하나여야 한다. 또한, coord타입이 number가 되어야 할 것으로 예상된다.<br>
그러나 사실은 TS가 오류를 정확히 찾아낸 것이 맞다. 앞에서 Vector3D는 봉인되었다고 가정했다. 그런데 다음 코드처럼 작성할 수도 있다.
``` ts
const vec3D = { x: 3, y: 4, z: 1, address: '123 Broadway' };
calculateLengthL1(vec3D); // 정상, NaN 반환
```

v는 어떤 속성이든 가질 수 있기 때문에, axix의 타입은 string이 될 수도 있다. 그러므로 앞서 본 것처럼 TS는 `v[axis]`가 어떤 속성이 될지 알 수 없기 때문에 number라고 확정할 수 없다. 정확한 타입으로 객체를 순회하는 것은 까다로운 문제다.<br>
결론은 루프보다는 모든 속성을 각각 더하는 구현이 더 낫다.

## 구조적 타이핑은 클래스와 관련된 할당문에서도 당황스러운 결과를 보여 준다.
``` ts
class C {
  foo: string;
  constructor(foo: string) {
    this.foo = foo;
  }
}
const c = new C('instance of C');
const d: C = { foo: 'object literal' }; // 정상
```

d가 `C타입`에 할당되는 이유를 알아보겠다. d는 string 타입의 foo 속성을 가진다. 게다가 하나의 매개변수로 호출이 되는 생성자를 가진다. 그래서 구조적으로는 필요한 속성과 생성자가 존재하기 때문에 문제가 없다. 만약 `C의 생성자`에 단순 할당이 아닌 연산 로직이 존재한다면, d의 경우는 생성자를 실행하지 않으므로 문제가 발생한다. 이러한 부분이 `C 타입`의 매개변수를 선언하여 C, C++, Java와 같은 언어와 매우 다른 특징이다.

## 테스트를 작성할 때는 구조적 타이핑이 유리하다.(유닛 테스팅을 손쉽게 할 수 있다.)
DB에 쿼리하고 결과를 처리하는 함수다.
``` ts
interface Author {
  first: string;
  last: string;
}
function getAuthors(database: PostgresDB): Author[] {
  const authorRows = database.runQuery(`SELECT FIRST, LAST FROM AUTHORS`);
  return authorRows.map(row => ({ first: row[0], last: row[1] }));
}
```

`getAuthors` 함수를 테스트하기 위해서는 mocking한 PostgresDB를 생성해야 한다. 그러나 구조적 타이핑을 활용하여 더 구체적인 인터페이스를 정의하는 것이 더 나은 방법이다.

``` ts
interface DB {
  runQuery: (sql: string) => any[];
}
function getAuthors(database: DB): Author[] {
  const authorRows = database.runQuery(`SELECT FIRST, LAST FROM AUTHORS`);
  return authorRows.map(row => ({ first: row[0], last: row[1] }));
}
```

`runQuery`메서드가 있기 때문에 실제 환경에서도 `getAuthors`에 PostgresDB를 사용할 수 있다. 구조적 타이핑 덕분에, `PostgresDB가 DB인터페이스를 구현하는지 명확히 선언할 필요가 없다.` TS는 그렇게 동작할 것이라는 것을 알아챈다.<br>
테스트를 작성할 때, 더 간단한 객체를 매개변수로 사용할 수도 있다.

``` ts
test('getAuthors', () => {
  const authors = getAuthors({
    runQuery(sql: string) {
      return [['Toni', 'Morrison'], ['Maya', 'Angelou']];
    }
  });
  expect(authors).toEqual([
    { first: 'Toni', last: 'Morrison' },
    { first: 'Maya', last: 'Angelou' },
  ]);
})
```

TS는 테스트 DB가 해당 interface를 충족하는지 확인한다. 그리고 `테스트 코드에는 실제 환경의 DB에 대한 정보가 불필요하다.` 심지어 mocking 라이브러리도 필요없다. 추상화(DB)함으로써, 로직과 테스트를 특정한 구현(PostgresDB)으로부터 분리한 것이다.<br>
테스트 이외에 구조적 타이핑의 또 다른 장점은 `라이브러리 간의 의존성을 완벽히 분리할 수 있다는 것`이다.