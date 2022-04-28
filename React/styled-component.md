# Styled Component
## Styled Component 사용
```javascript
import styled from "styled-components"

const Father = styled.div`
  display: flex;
`;

const Box = styled.div`
  background-color: "teal";
  width: 100px;
  height: 100px;  
`;

function App() {

  return (
    <Father>
      <Box/>
    </Father>
  );
}

export default App;
```
* styled component 내에 css를 정의하여 컴포넌트를 만들어 해당 컴포넌트를 계속해서 재사용할 수 있다.
## props 전달해 동적으로 css 바꾸기
```javascript
import styled from "styled-components"

const Father = styled.div`
  display: flex;
`;

const Box = styled.div`
  background-color: ${(props) => props.bgColor};
  width: 100px;
  height: 100px;  
`;

function App() {

  return (
    <Father>
      <Box bgColor="teal"/>
    </Father>
  );
}

export default App;
```
* props으로 css 인자 값을 전달해서 동적으로 스타일은 변경할 수 있다.
## 타입스크립트에서 styled component에 props 전달
```typescript
import styled from 'styled-components';
import Circle from './Circle';

function App() {
  return (
    <div>
      <Circle bgColor="teal"/>
      <Circle bgColor="tomato"/>
    </div>
  );
}

export default App;
```
```typescript
import styled from "styled-components"

//props의 타입이 ContainerProps
const Container = styled.div<ContainerProps>`
    width: 200px;
    height: 200px;
    background-color: ${props => props.bgColor};
    border-radius: 100px;
`;

interface ContainerProps {
    bgColor: string;
}

interface CircleProps {
    bgColor: string;
    borderColor?: string; // optional props
}

//Circle이 받은 props의 타입은 CircleProps
function Circle({bgColor}:CircleProps) {
    //인터페이스의 속성값이 전달되지 않았을 때(해당 값이 undefined 일때)의 default 값을 지정 가능
    return(
        <Container bgColor={bgColor} borderColor={borderColor ?? bgColor}/>
    );
}

export default Circle;
```
* 타입스크립트의 인터페이스는 다른 언어의 인터페이스와 달리 객체가 가져야할 메서드를 정의하는 것뿐만 아니라 객체가 가져야할 속성 값을 정의할 수 있다.
* 인터페이스의 속성 중에서 필수가 아닌 속성을 정의할 때는 `:` 대신에 `?:`를 사용한다. ex) borderColor?: string
* `??` 를 통해 인터페이스의 속성값이 전달되지 않았을 때(해당 값이 undefined 일때)의 default 값을 지정 가능하다.(그냥 파라미터로 전달 받을 때 default 값을 지정해줘도 된다.) ex) borderColor ?? bgColor
* 타입스크립트에서는 styled component에 props를 전달할 때 제네릭의 형태로 props의 타입을 명시한다.
## 컴포넌트 확장
```javascript
import styled from "styled-components"

const Father = styled.div`
  display: flex;
`;

const Box = styled.div`
  background-color: ${(props) => props.bgColor};
  width: 100px;
  height: 100px;  
`;

const Circle = styled(Box)`
  border-radius: 50px;
`;

function App() {

  return (
    <Father>
      <Box bgColor="teal"/>
      <Circle bgColor="tomato"/>
    </Father>
  );
}

export default App;
```
* 기존의 컴포넌트를 확장하여 새로운 컴포넌트를 생성할 수 있다. 새로운 컴포넌트는 기존의 컴포넌트와 같은 태그를 가진다.
* 새로운 컴포넌트의 css는 기존의 컴포넌트의 css를 상속받고 거기에 더하여 새로운 css를 추가할 수 있다.
## 컴포넌트의 태그 바꾸기
```javascript
import styled from "styled-components"

const Father = styled.div`
  display: flex;
`;


const Btn = styled.button`
  color: white;
  background-color: blue;
  font-size: small;
  height: 20px;
  width: 50px;
  text-align: center;
