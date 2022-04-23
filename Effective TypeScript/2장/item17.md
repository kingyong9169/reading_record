# 아이템 17: 변경 관련된 오류 방지를 위해 readonly 사용하기

## readonly를 사용하면 변경하면서 발생하는 오류를 방지할 수 있고, 변경이 발생하는 코드도 쉽게 찾을 수 있다.
아래는 삼각수(1, 1+2,1+2+3...)를 출력하는 코드
``` ts
function printTriangles(n: number) {
  const nums = [];
  for (let i = 0 ; i < n ; i++) {
    nums.push(i);
    console.log(arraySum(nums));
  }
}

function arraySum(arr: number[]) {
  let sum = 0, num;
  while((num = arr.pop()) !== undefined) {
    sum += num;
  }
  return sum;
}

> printTriangles(5);
0
1
2
3
4
```

이 함수(arraySum)는 배열 안의 숫자들을 모두 합칩니다. 그런데 계산이 끝나면 원래 배열이 전부 비게 됩니다. JS 배열은 내용을 변경할 수 있기 때문에, TS에서도 역시 오류 없이 통과하게 됩니다.<br>
오류의 범위를 좁히기 위해 arraySum이 배열을 변경하지 않는다는 선언을 해보겠습니다. readonly접근 제어자를 사용하면 됩니다.
``` ts
function arraySum(arr: readonly number[]) {
  let sum = 0, num;
  while((num = arr.pop()) !== undefined) {
  // readonly number[] 형식에 pop 속성이 없습니다.
    sum += num;
  }
  return sum;
}
```

`reaonly number[]`는 타입이고 `number[]`와 구분되는 몇 가지 특징이 있습니다.
- 배열의 요소를 읽을 수 있지만, 쓸 수는 없다.
- length를 읽을 수 있지만, 바꿀 수는 없다.(배열을 변경함)
- 배열을 변경하는 pop을 비롯한 다른 메소드를 호출X

`number[]`는 `reaonly number[]`보다 기능이 많기 때문에, `reaonly number[]`의 서브타입이 됩니다. 따라서 변경 가능한 배열을 `readonly` 배열에 할당할 수 있습니다. 하지만 그 반대는 불가능.

``` ts
const a: number[] = [1, 2, 3];
const b: readonly number[] = a;
const c: number[] = b;
// readonly number[]은 readonly이므로
// 변경 가능한 number[]에 할당될 수 없다.
```

## 함수가 매개변수를 수정하지 않는다면 readonly로 선언하는 것이 좋다.

매개변수를 readonly로 선언하면 생기는 일
- TS는 매개변수가 함수 내에서 변경이 일어나는지 체크
- 호출하는 쪽에서는 함수가 매개변수를 변경하지 않는다는 보장을 받는다.
- 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을 수도 있다.

JS(TS)에서는 명시적으로 언급하지 않는 한, 함수가 매개변수를 변경하지 않는다고 가정합니다. but 이러한 암묵적인 방법은 타입 체크에 문제를 일으킬 수 있습니다. 명시적인 방법을 사용하는 것이 컴파일러, 사람 모두에게 좋습니다.

이제 제대로 동작합니다.
``` ts
function arraySum(arr: readonly number[]) {
  let sum = 0, num;
  for(const num of arr) {
    sum += num;
  }
  return sum;
}
```

만약 함수가 매개변수를 변경하지 않는다면, readonly로 선언해야 합니다. 더 넓은 타입으로 호출할 수 있고, 의도치 않은 변경이 방지될 것입니다. 이로 인한 단점은 상대적으로 적습니다.

