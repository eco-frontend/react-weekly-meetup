state는 객체를 포함한 모든 종류의 자바스크립트 값을 가질 수 있다.

하지만 리액트 state를 직접 변경해선 안된다. 

객체를 업데이트하고 싶을 때 하는 방법을 알아보도록 하겠습니다.

> 학습 내용
> 
- 리액트 state에서 객체를 올바르게 업데이트하는 방법
- 중첩된 객체를 변경하지 않고 업데이트하는 방법
- 불변성이랑 무엇인지, 불변성을 지키는 방법
- Immer로 반복성을 줄여 객체를 복사하는 방법

## 변경(mutation)이란?

지금까지 숫자, 문자열, 논리형(boolean)을 다루었다.

- 자바스크립트 값들을 변경할 수 없거나, “읽기 전용”을 의미하는 “불변성”을 가진다.
- 값을 교체하기 위해서는 리렌더링이 필요하다.

```jsx
const [x, setX] = useState(0);
setX(5);
```

위 코드에서 보았듯 X는 0 → 5로 변경되었지만, 숫자 0은 바뀌지 않았다.

숫자, 문자열, boolean과 같이 자바스크립트에 정의되어있는 값들은 변경할 수 없다.

- 내장 type들은 바꿀 수 없다.

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
position.x = 5;
```

위코드에서 이러한 객체가 있다고 했을 때, 객체 그 자체의 내용을 바꿀 수 있다.

이것을 `mutation`이라고 한다. 

- State의 객체들이 기술적으로 변경(mutable)이 가능할지라도, 불변성(immutable) 한 것 처럼 다뤄야한다.
- 객체를 변경하는 대신 교체해야한다.

## State를 읽기 전용처럼 다루세요.

- state에 저장한 자바스크립트 객체는 어떤 것 이라도 **읽기 전용**처럼 다뤄야 한다.

```jsx
import { useState } from 'react';
export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        position.x = e.clientX;
        position.y = e.clientY;
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  )
}

```

- 위와 같은 코드가 있을 때 (preview 영역을 누르거나, 커서를 움직일 때 빨간 점이 이동해야 한다.
- 하지만 움직이지 않고, 초기 위치에 있다.

```jsx
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}
```

- 위 코드는 [이전 렌더링](https://ko.react.dev/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time)에서 position State에 할당된 객체를 바꾼다.
- 하지만, setState를 사용하지 않으면 리액트는 객체가 바뀌었는지 알 수 없다.
    - 그러므로 아무것도 하지 않는다.

> 마치 식사를 한 뒤에 주문을 바꾸려고 하는 것과 같다.
> 

- 이러한 경우 [리렌더링을 발생시키면,](https://react.dev/learn/state-as-a-snapshot#setting-state-triggers-renders) 새 객체를 생성하여 state 함수로 전달해라

```jsx
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

- setPosition은 React에 다음과 같이 요청한다.
    - position을 이 새로운 객체로 교체하라
    - 그리고 이 컴포넌트를 다시 렌더링하라

이제 영역을 누르거나, hover 시 빨간점이 포인터를 따라오는 것을 볼 수 있다.

<aside>
👌 지역 변경은 괜찮다.

</aside>

### DEEP DIVE - 지역 변경은 괜찮습니다.

- 아래 코드는 방금 생성한 새로운 객체를 변경하기 때문에 적절하다.

```jsx
const nextPosition = {};
nextPosition.x = e.clientX;
nextPosition.y = e.clientY;
setPosition(nextPosition);
```

- 위 코드는 아래처럼 작성할 수 있다.

```jsx
setPosition({
  x: e.clientX,
  y: e.clientY
});
```

- 변경(mutation)은 이미 state에 존재하는 객체를 변경할 때 문제가 된다.
- 방금 만든 객체를 수정하는 것은 아직 다른 코드가 해당 객체를 참조하지 않았기 때문에 괜찮다.
- 위 내용을 “지역 변경 local mutation” 이라고 한다.

## Spread(전개) 문법으로 객체 복사하기

