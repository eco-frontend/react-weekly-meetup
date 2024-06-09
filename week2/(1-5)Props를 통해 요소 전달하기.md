react 컴포넌트는 props를 이용해 서로 통신한다.

모든 부모 컴포넌트는 Props를 넘겨줌으로 몇몇의 정보를 자식 컴포넌트로 전달할 수 있다.

Props는 HTML 속성을 생각하게 할 수 있지만, 객체, 배열, 함수를 포함한 모든 Javascript 값을 전달할 수 있다.

## 학습 내용

1. 컴포넌트에 Props를 전달하는 방법
2. 컴포넌트에서 Props를 읽는(사용하는) 방법
3. Props의 기본값을 지정하는 방법
4. 컴포넌트에 JSX를 전달하는 방법
5. 시간에 따라 Props 가 변하는 방식

## 1. 친밀한 Props

- Props는  JSX tag에 정보를 넘길 수 있다.
    - EX) ClassName, src, alt, width 그리고 height의 img 태그와 같이 넘겨줄 수 있다.
    - img 태그에 전달할 수 있는 Props는 미리 정의되어 있다. (React Dom은 [HTML의 표준](https://html.spec.whatwg.org/multipage/embedded-content.html#the-img-element)을 따른다.)
    - Avatar 컴포넌트에 어떠한 Props든 전달 할 수 있다.

```jsx
function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/1bX5QH6.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}

export default function Profile() {
  return (
    <Avatar />
  );
}

```

## 2. 컴포넌트에 Props 넘겨주기

- 아래 Avatar 컴포넌트는 자식 컴포넌트에 Props를 전달하지 않고 있다.

```jsx
export default function Profile() {
  return (
    <Avatar />
  );
}
```

- 두 과정을 거쳐서 Avatar에 몇가지 Props 줄 수 있다. (아래 설명)
    1. 첫 번째로 Avatar에서 넘겨줄 몇가지 Props를 전달한다. 
    - 예시로 2가지의 Props를 전달합니다. person이라는 Object 타입, size number 타입
    
    ```jsx
    export default function Profile() {
      return (
        <Avatar
          person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
          size={100}
        />
      );
    }
    ```
    
    - 이제 Avatar 컴포넌트 내부에서 Props를 읽을 수 있다.
    
    1. 컴포넌트 내부에서 Props를 읽는(사용하는) 방법
    - 아래와 같이 function Avatar 바로 뒤에 ({ }) 내부에 person, size 쉼표로 구분해 읽을 수 있다.
    - 이렇게 하면 Avatar 코드 내에서 변수를 사용하는 것처럼 사용할 수 있다,
    
    ```jsx
    function Avatar({ person, size }) {
      // person and size are available here
    }
    ```
    
    - Avatar 내부에서 person와 size Props를 통해 랜더링하는 몇가지 로직을 추가하면 됩니다.
    - 이제 Avatar를 Props를 사용해 다양한 방식으로 렌더링 하도록 구성할 수 있다. 값을 조정해보세요.
    
    ```jsx
    import { getImageUrl } from './utils.js';
    
    function Avatar({ person, size }) {
      return (
        <img
          className="avatar"
          src={getImageUrl(person)}
          alt={person.name}
          width={size}
          height={size}
        />
      );
    }
    
    export default function Profile() {
      return (
        <div>
          <Avatar
            size={100}
            person={{ 
              name: 'Katsuko Saruhashi', 
              imageId: 'YfeOqp2'
            }}
          />
          <Avatar
            size={80}
            person={{
              name: 'Aklilu Lemma', 
              imageId: 'OKS67lh'
            }}
          />
          <Avatar
            size={50}
            person={{ 
              name: 'Lin Lanying',
              imageId: '1bX5QH6'
            }}
          />
        </div>
      );
    }
    ```
    
    - Props를 사용하면 부모컴포넌트와 자식 컴포넌트를 독립적으로 생각할 수 있다.
    - EX)  Avatar가 Props를 어떻게 사용하고 있는지 생각할 필요없이, Profile의 Person 과 size Props를 수정할 수 있다.
    - 또한 Profile 컴폰넌트를 보지도 않고 Avatar가 Props를 사용하는 방식을 바꿀 수 있다.
    - Props는 손잡이와 같다고 생각하면 된다. Props는 함수의 인자 값과 동일한 역할을 한다.
    - Props는 컴포넌트에 대한 인자다. React 컴포넌트 함수는 하나의 인자, Props 객체를 받는다.
    
    ```jsx
    function Avatar(props) {
      let person = props.person;
      let size = props.size;
      // ...
    }
    ```
    
    - 위 코드와 같이 전체 Props 자체를 필요로 하지 않기에, 개별 Props로 구조 분해 할당한다.
    
    > 주의!
    > 
    
    ```jsx
    //Props를 선언할 때 () 안에 {} 를 넣거나, 구조분해 할당하는걸 놓치지 마세요!!
    function Avatar({ person, size }) {
      // ...
    }
    
    function Avatar(props) {
      let person = props.person;
      let size = props.size;
      // ...
    }
    ```
    

## 3. Prop의 기본 값 지정하기

- 값이 지정되지 않았을 때, Prop에 기본값을 주길 원한다면, 변수 바로 뒤에 = 과 기본값을 넣어 구조 분해 할당을 할 수 있다.
- 아래 코드와 같이 size Prop이 없을 때 랜더링 된다면, size는 100으로 설정된다.
- 기본 값은 size가 undefined일 때 사용한다. (주의! ⚠️ null과 0으로 전달된다면, 기본값은 사용하지 않음)

```jsx
function Avatar({ person, size = 100 }) {
  // ...
}
```

## 4. JSX spread 문법으로 Props 전달하기

- 때때로 전달되는 Props는 반복적인 경우가 있다.
- 아래 코드를 살펴보자 person, size, isSepia, thickBorder가 그대로 상속되어지고 있다.

```jsx
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

- 반복적인 코드는 가독성을 높일 수 있어서 잘못된 것은 아니다. 하지만, 간결함이 중요할 때가 있다.
- Profile이 Avatar에서 하는 것 처럼 일부 컴포넌트는 그들의 모든 Props를 자식 컴포넌트에 전달한다.
- Props를 직접 사용하지 않기에 간결한 “spread” 문법을 사용하는 것이 합리적일 수 있다.

```jsx
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

- 아래와 같이 코드를 완성하면, Profile의 모든 Props를 각각 보내지 않고, Avatar로 전달한다.
- spread 문법은 다른 모든 컴포넌트에 이 구문을 사용한다면 문제가 있는 것이다.
- 종종 컴포넌트들을 분할하여 자식을 JSX로 전달해야 함을 나타낸다.  (아래에서 계속)

## 5. 자식 JSX에 전달하기

- 아래와 같은 방식으로 컴포넌트를 중첩하고 싶을 때가 있다.

```jsx
<Card>
  <Avatar />
</Card>
```

- JSX 태그 내에 콘텐츠를 중첩하면, 부모 컴포넌트는 해당 콘텐츠를 children 이라는 Prop으로 받을 것이다.
- EX) 아래의 Card 컴포넌트는 <Avatar/>로 설정된 Children Prop을 받아 이를 div에 렌더링한다.

