# 2-5. state 업데이트 큐

작성자: sikiy
날짜: 2024년 5월 15일

# 개요

state 변수를 설정하면 다음 렌더링이 큐에 들어간다. 그러나 때에 따라 다음 렌더링을 큐에 넣기 전에, 값에 대해 여러 작업을 수행하고 싶을 때도 있다. 이를 위해선 React가 state업데이트를 어떻게 배치하면 좋을지 이해하는 것이 도움이 된다.

## 학습 내용

- “batching”이란 무엇이며, React가 여러 state 업데이트를 처리하는 방법
- 동일한 state 변수에서 여러 업데이트를 연속으로 적용하는 방법

## React state batches 업데이트

setNumber(number + 1)를 세 번 호출하므로 “+3” 버튼을 클릭하면 세 번 증가할 것으로 예상할 수 있다.

- 코드
    
    ```jsx
    import { useState } from 'react';
    
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
    

이전 세션에서 기억할 수 있듯 각 렌더링 state값은 고정되어 있으므로, 첫 번째 렌더링의 이벤트 핸들러의 `number`값은 `setNumber(1)`을 몇 번 호출하든 항상 0이다.

```jsx
setNumber(0 + 1)
setNumber(0 + 1)
setNumber(0 + 1)
```

하지만 여기에는 한가지 요인이 더 있다. React는 state를 업데이트 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때 까지 기다린다. 이 때문에 리렌더링은 모든 `setNumber()` 호출이 완료된 이후에만 일어난다.

이는 음식점에서 주문 받는 웨이터를 생각해볼 수 있다. 웨이터는 첫 번째 요리를 말하자마자 주방으로 달려가지 않는다. 대신 주문이 끝날때까지 기다렸다가 주문을 변경하고, 심지어 테이블에 있는 다른사람의 주문도 받는다.

![Untitled](2-5%20state%20%E1%84%8B%E1%85%A5%E1%86%B8%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%B3%20%E1%84%8F%E1%85%B2%2029703dad541a4c0ab70e35554916af10/Untitled.png)

이렇게 하면 너무 많은 리렌더링이 발생하지 않고도 여러 컴포넌트에서 나온 다수의 state 변수를 업데이트할 수 있다. 하지만 이는 이벤트 핸들러와 그 안에 있는 코드가 완료될 때 까지 UI가 업데이트 되지 않는다는 의미이기도 하다. batching이라고도 하는 이 동작은 React 앱을 훨씬 빠르게 실행할 수 있게 해준다. 또한 일부 변수만 업데이트된 “반쯤 완성된” 혼란스러운 렌더링을 처리하지 않아도 된다.

React는 클릭과 같은 여러 의도적인 이벤트에 대해 batch하지 않으며, 각 클릭은 개별적으로 처리된다. React는 안전한 경우에만 batch를 수행하니 안심하자. 예를 들어 첫 번째 버튼 클릭으로 양식이 비활성화 되면 두 번째 클릭으로 양식이 다시 제출되지 않도록 보장한다.

## 다음 렌더링 전에 동일한 state변수를 여러 번 업데이트 하기

흔한 사례는 아니지만, 다음 렌더링 전에 동일한 state변수를 여러 번 업데이트 하고 싶다면 `setNumber(number + 1)`와 같이 쓰지 말고 `setNumber(n => n + 1)`과 같이 이전 큐의 state를 기반으로 다음 state를 계산하도록 하면 된다. 이는 단순히 state 값을 대체하는 것이 아니라 React에 “state 값으로 무언가를 하라”고 지시하는 방법이다.

- 코드
    
    ```jsx
    import { useState } from 'react';
    
    export default function Counter() {
      const [number, setNumber] = useState(0);
    
      return (
        <>
          <h1>{number}</h1>
          <button onClick={() => {
            setNumber(n => n + 1);
            setNumber(n => n + 1);
            setNumber(n => n + 1);
          }}>+3</button>
        </>
      )
    }
    ```
    

여기서 `n => n + 1`은 업데이터 함수(updater function)라고 부른다. 이를 state 설정자 함수에 전달할 때,

1. React는 이벤트 핸들러의 다른 코드가 모두 실행된 후에 이 함수가 처리되도록 큐에 넣는다.
2. 다음 렌더링 중에 React는 큐를 순회하여 최종 업데이트된 state를 제공한다.

```jsx
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

![Untitled](2-5%20state%20%E1%84%8B%E1%85%A5%E1%86%B8%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%B3%20%E1%84%8F%E1%85%B2%2029703dad541a4c0ab70e35554916af10/Untitled%201.png)

React는 `3`을 최종 결과로 저장하고 `useState`에서 반환한다.

이것이 위 예제 “+3”을 클릭하면 값이 3씩 올바르게 증가하는 이유다.

