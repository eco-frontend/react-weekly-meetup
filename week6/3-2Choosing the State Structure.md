상태를 잘 구조화하면 수정과 디버깅이 편한 컴포넌트와 버그가 끊임없이 발생하는 컴포넌트의 차이를 만들 수 있습니다.

<aside>
💡 배울 내용
- 하나 또는 여러개의 상태 변수를 사용하는 방법
- 상태 정리 시 피해야할 내용
- 상태 구조와 함께 일반적인 문제를 해결하는 방법

</aside>

## **Principles for structuring state**

상태를 가지고 있는 컴포넌트를 작성할 때, 사용하기 위한 많은 상태 변수를 선택할 수 있고 그 데이터의 모양을 어떻게 할지 정할 수 있습니다. 최적이 아닌 상태 구조로도 올바른 프로그램을 작성할 수 있지만, 더 나은 선택을 위해서 몇 가지 원칙이 있습니다.

1. 관련된 상태끼리 그룹화하기: 항상 같은 시점에 두개 이상의 상태를 업데이트한다면, 하나의 상태 변수로 합치는 것을 고려할 수 있습니다
2. 상태에서 모순 피하기: 여러 상태가 서로 모순되고 불일치 할 수 있는 방식으로 상태를 구성하는 것은 실수가 발생할 여지를 만듭니다. 이를 피하세요
3. 불필요한 상태 피하기: 컴포넌트의 props 또는 상태를 통해서 렌더링 동안 상태를 계산할 수 있다면, 컴포넌트의 상태에 해당 정보를 넣지 않아야합니다.
4. 상태의 중복 피하기: 여러개의 상태 변수 사이, 중첩된 객체에서 같은 데이터의 중복될 경우 동기화 하기 어렵습니다. 가능하다면 중복을 줄이세요.
5. 깊은 중첩 피하기: 깊게 계층화된 상태는 업데이트하기 쉽지 않습니다. 가능하다면 평탄화된 상태구조를 선호합니다.

이러한 원칙을 통한 목표는 실수없이 업데이트할 수 있는 상태를 쉽게 만들기 위함입니다. 상테에서 중복 데이터와 중복을 제거하는 것은 모든 데이터가 동기화 상태를 유지하는 데 도움이 됩니다. 이는 데이터베이스 엔지니어가 버그를 줄이기 위해 데이터베이스 구조를 정규화하는 방법과 유사합니다. 알버트 아인슈타인의 말을 인용한다면, 당 상태를 가능한한 단순하게 만들어야합니다, 더 단순하게가 아니라.

## **Group related state**

종종 하나의 상태 또는 여러 상태 사이에서 고민할 때가 있습니다.

```tsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);

또는

const [position, setPosition] = useState({ x: 0, y: 0 });
```

기술적으로, 당신은 이러한 접근들을 고려할 수 있습니다. 그러나 두개의 상태변수가 항상 함께 변한다면 하나의 상태로 정의하는 것이 좋을 수 있습니다. 그러면 마우스 커서를 움직이면 빨간 점의 두 좌표가 모두 업데이트되는 이 예제처럼 항상 동기화를 유지하는 것을 잊지 않을 것입니다.

```tsx
export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  
  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
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

또 다른 경우는 객체 또는 배열로 그룹화 하는 데이터가 얼마나 많은 상태를 가져야하는지 모르는 경우입니다. 예를 들어 유저가 폼에 커스텀 필드를 추가하는 경우 유용할 수 있습니다.

 

<aside>
💡 상태 변수가 객체인 경우 다른 필드를 명시적으로 복사하지 않고 하나의 필드만 업데이트할 수 없다는 것을 잊지 마세요. 예를들어 위 예제의 경우 `setPosition({ x: 100 })` 라고 할 수 없다. 왜냐하면 y 프로퍼티를 가지지 않았기 때문입니다. 대신에 x만 업데이트하길 원한다면, `setPosition({…position, x: 100})` 또는 상태를 분리하여 `setX(100)` 와 같이 쓸 수 있습니다.

</aside>

## **Avoid contradictions in state**

`isSending` 과 `isSent` 상태를 가진 호텔 만족도 조사 폼이 있습니다.

```tsx
export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Thanks for feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

