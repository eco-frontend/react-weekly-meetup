# Responding to Events
[Responding to Events – React](https://react.dev/learn/responding-to-events)

리액트에서는 JSX에 이벤트 핸들러를 추가할 수 있도록 제공하고있습니다. 이벤트 핸들러는 클릭, 호버, form 입력에 포커스 등과 같은 상호작용에 반응하여 트리거되는 함수입니다.

이번 장에서 우리가 배울 내용

- 이벤트 핸들러를 작성하는 여러 방법들
- 부모 컴포넌트로부터 이벤트 핸들링을 넘기는 방법
- 이벤트 전파 방법과 막는 방법들

## Adding event handlers

이벤트 핸들러를 추가하기위해서, 첫번째로 함수를 정의해야하고 JSX 태그에 적절한 props를 넘겨줘야합니다.

유저가 클릭을 했을때 메세지를 보이게 하는 기능을 버튼에 추가하고자 할 때 세가지 스텝이 있습니다.

- Button 컴포넌트안에서 호출할 handleClick 함수를 선언
- 함수안에서 `alert` 을 통해서 메세지를 보여주는 로직을 구현
- `onClick` props에 handleClick 함수를 `button`에 추가

```tsx
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

이벤트 핸들러 함수는 대개 컴포넌트 내부에서 정의되고 “handle+이벤트” 와 같은 네이밍을 가집니다.

컨벤션에 의해 이벤트 핸들러의 이름은 “handle”뒤에 이벤트 이름을 붙이는 것이 일반적입니다. `onClick={handleClick}` `onMouseEnter={handleMouseEnter}` 

또는 JSX 에서 인라인 핸들러를 정의할 수 있습니다. 인라인 핸들러는 함수가 짧은 경우 편리합니다.

```tsx
<button onClick={function handleClick() {
	alert('text');
}}>

<button onClick={() => {
	alert('text');
}}>
```

### Pitfall

함수는 이벤트핸들러에 전달하는 함수는 그대로 전달되어야하고 호출해선 안됩니다.

```tsx
<button onClick={handleClick}> // OK

const sum = (a, b) => a + b;

const handleClick = (calcFn) => (event) => {
	calcFn(event.target.value, 1)
}

const handleClick = (event) => {
	sum(event.target.value, 1)
}

<button onClick={handleClick(sum)}> // NOT 
```

차이는 미묘해보입니다. 첫번째 예제에서는 handleClick 함수는 onClick 이벤트 핸들러에 전달되고 리액트는 이를 기억하고 유저가 버튼을 클릭할 때만 호출합니다

두번째 예제는 `()` 가 클릭 없이 렌더링 동안에 함수를 즉시 실행하도록 합니다. JSX의 대괄호 안에서 자바스크립트가 바로 실행되기 때문입니다.

인라인으로 코드를 작성할 때, 각각 다른 방식으로 함정처럼 나타남. 아래와 같은 방법으로 전달하면 컴포넌트가 렌더링될 때마다 실행됩니다. 그래서 익명함수로 한번 감싸서 써야합니다

```tsx
<button onClick={() => alert('...')}> // OK

<button onClick={alert('...')}> // NOT
```

## Reading props in event handlers

컴포넌트의 내부에서 이벤트 핸들러를 선언하기 때문에 컴포넌트의 props에 접근할 수 있습니다.

```tsx
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="Playing!">
        Play Movie
      </AlertButton>
      <AlertButton message="Uploading!">
        Upload Image
      </AlertButton>
    </div>
  );
}

```

## Passing event handlers as props

종종 부모 컴포넌트에서 자식의 이벤트 핸들러를 지정하고 싶은 경우가 있습니다. 버튼 컴포넌트를 사용하는 위치에 따라서 각각 다른 함수를 실행하고 싶을 수 있습니다. 예를 들어 동영상 실행이나 다른 이미지 업로드 등

이를 위해서 컴포넌트가 부모로부터 이벤트 핸들러로 받는 prop을 아래 예제외 같이 전달합니다.

```tsx
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Playing ${movieName}!`);
  }

  return (
    <Button onClick={handlePlayClick}>
      Play "{movieName}"
    </Button>
  );
}

function UploadButton() {
  return (
    <Button onClick={() => alert('Uploading!')}>
      Upload Image
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="Kiki's Delivery Service" />
      <UploadButton />
    </div>
  );
}

```

Toolbar 컴포넌트는 PlayButton과 UploadButton을 렌더링합니다.

- PlayButton은 handlePlayClick을 onClick prop으로 버튼 내부에 전달합니다.
- UploadButton은 익명함수를 onClick prop으로 버튼 내부에 전달합니다.

Button 컴포넌트는 onClick이라는 prop을 받습니다. prop을 브라우저의 내장된 button요소에 onClick을 바로 전달합니다. 클릭 시에 전달된 함수를 호출하도록 리액트에 말합니다.

