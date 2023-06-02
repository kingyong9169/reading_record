# 타입 추론에 문맥이 어떻게 사용되는지 이해하기
ts는 타입을 추론할 때 단순히 값만 고려하지는 않습니다. 값이 존재하는 곳의 문맥까지도 살핀다. 그런데 문맥을 고려해 타입을 추론하면 가끔 이상한 결과가 나온다. 이때 타입 추론에 문맥이 어떻게 사용되는지 이해하고 있다면 제대로 대처할 수 있다.

js는 코드의 동작과 실행 순서를 바꾸지 않으면서 표현식을 상수로 분리해 낼 수 있다. 예를 들어, 다음 두 문장은 동일하다.

```js
// 인라인 형태
setLanguage('JavaScript');

// 참조 형태
let language = 'JavaScript';
setLanguage(language);
```

ts에서는 다음 리팩터링이 여전히 동작한다.
```ts
function setLanguage(language: string) { ... }

setLanguage('JavaScript'); // 정상

let language = 'JavaScript';
setLanguage(language); // 정상
```

이제 문자열 타입을 더 특정해서 문자열 리터럴 타입의 유니온으로 바꾼다고 가정한다.

```ts
type Language = 'JavaScript' | 'TypeScript' | 'Python';
function setLanguage(language: Language) { ... }

setLanguage('JavaScript'); // 정상

let language = 'JavaScript';
setLanguage(language); // string 형식의 인수는 'Language' 형식의 매개 변수에 할당될 수 없습니다.
```

인라인 형태에서 ts는 함수 선언을 통해 매개변수가 Language타입이어야 한다는 것을 알고 있다. 해당 타입에 문자열 리터럴 'JavaScript'는 할당 가능하므로 정상이다. but **이 값을 변수로 분리해내면, ts는 할당 시점에 타입을 추론한다.** 이번 경우는 string으로 추론했고, Language 타입으로 할당이 불가능하므로 오류가 발생했습니다.

이런 문제를 해결하는 두 가지 방법이 있다. 첫 번째 해법은 타입 선언에서 Language의 가능한 값을 제한하는 것이다.

```ts
let language: Language = 'JavaScript';
setLanguage(language); // 정상
```

const를 사용하여 타입 체커에게 language는 변경할 수 없다고 알려 준다. 따라서 ts는 language에 대해서 더 정확한 타입인 문자열 리터럴 'JavaScript'로 추론할 수 있다. 'JavaScript'는 Language 할당할 수 있으므로 타입 체크를 통과한다. 물론, language를 재할당해야 한다면 타입 선언이 필요하다.

그런데 이 과정에서 사용되는 문맥으로부터 값을 분리했다. 문맥과 값을 분리하면 추후에 근본적인 문제를 발생시킬 수 있다. 이제부터 이러한 문맥의 소실로 인해 오류가 발생하는 몇 가지 경우와 해결 방법을 살펴보겠다.

## 튜플 사용 시 주의점
문자열 리터럴 타입과 마찬가지로 튜플 타입에서도 문제가 발생한다. 이동이 가능한 지도를 보여 주는 프로그램을 작성한다고 생각해 보겠습니다.

```ts
// 매개변수는 (latitude, longitude) 쌍입니다.
function panTo(where: [number, number]) { ... }

panTo([10, 20]); // 정상

const loc = [10, 20];
panTo(loc); // number[] 형식의 인수는 [number, number]형식의 매개변수에 할당될 수 없다.
```

이전 예제처럼 여기서도 문맥과 값을 분리했다. 첫 번째 [10, 20]이 튜플 타입 [number, number]로 할당 가능하다. 두 번째 경우는 ts가 loc의 타입을 number[]로 추론하다. 많은 배열이 이와 맞지 않는 수의 요소를 가지므로 튜플 타입에 할당할 수 없다.

이를 고치려면
```ts
const loc: [number, number] = [10, 20];
panTo(loc); // 정상
```

또 다른 방법은 **상수 문맥**을 제공하는 것이다. const는 단지 값이 가리키는 참조가 변하지 않는 얕은 상수인 반면, **as const**는 그 값이 내부까지(deeply) 상수라는 사실을 ts에게 알려 준다.

