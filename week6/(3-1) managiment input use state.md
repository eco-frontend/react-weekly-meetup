# 3-1. State 관리하기. State를 사용해 Input 다루기

작성자: sikiy
날짜: 2024년 5월 23일

# 대단원: State 관리하기

애플리케이션이 커짐에 따라, state가 어떻게 구성되는지 그리고 데이터가 컴포넌트 간에 어떻게 흐르는지에 대해 의식적으로 파악하면 도움이 된다. 불필요하거나 중복된 state는 버그의 흔한 원인이다. 이 장에서는 state를 잘 구성하는 방법, state 업데이트 로직을 유지 보수 가능하게 관리하는 방법, 그리고 멀리 있는 컴포넌트 간에 state를 공유하는 방법에 대해 알아본다.

## **이 장에서는**

- [UI 변경을 state 변경으로 생각하는 방법](https://ko.react.dev/learn/reacting-to-input-with-state)
- [State를 잘 구조화하는 방법](https://ko.react.dev/learn/choosing-the-state-structure)
- [”State를 끌어올려” 컴포넌트 간에 공유하는 방법](https://ko.react.dev/learn/sharing-state-between-components)
- [State가 보존될지 초기화될지 컨트롤하는 방법](https://ko.react.dev/learn/preserving-and-resetting-state)
- [복잡한 State 로직을 함수로 통합하는 방법](https://ko.react.dev/learn/extracting-state-logic-into-a-reducer)
- [”Prop drilling” 없이 정보를 전달하는 방법](https://ko.react.dev/learn/passing-data-deeply-with-context)
- [앱이 커짐에 따라 state 관리를 확장하는 방법](https://ko.react.dev/learn/scaling-up-with-reducer-and-context)

## State를 사용해 Input 다루기

리액트는 선언적인 방식으로 UI를 조작한다. 개별적인 UI를 직접 조작하는 것 대신에 컴포넌트 내부에 여러 state를 묘사하고 사용자의 입력에 따라 state를 변경한다. 이는 디자이너가 UI를 바라보는 방식과 비슷하다.

### 학습 내용

- 명령형 UI 프로그래밍이 아닌 선언형 UI 프로그래밍을 하는 방법
- 컴포넌트에 들어갈 수 있는 다양한 시각적 state를 열거하는 방법
- 코드에서 다른 시각적 state 간의 변경을 트리거 하는 방법

### 선언형 UI와 명령형 UI 비교

UI 인터랙션을 디자인할 때 유저가 액션을 하면 어떻게 UI를 변경해야 할지 고민해본 적이 있을 것이다. 유저가 폼을 제출한다고 생각해보자.

- 폼에 무언가를 입력하면 “제출” 버튼이 **활성화**
- “제출” 버튼을 누르면 폼과 버튼이 **비활성화** 되고 스피너가 **나타날 것**이다.
- 네트워크 요청이 성공하면 폼은 **숨겨질 것**이고, “감사합니다.” 메세지가 **나타날** 것.
- 네트워크 요청이 실패하면 오류 메시지가 **보일 것**이고, 폼은 다시 **활성화 될 것**

위 내용은 **명령형 프로그래밍**에서 인터랙션을 구현하는 방법. UI를 조작하기 위해서는 발생한 상황에 따라 정확한 지침을 작성해야만 한다. 

![Untitled](3-1%20State%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20State%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%20Input%20%E1%84%83%E1%85%A1%E1%84%85%E1%85%AE%E1%84%80%E1%85%B5%20ca8b880326cb4f0d82c58914779b0b83/Untitled.png)

옆에 누군가를 태우고 차례대로 어디를 가야 할지 말해준다고 상상해보자.

옆에 탄 운전기사는 당신이 어디를 가고싶어 하는지 모른다. 그저 명령에 따를 뿐. (잘못된 지시를 하면 잘못된 곳에 도착하고 말 것.) 우리가 이것을 *명령형*이라 부르는 이유이다. 왜냐하면 컴퓨터에게 스피너에서부터 버튼까지 각각의 요소에 UI를 *어떻게* 업데이트 해야할지 “명령”을 내려야하기 때문..

아래의 명령형 UI 프로그래밍 예제는 리액트 *없이* 만들어진 폼이다. 브라우저에 내장된 [DOM](https://developer.mozilla.org/ko/docs/Web/API/Document_Object_Model)을 사용했다.

- 코드
    
    ```jsx
    async function handleFormSubmit(e) {
      e.preventDefault();
      disable(textarea);
      disable(button);
      show(loadingMessage);
      hide(errorMessage);
      try {
        await submitForm(textarea.value);
        show(successMessage);
        hide(form);
      } catch (err) {
        show(errorMessage);
        errorMessage.textContent = err.message;
      } finally {  
        hide(loadingMessage);
        enable(textarea);
        enable(button);
      }
    }
    
    function handleTextareaChange() {
      if (textarea.value.length === 0) {
        disable(button);
      } else {
        enable(button);
      }
    }
    
    function hide(el) {
      el.style.display = 'none';
    }
    
    function show(el) {
      el.style.display = '';
    }
    
    function enable(el) {
      el.disabled = false;
    }
    
    function disable(el) {
      el.disabled = true;
    }
    
    function submitForm(answer) {
      // 네트워크에 접속한다고 가정해봅시다.
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          if (answer.toLowerCase() === 'istanbul') {
            resolve();
          } else {
            reject(new Error('Good guess but a wrong answer. Try again!'));
          }
        }, 1500);
      });
    }
    
    let form = document.getElementById('form');
    let textarea = document.getElementById('textarea');
    let button = document.getElementById('button');
    let loadingMessage = document.getElementById('loading');
    let errorMessage = document.getElementById('error');
    let successMessage = document.getElementById('success');
    form.onsubmit = handleFormSubmit;
    textarea.oninput = handleTextareaChange;
    
    ```
    

위의 예시가 문제가 없다고 생각할 수 있지만, 위와 같이 UI를 조작하면 더 복잡한 시스템에선 난이도가 기하급수적으로 올라간다. 여러 다른 폼으로 가득 찬 페이지를 업데이트해야 한다고 생각해보라. 새로운 UI 요소나 새로운 인터랙션을 추가하려면 버그의 발생을 막기 위해 기존의 모든 코드를 주의 깊게 살펴봐야만 할 것.. (예를 들면 어떤 것을 보여주거나 숨기거나 하는 것을 잊어버릴지도 모른다.)

**리액트는 이러한 문제점을 해결하기 위해 만들어졌다.**

리액트에선 직접 UI를 조작할 필요가 없다. 컴포넌트를 직접 활성화하거나 비활성화하거나 보여주거나 숨길 필요가 없다. 대신에 **무엇을 보여주고 싶은지 선언**하기만 하면 된다. 그러면 리액트는 어떻게 UI를 업데이트 해야 할지 이해할 것이다.

![Untitled](3-1%20State%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20State%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%20Input%20%E1%84%83%E1%85%A1%E1%84%85%E1%85%AE%E1%84%80%E1%85%B5%20ca8b880326cb4f0d82c58914779b0b83/Untitled%201.png)

택시를 탄다고 생각해보자. 

운전기사에게 어디서 꺾어야 할지 알려주는게 아니라, 가고 싶은 곳을 말한다고 생각해 보라. 당신을 거기까지 데려다주는 것은 운전기사의 일이고 운전기사는 어쩌면 당신이 몰랐던 지름길을 알고 있을지도 모른다.

### UI를 선언적인 방식으로 생각하기

지금까지 폼을 선언적인 방식으로 구현하는 방법을 살펴보았다. 리액트처럼 생각하는 방법을 더 잘 이해하기 위해 UI를 리액트에서 다시 구현하는 과정을 아래에서 살펴본다.

1. 컴포넌트의 다양한 시각적 state를 **확인하기.**
2. 무엇이 state 변화를 트리거하는지 **알아내기.**
3. `useState`를 사용해서 메모리의 state를 **표현하기.**
4. 불필요한 state 변수를 **제거하기.**
5. state 설정을 위해 이벤트 핸들러를 **연결하기.**

### 첫 번째: 컴포넌트의 다양한 시각적 state 확인하기

여러가지 “state”를 가지고 있는 “상태 기계”라는 것을 컴퓨터 과학에서 들어본 적이 있을 것이다. 그리고 디자이너와 일한다면 다양한 “시각적 state”에 관한 모형을 본 적이 있을 것이다. 리액트는 디자인과 컴퓨터 과학의 사이에 있기 때문에 두 아이디어 모두에게서 영감을 받았다.

먼저 사용자가 볼 수 있는 UI의 모든 “state”를 시각화 해야 한다.

- **Empty**: 폼은 비활성화된 “제출” 버튼을 가지고 있다.
- **Typing**: 폼은 활성화된 “제출” 버튼을 가지고 있다.
- **Submitting**: 폼은 완전히 비활성화되고 스피너가 보인다.
- **Success**: 폼 대신에 “감사합니다” 메시지가 보인다.
- **Error**: “Typing” state와 동일하지만 오류 메시지가 보인다.

디자이너처럼 로직을 추가하기 전에 여러 state를 “목업” 하거나 “모형”을 만들고 싶을 것이다. 여기에 폼의 생김새와 관련된 모형이 있다고 생각해보자. 이 모형은 기본값이 `'empty'`인 `status` prop에 의해 컨트롤된다.

- 코드
    
    ```jsx
    export default function Form({
      status = 'empty'
    }) {
      if (status === 'success') {
        return <h1>That's right!</h1>
      }
      return (
        <>
          <h2>City quiz</h2>
          <p>
            In which city is there a billboard that turns air into drinkable water?
          </p>
          <form>
            <textarea />
            <br />
            <button>
              Submit
            </button>
          </form>
        </>
      )
    }
    ```
    

여기서 prop을 네이밍하는 것은 중요하지 않는다. 원하는 이름을 붙이면 된다. `status = 'empty'`를 `status = 'success'`로 변경하고 성공 메시지가 나타나는 것을 보라. 모형을 만듦으로써 로직을 연결하기 전에 UI에서 빠르게 테스트해볼 수 있다. 다음은 `status` prop에 의해 “컨트롤”되는 조금 더 구체적인 프로토타입이다.

- 코드
    
    ```jsx
    export default function Form({
      // 'submitting', 'error', 'success'로 한 번 변경해보세요:
      status = 'empty'
    }) {
      if (status === 'success') {
        return <h1>That's right!</h1>
      }
      return (
        <>
          <h2>City quiz</h2>
          <p>
            In which city is there a billboard that turns air into drinkable water?
          </p>
          <form>
            <textarea disabled={
              status === 'submitting'
            } />
            <br />
            <button disabled={
              status === 'empty' ||
              status === 'submitting'
            }>
              Submit
            </button>
            {status === 'error' &&
              <p className="Error">
                Good guess but a wrong answer. Try again!
              </p>
            }
          </form>
          </>
      );
    }
    
    ```
    

### DEEP DIVE | 많은 시각적 state를 한 번에 보여주기

[https://ko.react.dev/learn/reacting-to-input-with-state#:~:text=많은 시각적 state를 한 번에 보여주기](https://ko.react.dev/learn/reacting-to-input-with-state#:~:text=%EB%A7%8E%EC%9D%80%20%EC%8B%9C%EA%B0%81%EC%A0%81%20state%EB%A5%BC%20%ED%95%9C%20%EB%B2%88%EC%97%90%20%EB%B3%B4%EC%97%AC%EC%A3%BC%EA%B8%B0)

### 두 번째: 무엇이 state 변화를 트리거하는지 알아내기

![Untitled](3-1%20State%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20State%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%20Input%20%E1%84%83%E1%85%A1%E1%84%85%E1%85%AE%E1%84%80%E1%85%B5%20ca8b880326cb4f0d82c58914779b0b83/Untitled%202.png)

두 종류의 인풋 유형으로 state 변경을 트리거할 수 있다.

- 버튼을 누르거나, 필드를 입력하거나, 링크를 이동하는 것 등의 **휴먼 인풋**
- 네트워크 응답이 오거나, 타임아웃이 되거나, 이미지를 로딩하거나 하는 등의 **컴퓨터 인풋**

두 가지 경우 모두 **UI를 업데이트 하기 위해서는 state 변수를 설정해야 한다.** 

지금 만들고 있는 폼의 경우,

- **텍스트 인풋을 변경하면** (휴먼) 텍스트 상자가 비어있는지 여부에 따라 state를 *Empty*에서 *Typing* 으로 또는 그 반대로 변경
- **제출 버튼을 클릭하면** (휴먼) *Submitting* state를 변경
- **네트워크 응답이 성공적으로 오면** (컴퓨터) *Success* state를 변경
- **네트워크 요청이 실패하면** (컴퓨터) 해당하는 오류 메시지와 함께 *Error* state를 변경

<aside>
💡 휴먼 인풋은 종종 [이벤트 핸들러](https://ko.react.dev/learn/responding-to-events)가 필요할 수도 있다는 것도 기억할 것!

</aside>

이와 같은 흐름은 종이에 그려 시각화할 수 있다. 종이에 각각의 state를 라벨링 된 원으로 그리고 각각의 state 변화를 화살표로 이어보자. 이러한 과정을 통해 state 변화의 흐름을 파악할 수 있을 뿐 아니라 구현 전에 버그를 찾을 수도 있다.

![Untitled](3-1%20State%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20State%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%20Input%20%E1%84%83%E1%85%A1%E1%84%85%E1%85%AE%E1%84%80%E1%85%B5%20ca8b880326cb4f0d82c58914779b0b83/Untitled%203.png)

### 세 번째: 메모리의 state를 `useState` 로 표현하기

`useState`를 사용하여 컴포넌트의 시각적 state를 표현해야 한다. 이 과정은 단순함이 핵심이다. 각각의 state는 “움직이는 조각”이다. 그리고 **“움직이는 조각”은 적을수록 좋다.** 복잡한 것은 버그를 일으키기 마련이기 때문..ㅠㅠ

먼저, 반드시 필요한 state를 가지고 시작해보자.

```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
```

그리고 나서 앞서 필요하다고 나열했던 나머지 시각적 state도 살펴보자.

좋은 방법이 곧바로 떠오르지 않는다면 가능한 모든 시각적 state를 커버할 수 있는 *확실한* 것을 먼저 추가하는 방식으로 시작:

```jsx
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

처음으로 떠올린 생각이 최고의 방법은 아닐 수도 있지만 괜찮다. state를 리팩토링 하는 것도 과정 중 하나이다..!

### 네 번째: 불필요한 state 변수를 제거하기

state의 중복은 피하고 필수적인 state만 남겨두고 싶을 것이다. state 구조를 리팩토링하는 데 시간을 조금만 투자하면 컴포넌트는 더 이해하기 쉬워질 것이고 불필요한 중복은 줄어들 것이며 의도하지 않은 의미를 피할 수 있을 것이다. 리팩토링의 목표는 **state가 사용자에게 유효한 UI를 보여주지 않는 경우를 방지하는 것.**

여기에 state 변수에 관한 몇 가지 질문이 있다:

- **state가 역설을 일으키지는 않나요?** 예를 들면 `isTyping`과 `isSubmitting`이 동시에 `true`일 수는 없습니다. 이러한 역설은 보통 state가 충분히 제한되지 않았음을 의미합니다. 여기에는 두 boolean에 대한 네 가지 조합이 있지만 오직 유효한 state는 세 개뿐입니다. 이러한 “불가능한” state를 제거하기 위해 세 가지 값 `'typing'`, `'submitting'`, `'success'`을 하나의 `status`로 합칠 수 있습니다.
- **다른 state 변수에 이미 같은 정보가 담겨있진 않나요?** `isEmpty`와 `isTyping`은 동시에 `true`가 될 수 없습니다. 이를 각각의 state 변수로 분리하면 싱크가 맞지 않거나 버그가 발생할 위험이 있습니다. 이 경우에는 운이 좋게도 `isEmpty`를 지우고 `answer.length === 0`으로 체크할 수 있습니다.
- **다른 변수를 뒤집었을 때 같은 정보를 얻을 수 있진 않나요?** `isError`는 `error !== null`로도 대신 확인할 수 있기 때문에 필요하지 않습니다.

이러한 정리 과정을 거친 후에는 세 가지 (일곱 개에서 줄어든!) *필수* 변수만 남게 된다.

```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', or 'success'
```

어느 하나를 지웠을 때 정상적으로 작동하지 않는다는 점에서 이것들이 모두 필수라는 것을 알 수 있다.

### DEEP DIVE | Reducer를 사용하여 “불가능한” state 제거

[https://ko.react.dev/learn/reacting-to-input-with-state#:~:text=를 사용하여 “불가능한” state 제거](https://ko.react.dev/learn/reacting-to-input-with-state#:~:text=%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC%20%E2%80%9C%EB%B6%88%EA%B0%80%EB%8A%A5%ED%95%9C%E2%80%9D%20state%20%EC%A0%9C%EA%B1%B0)

<aside>
💡 여기 폼의 state를 나타내는데 충분한 세 가지 변수가 있습니다. 하지만 세 변수는 여전히 말이 안 되는 일부 중간 state를 가지고 있습니다. 예를 들면 `error`가 null이 아닌데 `status`가 `success`인 것은 말이 되지 않습니다. state를 조금 더 정확하게 모델링하기 위해서는 [리듀서로 분리](https://ko.react.dev/learn/extracting-state-logic-into-a-reducer)하는 방법도 있습니다. 리듀서를 사용하면 여러 state 변수를 하나의 객체로 통합하고 관련된 모든 로직도 합칠 수 있습니다!

</aside>

### 다섯 번째: state설정을 위해 이벤트 핸들러 연결하기

마지막으로 state 변수를 설정하기 위해 이벤트 핸들러를 연결하면 끝이다.

[[코드]](https://ko.react.dev/learn/reacting-to-input-with-state#:~:text=%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC%20%E2%80%9C%EB%B6%88%EA%B0%80%EB%8A%A5%ED%95%9C%E2%80%9D%20state%20%EC%A0%9C%EA%B1%B0)

이러한 코드가 기존의 명령형 프로그래밍 예제보다는 길지만 그래도 조금 더 견고하다. 모든 인터랙션을 state로 표현하게 되면 이후에 새로운 시각적 state가 추가되더라도 기존의 로직이 손상되는 것을 막을 수 있다. 또한 인터랙션 자체의 로직을 변경하지 않고도 각각의 state에 표시되는 항목을 변경할 수 있다.

### 요약

- 선언형 프로그래밍은 UI를 세밀하게 직접 조작하는 것(명령형)이 아니라 각각의 시각적 state로 UI를 묘사하는 것을 의미함.
- 컴포넌트를 개발할 때
    1. 모든 시각적 state를 확인하기
    2. 휴먼이나 컴퓨터가 state 변화를 어떻게 트리거 하는지 알아내기
    3. `useState`로 state를 모델링하기
    4. 버그와 모순을 피하려면 불필요한 state를 제거하기
    5. state 설정을 위해 이벤트 핸들러를 연결하기

[[챌린지]](https://ko.react.dev/learn/reacting-to-input-with-state#recap)