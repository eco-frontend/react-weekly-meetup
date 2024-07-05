# 3-7. Scaling Up with Reducer and Context

[Scaling Up with Reducer and Context – React](https://react.dev/learn/scaling-up-with-reducer-and-context)

리듀서는 컴포넌트의 상태(state) 업데이트 로직을 통합할 수 있습니다. 컨텍스트(Context)는 다른 컴포넌트에 정보를 더 깊이 전달할 수 있습니다. 리듀서와 컨텍스트를 결합하여 복잡한 화면의 상태를 함께 관리할 수 있습니다.

# **Combining a reducer with context**

리듀서 소개 예제에서 상태를 리듀서를 이용하여 관리하는 방법을 알아보았습니다. 리듀서 함수는 모든 상태의 업데이트 로직을 포함하고 아래 예제처럼 App의 맨 아래에 선언했습니다.

```tsx
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

리듀서는 이벤트 핸들러를 짧고 간결하게 유지할 수 있도록 도와줍니다. 그러나 앱이 커지면 또 다른 어려움을 마주할 수 있습니다. **현재 `task` 상태와 `dispatch` 함수는 `TaskApp` 컴포넌트의 최상단에서만 이용 가능 합니다.** 다른 컴포넌트가 작업 목록을 읽거나 변경하기 위해서는 현재 상태와 상태를 변경하는 이벤트 핸들러를 props를 통해서 명시적으로 아래로 전달해야합니다.

예를 들어, `TaskApp`은 작업 리스트와 이벤트 핸들러를 `TaskList`에 전달합니다.

```tsx
<TaskList
	tasks={tasks}
	onChangeTask={handleChangeTask}
	onDeleteTask={handleDeleteTask}
/>
```

그리고 `TaskList`는 `Task`에 이벤트 핸들러를 전달합니다.

```tsx
<Task 
	task={task}
	onChange={onChangeTask}
	onDelete={onDeleteTask}
/>
```

위와 같은 작업 예제에서는 잘 동작합니다. 그러나 10개 혹은 100개의 컴포넌트가 중간에 있다고 가정하면 모든 상태와 함수를 아래로 전달하는 것은 상당히 답답할 수 있습니다.

props를 통해서 이를 전달하는 대신에, tasks 상태와 dispatch 함수를 둘 다 컨텍스트에 넣고자 할 수 있습니다. 이 방법은 `TaskApp` 트리 아래에 위치하는 모든 컴포넌트가 “prop drilling” 없이 tasks와 dispatch actions을 읽을 수 있도록 합니다.

다음과 같은 방법으로 리듀서와 컨텍스트를 결합할 수 있습니다.

1. 컨텍스트를 생성합니다.
2. 컨텍스트안에 상태와 dispatch를 넣습니다.
3. 트리 안에서 컨텍스트를 사용합니다.

### Step 1: 컨텍스트 생성

`useReducer` 훅은 현재 `tasks`와 이를 업데이트하는 `dispatch` 함수를 반환합니다.

```tsx
const [tasks, dispatch] = useReduecer(tasksReducer, initialTasks);
```

트리 하위에 이를 전달하기 위해서 두개의 분리된 컨텍스트를 생성해야합니다.

- `TaskContext`는 작업의 현재 리스트를 제공합니다.
- `TaskDispatchContext`는 컴포넌트에서 action을 dispatch 하는 함수를 제공합니다.

별도의 파일에서 이를 내보내고(export) 나중에 다른 파일에서 가져올 수 있도록 합니다.

두 컨텍스트에 기본 값으로 null을 전달합니다. 실제 값은 `TaskApp` 컴포넌트에서 제공될 것 입니다.

```tsx
export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

### Step 2: 컨텍스트에 상태와 dispatch 넣기

이제 `TaskApp` 컴포넌트에 컨텍스트 둘 다 가져올 수 있습니다. `useReducer()`에 의해 반환된 `tasks`와 `dispatch`를 가져와 아래와 같이 전체 트리에 제공하세요.

이제 정보를 props를 통해서 컨텍스트에 전달합니다. 그 다음은 props 전달을 삭제할 것 입니다.

```tsx
import { TasksContext, TasksDispatchContext } from './TasksContext.js' 

export default function TaskApp() {
	const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
	
	return (
		<TasksContext.Provider value={tasks}>
			<TasksDispatchContext.Provider value={dispatch}>
				...
			</TasksDispatchContext.Provider>
		</TasksContext.Provider>
	)
}
```

### Step 3: 트리에서 컨텍스트 사용

이제 tasks 목록 또는 이벤트 핸들러를 컴포넌트 트리로 전달할 필요가 없습니다.

```tsx
<TasksContext.Provider value={tasks}>
	<TasksDispatchContext.Provider value={dispatch}>
		<AddTask />
		<TaskList />
	</TasksDispatchContext.Provider>
</TasksContext.Provider>
```

대신 필요한 컴포넌트에서는 `TaskContext`로부터 `task` 목록을 읽을 수 있습니다.

```tsx
export default function TaskList() {
	const tasks = useContext(TasksContext);
}
```

task 목록을 업데이트하기 위해서 필요한 컴포넌트에서 컨텍스트로부터 `dispatch`함수를 읽을 수 있고 호출할 수 있습니다.

```jsx
export default function AddTask() {
	const [text, setText] = useState('');
	const dispatch = useContext(TaskDispatchContext);
	
	return (
		<button onClick={() => {
			setText('');
			dispatch({
				type: 'added',
				id: nextId++,
				text: text
			});
		}}>Add</button>
	)
}
```

`TaskApp` 컴포넌트는 더 이상 어떤 이벤트 핸들러도 아래로 전달하지 않고 `TaskList` 역시 어떠한 이벤트 핸들러도 `Task` 컴포넌트에 전달하지 않습니다. 각 컴포넌트는 필요한 컨텍스트를 읽습니다.

```jsx
export default function TaskList() {
	const tasks = useContext(TasksContext);
	return (
		...
	)
}

function Task({ task }) {
	const [isEditing, setIdEditing] = useState(false);
	const dispatch = useContext(TasksDispatchContext);
	...
}
```

상태(state)는 `TaskApp`의 제일 상위에서 살아있고 `useReducer`를 통해 관리됩니다. 그러나 이제 컨텍스트를 가져와 트리 아래의 모든 컴포넌트에서 `tasks` 및 `dispatch`를 이용할 수 있습니다. 

# Moving all wiring into a single file

반드시 이런 방식으로 작성하지 않아도 되지만 리듀서와 컨텍스트 모두 하나의 파일로 이동하여 컴포넌트를 더 깔끔하게 정리할 수 있습니다. 현재 `TasksContext.js` 는 두 개의 컨텍스트 선언을 가지고 있습니다.

```tsx
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

이 파일은 더 복잡해질 것입니다. 리듀서를 같은 파일로 옮기고 새로운 `TasksProvider` 컴포넌트를 같은 파일 내에 선언합니다. 이 컴포넌트는 모든 것을 하나로 묶는 역할을 하게 됩니다.

1. 리듀서를 통해 상태를 관리합니다
2. 두 개의 컨텍스트를 하위 컴포넌트로 전달합니다
3. `children`을 `props`로 제공하기 때문에 JSX를 전달할 수 있습니다.

```tsx
export function TasksProvider({ children }) {
	const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
	
	return (
		<TasksContext.Provider value={tasks}>
			<TaskDispatchContext.Provider value={dispatch}>
				{children}
			</TaskDispatchContext.Provider>
		</TasksContext.Provider>
	)
}
```

이렇게 하면 TaskApp 컴포넌트에서 모든 복잡성과 연결이 제거됩니다.

```tsx
export default function TasksApp() {
	return (
		<TasksProvider>
			<AddTask />
			<TaskList />
		</TasksProvider>
	)
}
```

또한 `TaskContext.js`로 부터 컨텍스트를 사용하는 함수를 내보낼 수 있습니다.

```tsx
export function useTasks() {
	return useContext(TasksContext);
}

export function useTasksDispatch() {
	return useContext(TasksDispatchContext);
}
```

컴포넌트가 컨텍스트를 읽어야 하는 경우 이러한 함수를 통해 수행할 수 있습니다.

```tsx
const tasks = useTasks();
const dispatch = useTasksDispatch();
```

이렇게 해도 어떤 동작도 변하지 않습니다. 그러나 나중에 이러한 컨텍스트를 더 분할하거나 이러한 함수에 로직을 추가할 수 있습니다. 이제 컨텍스트와 리듀서 모두 `TaskContext.js`에 연결되어 있습니다. 이것은 컴포넌트를 깨끗하고 깔끔하게 유지하고 데이터를 어디서 가져오는지가 아닌 무엇을 노출할지에 더 초점을 맞출수 있도록 합니다.

```tsx
export default function TaskList() {
	const tasks = useTasks();
	
	return (
		<ul>
			{tasks.map((task) => (
				<li key={task.id}>
					<Task task={task} />
				</li>
			))}
		</ul>
	)
}
```

tasks를 다루는 방법을 알고있는 화면의 일부로 `TasksProvider`를, 이를 읽는 방법으로서 `useTasks`를, 하위 트리의 컴포넌트로부터 이를 업데이트하는 방법으로서 `useTasksDispatch`를 생각하면 됩니다.

### Note

useTasks와 useTasksDispatch와 같은 함수는 커스텀 훅이라고 불립니다. 함수의 이름이 use로 시작한다면 당신의 함수는 커스텀 훅으로 간주됩니다. 이렇게하면 안에서 다른 훅(useContext와 같이)을 사용할 수 있습니다.

앱이 커지게되면 위와 같은 context-reducer 쌍이 아주 많이 생길 수 있습니다. 이는 트리의 깊은 곳의 데이터에 접근하기를 원할때마다 큰 작업 없이 앱을 확장하고 상태를 끌어올릴 수 있는 강력한 방법입니다.

# Recap

- 어떤 컴포넌트에서라도 상태를 읽고 업데이트하기 위해서 컨텍스트와 함께 리듀서를 결합할 수 있습니다.
- 상태와 dispatch 함수를 하위 컴포넌트에 제공하는 방법
    1. state와 dispatch 함수를 위해서 두개의 컨텍스트를 생성합니다
    2. Reducer를 사용하는 컴포넌트에 두 context를 모두 제공합니다
    3. 하위 컴포넌트들에서 필요한 context를 사용합니다
- 더 나아가 하나의 파일로 합쳐서 컴포넌트들을 정리할 수 있습니다.
    - `Context`를 제공하는 `TasksProvider`와 같은 컴포넌트를 내보낼 수 있습니다.
    - 바로 사용할 수 있도록 `useTasks`와 `useTasksDispatch`같은 커스텀 훅을 내보낼 수 있습니다.
- context-reducer 조합을 앱에 여러개 만들 수 있습니다.

```jsx
const Component = () => {
	const ctx = useContext(C)
	const [a,b] = useState()

	return (
		<C.Provider value={a}>
			<CA />
			{ctx && <CB ctx={ctx} />} //이런식이 문제가 아니였을지...
		</C.Provider>
	)
}

const CA = () => {
	const ctx = useContext(C)
	
	return ...
}

const CB = () => { // 리렌더링 되니?
	return ...
}

const C = createContext()
```

```jsx
const Component = ({ children }) => {

	return (
		<C.Provider ={{ value }}> // change, everyone will re-render
			{children}
		</C.Provider>
	)
}

const Component = () => {
	const memoValue = useMemo(() => ({ value })) 
	
	return (
		<C.Provider ={memoValue}> // no-one re-rendering
			{children}
		</C.Provider>
	)
}
```
