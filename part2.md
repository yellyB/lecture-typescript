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
