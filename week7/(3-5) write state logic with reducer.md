# 3-5. state 로직을 리듀서로 작성하기

작성자: sikiy
날짜: 2024년 6월 5일

[state 로직을 reducer로 작성하기 – React](https://ko.react.dev/learn/extracting-state-logic-into-a-reducer)

# 개요

한 컴포넌트에서 state 업데이트가 여러 이벤트 핸들러로 분산되는 경우가 있다. 이 경우 컴포넌트를 관리하기 어려워 진다. 따라서, 이 문제 해결을 위해 state를 업데이트하는 모든 로직을 reducer를 사용해 컴포넌트 외부로 단일 함수로 통합해 관리할 수 있다.

## 학습 내용

- reducer 함수란 무엇인가
- `useState` 에서 `useReducer`로 리팩토링 하는 방법
- reducer를 언제 사용할 수 있는지
- reducer를 잘 작성하는 방법

## reducer를 사용하여 state 로직 통합하기

컴포넌트가 복잡해지면 컴포넌트의 state가 업데이트 되는 다양한 경우를 한눈에 파악하기 어려워 질 수 있다.

아래 예시코드에선 `TaskApp` 컴포넌트는 state에 `tasks` 배열을 보유하고 있고, 세 가지의 이벤트 핸들러를 사용해서 task를 추가, 제거 및 수정 한다.

- 예시코드
    
    ```jsx
    import { useState } from 'react';
    import AddTask from './AddTask.js';
    import TaskList from './TaskList.js';
    
    export default function TaskApp() {
      const [tasks, setTasks] = useState(initialTasks);
    
      function handleAddTask(text) {
        setTasks([...tasks, {
          id: nextId++,
          text: text,
          done: false
        }]);
      }
    
      function handleChangeTask(task) {
        setTasks(tasks.map(t => {
          if (t.id === task.id) {
            return task;
          } else {
            return t;
          }
        }));
      }
    
      function handleDeleteTask(taskId) {
        setTasks(
          tasks.filter(t => t.id !== taskId)
        );
      }
    
      return (
        <>
          <h1>Prague itinerary</h1>
          <AddTask
            onAddTask={handleAddTask}
          />
          <TaskList
            tasks={tasks}
            onChangeTask={handleChangeTask}
            onDeleteTask={handleDeleteTask}
          />
        </>
      );
    }
    
    let nextId = 3;
    const initialTasks = [
      { id: 0, text: 'Visit Kafka Museum', done: true },
      { id: 1, text: 'Watch a puppet show', done: false },
      { id: 2, text: 'Lennon Wall pic', done: false },
    ];
    
    ```
    

각 이벤트 핸들러는 state를 업데이트하기 위해 `setTasks`를 호출한다. 컴포넌트가 커질수록 그 안에서 state를 다루는 로직의 양도 늘어나게 된다. 

복잡성은 줄이고 접근성은 높이기 위해 컴포넌트 내부의 state 로직을 컴포넌트 외부의 `reducer 라고 하는` 단일 함수로 옮길 수 있다.

reducer는 state를 다루는 방법이고, 다음과 같은 세가지 단계에 걸쳐서 `useState`에서 `useReducer`로 바꿀 수 있다.

1. state를 설정하는 것에서 action을 dispatch 함수로 전달하는 것으로 바꾸기
2. reducer 함수 작성하기
3. 컴포넌트에서 reducer 사용하기

### 1단계: state를 설정하는 것에서 action을 dispatch 함수로 전달하는 것으로 바꾸기

현재 이벤트 핸들러는 state를 설정함으로써 **무엇을 할 것인지**를 명시한다.

```jsx
function handleAddTask(text) {
  setTasks([...tasks, {
    id: nextId++,
    text: text,
    done: false
  }]);
}

function handleChangeTask(task) {
  setTasks(tasks.map(t => {
    if (t.id === task.id) {
      return task;
    } else {
      return t;
    }
  }));
}

function handleDeleteTask(taskId) {
  setTasks(
    tasks.filter(t => t.id !== taskId)
  );
}
```

위 코드에서 state 설정 관련 로직을 전부 지우면 아래 세 가지 이벤트 핸들러가 남는다.

- 사용자가 “Add”를 눌렀을 때 호출되는 `handleAddTask(text)`
- 사용자가 task를 토글하거나 “저장”을 누르면 호출되는 `handleChangeTask(task)`
- 사용자가 “Delete”를 누르면 호출되는 `handleDeleteTask(taskId)`

reducer를 사용한 state 관리는 state 직접 설정하는 것과 약간 다르다. 

state를 설정해서 React에게 “무엇을 할 지”를 지시하는 대신, 이벤트 핸들러에서 “action”을 전달하여 “사용자가 방금 한 일”을 지정한다. (state 업데이트 로직은 다른 곳에 있다!)

즉, 이벤트 핸들러를 통해 “ `task`를 설정” 하는 대신 “task를 추가/변경/삭제”하는 action을 전달하는 것. 이러한 방식이 사용자의 의도를 더 명확하게 한다.

```jsx
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId
  });
}
```

`dispatch` 함수에 넣어준 객체를 “action” 이라고 한다.

```jsx
function handleDeleteTask(taskId) {
  dispatch(
    // "action" 객체:
    {
      type: 'deleted',
      id: taskId
    }
  );
}
```

이 객체는 일반적인 자바스크립트 객체이다. 이 안에 어떤 것이든 자유롭게 넣을 수 있지만, 일반적으로 어떤 상황이 발생하는지에 대한 최소한의 정보를 담고 있어야 한다. ( `dispatch` 함수 자체에 대한 부분은 이후 단계에서 다룰 예정이다.)

### 중요!

action 객체는 어떤 형태든 될 수 있다. 그렇지만 발생한 일을 설명하는 문자열 `type`을 넘겨주고 이외의 정보는 다른 필드에 담아서 전달하도록 하는 것이 일반적이다.

`type`은 컴포넌트에 따라 값이 다르며, 이 예시의 경우 ‘added’ 또는 ‘added_task’ 둘 다 괜찮다. 무슨 일이 일어나는지를 설명할 수 있는 이름을 넣어주면 된다.

```jsx
dispatch({
  // 컴포넌트마다 다른 값
  type: 'what_happened',
  // 다른 필드는 이곳에
});
```

### 2단계: reducer 함수 작성하기

reducer 함수는 state에 대한 로직을 넣는 곳이고, 이 함수는 현재의 state 값과 action 객체, 이렇게 두 개의 인자를 받고 다음 state 값을 반환한다.

```jsx
function yourReducer(state, action) {
  // React가 설정하게될 다음 state 값을 반환합니다.
}
```

React는 reducer에서 반환받은 값을 state에 설정한다.

이 예시에서 이벤트 핸들러에 구현 되어 있는 state 설정과 관련 로직을 reducer 함수로 옮기기 위해서 다음과 같이 해보자.

1. 첫 번째 인자에 현재 state(`tasks`) 선언하기
2. 두 번째 인자에 `action` 객체 선언하기
3. reducer에서 다음 state 반환하기. (React가 state에 설정하게 될 값)

다음은 state 설정과 관련 모든 로직을 reducer 함수로 마이그레이션 한 코드이다.

```jsx
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [...tasks, {
      id: action.id,
      text: action.text,
      done: false
    }];
  } else if (action.type === 'changed') {
    return tasks.map(t => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter(t => t.id !== action.id);
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}
```

reducer 함수는 state(`tasks`)를 인자로 받고 있기 때문에, 이를 컴포넌트 외부에서 선언할 수 있다. 이렇게 하면 들여쓰기 수준이 줄어들고 코드를 더 쉽게 읽을 수 있다.

### 중요!

위 코드에선 if/else 문을 사용하고 있지만 reducer 함수 안에서는 switch문을 사용하는게 규칙이다. 물론 결과는 같지만, switch 문으로 작성된 것이 한눈에 읽기 더 쉬울 수 있다. 

이제부터 이  문서에서 다룰 예시는 아래처럼 switch문을 사용한다.

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

각자 다른 `case` 속에서 선언된 변수들이 서로 충돌하지 않도록 `case` 블록을 중괄호인 `{`와 `}`로 감싸는 걸 추천한다. 또 `case`는 일반적인 경우라면 `return`으로 끝나야 한다. `return` 하는 것을 잊으면 코드가 다음 case로 “떨어져” 실수할 수 있다! (이 부분 실수 잦음)

아직 switch 문에 익숙하지 않다면, if/else 문을 사용해도 괜찮다는 것..!

### DEEP DIVE | 왜 reducer 라고 부르게 되었을까?

[state 로직을 reducer로 작성하기 – React](https://ko.react.dev/learn/extracting-state-logic-into-a-reducer#why-are-reducers-called-this-way)

reducer를 사용하면 컴포넌트 내부의 코드 양을 “줄일 수” 있지만, 실제로는 배열에서 사용하는 [`reduce()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) 연산의 이름에서 따 명명되었습니다.

`reduce()`는 배열의 여러 값을 단일 값으로 “누적”하는 연산을 수행합니다.

`const arr = [1, 2, 3, 4, 5];const sum = arr.reduce(  (result, number) => result + number); // 1 + 2 + 3 + 4 + 5`

`reduce`로 전달하는 함수는 “reducer”로 알려져 있습니다. 이 함수는 지금까지의 결과(result)와 현재 아이템(number)을 인자로 받고 다음 결과를 반환합니다. 비슷한 아이디어의 예로 React의 reducer는 지금까지의 state와 action을 인자로 받고 다음 state를 반환합니다. 이 과정에서 여러 action을 누적하여 state로 반환합니다.

`initialState`와 reducer 함수를 넘겨 받아 최종적인 state 값으로 계산하기 위한 `action` 배열을 인자로 받는 `reduce()` 메서드를 사용할 수도 있습니다.

### 3단계: 컴포넌트에서 reducer 사용하기

마지막으로 컴포넌트에 `taskReducer`를 연결할 차례이므로 React에서 `useReducer` hook을 불러와보자.

```jsx
import { useReducer } from 'react';
```

그런 다음, `useState`를

```jsx
const [tasks, setTasks] = useState(initialTasks)
```

아래 처럼 `useReducer`로 바꾸자

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks); 
```

`useReducer` 훅은 초기 state 값을 입력받아 유상태(stateful) 값을 반환한다는 점과 state를 설정하는 함수(useReducer의 경우는 dispatch 함수를 의미)의 원리를 보면 `useState`와 비슷하다. 

하지만, 조금 다른 점이 있다.

`useReducer`  훅은 두 개의 인자를 넘겨받는다.

1. reducer 함수
2. 초기 state 값

그리고 아래와 같이 반환한다.

1. state를 담을 수 있는 값
2. dispatch 함수 (사용자의 action을 reducer 함수에게 “전달”)

- 변경된 코드
    
    ```jsx
    import { useReducer } from 'react';
    import AddTask from './AddTask.js';
    import TaskList from './TaskList.js';
    
    export default function TaskApp() {
      const [tasks, dispatch] = useReducer(
        tasksReducer,
        initialTasks
      );
    
      function handleAddTask(text) {
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        });
      }
    
      function handleChangeTask(task) {
        dispatch({
          type: 'changed',
          task: task
        });
      }
    
      function handleDeleteTask(taskId) {
        dispatch({
          type: 'deleted',
          id: taskId
        });
      }
    
      return (
        <>
          <h1>Prague itinerary</h1>
          <AddTask
            onAddTask={handleAddTask}
          />
          <TaskList
            tasks={tasks}
            onChangeTask={handleChangeTask}
            onDeleteTask={handleDeleteTask}
          />
        </>
      );
    }
    
    function tasksReducer(tasks, action) {
      switch (action.type) {
        case 'added': {
          return [...tasks, {
            id: action.id,
            text: action.text,
            done: false
          }];
        }
        case 'changed': {
          return tasks.map(t => {
            if (t.id === action.task.id) {
              return action.task;
            } else {
              return t;
            }
          });
        }
        case 'deleted': {
          return tasks.filter(t => t.id !== action.id);
        }
        default: {
          throw Error('Unknown action: ' + action.type);
        }
      }
    }
    
    let nextId = 3;
    const initialTasks = [
      { id: 0, text: 'Visit Kafka Museum', done: true },
      { id: 1, text: 'Watch a puppet show', done: false },
      { id: 2, text: 'Lennon Wall pic', done: false }
    ];
    
    ```
    

위처럼 관심사를 분리하면 컴포넌트의 로직은 읽기 더 쉬워질 수 있다. 이렇게 하면 이벤트 핸들러는 action을 전달해줘서 무슨 일이 일어났는지에 관련한 것만 명시하고, reducer 함수 구현부에선 이에 대한 응답으로 어떤 state가 어떤 값으로 업데이트 될 지 결정하면 됨.

### `useState`와 `useReducer` 비교하기

reducer가 좋은 점만 있는 것은 아니다! 아래 표를 통해 비교해보자.

|  | useState | useReducer |
| --- | --- | --- |
| 코드 크기 | 비교적 적음
하지만 이벤트가 늘어날수록 늘어남. | 비교적 많음.
하지만 이벤트가 늘어날수록 적어짐. |
| 가독성 | 비교적 좋은 편.
하지만 로직이 늘어날수록 어려워 짐 | 비교적 안좋은 편.
하지만 로직이 늘어날수록 명확히 구분되어 읽을 수 있어 쉬워짐. |
| 디버깅 | 어렵다. | 쉽다.
하지만 더 많은 블록의 경우에 따라 로그를 찍어봐야 하는 단점은 있음. |
| 테스팅 | 컴포넌트에 의존되어 어려움. | 컴포넌트에 의존하지 않는 순수 함수이기 때문에 쉬운편. |
| 개인적인 취향 | 선호도의 문제. 개인의 선택. | 선호도의 문제. 개인의 선택. |

만약 일부 컴포넌트에서 잘못된 방식으로 state를 업데이트하는 것으로 인한 버그가 자주 발생하거나 해당 코드에 더 많은 구조를 도입하고 싶다면 reducer 사용을 권장한다!

심지어는 `useState`와 `useReducer`를 마음대로 섞고 배치해서 사용해도 괜찮다!

## reducer 잘 작성하기

reducer를 작성할 때 다음과 같은 두 가지 팁을 명심하라.

- reducer는 반드시 순수해야 한다.
    - state updater functions와 비슷하게 reducer는 렌더링 중에 실행된다. (action은 다음 렌더링까지 대기됨) 때문에 reducer는 반드시 순수 해야한다.
    - 요청을 보내거나 timeout을 스케줄링 하거나 사이드 이펙트(컴포넌트 외부에 영향을 미치는 작업)를 수행해서는 안된다.
    - reducer는 objects와 arrays를 변이 없이 업데이트 해야 한다.
- 각 action은 데이터 안에서 여러 변경들이 있더라도 하나의 사용자 상호작용을 설명해야 한다.
    - 사용자가 reducer가 관리하는 5개의 필드가 있는 양식에서 “재설정”을 누른 경우, 5개의 개별 `set_field` action 보다는 하나의 `reset_form` action을 전송하는 것이 더 합리적.
    - 모든 action을 reducer에 기록하면 어떤 상호작용이나 응답이 어떤 순서로 일어났는지 재구성할 수 있을만큼 로그가 명확해야 하고, 이는 디버깅에 도움이 된다!

## Immer로 간결한 reducer 작성하기

일반적인 state에서 객체와 배열을 업데이트 하는 것 처럼, Immer 라이브러리를 사용하면 reducer를 더 간결하게 작성할 수 있다.

이 라이브러리에서 제공하는 `useImmerReducer`를 사용하여 `push` 또는 `arr[i] =`로 값을 할당하므로써 state를 변경해보자.

- 코드
    - package.json
        
        ```jsx
        {
          "dependencies": {
            "immer": "1.7.3",
            "react": "latest",
            "react-dom": "latest",
            "react-scripts": "latest",
            "use-immer": "0.5.1"
          },
          "scripts": {
            "start": "react-scripts start",
            "build": "react-scripts build",
            "test": "react-scripts test --env=jsdom",
            "eject": "react-scripts eject"
          },
          "devDependencies": {}
        }
        ```
        
    
    ```jsx
    import { useImmerReducer } from 'use-immer';
    import AddTask from './AddTask.js';
    import TaskList from './TaskList.js';
    
    function tasksReducer(draft, action) {
      switch (action.type) {
        case 'added': {
          draft.push({
            id: action.id,
            text: action.text,
            done: false
          });
          break;
        }
        case 'changed': {
          const index = draft.findIndex(t =>
            t.id === action.task.id
          );
          draft[index] = action.task;
          break;
        }
        case 'deleted': {
          return draft.filter(t => t.id !== action.id);
        }
        default: {
          throw Error('Unknown action: ' + action.type);
        }
      }
    }
    
    export default function TaskApp() {
      const [tasks, dispatch] = useImmerReducer(
        tasksReducer,
        initialTasks
      );
    
      function handleAddTask(text) {
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        });
      }
    
      function handleChangeTask(task) {
        dispatch({
          type: 'changed',
          task: task
        });
      }
    
      function handleDeleteTask(taskId) {
        dispatch({
          type: 'deleted',
          id: taskId
        });
      }
    
      return (
        <>
          <h1>Prague itinerary</h1>
          <AddTask
            onAddTask={handleAddTask}
          />
          <TaskList
            tasks={tasks}
            onChangeTask={handleChangeTask}
            onDeleteTask={handleDeleteTask}
          />
        </>
      );
    }
    
    let nextId = 3;
    const initialTasks = [
      { id: 0, text: 'Visit Kafka Museum', done: true },
      { id: 1, text: 'Watch a puppet show', done: false },
      { id: 2, text: 'Lennon Wall pic', done: false },
    ];
    
    ```
    

reducer는 순수해야 하기 때문에 이 안에서는 state를 변형할 수 없다. 하지만, Immer에서 제공하는 특별한 `draft` 객체를 사용하면 안전하게 state를 변형할 수 있다.

Immer 내부적으로는 변경 사항이 반영된 `초안(draft)` 으로 state의 복사본을 생성한다. 이것이 `useImmerReducer`가 관리하는 reducer가 첫 번째 인수인 state를 변형할 수 있고 새로운 state 값을 반활할 필요가 없는 이유다.

## 요약

- `useState`에서 `useReducer`로 변환하려면:
    1. 이벤트 핸들러에서 action을 전달
    2. 주어진 state와 action에 대해 다음 state를 반환하는 reducer 함수를 작성.
    3. `useState`를 `useReducer`로 바꾸기
- reducer를 사용하면 코드를 조금 더 작성해야 하지만 디버깅과 테스트에 도움이 된다.
- reducer는 반드시 순수해야 한다.
- 각 action은 단일 사용자 상호작용을 설명해야 한다.
- 객체와 배열을 변이하는 스타일로 reducer를 작성하려면 Immer 라이브러리를 사용하는 것을 추천

## 챌린지 도전!

[state 로직을 reducer로 작성하기 – React](https://ko.react.dev/learn/extracting-state-logic-into-a-reducer#challenges)