굳이 찾아보자면 매개변수가 readonly로 선언되지 않은 함수를 호출해야 할 경우도 있다는 것입니다. 만약 함수가 매개변수를 변경하지 않고도 제어가 가능하다면 readonly로 선언하면 됩니다. 그런데 `어떤 함수를 readonly로 만들면`, `그 함수를 호출하는 다른 함수도 모두 readonly로 만들어야 합니다.` 그러면 인터페이스를 명확히 하고 타입 안전성을 높일 수 있기 때문에 꼭 단점이라고 볼 순 없습니다. but 다른 라이브러리에 있는 함수를 호출하는 경우라면, 타입 선언을 바꿀 수 없으므로 타업 단언문(`as number[]`)을 사용해야 합니다.

``` ts
et foo: {
    readonly bar: number;
} = {
        bar: 123
    };

function iMutateFoo(foo: { bar: number }) {
    foo.bar = 456;
}

iMutateFoo(foo); // foo 인자가 foo 파라미터에 의해 앨리어싱됨
console.log(foo.bar); // 456!
```
기본적으로 `readonly`는 내가 속성을 변경하지 못함을 보장하지만, 객체를 다른 함수에게 넘길 경우에는 이것이 보장되지 않고 그 다른 함수는 객체의 속성을 변경할 수 있다.(타입 호환성 문제 때문에 허용됨.)

``` ts
interface Foo {
    readonly bar: number;
}
let foo: Foo = {
    bar: 123
};

function iTakeFoo(foo: Foo) {
    foo.bar = 456; // 오류! bar는 readonly
}

iTakeFoo(foo); // // foo 인자가 foo 파라미터에 의해 앨리어싱됨
```
이처럼 어떤 변수를 readonly로 넘기면 매개변수 또한 readonly로 만들어야 합니다.

readonly를 사용하면 지역 변수와 관련된 모든 종류의 변경 오류를 방지할 수 있습니다.

``` ts
const lines = [
  blah,
  blah,
  blah...
];
```
라는 배열을 전달 인자로 함수 호출

``` ts
function parseTaggedText(lines: string[]): string[] {
  const paragraphs: string[][] = [];
  const currPara: string[] = [];

  const addParagraph = () => {
    if(currPara.length) {
      paragraphs.push(currPara);
      currPara.length = 0; // 배열 비우기
    }
  };

  for(const line of lines) {
    if(!line) addParagraph();
    else currPara.push(line);
  }
  addParagraph();
  return paragraphs;
}

[ [], [], [] ]
```

문제점은 별칭과 변경을 동시에 사용해 발생했습니다.
``` ts
paragraphs.push(currPara);
```

`currPara`의 내용이 삽입되지 않고 배열의 참조가 삽입되었습니다. `currPara`에 새 값을 채우거나 지운다면 동일한 객체를 참조하고 있는 `paragraphs` 요소에도 변경이 반영됩니다.

**문제의 코드**
``` ts
paragraphs.push(currPara);
currPara.length = 0;
```
이 코드는 새 단락을 `paragraphs`에 삽입하고 바로 지워 버립니다.<br>
문제는 `currPara`길이를 수정하고 `currPara.push`를 호출하면 둘 다 `currPara` 배열을 변경한다는 점입니다. `currPara`를 `readonly`로 선언하여 이런 동작을 방지할 수 있습니다.

**바꾼 결과**
``` ts
function parseTaggedText(lines: string[]): string[] {
  const paragraphs: string[][] = [];
  const currPara: readonly string[] = [];

  const addParagraph = () => {
    if(currPara.length) {
      paragraphs.push(currPara); // readonly string[] 형식의 인수는 string[] 형식의 매개변수에 할당할 수 없습니다.
      currPara.length = 0; // 읽기 전용 속성이기 때문에 length에 할당할 수 없습니다.
    }
  };

  for(const line of lines) {
    if(!line) addParagraph();
    else currPara.push(line); // readonly string[] 형식에 push 속성이 없습니다.
  }
  addParagraph();
  return paragraphs;
}
```

**해결코드**
``` ts
let currPara: readonly string[] = [];
// ...
currPara = []; // 배열을 비움
// ...
currPara = currPara.concat([line]);
```
- push와 달리 concat은 원본을 수정하지 않고 새 배열을 반환.
- let으로 바꿈으로써 가리키는 배열을 자유롭게 변경. 배열 자체는 변경X

