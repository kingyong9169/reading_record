# 아이템 35 : 데이터가 아닌, API와 명세를 보고 타입 만들기

## 코드의 구석 구석까지 타입 안전성을 얻기 위해 API 또는 데이터 형식에 대한 타입 생성을 고려해야 한다.
파일 형식, API, 명세 등 우리가 다루는 타입 중 최소한 몇 개는 프로젝트 외부에서 비롯된 것이다. 이러한 경우는 타입을 직접 작성하지 않고 자동으로 생성할 수 있다. 여기서 핵심은, 예시 데이터가 아니라 명세를 참고해 타입을 생성한다는 것이다. 명세를 참고해 타입을 생성하면 TS는 사용자가 실수를 줄일 수 있게 도와준다. 반면에 예시 데이터를 참고해 타입을 생성하면 눈앞에 있는 데이터들만 고려하게 되므로 예기치 않은 곳에서 오류가 발생할 수 있다.

``` ts
...
const { geometry } = f;
if(geometry) {
  if(geometry.type === 'GeometryCollection') {
    throw new Error('GeometryCollections are not supported.');
  }
  helper(geometry.coordinates); // 정상
}
```

TS는 타입을 체크하는 방법으로 도형의 타입을 정제할 수 있으므로 정제된 타입에 한해서 `geometry.coordinates`의 참조를 허용하게 된다. 차단된 `GeometryCollection`타입의 경우, 사용자에게 명확한 오류 메시지를 제공한다. but `GeometryCollection`타입을 차단하기보다는 모든 타입을 지원하는 것이 더 좋은 방법이기 때문에 조건을 분기해서 헬퍼 함수를 호출하면 모든 타입을 지원할 수 있다.
``` ts
...
const geometryHelper = (g: Geometry) => {
  if(geometry.type === 'GeometryCollection') {
    geometry.geometries.forEach(geometryHelper);
  } else {
    helper(geometry.coordinates); // 정상
  }
}

const { geometry } = f;
if(geometry) {
  geometryHelper(geometry);
}
```

API의 명세로부터 타입을 생성할 수 있다면 그렇게 하는 것이 좋다. 특히 GraphQL처럼 자체적으로 타입이 정의된 API에서 잘 동작한다.<br>
GraphQL API는 TS와 비슷한 타입 시스템을 사용하여, 가능한 모든 쿼리와 인터페이스를 명세하는 스키마로 이루어진다. 우리는 이러한 인터페이스를 사용해서 특정 필드를 요청하는 쿼리를 작성한다. 예를 들어, Github GraphQL API를 사용해서 저장소에 대한 정보를 얻는 코드는 다음처럼 작성할 수 있다.
``` ts
query {
  repository(owner: "Microsoft", name: "TypeScript") {
    createdAt
    description
  }
}

// 결과
{
  "data": {
    "repository": {
      "createdAt": "2014-06..",
      "description": "TypeScript is a.."
    }
  }
}
```

GraphQL의 장점은 특정 쿼리에 대해 TS 타입을 생성할 수 있다는 것이다. GeoJSON 예제와 마찬가지로 GraphQL을 사용한 방법도 타입에 null이 가능한지 여부를 정확하게 모델링할 수 있다. 다음 예제는 Github에서 오픈소스 라이선스를 조회하는 쿼리이다.
``` ts
query getLicense($owner: String!, $name: String!) {
  repository(owner: $owner, name: $name) {
    description
    licenseInfo {
      spdxId
      name
    }
  }
}
```
$owner와 $name은 타입이 정의된 GraphQL변수이다. 타입 문법이 TS와 매우 비슷하다. String은 GraphQL의 타입이다. TS에서는 string이 된다. 그리고 TS에서 string 타입은 null이 불가능하지만 GraphQL의 String타입에서는 null이 가능하다. 타입 뒤!는 null이 아님을 명시한다. GraphQL 쿼리를 TS 타입으로 변환해 주는 많은 도구가 존재한다. 그 중 하나는 Apollo이다. 다음은 Apollo를 어떻게 사용하는지 보여 준다.

``` ts
$ apollo client:codegen \
    --endpoint https://api.. \
    --includes license.graphql \
    --target typescript
```
쿼리에서 타입을 생성하려면 GraphQL 스키마가 필요하다. Apollo는 `api..`으로부터 스키마를 얻는다. 실행의 결과는 다음과 같다.
``` ts
export interface getLicense_repository_licenseInfo {
  __typename: "License";
  spdxId: string | null;
  name: string;
}

export interface getLicense_repository {
  __typename: "Repository";
  description: string | null;
  // 주석
  licenseInfo: getLicense_repository_licenseInfo | null;
}

export interface getLicense {
  // 주석
  repository: getLicense_repository_licenseInfo | null;
}

export interface getLicenseVariables {
  owner: string;
  name: string;
}
```

주목할 만한 점은 다음과 같다.
- 쿼리 매개변수(getLicenseVariables)와 응답 모두 인터페이스(getLicense)가 생성되었다.
- null 가능 여부는 스키마로부터 응답 인터페이스로 변환되었다. repository, description, licenseInfo, spdxId 속성은 null이 가능한 반면, name과 쿼리에 사용된 변수들은 그렇지 않다.
- 편집기에서 확인할 수 있도록 주석은 JSDoc으로 변환되었다. 이 주석들은 GraphQL스키마로부터 생성되었다.

## 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 명세로부터 코드를 생성하는 것이 좋다.
자동으로 생성된 타입 정보는 API를 정확히 사용할 수 있도록 도와준다. 쿼리가 바뀐다면 타입도 자동으로 바뀌며 스키마가 바뀐다면 타입도 자동으로 바뀐다. 타입은 단 하나의 원천 정보인 GraphQL스키마로부터 생성되기 때문에 타입과 실제 값이 항상 일치한다.<br>
만약 명세 정보나 공식 스키마가 없다면 데이터로부터 타입을 생성해야 한다. 이를 위해 quicktype같은 도구를 사용할 수 있다. but 생성된 타입이 실제 데이터와 일치하지 않을 수 있다는 점을 주의해야 한다. 예외적인 경우가 존재할 수 있다.

우리는 이미 자동 타입 생성의 이점을 누리고 있다. 브라우저 DOM API에 대한 타입 선언은 공식 인터페이스로부터 생성되었다. 이를 통해 복잡한 시스템을 정확히 모델링하고 TS가 오류나 코드상의 의도치 않은 실수를 잡을 수 있게 한다.
