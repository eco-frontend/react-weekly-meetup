일부 자바스크립트 함수는 순수하고 순수한 함수는 하나의 계산만 수행하고 그 이상을 수행하지 않음.

컴포넌트를 순수 함수로만 작성하면 코드 베이스가 커짐에 따라 당황스러운 버그와 예측할 수 없는 동작을 피할 수 있음.

이러한 이점을 얻기 위해서 몇가지 룰을 따라야한다.

- 순수성 그리고 버그를 피하는 방법
- 렌더링 단계에서 변경 사항을 제외하여 컴포넌트를 순수하게 유지하는 방법
- Strict Mode를 사용하여 컴포넌트의 실수를 찾는 방법

## 순수성?

컴퓨터 과학에서 (특히 함수형 프로그래밍의에서), 순수함수의 정의는 두가지의 특성을 따른다.

- 자신의 일에만 신경을 씁니다. 함수가 호출되기 전에 존재했던 어떤 객체 또는 변수도 변경하지 않음.
- 같은 입력에 같은 출력. 같은 입력이 주어지면 순수함수는 항상 같은 결과를 리턴함

수학에서의 순수 함수의 예제

$y = 2x$ 라는 함수가 존재할 때, $x = 2$ 라면 항상 $y = 4$ 이고  $x = 3$ 라면 항상 $y = 6$ 입니다.

여기서 $x = 3$ 인데 상황에 따라서 y가 다른 값을 가지지는 않음. 항상 6이라는 값을 가짐

```jsx
function double(number) {
	return 2 * number;
}
```

자바스크립트 함수로 표현한다면 위와 같은 함수가 됨. double 은 순수함수이다. 만약에 3을 인자로 넘긴다면 항상 6을 리턴함.

**리액트는 이러한 개념을 중심으로 설계되었습니다. 리액트는 작성한 모든 컴포넌트는 순수함수라고 가정합니다.**

이것은 리액트 컴포넌트가 같은 입력에 항상 같은 JSX를 리턴한다는 말이랑 같은 의미임.

```jsx
function Recipe({ drinkers }) {
	return (
    <ol>    
      <li>Boil {drinkers} cups of water.</li>
      <li>Add {drinkers} spoons of tea and {0.5 * drinkers} spoons of spice.</li>
      <li>Add {0.5 * drinkers} cups of milk to boil and sugar to taste.</li>
    </ol>
	)
}

export default function App() {
  return (
    <section>
      <h1>Spiced Chai Recipe</h1>
      <h2>For two</h2>
      <Recipe drinkers={2} />
      <h2>For a gathering</h2>
      <Recipe drinkers={4} />
    </section>
  );
}

Result
For two
Boil 2 cups of water.
Add 2 spoons of tea and 1 spoons of spice.
Add 1 cups of milk to boil and sugar to taste.

For a gathering
Boil 4 cups of water.
Add 4 spoons of tea and 2 spoons of spice.
Add 2 cups of milk to boil and sugar to taste.
```

`drinkers`에 `2`라는 값을 `Recipe` 컴포넌트로 넘겨주면, 컴포넌트는 `2 cups of water` 를 항상 포함하는 JSX를 리턴함.

만약에 `drinkers`에 `4`라는 값을 넘겨주면 `4 cups of water`를 포함하는 JSX를 항상 반환함

수학 공식과 다를 바 없음.

Recipe 컴포넌트로 생각해본다면, 요리과정에 새로운 재료를 넣지 않는다면, 같은 요리를 매번 얻을 수 있음.

여기서 요리라고 하는것은 JSX이고, JSX는 리액트가 렌더링하는데 사용된다.

## 부수효과: (의도하지 않은) 결과

리액트의 렌더링 과정은 항상 틀림없이 순수하다. 컴포넌트는 JSX만 반환해야하고 렌더링 이전에 존재하는 어떤 객체나 변수도 변경되서는 안됨. 왜냐면 순수하지 못하니까..!

컴포넌트의 룰이 파괴된 케이스

```jsx
let guest = 0;

function Cup() {
	guest = guest + 1;
	return <h2>Tea cup for guest #{guest}</h2>;
}

function TeaSet() {
	return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}

Result

Tea cup for guest #2
Tea cup for guest #4
Tea cup for guest #6
```

Cup 컴포넌트는 바깥에서 선언된 guest 변수를 읽고 쓰고있음. 즉, 이 컴포넌트를 여러번 호출하면 다른 JSX가 매번 생성된다. 그리고 다른 컴포넌트가 guest 변수를 읽고 있다면 렌더링된 시점에 따라 또 다른 JSX를 생성할 것임. 이것은 예측이 불가능하게 함!

