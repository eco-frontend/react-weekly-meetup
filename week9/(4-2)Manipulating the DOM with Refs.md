# 4-2. Manipulating the DOM with Refs

[Manipulating the DOM with Refs – React](https://react.dev/learn/manipulating-the-dom-with-refs)

리액트는 자동으로 렌더 결과와 일치하도록 DOM을 업데이트합니다. 그래서 컴포넌트는 자주 조작할 필요가 없습니다. 그러나 때때로 리액트에서 관리하는 DOM 요소에 접근이 필요할 지도 모릅니다. 예를 들면 노드에 focus를 맞추거나 노드를 스크롤하거나 또는 노드의 사이즈와 위치를 측정할 수 있습니다. 리액트에는 이러한 작업을 수행하는 기본 제공 방법이 없습니다 그래서 DOM 노드를 위해 ref가 필요합니다.

# 노드에서 ref를 얻기

리액트에서 관리하는 DOM 노드에 접근하기 위해서 첫번째로 useRef 훅을 import 합니다.

```jsx
import { useRef } from 'react';
```

그리고 이를 사용하기 위해서 컴포넌트 내부에 ref를 선언합니다.

```jsx
const myRef = useRef(null);
```

마지막으로 DOM 노드를 가져오길 원하는 곳의 JSX 태그의 `ref` 속성으로 ref를 전달합니다.

```jsx
<div ref={myRef}>
```

`useRef`훅은 `current`라고 불리는 단일 속성을 가진 객체를 반환합니다. 처음에 `myRef.current` 는 `null` 일 것 입니다. React가 이 `div`에 대한 DOM 노드를 생성할 때, 리액트는 이 노드에 대한 참조를 `myRef.current` 에 넣습니다. 이벤트 핸들러에서 DOM 노드에 접근하고 여기에 정의된 기본 제공 브라우저 API를 사용할 수 있습니다.

```jsx
myRef.current.scrollIntoView();
```

## 예제: text input에 포커싱하기

```jsx
export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

1. `useRef` 훅으로 `inputRef`를 선언합니다
2. `<input ref={inputRef}>` 와 같이 전달합니다. 이는 리액트에게 `input`의 DOM 노드를 `inputRef.current`에 넣으라고 지시합니다.
3. `handleClick` 함수 내부에서 input DOM 노드를 `inputRef.current`에서 읽고 `inputRef.current.focus()`와 같이 `focus()`를 호출합니다.
4. `handleClick` 이벤트 핸들러를 `button`의 `onClick`에 전달합니다.

ref의 가장 일반적인 사용 사례는 DOM을 조작하는 것이지만 useRef 훅은 타이머 ID와 같은 것들을 리액트 외부에 저장하는데 사용할 수 있습니다. 상태(state)와 유사하게 refs는 렌더링 사이에 유지됩니다. Ref는 설정할 때 리렌더링을 트리거하지 않는 상태 변수와 비슷합니다. [참조를 사용하여 값을 참조하기](https://react.dev/learn/referencing-values-with-refs)에 대해 읽어보세요

## 예제: 요소를 스크롤하기

컴포넌트에서 ref를 하나이상 가질 수 있습니다. 세 개의 이미지를 가진 캐러셀 예제가 있습니다. 각각의 버튼은 해당하는 DOM 노드에서 브라우저의 `scrollIntoView()` 메서드를 호출하여 이미지를 중앙에 배치합니다.

```jsx
import { useRef } from 'react';

export default function CatFriends() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function handleScrollToFirstCat() {
    firstCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>
          Tom
        </button>
        <button onClick={handleScrollToSecondCat}>
          Maru
        </button>
        <button onClick={handleScrollToThirdCat}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          <li>
            <img
              src="https://loremflickr.com/200/200"
              alt="Tom"
              ref={firstCatRef}
            />
          </li>
          <li>
            <img
              src="https://loremflickr.com/300/200"
              alt="Maru"
              ref={secondCatRef}
            />
          </li>
          <li>
            <img
              src="https://loremflickr.com/250/200"
              alt="Jellylorum"
              ref={thirdCatRef}
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

## Deep Dive: **How to manage a list of refs using a ref callback**

위의 예제에서 미리 정의된 수의 ref가 있습니다. 그러나 때때로 리스트의 각 아이템의 ref가 필요한 경우가 있거나 얼마나 많은 항목이 있는지 알 수 없는 경우도 있습니다. 이런식으로는 동작하지 않습니다. 

```jsx
<ul>
  {items.map((item) => {
    // Doesn't work!
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

Hook은 반드시 컴포넌트의 최상위에서 호출되어야한다는 규칙 때문입니다. 루프나 조건문 또는 map 함수 호출 내부에서는 useRef를 호출할 수 없습니다.

이 문제를 해결하는 하나의 방법은 부모 요소에서 단일 ref를 얻고 `querySelectorAll`과 같은 DOM 조작 메서드를 이용하여 개별 자식 노드를 “찾는” 것입니다. 그러나 이는 다루기가 힘들고 DOM 구조가 변경된다면 작동하지 않을 수 있습니다.

또 다른 해결책은 `ref` 속성에 함수를 전달하는 것입니다. 이를 [ref callback](https://react.dev/reference/react-dom/components/common#ref-callback) 이라고 부릅니다. 리액트는 ref를 설정할 때가 되면 DOM 노드와 함께 전달한 [ref callback](https://ko.react.dev/reference/react-dom/components/common#ref-callback)을 호출하고, ref를 지울 때 null을 전달합니다. 이를 통해 자체 배열이나 맵(Map)을 유지하고 index 또는 일종의 ID로 모든 ref에 접근할 수 있습니다.

```jsx
import { useRef, useState } from "react";

export default function CatFriends() {
  const itemsRef = useRef(null);
  const [catList, setCatList] = useState(setupCatList);

  function scrollToCat(cat) {
    const map = getMap();
    const node = map.get(cat);
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToCat(catList[0])}>Tom</button>
        <button onClick={() => scrollToCat(catList[5])}>Maru</button>
        <button onClick={() => scrollToCat(catList[9])}>Jellylorum</button>
      </nav>
      <div>
        <ul>
          {catList.map((cat) => (
            <li
              key={cat}
              **ref={(node) => {
                const map = getMap();
                if (node) {
                  // Add to the Map
                  map.set(cat, node);
                } else {
                  // Remove from the Map
                  map.delete(cat);
                }
              }}**
            >
              <img src={cat} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catList = [];
  for (let i = 0; i < 10; i++) {
    catList.push("https://loremflickr.com/320/240/cat?lock=" + i);
  }

  return catList;
}
```

이 예제에서는 itemsRef는 단일 DOM 노드를 가지고 있지 않습니다. 대신에 item ID와 DOM 노드로 연결된Map을 가지고 있습니다. ([Ref는 어떤 값이든 가질 수 있습니다](https://react.dev/learn/referencing-values-with-refs)!) 모든 리스트 아이템에 있는 ref callback은 Map의 업데이트를 처리합니다.

이것으로 이후 Map에서 개별 DOM 노드를 읽을 수 있습니다.

### Canary

이 예제에서는 ref callback cleanup 함수와 함께 Map을 다루기위한 또 다른 접근법을 보여줍니다.

```jsx
<li
  key={cat.id}
  ref={node => {
    const map = getMap();
    // Add to the Map
    map.set(cat, node);

    return () => {
      // Remove from the Map
      map.delete(cat);
    };
  }}
>
```

## 다른 컴포넌트의 DOM 노드 접근하기

`<input />` 과 같은 브라우저 요소를 출력하는 내장 컴포넌트에 ref를 주입할 때, 리액트는 ref의 current 프로퍼티를 해당 DOM 노드로 설정합니다(브라우저에서는 실제 `<input />` 과 같은)

그러나 <MyInput /> 과 같이 직접 만든 컴포넌트에 ref를 주입하고자 한다면 기본으로 null 값을 얻을 것 입니다. 여기 예제가 있습니다. 버튼을 클릭해도 input에 focus가 맞춰지지 않는다는 점 유의하세요.

```jsx
function MyInput(props) {
  return <input {...props} />;
}

/// ...

    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
```

이 문제를 발견할 수 있도록 리액트는 콘솔에 에러를 노출합니다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/fbb7d653-8769-4b52-b522-ec037ecdbb36/Untitled.png)

리액트는 기본적으로 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않습니다. 컴포넌트의 자식도 예외가 아닙니다! 이는 의도적인 설계 입니다. Ref는 신중하게 사용해야하는 탈출구입니다. 다른 컴포넌트의 DOM 노드를 직접 다루는 것은 코드가 더 쉽게 깨질 수 있도록 만듭니다.

대신에 컴포넌트의 DOM 노드를 노출하려는 경우 해당 동작을 선택적으로 노출할 수 있습니다. 컴포넌트는 자신의 참소를 자식 중 하나에 “forwards” 하도록 지정할 수 있습니다. 여기 `MyInput`이 `forwardRef` API를 사용하는 방법이 있습니다.

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

1. `<MyInput ref={inputRef} />` 은 리액트가 대응되는 DOM 노드를 inputRef.current에 넣을 수 있도록 설정합니다. 하지만 이는 `MyInput` 컴포넌트의 선택에 달려있습니다, 기본적으로는 이렇게 동작하지 않습니다.
2.  `MyInput` 컴포넌트는 `forwardRef`를 사용하여 선언되었습니다. 이는 `props` 다음에 선언된 **두 번째 `ref` 인수를 통해 상위의 `inputRef`를 받을 수 있도록 합니다.**
3. `MyInput`은 자체적으로 `ref`를 받으면 `input`의 내부에 전달하도록 합니다

이제 버튼을 클릭하면 input에 focus가 작동합니다.

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
// ...
  function handleClick() {
    inputRef.current.focus();
  }
return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
```

이는 디자인 시스템에서 버튼, 입력 등과 같은 로우 레벨 컴포넌트에서 ref를 통해 DOM 노드를 전달하기 위한 흔한 패턴입니다. 반면에 폼, 리스트, 또는 페이지 등의 고수준의 컴포넌트에서는 대개 의도하지 않은 DOM 구조 의존성 문제를 피하고자 그들의 DOM 노드를 노출하지 않습니다.

### Deep Dive: **Exposing a subset of the API with an imperative handle**

명령형으로 API의 하위 집합 노출하기

위의 예제에서는 `MyInput` 은 DOM input 요소를 노출하고 있습니다. 그리고 부모 컴포넌트에서 DOM 노드의 `focus()`를 호출할 수 있게 되었습니다. 하지만 이렇게 하면 부모 컴포넌트가 다른 작업(ex. CSS 스타일 변경 등)을 할 수도 있습니다. 특별한 경우에 기능적으로 제한하고 싶을 수 있는데, 이 떄 `useImperativeHandle`를 사용할 수 있습니다.

```jsx
const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});
```

여기 `realInputRef`는 실제 DOM 노드를 가지고 있습니다. 하지만  `useImperativeHandle`을 사용하여 리액트가 ref를 참조하는 부모 컴포넌트에 직접 구성한 객체를 전달하도록 지시합니다. 따라서 Form 컴포넌트 안쪽의 `inputRef.current`는 `focus` 메서드만 가지고 있습니다. 이러한 경우에는 ref가 “다루는 것”은 DOM 노드가 아니라 `useImperativeHandle` 호출 내부에서 만들어낸 커스텀한 객체가 됩니다.

## React가 ref를 부여할 때

리액트에서 모든 업데이트는 두 단계로 나눌 수 있습니다.

- 렌더링 단계에서는 리액트는 화면에 무엇을 노출할지 알아내도록 컴포넌트를 호출합니다.
- 커밋 단계에서는 리액트는 변화를 DOM에 적용합니다.

일반적으로 렌더링 동안에 ref에 접근하는 것을 원하지 않습니다. DOM 노드를 가지고 있는 ref도 마찬가지 입니다. 첫번째 렌더링에서 DOM 노드는 아직 생성되지 않았습니다. 그래서 `ref.current`는 null 입니다. 

그리고 업데이트들의 렌더링 동안에는 DOM 노드는 아직 업데이트되지 않은 상태입니다. ref를 읽기에 너무 이른 상황입니다.

대개 이벤트 핸들러로에서 ref를 접근할 것 입니다. 만약 ref를 사용하여 뭔가를 하더라도 이를 시행할 특정할 이벤트가 없을 떄 Effect가 필요할지도 모릅니다. 다음 페이지에서 Effects에 대해서 이야기해보겠습니다.

### Deep Dive: **Flushing state updates synchronously with flushSync**

flushSync로 state의 업데이트를 동적으로 플러시하기

새로운 할일을 추가하고 리스트의 마지막 자식으로 스크롤이 내려가는 아래 예제 코드를 보겠습니다. 어떤 이유에서인지 마지막에 추가된 할일 직전으로 항상 스크롤이 되는 것을 보세요.

```jsx
import { useState, useRef } from 'react';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([ ...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

이 문제는 아래 두 라인에 있습니다.

```jsx
setTodos([ ...todos, newTodo]);
listRef.current.lastChild.scrollIntoView()
```

리액트에서 [상태는 큐와 같이 업데이트](https://react.dev/learn/queueing-a-series-of-state-updates)됩니다. 대개 이것은 일반적으로 기대하는 동작입니다. 그러나 여기에서는 `setTodos`가 DOM을 바로 업데이트하지 않기 때문에 문제가 발생합니다. 그래서 할 일 목록의 마지막 노드로 스크롤 할 때, 할일은 아직 추가되지 않은 상태입니다. 이것이 스크롤이 항상 하나의 항목에 뒤쳐지는 이유입니다.

이를 고치기 위해서, 리액트가 DOM을 동기적으로 강제로 업데이트(flush)하도록 할 수 있습니다. 이를 수행하기 위해서 `flushSync` 를 `react-dom`으로부터 import 하고 `flushSync` 호출 내부에서 상태를 업데이트하면 됩니다.

```jsx
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
```

이것은 리액트에게 `flushSync`로 감싼 코드가 실행되고 난 후 DOM을 동기적으로 업데이트하도록 지시합니다. 그 결과 마지막 할일이 스크롤하기 전에 이미 DOM에 추가되어 있을 것 입니다.

## ref로 DOM을 조작하는 모범 사례

Ref는 탈출구입니다. “리액트 바깥으로 벗어날 떄”만 오로지 이를 사용해야합니다. 일반적인 예시는 focus, scroll 위치, 브라우저 API 호출 등(리액트가 노출하지 않는)의 작업이 이에 포함됩니다.

포커싱이나 스크롤과 같이 구조를 파괴하지 않는 행동을 하는 경우라면 문제를 마주하지 않을 것 입니다. 그러나 만약 당신이 DOM을 직접 수정하기를 시도한다면 리액트가 만들어낸 변화와 충돌하는 위험을 감수해야합니다.

이 문제를 이해하기 위해서 아래 예시는 환영 메세지와 두개의 버튼을 포함하고 있습니다. 첫번째 버튼은 일반적인 리액트에서 제공하는 조건부 렌더링과 상태를 이용하여 토글하고 있습니다. 두번째 버튼은 DOM API의 remove를 이용하여 리액트의 제어 밖에서 DOM 노드를 강제로 삭제합니다.

  

“Toggle with setState”를 여러번 눌러보세요. 메세지가 반복적으로 나타나거나 사라질 것입니다. 이후 “Remove from the DOM”을 눌러보세요. 이것은 강제적으로 노드를 삭제합니다. 마지막으로 “Toggle with setState”를 다시 눌러보세요.

```jsx
import { useState, useRef } from 'react';

export default function Counter() {
  const [show, setShow] = useState(true);
  const ref = useRef(null);

  return (
    <div>
      <button
        onClick={() => {
          setShow(!show);
        }}>
        Toggle with setState
      </button>
      <button
        onClick={() => {
          ref.current.remove();
        }}>
        Remove from the DOM
      </button>
      {show && <p ref={ref}>Hello world</p>}
    </div>
  );
}
```

DOM 요소를 직접 삭제한 후에 setState를 사용하여 다시 보이게하는 것은 충돌을 발생시킵니다. DOM을 변경했기 때문에 그리고 리액트는 이것을 올바르게 계속해서 다루는 방법을 모르기 때문에 발생합니다.

리액트에 의해 관리되는 DOM의 변경을 피하세요. 리액트에 의해 관리되는 수정, 자식 요소 추가, 또는 요소로부터 자식 삭제등은 위와 같이 시각적인 결과 또는 충돌로 이어집니다.

하지만 항상 이것을 할 수 없다는 의미는 아닙니다. 주의깊게 사용해야합니다. **안전하게 React가 업데이트할 이유가 없는 DOM 노드 일부를 수정할 수 있습니다.** 예를 들어 몇몇 `<div>`가 항상 빈 채로 JSX에 있다면, React는 해당 노드의 자식 요소를 건드릴 이유가 없습니다. 따라서 빈 노드에서 엘리먼트를 추가하거나 삭제하는 것은 안전합니다.

[Manipulating the DOM with Refs – React](https://react.dev/learn/manipulating-the-dom-with-refs#challenges)