코드가 동작하지만 불가능한 상태에 대한 문을 열어둡니다. 예를들어, `setIsSent`와 `setIsSending` 을 동시에 호출하는 것을 잊었다면, `isSending` 과 `isSent` 가 같은 시간에 `true` 인 상황이 생길 수 있습니다. 더 복잡한 컴포넌트라면 왜 이런 문제가 발생하는지 이해하기 어렵습니다.

`isSending`과 `isSent` 가 같은 시점에 `true` 여서는 안되기 때문에 이 두 변수를 `'typing'`(초깃값), `'sending'`, `'sent'` 세 가지 유효한 상태 중 하나를 가질 수 있는 `status` 상태 변수로 대체하는 것이 좋습니다.

```tsx
export default function FeedbackForm() {
  const [text, setText] = useState('');
  - const [isSending, setIsSending] = useState(false);
  - const [isSent, setIsSent] = useState(false);
  + const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    - setIsSending(true);
    + setStatus('sending');
    await sendMessage(text);
    - setIsSending(false);
    - setIsSent(true);
    + setStatus('sent');
  }
  
  + const isSending = status === 'sending';
  + const isSent = status === 'send'; 

  if (isSent) {
    return <h1>Thanks for feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}
```

가독성을 위해서 일부 상수를 선언할 수 있습니다.

```tsx
const isSending = status === 'sending';
const isSent = status === 'send'; 
```

그러나 상수들은 상태 변수가 아니기 때문에, 서로 동기화를 걱정할 필요가 없습니다.

## **Avoid redundant state**

렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있다면, 컴포넌트의 state에 해당 정보를 넣지 않아야 합니다**.**

아래 폼에 예제가 있습니다. 동작하지만 중복 상태를 찾을 수 있나요?

```tsx
export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

여기 폼은 세가지의 상태 변수(`firstName`, `lastName`, `fullName`)를 가지고 있습니다. 그러나 `fullName`은 중복입니다. `firstName`과 `lastName`으로 부터 `fullName`을 항상 렌더링 동안 계산할 수 있습니다. 그러니 상태에서 제거할 수 있습니다.

```tsx
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  - const [fullName, setFullName] = useState('');
  
  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    - setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    - setFullName(firstName + ' ' + e.target.value);
  }
```

여기, `fullName` 은 상태 변수가 아닙니다. 대신 렌더링동안에 계산됩니다.

```tsx
const fullName = firstName + ' ' + lastName;
```

결과적으로 change handlers는 더 이상 업데이트를 위한 특별한 동작을 하지 않습니다. `setFirstName` 또는 `setLastName`을 호출할 때, 리렌더링이 트리거되고 다음 `fullName`이 새 데이터로 계산될 것 입니다.

### Deep Dive: **Don’t mirror props in state**

다음 코드는 불필요한 state의 일반적인 예입니다.

```tsx
function Message({ messageColor }) {
	const [color, setColor] = useState(messageColor);
}
```

`color` 상태 변수는 `messageColor` props를 통해서 초기화됩니다. 문제는 부모 컴포넌트가 나중에 `messageColor`의 값을 다르게 전달해준다면(ex, ‘red’ 대신 ‘blue’와 같이), `color` 상태 변수는 업데이트 되지 않을 것 입니다. 상태는 오직 첫번째 렌더링 동안에만 초기화됩니다.

그 때문에 상태 변수의 일부 props를 “미러링”하면 혼란을 일으키는 이유입니다. 대신, `messageColor` props를 코드에 직접 사용하세요. 만약 더 짧은 이름을 주길 원한다면 constant를 사용하십시오

```tsx
function Message({ messageColor }) {
	const color = messageColor;
}
```

이러한 방법은 부모 컴포넌트로부터 전달된 props와 동기화를 잃지 않습니다.

props를 상태로 “미러링”하는 것은 특정 props에 대한 모든 업데이트를 무시하기를 원할 때에만 의미가 있습니다. 컨벤션에 따르면, props 이름의 시작에 `initial` 또는 `default` 를 추가하여 새로운 값이 무시됨을 명확하게 하세요.

```tsx
function Message({ initialColor }) {
	const [color, setColor] = useState(initialColor);
}
```

## **Avoid duplication in state**

당신이 간식을 선택할 수 있는 메뉴 목록 컴포넌트가 있습니다.

```tsx
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.title}
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Choose</button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