`;

function App() {

  return (
    <Father>
        <Btn as="a">link</Btn>
    </Father>
  );
}

export default App;
```
* as를 통해 컴포넌트의 태그를 다른 태그로 바꿀 수 있다. 위의 예시는 button 태그를 a 태그로 바꾸었음
## html 속성 정의하기
```javascript
import styled from "styled-components"

const Father = styled.div`
  display: flex;
`;

const Input = styled.input.attrs({placeholder: "hello", required: true})`
  background-color: aquamarine;
  height: 20px;
`;

function App() {

  return (
    <Father>
        <Input/>
    </Father>
  );
}

export default App;
```
* html 속성은 JSX 태그 내에서 정의할 수도 있지만 공통적인 태그라면 같은 속성을 반복적으로 쓰게 된다. 하지만 styled component 내에서 html 속성을 사용하면 이러한 반복적인 작업을 없앨 수 있다.
* attrs() 메서드 내에 오브젝트 형식으로 html 속성을 전달할 수 있다.
## 애니메이션 적용
```javascript
import styled from "styled-components"
import { keyframes } from "styled-components";

const Father = styled.div`
  display: flex;
`;

const rotation = keyframes`
  0% {
    transform: rotate(0deg);
    background-color: brown;
  }
  50% {
    transform: rotate(360deg);
    background-color: aqua;
  }
  100% {
    transform: rotate(0deg);
    background-color: yellow;
  }
`;

const AnimationBox = styled.div`
  background-color: "teal";
  width: 100px;
  height: 100px;
  animation: ${rotation} 5s linear infinite;
`;

function App() {

  return (
    <Father>
        <AnimationBox />
    </Father>
  );
}

export default App;
```
* keyframes를 통해 애니메이션을 정의할 수 있다.
## styled component 내에서 선택자 사용
```javascript
import styled from "styled-components"
import { keyframes } from "styled-components";

const Father = styled.div`
  display: flex;
`;

const squeeze = keyframes`
  from {
    font-size: 50px;
  }
  to {
    font-size: 30px;
  }
`;

const Emoji = styled.span``;

const AnimationBox = styled.div`
  background-color: "teal";
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  span {
    font-size: 50px;
    &:hover {
      animation: ${squeeze} 2s;
      animation-fill-mode: both;
    }
  }
  ${Emoji} {
    font-size: 5px
  }
  .emoji {
    &:active {
      opacity: 0;
    }
  }
`;

function App() {

  return (
    <Father>
        <AnimationBox>
            <div>
                <span className="emoji">😊</span>
                <Emoji>🐱‍🚀</Emoji>
            </div>
      </AnimationBox>
    </Father>
  );
}

export default App;
```
* styled component 내에서도 선택자를 사용하여 해당 컴포넌트 내에있는 컴포넌트를 선택할 수 있다.
* 내가 만든 컴포넌트를 선택할 때는 `${...}`를 통해 선택한다.
* hover나 active 같은 것을 사용할 때 기존에는 `span:hover`처럼 사용했다면 styled component 에서는 선택자가 컴포넌트 안에 있기 때문에 `&:hover`처럼 사용하면 해당 컴포넌트에 대한 hover를 정의할 수 있다.
## 테마
```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { ThemeProvider } from 'styled-components';
import App from './App';

const darkTheme = {
  backgroundColor: "#111"
}

const lightTheme = {
  backgroundColor: "white"
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <ThemeProvider theme={darkTheme}>
      <App />
    </ThemeProvider>
  </React.StrictMode>
);
```
```javascript
import styled from "styled-components"

const Father = styled.div`
  display: flex;
  height: 100vh;
  background-color: ${props => props.theme.backgroundColor};
`;

function App() {

  return (
    <Father/>
  );
}

export default App;
```
* 어떤 컴포넌트의 부모 컴포넌트로 `ThemeProvider`컴포넌트를 두면 해당 컴포넌트와 그 자식 컴포넌트들의 css 속성을 일괄적으로 변경할 수 있다.
* ThemeProvider 컴포넌트의 모든 자식 컴포넌트는 props로 ThemeProvider의 theme을 전달받는다. 그렇기 때문에 ThemeProvider 컴포넌트의 모든 자식 컴포넌트는 ThemeProvider의 theme으로 주어진 object에 접근할 수 있다.