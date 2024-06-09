각 컴포넌트는 독립된 state를 가진다.

React는 UI 트리에서의 위치를 통해 각 state가 어떤 컴포넌트에 속하는지 추적한다.

리렌더링마다 언제 state를 보존하고 또 state를 초기화할지 컨트롤할 수 있다.

학습 내용

- React가 언제 state를 보존하고 언제 초기화 하는지
- 어떻게 React가 컴포넌트의 state를 초기화하도록 강제할 수 있는지
- key와 타입이 state 보존에 영향을 어떻게 주는지

## State는 렌더트리의 위치에 연결된다.

React는 UI안에 있는 컴포넌트 구조로 [렌더 트리](https://react.dev/learn/understanding-your-ui-as-a-tree#the-render-tree)를 만든다.

컴포넌트에 state를 줄 때 state가 컴포넌트 안에 “살고” 있다고 생각할 수 있다.

하지만 사실 state는 react안에 있다.

react는 컴포넌트가 UI트리에 있는 위치를 이용해 react가 가지고 있는 각 state를 알맞은 컴포넌트와 연결한다.

```jsx
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}

```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/0458741e-c6d0-4228-a16a-7965225e6ece/Untitled.png)

위 코드를 보았을 때 카운터가 아래 이미지와 같이 트리 구조로 보인다. 

![preserving_state_tree.webp](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/743a0135-1480-4c43-89a9-c4c62b85f8a9/preserving_state_tree.webp)

- **이 둘은 각각 트리에서 자기 고유의 위치에 렌더링되어 있으므로 분리되어 있는 카운터다.**
- 일반적으로 React를 사용할 때 위치에 대해 생각할 필요 없지만, React가 어떻게 작동하는지 이해할 때 유용하다.

- React에서 화면의 각 컴포넌트는 완전히 분리된 state를 가진다.
- 예를 들어, 두 Counter 컴포넌트를 나란히 렌더링하면 각각 자신만의 독립된 score와 hover state를 가진다.
- 특정 카운터가 갱신되면, 해당 컴포넌트의 상태만 갱신되는 것을 확인할 수 있다.
    
    ![preserving_state_increment.webp](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/56729eff-fa6a-4630-8948-b303ca358e6a/preserving_state_increment.webp)
    

- React는 트리의 동일한 컴포넌트를 동일한 위치에 렌더링하는 동안 상태를 유지한다.
- 이를 확인하려면, 두 Counter를 모두 증가시키고, “Render the second counter” 체크박스의 해제하여 두 번째 컴포넌트를 제거 해보자, 그리고 다시 체크박스를 눌러 추가해보자

> 결과는 두 번째 카운터를 렌더링하지 않을 때 그 state가 완전히 사라지는 것을 확인해보자
> 

> 이는 React가 컴포넌트를 제거할 때 그 state도 같이 제거한다.
> 

![preserving_state_remove_component.webp](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/645c8e39-31c0-4f09-b096-b92cbb71ba1d/preserving_state_remove_component.webp)

- ”Render the second counter”를 누를 때 두 번째 Counter와 그 state는 처음부터 초기화되고(score = 0) DOM에 추가
    
    ![preserving_state_add_component.webp](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/53410713-edda-4c6b-92da-459c71eafcd0/preserving_state_add_component.webp)
    

- **React는 컴포넌트가 UI 트리에서 그 자리에 렌더링되는 한 state를 유지한다.**
- 만약 그것을 제거하거나 같은 자리에 다른 컴포넌트가 렌더링되면 React는 그 state를 버린다.

## **같은 자리의 같은 컴포넌트는 state를 보존한다.**

다음 예시에는 서로 다른 두 `<Counter />` 태그가 있다.