현재 `selectedItem` 상태 변수에 객체로서 선택된 아이템을 저장합니다. 그러나 훌륭하지는 않습니다. `selectedItem` 의 컨텐츠는 `item` 목록 내부의 아이템 중 하나와 같은 객체입니다. 이는 두가지 위치에 중복으로 아이템에 대한 정보가 있다는 것을 의미합니다.

이것이 왜 문제일까요? 각각의 아이템이 수정이 가능하다고 생각해봅시다

```tsx
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
 

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>What's your travel snack?</h2> 
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Choose</button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

당신이 아이템의 “Choose”를 첫번째로 클릭하고 수정한다면 input은 업데이트 되었지만 아래쪽의 라벨에는 수정한 내용이 반영되지 않습니다. 이는 당신이 중복된 상태를 가지고 있고 `selectedItem` 을 업데이트하는 것을 잊었기 때문입니다.

`selectedItem` 역시 업데이트 할 수 있지만, 쉬운 수정은 중복을 제거하는 것입니다. 이 예제에서는 `selectedItem` 객체(item 내부에 객체와 중복을 생성하는) 대신에, `selectedId`를 상태로 유지하고 그 아이디를 가지고 `items` 배열에서 `selectedItem` 을 찾을 수 있습니다.

```tsx
export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);

  const selectedItem = items.find(item =>
    item.id === selectedId
  );
  
  ...
  
              <button onClick={() => {
              setSelectedId(item.id);
            }}>Choose</button>
}

```

상태는 아래와 같이 중복으로 사용되었었습니다.

`items = [{ id: 0, title: 'pretzels'}, ...]`

`selectedItem = {id: 0, title: 'pretzels'}`

변경 이후는 아래와 같습니다.

`items = [{ id: 0, title: 'pretzels'}, ...]`

`selectedId = 0` 

중복은 사라지고 필수적인 상태만 남아있습니다!

이제 선택된 아이템을 수정한다면 메세지는 즉시 업데이트되어 보일 것 입니다. 이는 `setItems` 가 리렌더링을 트리거하고 `items.find()` 는 업데이트된 타이틀을 가지고있는 아이템을 찾을 것 입니다. 선택한 ID만 필수이므로 선택한 항목을 state로 유지할 필요가 없습니다. 나머지는 렌더링 동안에 계산할 수 있습니다.

## **Avoid deeply nested state**

행성, 대륙, 국가로 구성된 여행 계획을 상상해보세요. 아래 예제처럼 중첩된 객체나 배열을 사용하여 상태의 구조를 구성할 수 있습니다. 

https://ko.react.dev/learn/choosing-the-state-structure#avoid-deeply-nested-state

```tsx
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