```ts
const loc = [10, 20] as const;
panTo(loc); // readonly [10, 20] 형식은 readonly이며 변경 가능한 형식 [number, number]에 할당할 수 없다.
```

편집기에서 loc에 마우스를 올려 보면, 타입은 이제 number[]가 아니라 reaonly [10, 20]으로 추론됨을 알 수 있다. 그런데 안타깝게도 이 추론은 **너무 과하게** 정확하다. panTo의 타입 시그니처는 where의 내용이 불변이라고 보장하지 않는다. 즉, loc 매개변수가 readonly 타입이므로 동작하지 않는다.

따라서 any를 사용하지 않고 오류를 고칠 수 있는 최선의 해결책은 panTo 함수에 readonly 구문을 추가하는 것이다.

```ts
function panTo(where: readonly [number, number]) { ... }
const loc = [10, 20] as const;
panTo(loc); // 정상
```

타입 시그니처를 수정할 수 없는 경우라면 타입 구문을 사용해야 한다. `as const`는 문맥 손실과 관련한 문제를 깔끔하게 해결할 수 있지만, 한 가지 단점을 가지고 있다. 만약 타입 정의에 실수가 있다면, 오류는 타입 정의가 아니라 호출되는 곳에서 발생한다는 것이다. 특히 여러 겹 중첩된 객체에서 오류가 발생한다면 근본적인 원인을 파악하기 어렵다.

```ts
const loc = [10, 20, 30] as const; // 실제 오류는 여기서 발생한다.
panTo(loc); // readonly [10, 20, 30] 형식의 인수는 readonly [number, number] 형식의 매개변수에 할당될 수 없다. length 속성의 형식이 호환되지 않는다. 3형식은 2형식에 할당할 수 없다.
```

## 객체 사용 시 주의점
문맥에서 값을 분리하는 문제는 문자열 리터럴이나 튜플을 포함하는 큰 객체에서 상수를 뽑아낼 때도 발생한다.
```ts
type Language = 'JavaScript' | 'TypeScript' | 'Python';
interface GovernedLanguage {
  language: Language;
  organization: string;
}

function complain(language: GovernedLanguage) { ... }

complain({ language: 'JavaScript', organization: 'Microsoft' }); // 정상

const ts = {
  language: 'TypeScript',
  organization: 'Microsoft',
}
complain(ts); // { language: string; organization: string; } 형식의 인수는 GovernedLanguage 형식의 매개변수에 할당될 수 없다. language 속성의 형식이 호환되지 않는다. string 형식은 Language 형식에 할당할 수 없다.
```

ts객체에서 language의 타입은 string으로 추론된다. 이 문제는 타입 선언을 추가하거나, 상수 단언(as const)을 사용해 해결합니다.

## 콜백 사용 시 주의점
콜백을 다른 함수로 전달할 때, ts는 콜백의 매개변수 타입을 추론하기 위해 문맥을 사용한다.
```ts
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
  fn(Math.random(), Math.random());
}

callWithRandomNumbers((a, b) => {
  a; // number
  b; // number
  console.log(a + b);
});
```

callWithRandom의 타입 선언으로 인해 a와 b의 타입이 number로 추론된다. 콜백을 상수로 뽑아내면 문맥이 소실되고 noImplicitAny 오류가 발생하게 된다.

```ts
const fn = (a, b) => {
  a; // any
  b; // any
  console.log(a + b);
}
callWithRandomNumbers(fn);
```
이런 경우는 매개변수에 타입 구문을 추가해서 해결할 수 있다.

```ts
const fn = (a: number, b: number) => {
  console.log(a + b);
}
callWithRandomNumbers(fn);
```

또는 가능할 경우 전체 함수 표현식에 타입 선언을 적용하는 것이다.

## 요약
- 타입 추론에서 문맥이 어떻게 쓰이는지 주의해서 살펴봐야 한다.
- 변수를 뽑아서 별도로 선언했을 때 오류가 발생한다면 타입 선언을 추가해야 한다.
- 변수가 정말로 상수라면 상수 단언을 사용해야 한다. but 상수 단언을 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생하므로 주의해야 한다.