```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
        <Counter isFancy={false} /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

- 체크 박스를 선택하거나 선택 해제할 때 카운터 state는 초기화되지 않는다.
- isFancy가 true이든 false이든 <Counter />는 같은 자리에 있다.
- root App 컴포넌트가 반환한 div의 첫 번째 자식으로 있다.
    
    ![`Counter`는 같은 자리에 있기 때문에 `App` 상태의 갱신은 `Counter`를 초기화시키지 않는다
    
    ](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/21ae4d69-d62c-44d0-85de-3c030d7e539f/preserving_state_same_component.webp)
    
    `Counter`는 같은 자리에 있기 때문에 `App` 상태의 갱신은 `Counter`를 초기화시키지 않는다
    
    <aside>
    ⚠️ 주의! React는 JSX 마크업에서가 아닌 UI 트리에서의 위치에 관심이 있다는 것을 기억!
    
    </aside>
    
    ```jsx
    import { useState } from 'react';
    
    export default function App() {
      const [isFancy, setIsFancy] = useState(false);
      if (isFancy) {
        return (
          <div>
            <Counter isFancy={true} />
            <label>
              <input
                type="checkbox"
                checked={isFancy}
                onChange={e => {
                  setIsFancy(e.target.checked)
                }}
              />
              Use fancy styling
            </label>
          </div>
        );
      }
      return (
        <div>
          <Counter isFancy={false} />
          <label>
            <input
              type="checkbox"
              checked={isFancy}
              onChange={e => {
                setIsFancy(e.target.checked)
              }}
            />
            Use fancy styling
          </label>
        </div>
      );
    }
    
    function Counter({ isFancy }) {
      const [score, setScore] = useState(0);
      const [hover, setHover] = useState(false);
    
      let className = 'counter';
      if (hover) {
        className += ' hover';
      }
      if (isFancy) {
        className += ' fancy';
      }
    
      return (
        <div
          className={className}
          onPointerEnter={() => setHover(true)}
          onPointerLeave={() => setHover(false)}
        >
          <h1>{score}</h1>
          <button onClick={() => setScore(score + 1)}>
            Add one
          </button>
        </div>
      );
    }
    ```
    
- 체크 박스를 선택할 때 state가 초기화될 거라고 생각했을 수도 있지만 그렇지 않다.
- **두 `<Counter />` 태그가 같은 위치에 렌더링되기** 때문
- react는 함수 안 어디에 조건문이 있는지 모른다.
- React는 반환하는 트리만 “본다”
- 그들이 같은 “주소”를 갖는다고 생각할 수 있다.
    - root의 첫 번째 자식으로, 이것이 어떻게 로직을 만들었는지와 상관없이 React가 이전과 다음 렌더링 사이에 컴포넌트를 맞추는 방법이다.

### 같은 위치의 다른 컴포넌트는 state를 초기화 한다.

- 다음 예시에서 체크 박스를 선택하면 `<Counter>`가 `<p>`로 교체된다.

```jsx
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>See you later!</p> 
      ) : (
        <Counter /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        Take a break
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

- 여기서 같은 자리의 ***다른*** 컴포넌트 타입으로 바꾼다.
- 처음에는 `<div>`가 `Counter`를 갖고 있다.
- 하지만 p로 바꾸면 React는 UI 트리에서 Counter와 그 state를 제거한다.

![`Counter` 가 `p` 로 바뀌면, `Counter` 는 삭제되고 `p` 가 추가된다.](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/a83cc999-e198-488d-bcdf-a8b4d06081ac/preserving_state_diff_pt1.webp)

`Counter` 가 `p` 로 바뀌면, `Counter` 는 삭제되고 `p` 가 추가된다.

![다시 되돌리면, `p` 는 삭제되고 `Counter` 가 추가된다..
](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/5a0ae49d-5c66-456d-87f9-a08605b70feb/preserving_state_diff_pt2.webp)

다시 되돌리면, `p` 는 삭제되고 `Counter` 가 추가된다..

- 또한 **같은 위치에 다른 컴포넌트를 렌더링할 때 컴포넌트는 그 전체 서브 트리의 state를 초기화한다.**

```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} /> 
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}

```

- 체크 박스를 선택할 때 카운터 state가 초기화된다.
- div의 첫 번째 자식으로 Counter를 렌더링하는 것에서 section의 첫번째 자식으로 바꾼다.
- 자식 div가 DOM에서 제거될 때 그것의 전체 하위 트리(Counter와 state를 포함해서)는 제거된다.

![`section` 이 `div` 로 바뀌면, `section` 은 삭제되고 새로운 `div` 가 추가됩니다

](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/a5d81416-3b5a-4d15-ab57-cae67e2c63c3/preserving_state_diff_same_pt1.webp)

`section` 이 `div` 로 바뀌면, `section` 은 삭제되고 새로운 `div` 가 추가됩니다

![다시 되돌리면, `div` 는 삭제되고 새로운 `section` 이 추가됩니다.](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/0b7376b4-37a7-4d47-9a08-55af761618af/preserving_state_diff_same_pt2.webp)

다시 되돌리면, `div` 는 삭제되고 새로운 `section` 이 추가됩니다.

- 경험상(rule of thumb) **리렌더링할 때 state를 유지하고 싶다면, 트리 구조가 “같아야” 한다.** 만약 구조가 다르다면 React가 트리에서 컴포넌트를 지울 때 state로 지우기 때문에 state가 유지되지 않습니다.

<aside>
⚠️ 주의! 컴포넌트 함수를 중첩해서 정의하면 안되는 이유

</aside>

- 여기, `MyComponent` 안에서 `MyTextField` 컴포넌트 함수를 정의하고 있다.

```jsx
import { useState } from 'react';

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>Clicked {counter} times</button>
    </>
  );
}

-----
export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  const MyTextField = useCallback(() => {
    // eslint-disable-next-line
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  },[])

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>Clicked {counter} times</button>
    </>
  );
}
```

버튼을 누를 때마다 입력 state가 사라집니다! 이것은 `MyComponent`를 렌더링할 때마다 *다른* `MyTextField` 함수가 만들어지기 때문입니다. 

따라서 같은 함수에서 *다른* 컴포넌트를 렌더링할 때마다 React는 그 아래의 모든 state를 초기화한다. 

이런 문제를 피하려면 **항상 컴포넌트를 중첩해서 정의하지 않고 최상위 범위에서 정의해야 한다.**

## 같은 위치에서 state를 초기화하기

- 기본적으로 React는 컴포넌트가 같은 위치를 유지하면 state를 유지한다.
- 보통 이것이 원하는 행동이기 때문에 기본 동작으로서 타당하다. 그러나 가끔 컴포넌트 state를 초기화하고 싶을 때가 있다.
- 두 선수가 턴마다 자신의 점수를 추적하는 앱을 한번 보자.

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

- 지금은 선수를 바꿀 때 점수가 유지되어지지만, 두 Counter가 같은 위치에 나타나기 때문에 React는 person Props가 변경된 같은 Counter로 봅니다.
- 하지만, 개념적으로 app에는 두 개의 분리된 카운터가 있어야한다. 그들은 UI에 같은 위치에 나타나지만, 하나는 Taylor의 카운터이고, 다른 하나는 Sarah의 카운터이다.

이 둘을 바꿀 때 state를 초기화하기 위한 두가지 방법이 있다.

1. 다른 위치에 컴포넌트를 렌더링하기
2. 각 컴포넌트에 `key`로 명시적인 식별자를 제공하기

### 선택지 1: 다른 위치에 컴포넌트를 렌더링하기

두 `Counter`가 독립적이기를 원한다면 둘을 다른 위치에 렌더링할 수 있다.

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

- 처음에는 `isPlayerA`가 `true`입니다. 따라서 첫 번째 자리에 `Counter`가 있고 두 번째 자리는 비어있다.
- ”Next player”를 클릭하면 첫 번째 자리는 비워지고 두 번째 자리에 `Counter`가 온다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/a7a42db7-6ae0-4388-920b-2b031116d9dd/Untitled.png)