function PlaceTree({ place }) {
  const childPlaces = place.childPlaces;
  return (
    <li>
      {place.title}
      {childPlaces.length > 0 && (
        <ol>
          {childPlaces.map(place => (
            <PlaceTree key={place.id} place={place} />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);
  const planets = plan.childPlaces;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planets.map(place => (
          <PlaceTree key={place.id} place={place} />
        ))}
      </ol>
    </>
  );
}

```

여기에 버튼을 추가해서 이미 방문한 장소를 삭제하길 원한다고 해봅시다. 어떻게 하면 좋을까요? 중첩된 객체를 업데이트하기 위해서는 변경되는 부분부터 복사본을 모두 만들어야합니다. 깊게 중첩되어있는 장소를 삭제하기 위해서는 상위 체인을 모두 복사해야합니다. 이러한 코드는 매우 장황할 수 있습니다.

상태가 업데이트하기 쉽지 않게 엄청 중첩되어있다면, 평탄화하는 것을 고려해보세요. 데이터를 재구조화하는 한가지 방법이 있습니다. 각 `place`가 *자식 장소*의 배열을 가지는 트리 구조 대신, 각 장소가 *자식 장소 ID*의 배열을 가지도록 할 수 있습니다. 그런다음 각 장소 ID에서 해당 장소로 매핑을 저장합니다

as-is

```tsx
export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Earth',
    childPlaces: [{
      id: 2,
      title: 'Africa',
      childPlaces: [{
        id: 3,
...
```

to-be

```tsx
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
```

현재 상태가 평탄화(또는 정규화)되었고, 중첩된 아이템을 업데이트하는 것이 쉬워졌습니다.

장소를 제거하기 위해서 다음 두 단계의 상태만 업데이트하면 됩니다.

- 부모 장소의 업데이트된 버전은 제거된 ID를 `childIds` 배열에서 제외해야 합니다.
- 루트 테이블 객체의 업데이트된 버전에는 상위 위치의 업데이트된 버전이 포함되어야 합니다.

```tsx
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);

  function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // Create a new version of the parent place
    // that doesn't include this child ID.
    const nextParent = {
      ...parent,
      childIds: parent.childIds
        .filter(id => id !== childId)
    };
    // Update the root state object...
    setPlan({
      ...plan,
      // ...so that it has the updated parent.
      [parentId]: nextParent
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Complete
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

이와 같이 상태를 원하는 만큼 중첩할 수 있지만, 평탄화하면 많은 문제를 해결할 수 있습니다. 이렇게하면 상태를 더 쉽게 업데이트 할 수 있고 중첩된 객체의 다른 부분에 중복이 생기지 않도록 할 수 있습니다.

### Deep Dive: **Improving memory usage**

이상적으로는 “테이블”객체에서 삭제된 항목(및 그 하위 항목)도 제거하여 메모리 사용량을 개선하는 것이 좋습니다. 이 버전은 그렇게하고, immer를 사용하여 업데이트 로직을 더욱 간결하게 만듭니다.

```tsx
import { useImmer } from 'use-immer';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, updatePlan] = useImmer(initialTravelPlan);

  function handleComplete(parentId, childId) {
    updatePlan(draft => {
      // Remove from the parent place's child IDs.
      const parent = draft[parentId];
      parent.childIds = parent.childIds
        .filter(id => id !== childId);

      // Forget this place and all its subtree.
      deleteAllChildren(childId);
      function deleteAllChildren(id) {
        const place = draft[id];
        place.childIds.forEach(deleteAllChildren);
        delete draft[id];
      }
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Complete
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

때로는 중첩된 상태의 일부를 하위컴포넌트로 이동하여 상태의 중첩을 줄일 수 있습니다. 이는 항목의 마우스오버 등과 같은 저장할 필요가 없는 일시적인 UI 상태에 효과적입니다.

## 정리

- 두가지 상태가 같이 업데이트 된다면 하나로 합치는 것을 고려하세요
- “불가능한” 상태를 만드는 것을 피하기 위해 상태 변수를 신중하게 선택하세요
- 상태를 업데이트할 때 실수할 가능성을 줄이는 방식으로 구성하세요
- 동기화 상태를 유지할 필요가 없도록 중복 및 중복 상태를 피하세요
- 특별히 업데이트를 막아야하는 경우가 아니라면 props를 state에 넣지 마세요
- 선택과 같은 UI 패턴에서는 객체 대신 ID 또는 인덱스를 상태로 유지하세요.
- 깊이 중첩된 상태를 업데이트하는 것이 복잡하다면, 평탄화하세요.