### state를 교체한 후 업데이트 하면 어떻게 될까?

이 이벤트 핸들러는 어떨까? 다음 렌더링에서 `number`가 어떻게 될까?

```jsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

이 이벤트 핸들러가 React에게 지시하는 작업은 다음과 같다.

1. `setNumber(number + 5)`: `number`는 `0` 이므로 `setNumber(0 + 5)`이다. React는 큐에 “5로 바꾸기”를 추가한다.
2. `setNumber(n => n+1)` : `n => n+1` 는 업데이터 함수이다. React는 해당 함수를 큐에 추가한다.

다음 렌더링하는 동안 React는 state큐를 순회한다.

![Untitled](2-5%20state%20%E1%84%8B%E1%85%A5%E1%86%B8%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%B3%20%E1%84%8F%E1%85%B2%2029703dad541a4c0ab70e35554916af10/Untitled%202.png)

react는 `6`을 최종 결과로 저장하고 `useState`에서 반환한다.

<aside>
💡 중요함!

`setState(5)`가 실제로는 `setState(n => 5)`처럼 동작하지만 `n`이 사용되지 않는다는 것을 눈치챘을 것.

</aside>

### 업데이트 후 state를 바꾸면 어떻게 될까?

다음 렌더링 예에선 `number`가 어떻게 될까?

```jsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

1. `setNumber(number + 5)`: `number`는 `0`이므로 `setNumber(0 + 5)`이다. React는 “5로 바꾸기”를 큐에 추가한다.
2. `setNumber(n => n+1)` : `n => n +1` 는 업데이터 함수다. React는 함수를 큐에 추가한다.
3. `setNumber(42)`: React는 “`42`로 바꾸기”를 큐에 추가한다.

![Untitled](2-5%20state%20%E1%84%8B%E1%85%A5%E1%86%B8%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%B3%20%E1%84%8F%E1%85%B2%2029703dad541a4c0ab70e35554916af10/Untitled%203.png)

그런 다음 React는 `42`를 최종 결과로 저장하고 `useState`에서 반환한다.

요약하자면 `setNumber` state 설정자 함수에 전달할 내용은 다음과 같이 생각할 수 있다:

- 업데이터 함수 (예. `n => n+1`)가 큐에 추가된다.
- 다른 값 (예. 숫자 `5`)은 큐에 “ `5` 로 바꾸기”를 추가하며, 이미 이전 큐에 대기중인 항목은 무시한다.

이벤트 핸들러가 완료되면 React는 리렌더링을 실행한다. 리렌더링하는 동안 React는 큐를 처리한다. 업데이터 함수는 렌더링 중에 실행되므로, 업데이터 함수는 순수해야 하며 결과만 반환해야 한다. 업데이터 함수 내부에서 state를 변경하거나 다른 사이드 이펙트를 실행하려고 하지 말라. Strict모드에서 react는 각 업데이터 함수를 두 번 실행(두번째 결과는 버림)하여 실수를 찾을 수 있도록 도와준다.

### 명명 규칙

업데이터 함수 인수의 이름은 state변수의 첫 글자로 지정하는 것이 일반적이다.

```jsx
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

좀 더 자세한 코드를 선호하는 경우 `setEnabled(enabled => !enabled)`와 같이 전체 state 변수 이름을 반복하거나, `setEnabled(prevEnabled => !prevEnabled)`와 같은 접두사를 사용하는 것이 널리 사용되는 규칙이다.

## 요약

- state를 설정하더라도 기존 렌더링의 변수는 변경되지 않으며, 대신 새로운 렌더링을 요청
- React는 이벤트 핸들러가 실행을 마친 후 state 업데이트를 처리합니다. 이를 batching이라고 함.
- 하나의 이벤트에서 일부 state를 여러 번 업데이트하려면 `setNumber(n => n + 1)` 업데이터 함수를 사용할 수 있다.

## 챌린지

[state 업데이트 큐 – React](https://ko.react.dev/learn/queueing-a-series-of-state-updates#challenges)

하은

```jsx
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  // TODO: do something with the queue...
  queue.forEach((q) => {
    if(typeof q === 'function') {
      finalState = q(finalState)
    }else {
      finalState = q
    }
  })

  return finalState;
}
```

정민 

```jsx
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  // TODO: do something with the queue...
	
	return queue.map((v) => {
		if(typeof v === 'function'){
			return finalState = v(finalState)
		}else{
			return finalState = finalState 
		}
	})

}

```

은식

```jsx
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  // TODO: do something with the queue...

  queue.map((qu) => {
    let qui = qu
    if (typeof qu !== 'function') {
      qui = () => qu
    }

    finalState = qui(finalState)
  })

  return finalState;
}

```
