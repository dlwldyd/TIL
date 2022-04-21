# Effect
## useEffect
```javascript
function App() {
  const [counter, setCounter] = useState(0);
  const [keyword, setKeyword] = useState("");
  const onClick = () => setCounter(counter => counter + 1);
  const onChange = (event) => setKeyword(keyword => event.target.value);
  console.log("랜더링 될 때마다 실행")
  useEffect(() => console.log("처음 랜더링 시에만 실행"), []);
  useEffect(() => console.log("랜더링 시에 keyword 값이 바뀌었으면 실행", keyword), [keyword]);
  useEffect(() => console.log("랜더링 시에 counter 값이 바뀌었으면 실행"), [counter]);
  return (
    <div>
      <input type="text" onChange={onChange} value={keyword}></input>
      <h1>{counter}</h1>
      <button onClick={onClick}>click me</button>
    </div>
  );
}
```
* 컴포넌트가 이벤트로 인해 랜더링 될 때 Effect를 사용하지 않으면 컴포넌트의 모든 함수가 실행된다. 따라서 만약 API를 호출한다던가 하는 로직이 컴포넌트 내에 있으면 컴포넌트가 랜더링이 될 때마다 API를 호출하게 될 것이다. 이러한 일을 방지하기 위해 사용하는 것이 Effect이다.
* useEffect()의 첫 번째 파라미터는 실행하길 원하는 함수이다. 두 번째 파라미터는 어떤 값이 바뀌었을 때 해당 함수가 실행될지가 배열의 형태로 온다. 만약 두 번째 파라미터가 비어있으면 해당 컴포넌트가 생성될 때만 실행한다.
## Cleanup
```javascript
function Hello() {
  useEffect(() => {
    console.log("created");
    return () => console.log("destroyed");
  })
  return <h1>Hello</h1>;
}

function App() {
  const [showing, setShowing] = useState(false);
  const onClick = () => setShowing((prev) => !prev);
  return (
    <div>
      <button onClick={onClick}>{showing ? "Hide" : "Show"}</button>
      {showing ? <Hello/> : null}
    </div>
  )
}
```
* Cleanup 함수는 컴포넌트가 삭제될 때 실행된다.
* useEffect()의 반환 값을 Cleanup 함수로 함으로써 해당 컴포넌트가 삭제될 때 Cleanup 함수의 로직을 실행할 수 있다.