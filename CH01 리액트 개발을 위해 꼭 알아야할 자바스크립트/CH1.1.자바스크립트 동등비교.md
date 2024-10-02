# 📖 1.1. 자바스크립트의 동등비교

자바스크립트의 동등비교는

> 1. 리액트 함수형 컴포넌트의 훅을 사용할때 의존성 배열의 동작원리
> 2. 재렌더링이 일어나는 이유 (props 의 동등 비교 : 객체의 얕은 비교)

에서 사용되기에 중요하다.

## ✏️1.1.1. 자바스크립트의 데이터 타입과 값을 저장하는 방식의 차이

크게 원시타입과 객체 타입으로 나눌 수 있다.

|                    | 원시타입                                                               | 객체타입                                                                                                    |
| ------------------ | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| 종류               | `boolean`, `null`, `undefined`, `number`, `string`, `symbol`, `bigint` | 배열, 함수, 정규식, 클래스 등 원시타입이 아닌 모든 것                                                       |
| 값을 저장하는 방식 | 불변 형태의 값.<br/> 변수 할당시점에 메모리 영역을 차지하고 저장됨     | 객체는 프로퍼티의를 변경할 수 있기 때문에 변경 가능한 형태로 저장. <br/> 복사시에도 값이 아닌 참조를 전달함 |

## ✏️1.1.2. 자바스크립트 동등비교

### 동등비교 연산자

#### 1. `==` 연산자

동등 비교 전, 좌항과 우항이 동일 타입이 아닌 경우 `강제로 타입 캐스팅 후 비교` <br/>

```js
5 == "5"; // true
```

#### 2. `===` 연산자

동등 비교시, 좌항과 우항이 동일 타입이 아닌 경우 `false` 반환 <br/>

```js
5 === "5"; // false
```

3. `Object.is()` 메서드

`===` 와 유사하지만, 만족하지 않는 특이 케이스에 대해 다른 결과를 반환

```js
-0 === +0; // true
Object.is(-0, +0); // false

Number.NaN == NaN; // false
Object.is(Number.NaN, NaN); // true

NaN === 0 / 0; // false
Object.is(NaN, 0 / 0); // true

Object.is({}, {}); // false

const a = { key: "value" };
const b = a;
Object.is(a, b); // true

a === b; // true
```

### 객체의 동등비교

객체는 값이 아닌 `참조`를 저장하기 때문에, 동일하게 선언한 객체라도 `다른 참조를 바라보기 때문에` `false` 를 반환

### 리액트의 동등비교

리액트에서는 `Object.is()` 를 사용하는 `shallowEqual()` 을 통해 동등비교를 진행한다 <br/>
`Object.is()` 를 실행하지 못하는 경우, 객체 간의 얕은 비교를 수행.

❗️ 얕은 비교 : 객체의 첫 번째 깊이에 존재하는 값만 비교 <br/>
(일반적인 케이스에서 props 는 얕은 비교로도 충분하기 때문)

❗️ props 내부에 또 다른 객체를 넘겨줄 경우, 리액트 렌더링이 예상치 못하게 작동할 수 있음

❗️ props 가 깊어지는 경우, React.memo 는 컴포넌트에 실제 변경된 값이 없음에도 메모이제이션 된 컴포넌트 반환 안될 수 있음

❗️ `hasOwnProperty` : https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty

```ts
function shallowEqual(objA, objB): boolean {
    // 1. Object.is 실행
    if (Object.is(objA, objB)) return true;

    if (typeof objA !== "object" || objA === null || typeof objB !== "object" || objB === null) return false;

    // 2. 두 객체의 key 개수가 다르면 false 반환
    const keysA = Object.keys(objA);
    const keysB = Object.keys(objB);
    if (keysA.length !== keysB.length) return false;

    // 3. objA 의 키를 기준으로 objB 에 같은 키가 있고, 값이 동일한지 확인
    for (let i = 0; i < keysA.length; i++) {
        const currentKey = keysA[i];
        if (!hasOwnProperty.call(objB, currentKey) || !Object.is(objA[currentKey], objB[currentKey])) return false;
    }
    return true;
}
```
