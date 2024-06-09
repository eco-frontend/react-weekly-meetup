# 첫 번째 컴포넌트 작성 방법

- 컴포넌트는 React의 핵심 개념 중 하나
- 컴포넌트는 사용자 인터페이스를 구축하는 기반

### 학습 내용

- 컴포넌트는 무엇인가?
- React 애플리케이션에서 컴포넌트의 역할
- 첫 번째 React 컴포넌트를 작성하는 방법

---

## 컴포넌트 UI 구성 요소

- 웹에서 HTML을 통해 <h1>, <li> 같은 태그를 사용하여 풍부한 구조의 문서를 만들 수 있다.

```jsx
<article>
  <h1>My First Component</h1>
  <ol>
    <li>Components: UI Building Blocks</li>
    <li>Defining a Component</li>
    <li>Using a Component</li>
  </ol>
</article>
```

- 프로젝트가 성장함에 따라 이미 작성한 컴포넌트를 재사용하여 많은 디자인을 구성할 수 있고, 개발 속도가 빨라진다.
- UI library와 같은 React 오픈 소스 커뮤니티에서 수천 개의 컴포넌트로 프로젝트를 빠르게 시작할 수 있다.
    - [Chakra UI](https://chakra-ui.com/), [Material UI,](https://mui.com/) [ANTD UI](https://ant.design/), [Next UI](https://nextui.org/)

## 컴포넌트 정의

- React 컴포넌트는 마크업으로 뿌릴 수 있는 JavaScript 함수다.

```jsx
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

## 컴포넌트 빌드

1. 컴포넌트 내보내기
    - export default 접두사는 표준 Javascript 구문이다.
        - 해당 부분을 사용하면 나중에 다른 파일에서 가져올 수 있도록 한다.

1. 함수 정의
    - function 함수명() {}을 사용하면 함수명 이라는 이름의 javascript 함수를 정의할 수 있다.
        - function Test() {} 의 경우 Test 라는 컴포넌트 (javascript 함수)
        - hook을 사용하면 const Test = () ⇒ {} 으로 변환이 가능하다.
        
        ```jsx
        const **P**rofile = () => {
          return (
            <img
              src="https://i.imgur.com/MK3eW3Am.jpg"
              alt="Katherine Johnson"
            />
          )
        }
        ```
        
    - React 컴포넌트는 일반 JavaScript 함수 이지만, 이름은 대문자로 시작해야 한다.
        - 대문자가 아니면 작동하지 않는다.

1. 마크업 추가하기
    - 컴포넌트 내에 HTML 태그들을 추가한다.

- 괄호가 없으면 return 뒷 라인에 있는 모든 코드가 무시된다.

## 컴포넌트 사용

- 컴포넌트를 정의하고 다른 컴포넌트 내에 중첩할 수 있다.

```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

- 브라우저에 표시 되는 내용으로 **대소문자의 차이에 주목**
    - 소문자 태그들은 HTML 태그를 의미
    - 대문자로 시작하는 태크는 컴포넌트를 사용하고자 한다는 의미

```jsx
<section>
  <h1>Amazing scientists</h1>
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
</section>
```

## 컴포넌트 중첩 및 구성

- 컴포넌트는 일반 JavaScript 함수로 같은 파일에 여러 컴포넌트를 포함할 수 있다.
- 컴포넌트는 작거나, 서로 밀접하게 관련되어 있을 때 편리하다.
- 파일이 너무 커지게 된다면, 다음 파트에서 배울 import와 export 기능을 이용
- **컴포넌트 내에서 중첩해서 사용한다면, 사용은 가능하지만 매우 느리고 버그를 발생시킨다.**
    
    ```jsx
    //as-is
    export default function Gallery() {
      // 🔴 절대 컴포넌트 안에 다른 컴포넌트를 정의하면 안 됩니다!
      function Profile() {
        // ...
      }
      // ...
    }
    
    //to-bo
    export default function Gallery() {
      // ...
    }
    
    // ✅ 최상위 레벨에서 컴포넌트를 선언합니다
    function Profile() {
      // ...
    }
    ```
    

# 요약

- React를 사용하면 앱의 재사용한 가능한 UI 요소인 컴포넌트를 만들 수 있다.
- React 앱에서 모든 UI는 컴포넌트다.
- React 컴포넌트는 다음 몇 가지를 제외하고 일반적인 Javascript 함수다.
    - 컴포넌트의 이름은 항상 대문자로 시작한다.
    - JSX 마크업을 반환한다.
    

[문제1](https://codesandbox.io/p/sandbox/react-dev-3kp7gh?file=%2Fsrc%2FApp.js%3A1%2C20&utm_medium=sandpack)

[문제2](https://codesandbox.io/s/5kvyr2?file=%2Fsrc%2FApp.js&utm_medium=sandpack)

[문제3](https://ko.react.dev/learn/your-first-component)