- 이전 예시에서 position 객체는 현재 커서 위치에 항상 새롭게 생성된다.
- 존재하는 데이터를 새로 생성하는 객체에 포함하고 싶을 수 있다.
    - 단 한개의 필드만 수정하고, 나머지 모든 필드는 이전 값을 유지하고 싶을 때

```jsx

  firstName: e.target.value, // input의 새로운 first name
  lastName: person.lastName,
  email: person.email

 function handleFirstNameChange(e) {
    person.firstName = e.target.value;
 }
```

- 위와 같은 코드는 이전 렌더의 state를 변경한다.
- 원하는 동작을 정확히 얻기위해서는 새로운 객체를 생성하여 setPerson에 전달해줘야한다.

```jsx
setPerson({
  firstName: e.target.value, // input의 새로운 first name
  lastName: person.lastName,
  email: person.email
});
```

- 위와 같이 단하나의 필드가 바뀌었기 때문에 존재하는 다른 데이터를 복사해야한다.

하지만 … [객체 전개](https://react.dev/a-javascript-refresher#object-spread) 구문을 사용하면 모든 속성(property)를 각각 복사하지 않아도 된다.

```jsx
setPerson({
  ...person, // 이전 필드를 복사
  firstName: e.target.value // 새로운 부분은 덮어쓰기
});
```

<aside>
👌 여러 필드에 단일 이벤트 핸들러 사용하기

</aside>

### DEEP DIVE - 여러 필드에 단일 이벤트 핸들러 사용하기

- [ ] 괄호를 객체 안에 사용하여 동적 이름을 가진 프로퍼티를 명시할 수 있다.

아래는 이전 예제와 같지만 다른 세개의 이벤트 핸들러 대신 하나의 이벤트 핸들러를 사용하는 예제가 있다. 

- 이전 코드
    
    ```jsx
    import { useState } from 'react';
    
    export default function Form() {
      const [person, setPerson] = useState({
        firstName: 'Barbara',
        lastName: 'Hepworth',
        email: 'bhepworth@sculpture.com'
      });
    
      function handleFirstNameChange(e) {
        setPerson({
          ...person,
          firstName: e.target.value
        });
      }
    
      function handleLastNameChange(e) {
        setPerson({
          ...person,
          lastName: e.target.value
        });
      }
    
      function handleEmailChange(e) {
        setPerson({
          ...person,
          email: e.target.value
        });
      }
    
      return (
        <>
          <label>
            First name:
            <input
              value={person.firstName}
              onChange={handleFirstNameChange}
            />
          </label>
          <label>
            Last name:
            <input
              value={person.lastName}
              onChange={handleLastNameChange}
            />
          </label>
          <label>
            Email:
            <input
              value={person.email}
              onChange={handleEmailChange}
            />
          </label>
          <p>
            {person.firstName}{' '}
            {person.lastName}{' '}
            ({person.email})
          </p>
        </>
      );
    }
    
    ```
    
- 현재 코드
    
    ```jsx
    import { useState } from 'react';
    
    export default function Form() {
      const [person, setPerson] = useState({
        firstName: 'Barbara',
        lastName: 'Hepworth',
        email: 'bhepworth@sculpture.com'
      });
    
      function handleChange(e) {
        setPerson({
          ...person,
          [e.target.name]: e.target.value
        });
      }
    
      return (
        <>
          <label>
            First name:
            <input
              name="firstName"
              value={person.firstName}
              onChange={handleChange}
            />
          </label>
          <label>
            Last name:
            <input
              name="lastName" // 해당 프로퍼티의 이름을 e.target.name DOM 엘리먼트 name
              value={person.lastName}
              onChange={handleChange}
            />
          </label>
          <label>
            Email:
            <input
              name="email"
              value={person.email}
              onChange={handleChange}
            />
          </label>
          <p>
            {person.firstName}{' '}
            {person.lastName}{' '}
            ({person.email})
          </p>
        </>
      );
    }
    
    ```
    

## 중첩된 객체 갱신하기

- 아래와 같이 중첩된 객체 구조가 있다고 생각해보자.

```jsx
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});
```

- person.artwork.city를 업데이트 하고 싶다면, 변경하는 방법은 명백하다.

```jsx
person.artwork.city = 'New Delhi';

//하지만 리액트에서 state를 변경할 수 없는 읽기 전용으로 다루어야한다.
//city를 바꾸기 위해 먼저 이전 객체의 데이터로 생선된, 새로운 adnetwork 생성뒤 person 객체를 만들어야한다.

const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);

//또는 아래와 같이 단순하게 함수를 호출할 수 있다. (세 단계를 하나로)
setPerson({
  ...person, // 다른 필드 복사
  artwork: { // artwork 교체
    ...person.artwork, // 동일한 값 사용
    city: 'New Delhi' // 하지만 New Delhi!
  }
});
```

<aside>
⚠️ DEEP-DIVE 객체들은 사실 중첩되어 있지 않다.

</aside>

- 아래와 같은 객체는 코드에서 “중첩되어” 나타난다.

```jsx
let obj = {
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
};
```

- 중첩은 객체의 동작에 대해 생각하는 부정확한 방법이다.
- 코드가 실행 될 때 중첩된 객체라는 것은 없다.
    - 사실 두 개의 다른 객체를 보는 것이다.

```jsx
let obj1 = {
  title: 'Blue Nana',
  city: 'Hamburg',
  image: 'https://i.imgur.com/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};
```

- obj1은 obj2 “안”에 없다.
- obj3 또한 obj1을 “가리킬” 수 있기 때문
- obj3.artwork.city를 변경하려 했다면, obj.artwork.city, [obj.city](http://obj.city) 둘 다 영향을 미칠 것 이다.
- 이는 obj3.artwork, objartwork와 obj1이 같은 객체이기 때문
- 객체를 “중첩된” 것으로 생각하면 어려울 수 있다. 하지만 그것들은 프로퍼티를 통해 서로를 “가리키는” 각각의 객체들 이다.

## Immer로 간결한 갱신 로직 작성하기

state가 깊이 중첩되어 있다면, [평탄화를](https://react.dev/learn/choosing-the-state-structure#avoid-deeply-nested-state) 고려해 봐도 좋다.

하지만 구조를 바꾸고 싶지 않다면, 중첩 전개할 수 있는 방법이 있다.

- [immer](https://github.com/immerjs/use-immer) 는 편리하고, 변경 구문을 사용할 수 있게 해주며 복사본 생성을 도와주는 인기 라이브러리 이다.

```jsx
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```

<aside>
⚠️ DEEP- DIVE Immer는 어떻게 작동할까요?

</aside>

- Immer가 제공하는 `draft`는 [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)라고 하는 아주 특별한 객체 타입으로, 당신이 하는 일을 “기록” 한다.
- 객체를 원하는 만큼 자유롭게 변경할 수 있는 이유이다.
- Immer는 내부적으로 `draft`의 어느 부분이 변경되었는지 알아내어, 변경사항을 포함한 완전히 새로운 객체를 생성합니다.

- 이벤트 핸들러가 얼마나 간결해 졌는지 보라, 하나의 컴포넌트 안에서 원하는 만큼 useState와 useImmer를 섞어서 사용할 수 있다.

요약

- 리액트의 모든 state를 불변한 것으로 대하세요.
- state에 객체를 저장할 때, 객체를 변경하는 것은 렌더링을 발생시키지 않으며 이전 렌더 “스냅샷”의 state를 바꿀 것입니다.
- 객체를 변경하는 대신 *새로운* 객체를 생성하여 state를 설정함으로써 리렌더링을 일으키세요.
- 객체의 복사본을 만들기 위해 `{...obj, something: 'newValue'}` 객체 전개 구문을 사용할 수 있습니다.
- 전개 구문은 얕습니다. 그것은 한 레벨 깊이만 복사합니다.
- 중첩된 객체를 업데이트하기 위해서는 변경하는 부분에서부터 시작하여 객체의 모든 항목의 복사본을 만들어야 합니다.
- 반복적인 복사 코드를 줄이기 위해서 Immer를 사용하세요.