$y=2x$ 라는 공식으로 다시 돌아가서, $x = 2$ 일때 우리가 $y=4$ 라는 결과를 신뢰할 수 없다고 가정하자. 우리 테스트들은 실패할지도 모르고, 사용자들은 당황할 수 있으며, 비행기가 하늘에서 떨어지기까지..? ㅋㅋ 얼마나 혼란스러운 버그가 발생할지 짐작도 안감.

위 예제에서 guest를 props로 변경해서 바꿔보자

```jsx
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}

Result
Tea cup for guest #1
Tea cup for guest #2
Tea cup for guest #3
```

컴포넌트는 순수해졌고, JSX는 오로지 guest props에만 의존하게 되었음.

일반적으로 컴포넌트가 특정한 순서로 렌더링 되는지 기대할 수는 없음. 그치만 딱히 문제가 되지 않는 이유는 $y=2x$ 를 이전에 호출하거나 $y=5x$를 이후에 호출하거나 두 수식은 각각 독립적으로 해결됨.

같은 이유로 각각의 컴포넌트는 본인만 생각하고 렌더링을 수행하는 동안 다른 컴포넌트에 의존하거나 조율하지 않음. 렌더링은 학교 시험이랑 비슷함 ㅋㅋ 각각의 컴포넌트는 자기 자신의 JSX만 계산하니까..!

## Deep Dive # Strict Mode로 순수하지 않은 계산 감지하기

리액트에서는 렌더링 동안 읽을 수 있는 세가지의 입력이 있음. (props, state, context) 이 입력은 항상 읽기 전용으로 취급해야함.

사용자 입력에 따라서 어떤 응답을 변경해야할 때 변수를 직접 쓰는거 대신에 상태를 설정해야함. 컴포넌트가 렌더링되는 동안에는 기존 변수나 객체를 변경해서는 절대 안됨.

리액트에서는 각각의 컴포넌트의 함수가 개발 모드에서는 두번 호출되도록하는 Strict Mode 라는것을 제공한다.

컴포넌트 함수를 두번씩 호출하면서 Strict Mode는 규칙이 깨졌는지 찾을 수 있도록 도와줌.

원래 예제에서 1,2,3 대신 Guest #2, Guest #4, Guest #6으로 보여지는것을 알 수 있음. 원래 함수가 순수하지 못했고, 그래서 2번씩 호출되면서 깨진거임. 그러나 순수함수로 변경한 버전은 두 번씩 호출해도 동작하고 있음.

순수함수는 계산만 수행하기 때문에 두번씩 호출해도 어떤 것도 변하지 않음. 단지 double(2) 라고 두번씩 호출해도 결과가 바뀌지 않는 것과 비슷함. 그리고 $y=2x$를 두 번 풀어도 y는 변하지 않음. 항상 같은 입력에 같은 출력을 가지기 때문에.

Strict Mode는 프로덕션 환경에서는 적용되지 않아서 사용자의 앱 속도가 느려지지 않음. Strice Mode를 사용하려면 root 컴포넌트를 `<React.StrictMode>` 로 감싸면 되고, 일부 프레임워크는 기본값으로 제공함.   

## 로컬 변경: 컴포넌트의 작은 비밀

앞선 예제에서 문제는 컴포넌트가 렌더링되는 동안 이전에 존재했던 변수를 변경하고 있던 것. 이런 것은 무섭게 들리게 하려고 mutation 이라고 부르기도 한다. 순수 함수는 함수 스코프 또는 객체의 바깥의 변수(함수가 호출되기 이전에 만들어졌던)를 변경하지 않는다. 순수하지 못하니가..!

그러나 렌더링 하는 동안 생성한 변수나 객체를 변경하는 것은 전혀 문제가 되지 않음! 예를 들면 `cups` 변수를 할당하기 위해 빈 배열을 생성하고 `push` 를 한다고 가정해보자.

