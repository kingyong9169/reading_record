# 아이템 24:일관성 있는 별칭 사용하기


## 별칭은 TS가 타입을 좁히는 것을 방해한다. 따라서 변수에 별칭을 사용할 때는 일관되게 사용해야 한다.
> 어떤 값에 새 이름 할당 예제
``` ts
const borough = { name: 'Brooklyn', location: [40.688, -73.979] };
const loc = borough.location;
```

`borough.location` 배열에 `loc`이라는 별칭을 만들었다. 별칭의 값을 변경하면 원래 속성값에서도 변경된다.

그런데 별칭을 남발해서 사용하면 제어 흐름을 분석하기 어렵다. 모든 언어의 컴파일러 개발자들은 무분별한 별칭 사용으로 골치를 썩고 있다. TS에서도 마찬가지로 별칭을 신중하게 사용해야 합니다. 그래야 코드를 잘 이해할 수 있고, 오류를 쉽게 찾을 수 있다.

> 다각형을 표현하는 자료구조
``` ts
interface Coordinate {
  x: number;
  y: number;
}

interface BoundingBox {
  x: [number, number];
  y: [number, number];
}

interface Polygon {
  exterior: Coordinate[];
  holes: Coordinate[][];
  bbox?: BoundingBox;
}
```

다각형의 가하학적 구조는 `exterior`와 `holes` 속성으로 정의된다. `bbox`는 필수가 아니라 최적화 속성이다. `bbox`속성을 사용하면 어떤 점이 다각형에 포함되는지 빠르게 체크할 수 있다.

``` ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  if(polygon.bbox) {
    if(pt.x < polygon.bbox.x[0] || pt.x > polygon.bbox.x[1] ||
        pt.y < polygon.bbox.y[0] || pt.y > polygon.bbox.y[1]) {
      return false;      
    }
  }
}
```
이 코드는 잘 동작하지만(타입 체크도 통과) 반복되는 부분이 존재한다. 특히 `polygon.bbox`는 3줄에 걸쳐 5번이나 등장한다.

> 중복을 줄이기 위해 임시 변수 사용
``` ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if(polygon.bbox) {
    if(pt.x < box.x[0] || pt.x > box.x[1] || // 객체가 undefined일 수 있다.
        pt.y < box.y[0] || pt.y > box.y[1]) { // 객체가 undefined일 수 있다.
      return false;      
    }
  }
}
```
(`strickNullChecks`를 활성화했다고 가정)

이 코드는 동작하지만 편집기에서 오류로 표시된다. 그 이유는 `polygon.bbox`를 별도의 별칭으로 만들었고 제어 흐름 분석을 방해했기 때문이다.

> box, polygon.bbox의 타입 조사
``` ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  polygon.bbox // BoundingBox | undefined
  const box = polygon.bbox;
  box // BoundingBox | undefined
  if(polygon.bbox) {
    polygon.bbox // BoundingBox
    box // BoundingBox | undefined
    if(pt.x < box.x[0] || pt.x > box.x[1] || // 객체가 undefined일 수 있다.
        pt.y < box.y[0] || pt.y > box.y[1]) { // 객체가 undefined일 수 있다.
      return false;      
    }
  }
}
```

속성 체크는 `polygon.bbox`의 타입을 정제했지만 `box`는 그렇지 않았기 때문에 오류가 발생했다. 이러한 오류는 `별칭은 일관성 있게 사용한다`는 기본 원칙을 지키면 방지할 수 있다.

> 속성 체크에 box를 사용
``` ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if(box) {
    if(pt.x < box.x[0] || pt.x > box.x[1] ||
        pt.y < box.y[0] || pt.y > box.y[1]) {
      return false;      
    }
  }
}
```

## 비구조화 문법을 사용해서 일관된 이름을 사용하는 것이 좋습니다.
타입 체커의 문제는 해결되었지만 코드를 읽는 사람에게는 문제가 남아 있다. `box`, `bbox`는 같은 값인데 다른 이름을 사용한 것이다. 객체 비구조화를 이용하면 보다 간결한 문법으로 일관된 이름을 사용할 수 있다. 배열과 중첩된 구조에서도 역시 사용할 수 있다.
``` ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const { bbox } = polygon;
  if(bbox) {
    if(pt.x < box.x[0] || pt.x > box.x[1] ||
        pt.y < box.y[0] || pt.y > box.y[1]) {
      return false;      
    }
  }
}
```

but 객체 비구조화를 이용할 때 두 가지를 주의해야 한다.
1. 전체 `bbox`속성이 아니라 x와 y가 선택적 속성일 경우에 속성 체크가 더 필요하다. 따라서 타입의 경계에 null값을 추가하는 것이 좋다.
2. `bbox`에는 선택적 속성이 적합했지만 `holes`는 그렇지 않다. `holes`가 선택적이라면, 값이 없거나 빈 배열([])이었을 것이다. 차이가 없는데 이름을 구별한 것이다. 빈 배별은 `holes없음`을 나타내는 좋은 방법이다.

## 함수 호출이 객체 속성의 타입 정제를 무효화할 수 있다는 점을 주의해야 한다. 속성보다 지역 변수를 사용하면 타입 정제를 믿을 수 있다.
별칭은 타입 체커뿐만 아니라 런타임에도 혼동을 야기할 수 있다.
``` ts
const { bbox } = polygon;
if(!bbox) {
  calculatePolygonBbox(polygon); // polygon.bbox가 채워진다.
  // 이제 polygon.bbox와 bbox는 다른 값을 참조한다.
}
```

TS의 제어 흐름 분석은 지역 변수에는 꽤 잘 동작한다. but 객체 속성에서는 주의해야 한다.

``` ts
function fn(p: Polygon) { /* ... */ }

polygon.bbox // BoundingBox | undefined
if(polygon.bbox) {
  polygon.bbox // BoundingBox
  fn(polygon);
  polygon.bbox // BoundingBox
}
```

fn(polygon) 호출은 `polygon.bbox`를 제거할 가능성이 있으므로 타입을 `BoundingBox | undefined`로 되돌리는 것이 안전할 것이다. but 함수를 호출할 때마다 속성 체크를 반복해야 하기 때문에 좋지 않다. 그래서 TS는 함수가 타입 정제를 무효화하지 않는다고 가정한다. but 실제로는 무효화될 가능성이 있다. `polygon.bbox`로 사용하는 대신 `bbox` 지역 변수로 뽑아내서 사용하면 `bbox`의 타입은 정확히 유지되지만, `polygon.bbox`의 값과 같게 유지되지 않을 수 있다.
