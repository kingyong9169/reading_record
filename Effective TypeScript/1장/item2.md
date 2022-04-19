# 아이템 2: TS 설정 이해하기
다음 코드가 오류 없이 타입 체커를 통과할 수 있을지 생각해 보겠습니다.
``` ts
function add(a, b) {
  return a + b;
}
add(10, null);
```

설정이 어떻게 되어 있는지 모른다면 대답할 수 없는 질문입니다. TS컴파일러는 매우 많은 설정을 갖고 있습니다. 현재 시점에서는 설정이 거의 100개에 이릅니다.<br>
이 설정들은 커맨드 라인에서 사용할 수 있습니다.<br>
`$ tsc --noImplicitAny program.ts`

`tsconfig.json` 설정 파일을 통해서도 가능합니다.
``` json
{
  "compilerOptions": {
    "noImplicitAny": true
  }
}
```
`가급적 설정 파일을 사용하는 것이 좋다.` 그래야만 TS를 어떻게 사용할 계획인지 동료들이나 다른 도구들이 알 수 있다. 설정 파일은 `tsc --init`만 실행하며 간단히 생성된다.

TS의 설정들은 어디서 소스 파일을 찾을지, 어떤 종류의 출력을 생성할지 제어하는 내용이 대부분. `그런데 언어 자체의 핵심 요소들을 제어하는 설정도 있다. 대부분의 언어에서는 허용하지 않는 고수준 설계의 설정이다.` 설정을 제대로 사용하려면, `noImplicitAny`와 `strictNullChecks`를 이해해야 한다.<br>

## noImplicitAny
`noImplicitAny`는 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어한다. 위의 `add`코드는 `noImplicitAny`이 `false`일 때 유효하다.<br>
이 함수의 타입은 `function add(a: any, b: any): any`이다.<br>
any 타입을 매개변수에 사용하면 타입 체커는 속절없이 무력해진다. any는 유용하지만 매우 주의해서 사용해야 한다.

그런데 같은 코드임에도 `noImplicitAny`가 설정되었다면 오류가 된다. _이 오류들은 명시적으로 `: any`라고 선언해 주거나 더 분명한 타입을 사용하면 해결할 수 있다._
``` ts
function add(a: number, b: number) {
  return a + b;
}
```

TS는 타입 정보를 가질 때 가장 효과적이기 때문에, 되도록이면 `noImplicitAny`를 설정해야 한다. 그러면 TS가 문제를 발견하기 수월해지고, 코드의 가독성이 좋아지며, 개발자의 생산성이 향상된다. **참고로 `noImplicitAny` 해제는 JS로 되어 있는 기존 코드를 TS로 마이그레이션할 때에만 필요하다.**

## strictNullChecks
null과 undefined가 모든 타입에서 허용되는지 확인하는 설정이다.
`const x: number = null;`
- false일 때
  - 정상
- true일 때
  - null형식은 number형식에 할당할 수 없습니다.

null대신 undefined를 써도 같은 오류가 난다. 만약 null을 허용하려고 한다면 의도를 명시적으로 드러냄으로써 오류를 고칠 수 있다.<br>
`const x: number | null = null;`
만약 null을 허용하지 않으려면, 이 값이 어디서부터 왔는지 찾아야 하고, null을 체크하는 코드나 단언문을 추가해야 한다.

``` ts
const el = document.getElementById('status');
// el.textContent = 'Ready' => null인 것 같습니다.
if(el) el.textContent = 'Ready'; // null 제외
el!.textContent = 'Ready' // el이 null이 아님을 단언한다.
```

`strictNullChecks`는 nullrhk undefined 관련된 오류를 잡아 내는 데 많은 도움이 되지만, 코드 작성을 어렵게 한다. `strictNullChecks`를 설정하려면 `noImplicitAny`를 먼저 설정해야 한다.<br>
`strictNullChecks` 설정 없이 개발하기로 선택했다면 `undefined는 객체가 아닙니다.`라는 끔찍한 런타임 오류를 주의해야 한다. 프로젝트가 거대해질수록 설절 반경은 어려워질 것이므로, 가능한 한 초반에 설정하는 게 좋다.<br>
만약 언어에 의미적으로 영향을 미치는 설정들을(noImplicitThis, strictFunctionTypes 등) 체크하고 싶다면 `strict`설정을 하면 된다. 대부분의 오류를 잡아내줄 것이다.