```jsx
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
      <p>111</p>
    </Card>
  );
}

```

- <Card> 내부의 <Avatar>를 텍스트로 바꾸어 <Card> 컴포넌트가 중첩된 콘텐츠를 어떻게 감싸는지 확인해 보자.
- 내부에서 무엇이 렌더링 되는지 “알” 필요는 없다. 이 유연한 패턴은 많은 곳에서 볼 수 있다.
- children prop을 가지고 있는 컴포넌트는 부모컴포넌트가 임의의 JSX로 “채울” 수 있는 “구멍”이 있는 것으로 생각할 수 있다.
- 패널, 그리드 등 시각적 래퍼에 종종 children prop을 사용한다.
    
    !https://ko.react.dev/images/docs/illustrations/i_children-prop.png
    

## 6. 시간에 따라 Props가 변하는 방식

- 아래 Clock 컴포넌트는 부모컴포넌트로 부터 color와 time이라는 두가지 Props를 받는다
- 아래 Select box의 색상을 바꿨을 때 (바로 반응)

```jsx
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```

- 이 예제는 **컴포넌트가 시간에 따라 다른 Props를 받을 수 있음**을 나타낸다.
- Props는 항상 고정되어 있지 않다.
- time prop는 매초 변경되어지고, color prop은 다른 색상을 선택하면 바뀐다.
- 이를 통해 Props는 컴포넌트의 **데이터 처음에만 반영하는 것이 아닌, 모든 시점에 반영**된다.

- 그러나 Props는 컴퓨터 과학에서 “변경할 수 없다”라는 의미의 [불변성](https://en.wikipedia.org/wiki/Immutable_object)을 가지고 있다.
- 컴포넌트가 Props를 변경해야하는 경우 (예: 사용자의 상호작용, 새로운 데이터의 응답), 부모 컴포넌트에 다른 Props, 즉 새로운 객체를 전달하도록 “요청” 해야한다. 그러면, 이전의 Props는 만료되어지고, 자바스크립트 엔진은 기존 Props가 차지했던 메모리를 회수한다.

- “Props 변경”을 시도하지 마라, 선택한 색을 변경하는 등 사용자 입력에 반응해야하는 경우  [State: 컴포넌트의 메모리](https://ko.react.dev/learn/state-a-components-memory)에서 배울 “set state”가 필요하다.

# 요약

- Props를 전달하려면 HTML 속성(어트리뷰트) 사용할 때와 마찬가지로 JSX에 props를 추가한다.
- Props를 읽으려면 `function Avatar({ person, size })` 구조 분해 할당 문법을 사용합니다.
- `size = 100` 과 같은 기본값을 지정할 수 있다, 이는 누락되거나 `undefined` 인 props에 사용된다.
- 모든 props를 `<Avatar {...props} />`로 전달할 수 있다. JSX spread 문법을 사용할 수 있지만 과도하게 사용하지 마라! ⚠️
- `<Card><Avatar /></Card>`와 같이 중첩된 JSX는 `Card`컴포넌트의 자식 컴포넌트로 나타난다.
- Props는 읽기 전용 스냅샷으로, 렌더링 할 때마다 새로운 버전의 props를 받는다.
- Props는 변경할 수 없다. 상호작용이 필요한 경우 state를 설정해야 한다.

//부모 컴포넌트에서 사용하는 setState를 Props를 받는 것은 안티패턴이 될 수 있다.

- 차라리 onchange를 넘기는 것이 좋다.

//children으로 넘기는 순간 하위 컴포넌트는 생각할 필요가 없다.

```jsx
const obj = { a: 'b' }

obj.a = 'c'

props = { a: 'b' }

props.a = 'c' 

const Component = (props) => {
	props.title = "c" 
	
	Object.freeze(props) //?
	
	return <div>{props.title}</div> // ?? "c" "a" 
}

const Parent = () => {
	return <Component title="a" />
}

```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/8752dd40-a132-459f-b4b9-9cf8bd25ebe5/Untitled.png)

```tsx
const AComponent = () => {
	const [state, _] = useState({ abc: '123'});

	// 123;

	return (
		<BComponent state={state} />
	)
}

const BComponent = ({ state }) => {
	state.abc = '456';

	return (
		<div>
			{state.abc} // 456
		</div>
	)
}
```
