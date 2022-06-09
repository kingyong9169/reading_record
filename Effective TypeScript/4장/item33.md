# 아이템 33: string 타입보다 더 구체적인 타입 사용하기

## 문자열을 나말하여 선언된 코드를 피하자. 모든 문자열을 할당할 수 있는 string타입보다는 더 구체적인 타입을 사용하는 것이 좋다.
string 타입의 범위는 매우 넓다. string타입으로 변수를 선언하려 한다면, 혹시 그보다 더 좁은 타입이 적절하지는 않을지 검토해 보아야 한다.<br>
음악 컬렉션을 만들기 위해 앨범의 타입을 정의한다고 가정한다.
``` ts
interface Album {
  artist: string;
  title: string;
  releaseDate: string; // YYYY-MM-DD
  recordingType: string; // "live" or "studio"
}
```
string 타입이 남발된 모습니다. 게다가 주석에 타입 정보를 적어 둔 걸 보면 현재 interface가 잘못되었다는 것을 알 수 있다. 다음 예시처럼 Album 타입에 엉뚱한 값을 설정할 수 있다.

``` ts
const kindOfBlue: Album = {
  artist: 'Miles Davis',
  title: 'Kind of Blue',
  releaseDate: 'August 17th, 1959', // 날짜 형식이 다름
  recordingType: 'studio', // 대문자여서 오타
} // but 정상
```

`releaseDate` 필드의 값은 주석에 설명된 형식과 다르며, `recordingType` 필드의 값 `"studio"`는 소문자 대신 대문자가 쓰였다. but 이 두 값 모두 문자열이고, 해당 객체는 `Album` 타입에 할당 가능하며 타입 체커를 통과한다. 또한, `string` 타입의 번위가 넓기 때문에 제대로 된 `Album` 객체를 사용하더라도 매개변수 순서가 잘못된 것이 오류로 드러나지 않는다.
``` ts
function recordRelease(title: string, date: string) {}
recordRelease(kindOfBlue.releaseDate, kindOfBlue.title); // 오류여야 하지만 정상
```

`recordRelease` 함수의 호출에서 매개변수들의 순서가 바뀌었지만, 둘 다 문자열이기 때문에 타입 체커가 정상으로 인식한다. `string` 타입이 남용된 코드를 `문자열을 남발하여 선언되었다.`고 표현하기도 한다.

## 변수의 범위를 보다 정확히 표현하기 위해서는 유니온 타입을 사용하면 된다. 타입 체크를 더 엄격히 할 수 있고 생산성을 향상시킬 수 있다.
앞의 오류를 방지하기 위해 타입의 범위를 좁히는 방법이 좋다. `releaseDate`필드는 `Date`객체를 사용해서 날짜 형식으로만 제한하는 것이 좋다. `recordingType`은 `"live"`, `"studio"` 단 두 개의 값으로 유니온 타입을 정의할 수 있다.(`enum`을 사용할 수도 있지만 일반적으로 추천하지 않는다. 아이템 53 참고)

``` ts
type RecordingType = "live" | "studio";

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}
```

이런 방식에는 3가지 장점이 있다.
1. 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지된다.<br>
예를 들어, 특정 레코딩 타입의 앨범을 찾는 함수를 작성한다.
``` ts
function getAlbumOfType(recordingType: string): Album[] {
  //...
}
```
`getAlbumOfType`함수를 호출하는 곳에서 `recordingType`의 값이 `string`타입이어야 한다는 것 외에는 다른 정보가 없다. 함수를 사용하는 사람은 `recordingType`이 `"studiio"`, `"live"`여야 한다는 것을 알 수 없다.

2. 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있고 편집기에서 확인할 수 있다.
``` ts
/* 이 녹음은 어떤 환경에서 이루어졌는지? */
type RecordingType = "live" | "studio";
```
위 함수에서 매개변수 타입을 `string`대신 `RecordingType`으로 바꾸면, 함수를 사용하는 곳에서 `RecordingType`의 설명을 볼 수 있다.

3. `keyof` 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해진다.<br>
함수의 매개변수에 `string`을 잘못 사용하는 일은 흔하다. 어떤 배열에서 한 필드의 값만 추출하는 함수를 작성한다고 생각해 본다.
``` ts
function pluck(records, key) {
  retirm records.map(r => r[key]);
}
```

`pluck` 함수의 시그니처를 다음처럼 작성할 수 있다.
``` ts
function pluck(records: any[], key: string): any[] {
  return records.map(r => r[key]);
}
```

타입 체크가 되긴 하지만 `any` 타입이 있어서 정밀하지 못하다. 특히 반환 값에 `any`를 사용하는 것은 매우 좋지 않은 설계이다. 제너릭 타입을 도입하여 이를 개선한다.
``` ts
function pluck<T>(records: T[], key: string): any[] {
  return records.map(r => r[key]); // {} 형식에 인덱스 시그니처가 없으므로 요소에 암시적으로 any 형식이 있다.
}
```
이제 TS는 `key`의 타입이 `string`이기 때문에 범위가 너무 넓다는 오류를 발생시킨다. `Album`의 배열을 매개변수로 전달하면 기존의 `string`타입의 넓은 범위와 반대로, `key`는 단 4개의 값(`"artist", "title", "releaseDate", "recordingType"`)만이 유효하다.

## 객체의 속성 이름을 함수 매개변수로 받을 때는 string보다 keyof T를 사용하는 것이 좋다.
여기서 `keyof`를 사용하여 해결한다.
``` ts
function pluck<T>(records: T[], key: keyof T) {
  return records.map(r => r[key]); // {} 형식에 인덱스 시그니처가 없으므로 요소에 암시적으로 any 형식이 있다.
}
```
이 코드는 타입 체커를 통과한다. 또한 TS가 반환 타입을 추론할 수 있게 해 준다.
``` ts
function pluck<T>(records: T[], key: keyof T): T[keyof T][]
```

T[keyof T]는 T 객체 내의 가능한 모든 값의 타입이다. but key의 값으로 하나의 문자열을 넣게 되면, 그 범위가 너무 넓어서 적절한 타입이라고 보기 어렵다.
``` ts
const releaseDates = pluck(albums, 'releaseDate'); // 타입 (string | Date)[]
```

releaseDates의 타입은 (string | Date)[]가 아니라 Date[]이어야 한다. 이런 의미에서 keyof T는 string에 비하면 훨씬 범위가 좁기는 하지만 그래도 여전히 넓다. 따라서 범위를 더 좁히기 위해서, keyof T의 부분 집합(아마도 단일 값)으로 두 번째 제너릭 매개변수를 도입해야 한다.
``` ts
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map(r => r[key]);
}
```

``` ts
pluck(albums, 'releaseDate'); // Date[]
pluck(albums, 'artist'); // string[]
pluck(albums, 'recordingType'); // RecordingType[]
```

매개변수 타입이 정밀해진 덕분에 언어 서비스는 Album의 키에 자동 완성 기능을 제공할 수 있게 해 준다.<br>
string은 any와 비슷한 문제를 가지고 있다. 따라서 잘못 사용하게 되면 무효한 값을 허용하고 타입 간의 관계도 감추어 버린다. 이러한 문제점은 타입 체커를 방해하고 실제 버그를 찾지 못하게 만든다. `TS에서 string의 부분 집합을 정의할 수 있는 기능은 js코드에서 타입 안전성을 크게 높인다.` 보다 정확한 타입을 사용하면 오류를 방지하고 코드의 가독성도 향상시킨다.