```jsx
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaGathering() {
  let cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

`cups` 변수 또는 빈 배열이 `TeaGathering` 함수 바깥에서 만들어졌다면 이것은 큰 문제를 일으킬 수 있음. 배열에 값을 추가하면 기존 객체를 변경하게 되니까.

그러나, 같은 렌더링 동안에만 TeaGathering 안에서 변수들을 생성하고 있기 때문에 괜찮음. TeaGathering 바깥에 어떤 코드도 이런 일이 발생했다는 사실을 알 수 없음. 이를 local mutation 이라고 부른다. 

## 부수 효과를 일으키는 경우

함수형 프로그래밍은 순수성에 크게 의존하지만, 어떤 순간이나 어딘가에나 무언가 바뀌어야 한다면? 이것이 프로그래밍의 핵심!

화면을 업데이트 하거나, 애니메이션을 시작하거나, 데이터를 변경하는 등의 행위를 **부수효과** 라고 부른다. 이것들은 렌더링 중이 아니라 “**부수적으로**” 일어나는 일

**리액트에서 부수효과는 일반적으로 이벤트 핸들러 내부에 속한다**. 이벤트 핸들러는 어떤 동작을 수행할 때 리액트가 실행하는 함수이다. 예를 들면 버튼을 클릭할 때 등. 이벤트 핸들러는 컴포넌트 내부에 정의되어 있지만 렌더링 동안에 실행되지는 않음. 그래서 이벤트 핸들러는 순수할 필요가 없다

모든 옵션을 다 찾아보아도 부수효과에 적합한 이벤트 핸들러를 찾을 수 없는 경우에는 컴포넌트 내부에서 `useEffect` 를 호출해서 JSX 리턴에 첨부할 수 있음.

이것은 나중에 렌더링 후 부수효과가 허용될 때 리액트가 실행하도록 지시할 수 있다. 그러나 이러한 접근은 최후의 수단으로 사용해야함.

가능하다면 렌더링만으로 로직을 표현하고 이 방법을 통해서 얼마나 많은 것을 얻을 수 있는지 놀랄지도 모른다.

## Deep Dive # 리액트가 순수성을 중요하게 생각하는 이유

순수 함수를 작성하기 위해서는 약간의 습관과 훈련이 필요하다. 그러나 다음과 같은 기회도 얻을 수 있음

- 컴포넌트가 다른 환경에서 동작할 수도 있음. 예를들면 서버에서..! 같은 입력에 같은 결과를 반환하기 때문에 하나의 컴포넌트는가 많은 사용자 요청을 처리할 수 있다.
- 입력이 변경되지 않은 컴포넌트의 렌더링을 건너뛰면(memo) 성능을 향상시킬 수 있다. 순수 함수는 항상 같은 결과를 리턴하기 때문에 캐시해도 안전하다.
- 어떤 데이터가 깊은 컴포넌트 트리의 중간에서 렌더링이 되는 경우 리액트는 오래된 렌더링을 완료하기 위해 시간을 소모하지 않고 다시 렌더링을 시작할 수 있음. 순수성은 언제든지 계산을 중단해도 안전하다.

리액트의 모든 새로운 기능들은 순수성을 활용해서 만들어졌다. 데이터 페칭 부터 애니메이션, 성능에 이르기까지 컴포넌트를 순수하게 유지하면 리액트 패러다임의 강력한 기능을 활용할 수 있다.

## 정리

- 컴포넌트는 순수해야한다(무조건)
    - 렌더링 전에 있던 객체나 변수를 변경해서는 안되고 자체적으로 처리한다
    - 같은 입력에 같은 결과를 가진 JSX를 항상 리턴해야한다.
- 어떤 시점에 렌더링이 일어나든지 컴포넌트는 서로의 렌더링 순서에 의존해서는 안된다.
- 컴포넌트가 렌더링에 사용되는 입력을 변경해서는 안됨. props, state, context 가 포함되어있음. 화면을 업데이트하기 위해서 기존 객체를 변경하는 대신에 setState를 이용하세요
- 반환하는 JSX에서 컴포넌트의 로직을 표현하기 위해서 노력해야한다. “무언가를 변경” 해야할 때는 일반적으로 이벤트 핸들러에서 이를 수행하고 최후의 수단으로만 useEffect를 사용하는게 좋음
- 순수함수를 작성하는 것은 연습이 필요하긴하지만 리액트 패러다임의 힘을 발휘한다.

```jsx
const Componenot = () => {
  const [state, setState] = useState(false)

	useEffect(() => {
		api.fetch()
		false  true
	}, [state])

	return <button onClick={() => {setState()}}/>
}

const Component = () => {
	const [state, setState] = useState(false)
	
	const handleClick = () => {
		const restul = api.fetch();
		setState(result)
	}

	return <button className={state ? 'good' : 'bad'} onClick={handleClick}/>
}
```

[next.config.js Options: reactStrictMode](https://nextjs.org/docs/app/api-reference/next-config-js/reactStrictMode)

```jsx
const Map = () => {
	
}

const Point = ({ let, lng }) => {
  const [data, setData] = useState()
	
	useEffect(() => {
		const response = fetchAPI(let, lng)
		setDate(response)
	}, [let, lng])
	
	return ///
}
```

```jsx
const Component = () => {
	const router = useRouter()
	const query = router.query.asdf
	
	return (
		...
		
	)
}
```