but 여전히 paragraphs에 대한 오류가 남음. 
> 집합의 관계로 봤을 때, currPara가 더 큰 집합. 붕어빵과 붕어빵 틀의 관계

해결방법은 3가지
1. `currPara`의 복사본 만들기: `paragraphs.push([...currPara]);`
2. `paragraphs`를 `readonly string[]`배열로 변경: `const paragraphs: (readonly string[])[] = [];`<br>
여기서 괄호가 중요한데 `readonly string[][] = []`은 readonly 배열의 변경 가능한 배열이 아니라 변경 가능한 배열의 readonly 배열이다.<br>
말이 어렵지만 `readonly`를 여러 개 가지는 배열(수정 가능) vs `readonly` 배열(수정 불가능)
3. 배열의 `readonly` 속성을 제거하기 위해 단언문 사용
`paragraphs.push(currPara as string[]);`
바로 다음 문장에서 `currPara`를 새 배열에 할당하므로, 매우 공격적인 단언문처럼 보이지는 않는다.

## const와 readonly의 차이점
**공통점**
초기 때 할당된 값을 변경할 수 없다.

**차이점**
1. const
- 변수 참조를 위한 것이다.
- 변수에 다른 값을 할당할 수 없다.
- 재할당 방지
2. readonly
- 속성을 위한 것이다.
- 값 바꾸는 것 방지
``` ts
type readonlyA = {
  readonly barA: { baz: string }
};

const y: readonlyA = { barA: {baz: 'quux'} };
y.barA = 'quux'; // 변경 불가
y.barA.baz = 'zebranky'; // 변경 가능
```
readonly는 어떤 객체의 속성을 변경할 수 없도록 할 수 있다. 다만 얕게 동작한다.

## readonly는 얕게 동작한다.
`readonly`는 얕게 동작한다는 것에 유의하며 사용해야 합니다. 앞에서 이미 `readonly string[][]`을 봤습니다. 만약 객체의 `readonly` 배열이 있다면, 그 객체 자체는 `readonly`가 아닙니다.

``` ts
const dates: readonly Date[] = [new Date()];
dates.push(new Date()); // readonly Date[] 형식에 push 속성이 없습니다.
dates[0].setFullYear(2037); // 정상
```

비슷한 경우가 `readonly`의 사촌 격이자 객체에 사용되는 `Readonly` 제네릭에도 해당된다.

``` ts
interface Outer {
  inner: {
    x: number;
  }
}
const o: Readonly<Outer> = { inner: { x: 0 }};
o.inner = { x: 1 }; // 읽기 전용 속성이기 때문에 inner에 할당할 수 없습니다.
```

``` ts
type T = Readonly<Outer>;

type T = { // 동일
  readonly inner: {
    x: number;
  };
}
```
여기서 중요한 것은 `readonly` 접근제어자는 `inner`에 적용되는 것이지 `x`는 아니라는 것입니다. 현재 시점에는 깊은 `readonly` 타입이 기본으로 지원되지 않지만, 제너릭을 만들면 깊은 `readonly` 타입을 사용할 수 있습니다. but 제너릭은 만들기 까다롭기 때문에 라이브러리를 사용하는 게 낫습니다. 예를 들어, `ts-essentials`에 있는 `DeepReadonly` 제너릭을 사용하면 됩니다.

인덱스 시그니처에도 `readonly`를 쓸 수 있습니다. 읽기는 허용하되 쓰기를 방지하는 효과가 있습니다.
``` ts
let obj: { readonly [k: string]: number } = {};
// 또는 Readonly<{[k: string]: number}>
obj.hi = 45; // ...형식의 인덱스 시그니처는 읽기만 허용됩니다.
obj = {...obj, hi: 12}; // 정상
```
