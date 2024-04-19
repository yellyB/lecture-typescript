# 섹션3: React 타입 분석

### UMD 모듈과 tsconfig.json

- 리액트에서 컴포넌트는 함수! props를 받아 JSX를 리턴하는 함수다: (props) ⇒ JSX

### 함수 컴포넌트 (FC vs VFC)

- 정의해놓은 컴포넌트에 커서 올려보면 JSX.Element 로 나오는데 타입 찾아 들어가보면 결국 React.ReactElement랑 동일함 (* React 타입파일에서 namespace JSX로 찾으면 됨)
- 컴포넌트는 JSX.Element로 타입 추론이 되기 때문에 타입 작성 잘 안함. 그래도 하고 싶다면 해도 되는데 이미 만들어져 있는 “FunctionComponent(=FC)” 타입을 사용하는게 좋다. (이유: props에 직접 타이핑하기 보다는 앞에 써줘서 타입 추론 알아서 되는데, 리액트가 이미 통째로 만들어놨는데 개별로 props를 사용할 필요 없어서)
    - 이 타입은 FunctionComponent<P = {}> 형태라서 작성할때도 동일한 위치에 타이핑을 해주자.
    
    ```tsx
    interface P { name: string, title: string }
    
    const TempComponent: FunctionComponent<P> = (props) => {
        ....
    }
    ```
    
- 리액트 17버전에는 VFC(v=void)가 있는데 FC와 차이점은 children이 없다는 것 (18버전에는 둘다 children 제공 안해줌. 그래서 겹치는 VFC는 없어짐)
    - 그래서 children을 쓰고 싶다면 직접 타이핑 해주기 ( children?: ReactNode | undefined )


<br/><br/><br/>


### useState, useEffect 타이핑

```tsx
// lazy init
// 사용하는 상황: 넣어주려는 초기값이 복잡한 함수라면, 리렌더링 될때마다 초기화되니 한번만 호출되도록 하려고 사용
const [state, setState] = useState(() => { return 'temp' });
```

- await는 타입스크립트 힌트(이미지의 커서 올린 부분)에 Promise가 있어야 쓸 수 있음
![image](https://github.com/yellyB/lecture-typescript/assets/50893303/41d567ef-deba-4d72-82b1-8b879b85c662)

- useEffect는 타입스크립트에선 콜백의 리턴 타입이 void로 고정되어 있어서 다음과 같이 async로 사용할 수 없다. (단, 클린업함수는 따로 타이핑 되어어서 예외)  
![image](https://github.com/yellyB/lecture-typescript/assets/50893303/742b582d-6235-4a42-9131-b522bab511fd)
    - async를 쓰면 리턴타입이 Promise void가 되어서 고정된 타입인 void와 맞지 않게 됨
    - 그래서 자바스크립트 리액트에선 가능하지만 타입스크립트 리액트에선 불가능
    - 그럼 타입스크립트에선 useEffect안에서 동기처리 어떻게 하느냐? 아래처럼  
      ![image](https://github.com/yellyB/lecture-typescript/assets/50893303/3f558399-d891-4107-bed7-e6b229d2596a)

