# 아이템 25: 비동기 코드에는 콜백 대신 async함수 사용하기

과거 JS에서는 비동기 동작을 모델링하기 위해 콜백을 사용했다. 그렇기 때문에 악명 높은 `콜백 헬`을 필연적으로 마주할 수 밖에 없었다.
``` ts
fetchURL(url1, function(response1) {
  fetchURL(url1, function(response2) {
    fetchURL(url1, function(response3) {
      // ...
      console.log(1);
    });
    console.log(2);
  });
  console.log(3);
});
console.log(4);

// 4
// 3
// 2
// 1
```

로그에서 보면 알 수 있듯이, 실행의 순서는 코드의 순서와 반대이다. 이러한 콜백이 중첩된 코드는 직관적으로 이해하기 어렵다. 요청들을 병렬로 실행하거나 오류 상황을 빠져나오고 싶다면 더욱 혼란스러워진다.

ES2015는 콜백 지옥을 극복하기 위해 프로미스 개념을 도입했다. 프로미스는 미래에 가능해질 어떤 약속을 나타낸다.(future라고 부르기도 함)

> promise사용
``` ts
const page1Promise = fetch(url1);
page1Promise.then(response1 => {
  return fetch(url2);
}).then(response2 => {
  return fetch(url3);
}).then(response3 => {
  // ...
}).catch(error => {
  // ...
});
```
코도의 중첩도 적어졌고 실행 순서도 코드 순서와 같아졌다. 또한 오류를 처리하기도, `Promise.all`같은 고급 기법을 사용하기도 더 쉬워졌다.<br>
`ES2017`에서는 `async`와 `await` 키워드를 도입하여 콜백 지옥을 더욱 간단하게 처리할 수 있게 됐다.
``` ts
async function fetchPages() {
  const response1 = await fetch(url1);
  const response2 = await fetch(url2);
  const response3 = await fetch(url3);
}
```

`await`키워드는 각각의 프로미스가 처리(리졸브)될 때까지 `fetchPages`함수의 실행을 멈춘다. `async`함수 내에서 `await`중인 프로미스가 거절(리젝트)되면 예외를 던진다. 이를 통해 일반적인 `try/catch`구문을 사용할 수 있다.

``` ts
async function fetchPages() {
  try {
    const response1 = await fetch(url1);
    const response2 = await fetch(url2); 
    const response3 = await fetch(url3);
  } catch {
    // ...
  }
}
```

`ES5`또는 더 이전 버전을 대상으로 할 때, TS 컴파일러는 `async`와 `await`가 동작하도록 정교한 변환을 수행한다. 다시 말해 TS는 런타임에 관계없이 `async/await`을 사용할 수 있다.

콜백보다는 프로미스, `async/await`를 사용해야 하는 이유이다.
- 콜백보다는 프로미스가 코드를 작성하기 쉽다.
- 콜백보다는 프로미스가 타입을 추론하기 쉽다.

> 병렬로 페이지 로드 -> Promise.all 사용
``` ts
async function fetchPages() {
  const [res1, res2, res3] = await Promise.all([
    fetch(url1), fetch(url2), fetch(url3)
  ]);
}
```
이 경우 `await`와 구조 분해 할당이 찰떡궁합니다.<br>
TS는 세 가지 response 변수 각각의 타입을 `Response`로 추론한다. but 콜백 스타일로 동일한 코드를 작성하려면 더 많은 코드와 타입 구문이 필요하다.
``` ts
function fetchPagesCB() {
  let numDone = 0;
  const responses: string[] = [];
  const done = () => {
    const [res1, res2, res3] = responses;
    // ...
  };
  const urls = [url1, url2, url3];
  urls.forEach((url, i) => {
    fetchURL(url, r => {
      responses[i] = url;
      numDone++;
      if(numDone === urls.length) done();
    });
  });
}
```

이 코드에 오류 처리를 포함하거나 `Promise.all`같은 일반적인 코드로 확장하는 것은 쉽지 않다.

한편 입력된 프로미스들 중 첫 번째가 처리될 때 완료되는 `Promise.race`도 타입 추론과 잘 맞다. `Promise.race`를 사용하여 프로미스에 타입아웃을 추가하는 방법은 흔하게 사용되는 패턴이다.

``` ts
function timeouot(millis: number): Promise<never> {
  return new Promise((resolve, reject) => {
    setTimeout(() => reject('timeout'), millis);
  });
}

async function fetchWithTImeout(url: string, ms: number) {
  return Promise.race([fetch(url), timeout(ms)]);
}
```

타입 구문이 없어도 `fetchWithTimeout`의 반환 타입은 `Promise<Response>`로 추론된다. 추론이 동작하는 이유를 살펴보면 흥미로운 점을 발견할 수 있다. `Promise.race`의 반환 타입은 입력 타입들의 유니온이고, 이번 경우는 `Promise<Response | never>`가 된다. but never(공집합)와의 유니온은 아무런 효과가 없으므로, 결과가 `Promise<Response>`로 간단해진다. 프로미스를 사용하면 TS의 모든 타입 추론이 제대로 동작한다.<br>
가끔 프로미스를 직접 생성해야 할 때, 특히 `setTimeout`과 같은 콜백 API를 래핑할 경우가 있다. but 선택의 여지가 있다면 일반적으로는 프로미스를 생성하기보다는 `async/await`을 사용해야 한다. 그 이유는 다음 두 가지다.
- 일반적으로 더 간결하고 직관적인 코드가 된다.
- async 함수는 항상 프로미스를 반환하도록 강제된다.

``` ts
// function getNumber(): Promise<number>
async function getNumber() {
  return 42;
}
```
async 화살표 함수를 만들 수도 있다.
``` ts
const getNumber = async = () => 42; // 타입이 () => Promise<number>
```

프로미스를 직접 생성하면 다음과 같다.
``` ts
const getnUmber = () => Promise.resolve(42); // 타입이 () => Promise<number>
```

즉시 사용 가능한 값에도 프로미스를 반환하는 것이 이상하게 보일 수 있지만, 실제로는 비동기 함수로 통일하도록 강제하는 데 도움이 된다. 함수는 항상 동기 또는 비동기로 실행되어야 하며 절대 혼용해서는 안된다. 예를 들어, `fetchURL`함수에 캐시를 추가하기 위해 다음처럼 시도해 봤다고 가정해본다.
``` ts
// 이렇게 하지 말 것!
const _cache: { [url: string]: string } = {};
function fetchWithCache(url: string, callback: (text: string) => void) {
  if(url in _cache) {
    callback(_cache[url]);
  } else {
    fetchURL(url, text => {
      _cache[url] = text;
      callback(text);
    });
  }
}
```
코드가 최적화된 것처럼 보일지 몰라도, 캐시된 경우 콜백 함수가 동기로 호출되기 때문에 `fetchWithCache`함수는 이제 사용하기가 무척 어려워진다.

``` ts
let requestStatus: 'loading' | 'success' | 'error';
function getUser(userId: string) {
  fetchWithCache(`/user/${userId}`, profile => {
    requestStatus = 'success';
  });
  requestStatus = 'loading';
}
```
`getUser`를 호출한 후에 `requestStatus`의 값은 온전히 `profile`이 캐시되었는지 여부에 달렸다.