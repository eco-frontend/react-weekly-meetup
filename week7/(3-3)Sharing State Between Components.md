[Sharing State Between Components – React](https://react.dev/learn/sharing-state-between-components)

때때로 두 개의 컴포넌트의 상태를 항상 같이 업데이트하기를 원할 수 있습니다. 이를 수행하기 위해서 각각의 컴포넌트의 상태를 제거하고, 가까운 공통 부모 컴포넌트로 이동하고 props를 통해서 상태를 전달한다. 이것은 상태를 끌어올리기(**lifting state**)라고 하며 리액트 코드를 작성할 때 가장 흔히 하는 일 중 하나입니다.

# **Lifting state up by example**

`Accordion` 컴포넌트는 두 개의 `Panel`을 렌더링 합니다. 각 `Panel` 컴포넌트는 boolean 형태의 isActive state로 컨텐츠의 표시 여부를 판단합니다.

하나의 패널의 버튼을 누르더라도 다른 패널에는 영향을 미치지 않고 독립적입니다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/3e7ceaf4-0cba-4596-ae99-30d87e247697/Untitled.png)

이제 두 개의 패널 중 한번에 하나의 패널만 열리도록 변경하고자 합니다. 두 번째 패널을 열기 위해서 첫 번째 패널을 닫아야합니다. 어떻게 해야할까요?

두 개의 패널을 조정하기 위해서, 세 가지의 스텝으로 “상태 끌어올리기(**lift state up**)”가 필요합니다. 

1. 자식 컴포넌트에서 상태를 제거하세요
2. 공통된 부모에서 데이터를 하드코딩한 값으로 전달하세요.
3. 공통 부모에 state를 추가하고 이벤트 핸들러를 통해서 둘 다 전달하세요.

이 방법으로 `Accordion` 컴포넌트가 두 `Panel`을 조정하고 한 번에 하나만 열리도록 할 수 있습니다.

### Step1: 자식 컴포넌트에서 상태를 제거하세요

`Panel`의 `isActive`의 제어를 부모컴포넌트로 줄 수 있습니다. 이것은 부모 컴포넌트가 `isActive`를 `Panel`에 `props`로서 대신 전달한다는 것을 의미합니다. 아래 코드를 `Panel` 컴포넌트에서 제거하는 것 부터 시작해봅니다.

그리고 isActive를 Panel의 props 목록에 추가합니다.

```jsx
function Panel({ title, children, isActive }) {
  // - const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Show
        </button>
      )}
    </section>
  );
}
```

`Panel`의 부모 컴포넌트는 props의 전달을 통해서 `isActive`를 제어할 수 있습니다. 반대로 `Panel` 컴포넌트는 `isActive`의 값을 더 이상 제어하지 않습니다. 이제 부모 컴포넌트에 달려 있습니다! 

### Step2: 공통된 부모에서 데이터를 하드코딩한 값으로 전달하세요

상태를 끌어올리기 위해서 조정하려는 두 자식 컴포넌트의 가장 가까운 공통 부모 컴포넌트에 두어야합니다.

이 예제에서는 Accordion 컴포넌트입니다. 두개의 패널 그리고 그들의 props를 제어할 수 있기 때문에 현재 활성화된 패널의 “source of truth”가 됩니다. `Accordion` 컴포넌트가 하드코딩된 `isActive`의 값을 두 패널에 전달할 수 있도록 만드세요.(ex. `true`)

```jsx
export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About" **isActive={true}**>
        With a population of about 2 million, Almaty is Kazakhstan's largest city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology" **isActive={true}**>
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples". In fact, the region surrounding Almaty is thought to be the ancestral home of the apple, and the wild <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}
```

### Step3: 공통 부모에 상태를 추가하세요

상태를 끌어올리는 것은 종종 state로 저장하고있는 것의 특성을 바꿉니다.

이 케이스에서는 하나의 패널이 한번에 하나만 활성화되어야 합니다. 이는 `Accordion` 공통 부모 컴포넌트가 패널의 활성화된 것 중 하나를 추적해야한다는 것을 의미합니다. `boolean` 값 대신에 활성화 `Panel`의 인덱스로서 숫자를 이용해보세요.

```jsx
const [activeIndex, setActiveIndex] = useState(0);
```

`activeIndex` 가 `0` 일 때, 첫번째 패널이 활성화되고 `1` 일 때 두번째가 활성화됩니다.

각 `Panel`의 “show” 버튼을 클릭하는 것은 `Accordion`의 활성화 인덱스를 변화하기 위해서 필요합니다.

`Panel`은 `Accordion`의 내부에 정의되어 있기 때문에 `activeIndex` 상태를 직접 설정할 수 없습니다.  `Accordion` 컴포넌트는 `Panel` 컴포넌트가 상태(state)를 변경할 수 있음을 props로서 이벤트 핸들러를 전달하는 방식을 이용하여 명시적으로 허용해야합니다.

```jsx
<>
  <Panel
    isActive={activeIndex === 0}
    onShow={() => setActiveIndex(0)}
  >
    ...
  </Panel>
  <Panel
    isActive={activeIndex === 1}
    onShow={() => setActiveIndex(1)}
  >
    ...
  </Panel>
</>
```

Panel 내부의 버튼은 클릭 이벤트 핸들러로서 onShow props를 이용할 수 있습니다.