디자인 시스템을 사용할 때, 특정한 동작이 없이 버튼과 같은 컴포넌트에 스타일링을 포함해서 공통으로 만듭니다. 그리고 PlayButton, UploadButton처럼 이벤트 핸들러를 전달합니다.

## Naming event handler props

button이나 div와 같은 내장 컴포넌트는 onClick과 같은 브라우저 이벤트만 지원합니다. 그러나 직접 컴포넌트를 만들 때는 이벤트 핸들러 props의 이름을 적절하게 만들 수 있습니다.

컨벤션에 의하면 이벤트 핸들러의 props는 “on”으로 시작하고 뒤에는 대문자로 시작해야합니다

예를들어, 버튼 컴포넌트의 onClick prop처럼 onSmash 와 같은 이름을 사용할 수 있습니다.

→ 아래 예제처럼 이름을 꼭 이벤트와 동일하게 맞추지 않아도 된다는 얘기..!

```tsx
function Button({ onSmash, children }) {
  return (
    <button onClick={onSmash}>
      {children}
    </button>
  );
}
```

컴포넌트가 여러가지 상호작용을 제공할 경우, 컨셉에 맞게 이벤트 핸들러 프로퍼티 이름을 지정할 수 있습니다.

예를 들어, Toolbar 컴포넌트의 경우 onPlayMovie, onUploadImage와 같은 이벤트 핸들러를 받을 수 있습니다.

```tsx
function Toolbar({ onPlayMovie, onUploadImage }) {
  return (
    <div>
      <Button onClick={onPlayMovie}>
        Play Movie
      </Button>
      <Button onClick={onUploadImage}>
        Upload Image
      </Button>
    </div>
  );
}

export default function App() {
  return (
    <Toolbar
      onPlayMovie={() => alert('Playing!')}
      onUploadImage={() => alert('Uploading!')}
    />
  );
}
```

여기서 알아야할 점은 App 컴포넌트는 Toolbar 컴포넌트가 onPlayMovie과 onUploadImage로 어떤 작업을 할 지 알 필요가 없습니다. 그 이벤트들은 Toolbar 컴포넌트의 세부 구현입니다.

Toolbar는 버튼에 대해서 onClick 핸들러로 전달하지만 나중에 키보드 단축키 등을 사용해야하는 경우에도 트리거해야하면 동작에 따라서 상호작용의 이름을 정했기 때문에 사용 방식을 유연하게 바꿀 수 있습니다.

### Note

 이벤트 핸들러에 적절한 HTML태그를 사용해야합니다. 예를들어 클릭을 다루기 위해서, div 태그에 onClick 이벤트를 사용하는 것 보다 button 태그의 onClick 이벤트를 사용하는 것입니다. 실제 브라우저의 button 태그를 이용하면 키보드 탐색과 같은 기본 제공 브라우저 동작을 사용할 수 있습니다. 만약 버튼의 기본 브라우저 스타일을 좋아하지 않거나 링크 또는 다른 UI 처럼 보이게 하고싶으면, CSS를 이용하면 됩니다.

## Event propagation

이벤트 핸들러는 컴포넌트 하위의 모든 컴포넌트로 부터 이벤트를 캐치할 수 있습니다. 우리는 이를 이벤트 버블링, 이벤트 전파 라고 부릅니다. 이벤트가 발생하는 위치에서 시작해서 트리를 따라서 올라갑니다.

아래 예제의 div는 두개의 버튼을 가지고 있습니다. div 그리고 각 버튼은 onClick 핸들러를 가지고 있습니다. 버튼을 클릭할 때 어떤 핸들러가 발생할까욥

```jsx
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <button onClick={() => alert('Playing!')}>
        Play Movie
      </button>
      <button onClick={() => alert('Uploading!')}>
        Upload Image
      </button>
    </div>
  );
}
```

버튼을 클릭하면, 버튼의 onClick 이벤트가 첫번째로 발생하고, 이어서 div의 onClick 이벤트가 동작합니다. 두개의 메세지가 나타나게 됩니다. 만약 툴바만 클릭하면 div의 onClick만 동작합니다.

### Pitfall

리액트의 모든 이벤트는 전파됩니다. onScroll은 연결된 JSX 태그에서만 작동합니다. 

→ onScroll 이벤트는 기본적으로 DOM 이벤트 버블링을 지원하지 않습니다. 이벤트가 발생한 요소로부터 전파가 되지 않기때문에 React도 이를 따른다고합니다.

## Stopping propagation

이벤트 핸들러는 이벤트 객체를 유일한 인자로 받습니다. 컨벤션에 따르면 대개 이벤트는 `e` 라고 합니다. 이 객체를 이용해서 이벤트에 대한 정보를 읽을 수 있습니다.

