# State as a Snapshot

[State as a Snapshot – React](https://react.dev/learn/state-as-a-snapshot)

상태 변수는 읽고 쓸 수 있는 일반적인 자바스크립트의 변수처럼 보이곤 합니다. 그러나 상태는 스냅샷과 더 유사하게 동작합니다. 설정은 이미 가지고 있는 상태 변수를 변경하지 않고 대신 리렌더링이 트리거됩니다.

### 이번 파트에서 배울 내용

- state를 변경하고 리렌더링을 트리거하는 방법
- 언제 그리고 어떻게 상태를 업데이트하는지
- 왜 상태는 변경하자마자 바로 업데이트 되지 않는지
- 이벤트 핸들러가 상태의 스냅샷에 접근하는 방법

## **Setting state triggers renders**

클릭과 같이 유저 이벤트에 반응하여 사용자 인터페이스가 직접 변경된다고 생각할 수 있습니다. 이 동작의 멘탈 모델은 리액트에서 동작하는 방식과는 조금 다를 수 있습니다. 이전 페이지에서 리액트에서 상태를 설정하고 리렌더링을 요청하는 것을 설명했습니다. 즉, 이벤트에 반응하기 위해서는 상태를 업데이트 해야합니다.

예제에서는 “send” 버튼을 누를 때 `setIsSent(true)` 가 리액트에게 리렌더링하도록 말합니다.

```tsx
export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Hi!');
  if (isSent) {
    return <h1>Your message is on its way!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

1. `onSubmit` 이벤트가 실행됩니다
2. `setIsSent(true)` 는 `isSent` 를 `true` 로 설정하고 새로운 렌더링을 큐에 추가합니다.
3. 리액트는 새로운 `isSent` 값에 따라 컴포넌트를 리렌더링합니다

## **Rendering takes a snapshot in time**

**“리렌더링”**은 리액트가 컴포넌트(함수)를 호출하는 것을 의미합니다. 함수에서 반환된 JSX는 시간상의 UI 스냅샷과 유사합니다. 컴포넌트의 props, 이벤트 핸들러, 지역 변수는 모두 렌더링 당시의 상태(state)로 계산된 값입니다.

사진이나 동영상 프레임과는 다르게 UI 스냅샷은 상호작용? 대화형?(interactive)이 가능합니다. 스냅샷은 입력에 반응하여 특정한 동작을 하는 이벤트 핸들러와 같은 로직을 포함하고 있습니다. 리액트는 스냅샷과 일치하도록 화면을 업데이트하고 이벤트 핸들러와 연결합니다. 그 결과, 버튼을 누르면 JSX의 클릭 핸들러가 발동됩니다.

리액트가 컴포넌트를 리렌더링 할 때

1. 리액트는 함수를 다시 호출합니다.
2. 함수는 새로운 JSX 스냅샷을 반환합니다.
3. 리액트는 함수에서 리턴된 스냅샷과 일치하는 화면으로 DOM을 업데이트합니다.

컴포넌트 메모리로서 상태는 함수가 반환한 후에 사라지는 일반적인 변수와 다릅니다. 상태는 실제로 함수 외부에 있는 것처럼 리액트 자체에 **존재**합니다. 리액트가 컴포넌트를 호출하면 특정 렌더링에 대한 상태의 스냅샷을 제공합니다. 컴포넌트는 렌더링의 state값을 사용하여 계산된 새로운 props와 이벤트 핸들러가 포함된 UI의 스냅샷을  JSX에 반환합니다.

https://github.com/eco-frontend/react-weekly-meetup/assets/33442498/9261b5e7-aea7-432d-8eb4-b3a609d28e8f

위 작업에 대한 작은 실험 예제입니다. +3 버튼을 클릭하면 카운터가 3으로 올라갈 것 이라고 예측할 수 있습니다.

```tsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

실제로 실행해보면 숫자는 클릭한번에 한번만 증가하고있습니다..!

**설정된 상태는 오로지 다음 렌더링 때만 변합니다**. 첫 렌더링 때 `number`는 0입니다. 따라서 해당 렌더링의 onClick 핸들러에서 `setNumber(number+1)` 이 호출된 후에도 여전히 0입니다.

1. setNumber(number + 1); // number = 0, 0 + 1
    
    리액트는 다음 렌더링 때 number를 1로 변경할 준비를 합니다.
    
2. setNumber(number + 1); // number = 0, 0 + 1
    
    리액트는 다음 렌더링 때 number를 1로 변경할 준비를 합니다.
    
3. setNumber(number + 1); // number = 0, 0 + 1
    
    리액트는 다음 렌더링 때 number를 1로 변경할 준비를 합니다.
    

`setNumber(number + 1)`이 3번 호출되었을지라도, 이번 렌더링의 이벤트 핸들러의 `number`는 항상 0입니다. 그래서 상태를 1로 변경하는 작업을 세번 실행한 것입니다. 이벤트 핸들러가 종료된 후에 리액트가 `number`를 3이 아닌 1로 컴포넌트를 리렌더링 하는 이유입니다. 

코드에서 state 변수를 해당 값으로 대입하여 이를 시각화할 수도 있습니다. 이 렌더링에서 `number` state 변수는 `0`이므로 이벤트 핸들러는 다음과 같습니다.

```tsx
<button onClick={() => {
  setNumber(0 + 1);
  setNumber(0 + 1);
  setNumber(0 + 1);
}}>+3</button>
```

다음 렌더링에 number는 1이므로 아래 코드와 같습니다.

```tsx
<button onClick={() => {
  setNumber(1 + 1);
  setNumber(1 + 1);
  setNumber(1 + 1);
}}>+3</button>
```

## **State over time**

아래 코드의 동작을 추측해보세요.

```tsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        alert(number);
      }}>+5</button>
    </>
  )
}
```

알럿에서 0을 보여주는 것을 추측할 수 있습니다.

그러나 alert에 타이머를 추가해서 컴포넌트가 리렌더된 후에만 보여지게 하면 어떨까요?

```tsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setTimeout(() => {
          alert(number);
        }, 3000);
      }}>+5</button>
    </>
  )
}

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber((prev) => {
	        setTimeout(() => {
	          alert(prev);
	        }, 3000);
        
	        return prev + 5
        });
      }}>+5</button>
    </>
  )
}
```

대체 메서드를 사용하면 alert에 전달된 상태의 스냅샷을 볼 수 있습니다.

```tsx
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

리액트에 저장된 상태는 alert가 실행되는 시점에 변경되었을 수 있지만, 상태의 스냅샷을 이용하는 건 이미 예약되어있던 것 입니다.

이벤트 핸들러의 코드가 비동기 동작이더라도 state 변수의 값은 렌더링 내에서 절대로 변하지 않습니다. 렌더링의 `onClick`내부에서 `number`의 값은 `setNumber(number + 5)`가 호출된 이후에도 계속해서 0입니다. 

이 값은 컴포넌트를 호출해 React가 UI의 스냅샷을 찍을 때 고정된 값입니다.

이벤트 핸들러가 타이밍 실수를 줄이는 방법을 보여주는 예시입니다. 5초 지연 후 메세지를 보냅니다.

시나리오를 상상해보세요

1. “send” 버튼을 누르면, Alice에게 “Hello” 를 보냅니다
2. 5초가 지나기전에, To 필드를 Bob으로 변경합니다

alert에 어떤 메세지가 보인다고 예상이 되나요? “You said Hello to Alice” 또는  “You said Hello to Bob”?

```tsx
export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Hello');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`You said ${message} to ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{' '}
        <select
          value={to}
          onChange={e => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

리액트는 렌더링의 이벤트 핸들러 내에서 상태 값을 고정으로 유지합니다. 코드가 실행되는 동안 state가 변경되었는지 걱정하지 않아도 괜찮습니다!

그러나 리렌더링 전에 최신 상태값을 읽기 원한다면 다음 챕터의 state갱신 함수를 사용하면 됩니다.