이것은 완벽하게 상태가 끌어올려진 것(**lift up state**) 입니다! 공통 부모 컴포넌트안으로 상태를 이동하는 것은 두 패널을 조정할 수 있게합니다. 두개의 “보여진다”는 플래그 대신에 활성화된 인덱스를 사용하는 것은 하나의 패널이 주어진 시점에만 활성화된다는 것을 보장합니다.  그리고 자식으로 전달한 이벤트핸들러는 부모의 상태를 변경하는 것을 허락합니다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c5f6306-1a4b-466e-85d8-5d3d52bbcf19/d9d1ebb0-ff77-414a-b20b-3cb2dcce3b09/Untitled.png)

## Deep Dive: **Controlled and uncontrolled components**

“uncontrolled”된 일부 지역 상태(state)를 갖는 컴포넌트를 호출하는 것은 일반적입니다. 예를들어, 원래의 `Panel` 컴포넌트(isActive를 상태로가지는)는 부모가 패널의 활성 또는 비활성 여부에 영향을 줄 수 없기 때문에 비제어(uncontrolled) 컴포넌트입니다. 

반대로, 컴포넌트의 중요한 정보가 지역 상태(local state)가 아닌 props에 의해서 만들어진다면 “제어된다(controlled)”라고 말할지도 모릅니다. 이를 통해 부모 컴포넌트가 동작을 완전 특정할 수 있습니다. 제일 마지막의 `Panel` 컴포넌트(isActive를 props로 가지는)는 `Accordion` 컴포넌트에 의해서 제어됩니다.  

비제어 컴포넌트(uncontrolled component)는 필수 설정이 적기 때문에 부모 컴포넌트에서 쉽게 사용할 수 있습니다. 그러나 비제어 컴포넌트들은 여러 컴포넌트를 함께 조정해야할 때 유연성이 부족합니다. 제어 컴포넌트는(controlled component) 엄청 유연하지만 항상 부모 컴포넌트에서 props를 통해 전체적으로 설정해주어야합니다.

실제로 “controlled”와 “uncontrolled”는 엄격하게 기술적인 용어가 아닙니다. 각각의 컴포넌트는 대개 지역 상태와 props 둘 다 섞어서 이용하고 있습니다. 그러나 이러한 구분은 컴포넌트의 설계와 제공하는 기능에 관해서 설명하는데 유용한 방법 입니다.

컴포넌트를 작성할 때 어떤 정보들이 props를 통해 controlled 되어야하고, state를 통해 uncontrolled 되어야하는지 고려하세요. 그렇지만 언제든 마음이 바뀔 수 있고 나중에 리팩토링 할 수 있습니다.

```jsx
const Parent = () => {
	const [state, setState] = useState()
	
	return <Child state={state}/>
}

const Child = ({ state }) => {
	
	return <div>
					<ChildChild state={state}/>
					</div>
}

const ChildChild = ({ state }) => {
	return <div></div>
}

///////

const Parent = () => {
	const [state, setState] = useState()
	
	return <Child><ChildChild state={state} /></Child>
}

const Child = ({ children }) => {
	return <div>{children}</div>
}
```

## **A single source of truth for each state(각 상태를 위한 ssot)**

리액트 애플리케이션에서 많은 컴포넌트는 그들의 상태를 가질 수 있습니다. 일부 상태는 입력처럼(트리의 제일 하단의 컴포넌트인) leaf 컴포넌트와 가까운 곳에서 “live(살아있음, 생존)” 합니다. 또 다른 상태는 앱의 상단과 가까운 곳에서 “생존” 합니다. 예를 들어, 클라이언트 사이드의 라우팅 라이브러리도 대개 리액트의 상태로 현재 라우터를 state로 저장하고, props를 통해 전달하도록 구현되어있습니다.

각각 상태의 고유한 state에 대해서 어떤 컴포넌트가 가지고(소유) 있을지 선택할 수 있습니다. 이러한 원칙은 [SSOT](https://en.wikipedia.org/wiki/Single_source_of_truth)를 갖는 것으로 알려져있습니다. 한 장소에서만 모든 상태(state)가 살아있음(존재)을 의미하는것이 아니라 그 정보를 가지고 있는 특정한 컴포넌트가 있다는 것을 의미합니다. 컴포넌트 사이에 공유된 상태(state)의 중복 대신에 공통된 부모에게 끌어올리고(lift it up), 필요로 하는 자식에게 아래로 전달하세요.

당신의 앱은 계속 변할 것 입니다. 각 상태가 어디에 “live”해야 할지 고민하는 동안 상태를 내리거나 되돌리거나 한다는 것은 일반적인 일입니다. 모든 것은 과정입니다.

예제와 같이 더 많은 컴포넌트와 함께 이를 느끼고 보려면 [Thinking in React](https://ko.react.dev/learn/thinking-in-react)를 읽어보세요.

## 요약

- 두 컴포넌트를 조정하고 싶을 때 state를 그들의 공통 부모로 이동합니다
- 그리고 공통 부모로부터 props를 통해 정보를 전달합니다.
- 마지막으로 이벤트 핸들러를 전달해 자식에서 부모의 state를 변경할 수 있도록합니다.
- 컴포넌트를 제어할지 비제어할지 고려하면 유용합니다.