이벤트 객체를 통해서 전파를 멈출수도 있습니다. 이벤트가 상위 컴포넌트에 도달하지 못하도록 하려면, `e.stopPropagation()` 을 호출하면 됩니다.

```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}
```

1. 리액트는 button에 전달된 onClick 이벤트를 호출합니다.
2. 버튼 컴포넌트에 정의된 핸들러는 버블링되는 이벤트를 막기위해 e.stopPropagation() 을 호출하고 Toolbar 컴포넌트로 부터 전달 받은 onClick 함수를 호출합니다.
3. Toolbar 컴포넌트에 정의된 함수에 의해서 버튼의 알럿이 표시됩니다.
4. 전파가 멈추었기 떄문에, 부모 div의 onClick 핸들러는 동작하지 않습니다.

e.stopPropagation()의 결과로 버튼을 클릭하면 두개의 알림이 아닌 하나의 알림만 표시됩니다. 

버튼 클릭하는 것은 주변에 툴바를 클릭하는 것이랑 다르기 때문에 이 UI에서는 전파를 멈추는 것이 좋습니다.

### Deep Dive: **Capture phase events**

흔하지 않은 경우지만 자식 요소의 모든 이벤트를 캐치하고 싶을 수 있습니다. 전파가 중단되더라도요.

예를 들어 데이터 분석을 위해 전파 로직에 관계없이 모든 클릭을 로깅하기를 원할 수 있습니다. 그렇다면 이벤트 이름 뒤에 Capture을 추가하면됩니다.

```jsx
<div onClickCapture={() => { /* this runs first */ }}>
  <button onClick={e => e.stopPropagation()} />
  <button onClick={e => e.stopPropagation()} />
</div>
```

1. 아래로 이동하면서 모든 onClickCapture 핸들러를 호출합니다.
2. 클릭된 요소의 onClick 핸들러를 실행합니다.
3. 위쪽으로 이동하여 모든 onClick 핸들러를 호출합니다.

Capture 이벤트는 라우터 또는 분석과 같은 코드에 유용합니다. 그러나 앱 코드에서는 사용하지 않는게..

## **Passing handlers as alternative to propagation**

아래 코드에서 코드 한 줄을 실행한 다음 부모가 전달한 onClick 프로퍼티를 호출하는 방식을 살펴보세요.

```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      // logic
      onClick();
    }}>
      {children}
    </button>
  );
}
```

부모의 onClick 이벤트 핸들러를 호출하기 전에 코드를 더 작성할 수도 있습니다. 이러한 패턴은 전파에 대한 대안을 제공합니다. 자식 컴포넌트가 이벤트를 처리하는 동시에 부모 컴포넌트가 몇가지 추가적인 동작을 지정할 수 있도록합니다. 전파와 달리 이는 자동으로 되는 것은 아닙니다. 그렇지만 이러한 패턴의 장점은 특정한 이벤트의 결과로 실행되는 전체 코드 체인을 명확하게 추적할 수 있다는 점입니다.

만약 전파에 의존하고 있고 어떤 핸들러가 실행되고 왜 실행되는지 추적하기 어려운 경우 이러한 방법을 대신 사용해보세요.

## **Preventing default behavior**

몇몇 브라우저의 이벤트는 기본 동작과 연관되어 있습니다. 예를 들어 form 내부의 버튼을 클릭했을 때 일어나는 form의 전송 이벤트는 전제 페이지를 다시 로드합니다.

```jsx
export default function Signup() {
  return (
    <form onSubmit={() => alert('Submitting!')}>
      <input />
      <button>Send</button>
    </form>
  );
}
```

이 때 e.preventDefault()를 호출하여 이러한 상황을 멈출 수 있습니다. stopPropagation() 과 혼동하지 마십시오. 둘다 유용하지만 관련은 없습니다.

- e.stopPropagation() 은 위의 태그에 연결된 이벤트 핸들러의 실행을 중지합니다.
- e.preventDefault()는 몇 가지 이벤트에 대해서 기본 브라우저 동작을 방지합니다.

## **Can event handlers have side effects?**

이벤트 핸들러는 부수효과(사이드이펙트)가 가장 많이 발생하는 곳입니다..!

렌더링 함수와는 다르게 이벤트핸들러는 순수할 필요가 없어요, 그래서 어떤걸 변경하기에 최적의 장소입니다. 예를들어 input의 입력 값을 입력에 대한 응답으로 변경하는 경우, 또는 버튼을 누르는 응답으로 리스트를 변경하는 경우. 그러나 일부 정보를 변경하기 위해서 첫번째로 정보를 저장하는 방법이 필요합니다. 리액트에서는 컴포넌트의 메모리인 state(상태)를 이용하여 이 작업을 수행합니다.
