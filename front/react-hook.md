# React Hook
## useEffect
### useEffect란
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
### Cleanup
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
## useState
### State를 사용하기 전
```javascript
const root = document.getElementById("root");
let cnt = 0;

function cntUp() {
    cnt += 1;
    ReactDOM.render(<Container/>, root);
}
const Container = () => (
    <div>
        <h3>Total Clicks : {cnt}</h3>
        <button onClick={cntUp}>Click me</button>
    </div>
);
ReactDOM.render(<Container/>, root);
```
* State를 사용하지 않으면 이벤트로 인해 업데이트된 데이터를 화면에 보여주기 위해 ReactDOM.render(<Container/>, root)를 호출해 다시 화면을 렌더링 해줘야 한다.
### State 사용
```javascript
//onChange
const root = document.getElementById("root");
const Container = () => {
    const [number, setNumber] = React.useState(0);
    function num(event) {
        setNumber(event.target.value);
    }
    return (
        <div>
            <h3>Number : {number}</h3>
            <label htmlFor="number">숫자를 입력해 주세요</label>
            <input id="number" type="number" value={number} onChange={num}></input>
        </div>
    );   
}
ReactDOM.render(<Container/>, root);
```
```javascript
//onClick
const root = document.getElementById("root");
const Container = () => {
    //useState의 파라미터로 cnt의 default값(최초의 값)이 들어간다.
    const [cnt, setCnt] = React.useState(0);
    function cntUp() {
        // setCnt(cnt + 1); 값을 넣을 수도 있다.
        setCnt((cnt) => cnt + 1); //함수를 넣을 수도 있다. State의 첫 번째 인자값을 사용하는 경우 이 방법이 위의 방법보다 더 좋은 방법
    }
    return (
        <div>
            <h3>Total Clicks : {cnt}</h3>
            <button onClick={cntUp}>Click me</button>
        </div>
    );   
}
ReactDOM.render(<Container/>, root);
```
* State를 사용하면 이벤트 발생시 호출되는 함수마다 화면을 렌더링 하는 ReactDOM.render()를 호출할 필요가 없다.
* React.useState()는 ReactDOM.render()에 들어갈 컴포넌트 내에 넣어야 한다. 컴포넌트 밖에 넣으면 동작하지 않는다.
* React.useState()의 반환값은 길이 2인 배열인데 첫번째 인자는 렌더링할 데이터이고, 2번째 인자는 modifier(자바의 setter라 생각하면 된다.)이다.
* modifier가 호출되면 자동으로 ReactDOM.render(<Container/>, root)가 호출된다.
* modifier를 거치지 않고 State의 값을 바꿀 수 없다. 또한 modifier는 이전의 State를 새로운 값으로 __무조건__ 대체한다. 예를 들면 배열을 사용할 때 이전의 배열에 push()를 사용해 값을 추가하지 못하고 새로운 배열을 만들어서 반환해야한다.
## useNavigate
```javascript
import { useNavigate } from "react-router";

...

function VideoChat() {

    ...

    const navigate = useNavigate();

    const onClick = () => {
        if(!loginState) {
            navigate("/login");
        }
        setCreateRoomOpen(modalOpen => true);
    }

    ...

    return (

        ...

        <CreateChatRoom onClick={onClick}>채팅방 생성</CreateChatRoom>

        ...
    )
```
* useNavigate는 특정 이벤트가 발생할 때 유저를 redirect 하기위해 사용된다.
* navigate("/login", {replace : true}) 처럼 두 번째 파라미터의 객체에 replace : true를 주면 뒤로가기를 하더라도 이전의 페이지로 돌아오지 못하게 할 수 있다.
## useLocation
```javascript
function SearchRoom() {

    ...
    
    const getRoomKey = async (roomId: string | null) => {
        try {
            const {roomKey} = await (await axios.post(`${myData.domain}/api/getRoomKey`, {
                roomId,
                password: roomPassword
            }, {
                withCredentials: true
            })).data;
            navigate(`/video-chat`, {state: roomKey});
        } catch(err: unknown | AxiosError) {
            handleAxiosException(err);
        }
    }

    // 채팅방에 들어가기위해 클릭
    const onClick = (event: ClickEvent) => {        
        getRoomKey(event.target.getAttribute("data-roomid"));
    }

    ...

    return(

        ...

        <tbody>
            {rooms.map((room, idx) => 
            <Tr key={idx}>
                    <TdRoomName><Room data-roomid={room.roomId} data-roomname={room.roomName} onClick={room.locked ? onClickLocked : onClick}>{room.roomName} {room.locked ? <FaLock /> : null}</Room></TdRoomName>
                <TdNumber>{room.count}/9</TdNumber>
            </Tr>)}
        </tbody>

        ...

    )
}
```
```javascript
function VideoChat() {

    const {state: roomKey} = useLocation();

    ...

}
```
* useNavigate의 두 번째 파라미터의 객체에 state : (보내고자 하는 데이터)를 주면 리다이렉트된 페이지에서 useLocation을 통해 해당 데이터를 받을 수 있다.
* useParams와 달리 url에 데이터가 노출되지 않는다.
## useParams
```javascript
function App() {
  
  return (
    <Router>
      <Routes>

        ...

        <Route path='/video-chat/:roomKey' element={<ValidateAuth element={<VideoChat />} />} />

        ...
      </Routes>
    </Router>
  );
}
```
```javascript
function VideoChat() {

    const { roomKey } = useParams();

    ...

}
```
* useParams를 통해 path variable을 불러올 수 있다.
* 노출되면 안되는 데이터는 useParams를 사용하기 보다는 useLocation을 사용하는 것이 좋다.
* 쿼리스트링은 useParams가 아닌 queryString.parse(window.location.search)를 사용해 객체형태로 불러올 수 있다.