> 각 `Counter`의 state는 DOM에서 지워질 때마다 제거됩니다. 이것이 버튼을 누를 때마다 초기화되는 이유입니다.
> 

이 방법은 같은 자리에 적은 수의 독립된 컴포넌트만을 가지고 있을 때 편리하다. 이 예시에서는 두 개밖에 없기 때문에 JSX에서 각각 렌더링하기 번거롭지 않다.

### **선택지 2: key를 이용해 state를 초기화하기**

State를 초기화하는 좀 더 일반적인 방법이 또 있습니다.

배열을 렌더링할 때 key를 봤을텐데, key는 배열을 위한 것만은 아니다.

- React가 컴포넌트를 구별 할 수 있도록 key를 사용할 수도 있다.
- key를 사용하면 React에게 단지 첫 번째 카운터나 두 번째 카운터가 아니라 특정한 카운터라고 말해줄 수 있다.

- `<Counter />`는 JSX에서 같은 위치에 나타나지만, state를 공유하지는 않습니다.

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

Taylor와 Sarah를 바꾸지만, state를 유지하지는 않습니다. **당신이 다른 `key`를 주었기** 때문이다.

```jsx
{isPlayerA ? (
  <Counter key="Taylor" person="Taylor" />
) : (
  <Counter key="Sarah" person="Sarah" />
)}
```

