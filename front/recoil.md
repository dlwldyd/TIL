# Recoil
## 사용하기 전
```typescript

...


const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);
root.render(
  <React.StrictMode>    
    <RecoilRoot>
      <ThemeProvider theme={MyTheme}>
        <App />
      </ThemeProvider>
    </RecoilRoot>
  </React.StrictMode>
);

...

```
* Recoil을 사용하기 전에 index.tsx에 있는 <App /> 을 <RecoilRoot>로 감싸줘야 한다.
## Recoil 사용하기
```typescript
import { atom } from "recoil";
import { recoilPersist } from "recoil-persist";

const { persistAtom } = recoilPersist({ // 새로고침하거나 페이지를 닫아도 데이터를 유지하기 위해 사용
    key: "recoil-persist-atom",
    storage: sessionStorage // default : localStorage
})

export const loginStateAtom = atom<boolean>({
    key: "loginState", 
    default: false, 
    effects_UNSTABLE: [persistAtom]
});

export const usernameAtom = atom<string>({
    key: "username", 
    default: "", 
    effects_UNSTABLE: [persistAtom]
});

export const nicknameAtom = atom<String>({
    key: "nickname", 
    default: "", 
    effects_UNSTABLE: [persistAtom]
});
```
* Recoil을 사용해 모든 컴포넌트에서 접근하려면 atom({key, default})를 사용하여 변수를 만들고 export를 해주면 된다.
* key는 Recoil이 관리하는 변수에 접근하기 위한 key 값이고 default는 초기값이다.
* 해당 값을 페이지를 닫거나 새로고침해도 삭제되지 않게 하려면 sessionStorage나 localStorage를 사용해야 하는데 recoilPersist를 사용하면 해당 값을 스토리지에 저장할 수 있다. 이 때 스토리지에 저장되길 원하는 atom에 effects_UNSTABLE을 넣어줘야 한다.
```typescript

...

const [nickname, setNickname] = useRecoilState(nicknameAtom);

const username = useRecoilValue(usernameAtom);

const setUsername = useSetRecoilState(usernameAtom);

...

const changeNickname = async () => {
    try {
        const status = (await axios.get(`${myData.domain}/api/changeNickname?nickname=${nicknameInput}`, {
            withCredentials: true,
        })).status;
        if(status === 200) {
            setNickname(nickname => nicknameInput);
            setEditNicknameOpen(editNicknameOpen => false);
            setNicknameInput(nickname => "");
        }
    } catch(err: unknown | AxiosError) {
        handleAxiosException(err);
    }
}
```
* atom 값을 가져올 때는 useRecoilState("atomKey")를 사용하여 가져올 수 있다. 사용법은 useState()와 같다.
* 값만을 가져오거나 setter만을 가져오고 싶을 때는 useRecoilValue("atomKey")나 useSetRecoilState("atomKey")를 사용하면 된다.