- key를 명시하면 React는 부모 내에서 순서 대신에 key 자체를 위치한 일부로 사용한다.
- 이것이 컴포넌트를 JSX에서 같은 자리에 렌더링하지만, React 관점에서는 다른 카운터인 이유
- 카운터가 화면에 나타날 때 마다 state가 새로 만들어진다. (카운터가 제거되면 state도 제거)

> key가 전역적으로 유일하지 않다는 것을 기억해야 합니다. key는 오직 부모 안에서만 자리를 명시합니다.
> 

### **key를 이용해 폼을 초기화하기**

key로 state를 초기화하는 것은 특히 폼을 다룰 때 유용하다.

다음 채팅 앱에서 <Chat> 컴포넌트는 입력 state를 가지고 있다.

[코드가 여러개여서 링크를 첨부](https://react.dev/learn/preserving-and-resetting-state#resetting-a-form-with-a-key)

- 입력란에 타이핑한 후 “Alice”나 “Bob”을 눌러 다른 수신자를 선택해봅시다.
- <Chat> 은 이 트리의 같은 곳에서 렌더링되어지기 때문에 입력값이 유지된다.
- 많은 앱에서 이런 동작을 원하겠지만, 채팅앱에서는 아니다.

컴포넌트에 Key를 추가 해봅시다.

<aside>
⚠️ 제거된 컴포넌트의 state를 보존하기

</aside>

- 실제 채팅 앱에서는 이전 수신자를 선택했을 때 입력 값이 복구되는 것을 원할 것이다.

- 현재 채팅만 렌더링하는 대신, 모든 채팅을 렌더링하고 CSS로 안보이게 할 수 있다.
    - 채팅은 트리에서 사리지지 않을 것 이고 state는 유지된다.
    - 숨겨진 트리가 크고 많은 DOM 노드를 가지고 있다면 매우 느려질 것
- state를 상위로 올리고 각 수신자의 임시 메시지를 부모 컴포넌트에 가지고 있을 수 있다.
    - 일반 적인 해법이다.
- React state 이외의 다른 저장소를 이용할 수 있다.
    - localStorage에 메시지를 저장하고 이를 이용해 Chat 컴포넌트를 초기화 할 수 있다.

요약

- React는 같은 컴포넌트가 같은 자리에 렌더링되는 한 state를 유지한다.
- state는 JSX 태그에 저장되지 않는다. state는 JSX로 만든 트리 위치와 연관된다.
- 컴포넌트에 다른 Key를 주어서 그 하위 트리를 초기화 하도록 강제할 수 있다.
- 중첩해서 컴포넌트를 정의하면 원치 않게 state가 초기화 될 수 있기 때문에 그렇게 하지말아야 한다.
