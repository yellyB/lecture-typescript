# 섹션3: Axios 타입 분석

### 다양한 방식으로 사용 가능한 axios

- axios 공식 문서에 옆에 ts가 붙어있었는데, 이러면 설치할 때 @types/axios 이런거 안해도 됨
- import axios from ‘axios’ 해서 이거 타고 라이브러리 파일 들어가보면, 가장 마지막을 확인해야 한다.
→ export default axios 가 있군.
→ 만약 이 파일에 “export =” 이런게 있으면 CommonJS임. 이경우엔 임포트할때 import axios = require(’axios’)로 써야함
    (하지만 ex module introp?를 키면 ESM이랑 임포트 똑같이 쓸 수 있다.)
    
    * ESModule Interop: TypeScript에서 ECMAScript 모듈을 CommonJS 모듈 빙식으로 불러올 때 생기는 차이를 해결하기 위한 설정. "esModuleInterop": true 로 설정한다.
    (Interop: "Interoperability"의 줄임말, **상호 운용성)**
    
- axios는 저수준인 **fetch**에 **여러 기능**을 붙여놓은것.
- axios는 브라우저의 ky나 노드의 got과는 다르게 멀티 플랫폼에 사용하기 좋다.
- axios의 타입을 확인해보면 axios 는 클래스이자, 함수이자, 객체이다. 그래서 new Axios(), axios(), axios.get() 세가지 방식 모두 사용 가능

### ts-node 사용하기

- const response = await axios.get(’~~’)해온것의 타입을 보면 any로 되어있는데, 이는 get의 타입이 get<T = any, …> 으로 되어있기 때문임. T를 안넣어줘서 기본값으로 any가 된것.
- 작성한 .ts 파일을 실행하려면 먼저 js로 변환하고 실행해줘야함.
    - 명령어는 “npx tsx” → “node axios.js” 두가지를 통해 실행한다.
    - “npx tsx” 로 js 로 변환한 파일을 보면 복잡해져있는데, 이는 CSM, ESM을 모두 지원하기 위해 트릭을 준 것
    - 이게 귀찮기 때문에 명령어 한번으로 위 과정을 해결해주는 ts-node를 설치해보자. 설치 후 “npx ts-node axios” 해주면 됨

### 제네릭을 활용한 Response 타이핑

- axios의 response 타이핑을 해보자.
    
    ```jsx
    [axios 라이브러리 타입 파일]
    get<T = any, R = AxiosResponse<T>, D = any>(url: string, config?: ..): Promise<R>;
    
    interface AxiosResponse<T = any, D = any> {
    	data: T;
    	...
    }
    
    [구현파일.ts]
    interface Pose {
    	userId: number;
    	id: number;
    }
    
    const response = await axios.get<Post>('~~url~~');
    ```
    

### AxiosError와 unknown error 대처법

```jsx
try {
	...
} catch (error){
	console.log(error.response.data);  // error: 타입 에러
}
```

- 위 같이 쓰면 타입 에러 난다. 왜? catch 문의 error는 unknown이기 때문.
⇒ unknown 타입인 이유: try 안에 있는 코드가 axios 호출밖에 없더라도, 만약 여기서 문법 에러가 날 수도 있고 모르는 것이기 때문이다.
- 해결법1:

```jsx
try {
	...
} catch (error){
	const errorResponse = (error as AxiosError).response;
	console.log(errorResponse?.data);
	errorResponse?.data;
}
```

⇒ unknown 이라 가장 쉬운 방법은 as로 타입 알려줌 + 변수에 저장해두면 나중에 쓸때도 타입 적용되어 있음

- 근데 문제는.. 좋은 방법이 아니란것. 왜냐하면 error가 AxiosError 타입이 아닐경우엔 어쩔?
- 해결법2: 커스텀 타입 가드 (+알파)

```jsx
try {
	...
} catch (error){
	if (error instanceof AxiosError){
		error.response;  // error가 AxiosError 으로 제대로 나옴
	}
	// 또는 axios 에서 제공해주는 타입으로도 검사 가능. (isAxiosError는 is 키워드 쓴 타입 가드임)
	if (axios.isAxiosError(error)){
		error.response;  // error가 AxiosError 으로 제대로 나옴
	}
}

	// 참고: AxiosError의 isAxiosError 타입은 아래와 같음
isAxiosError(payload: any): payload is AxiosError;
```

+ AxiosError는 클래스도 되어있는데, 인터페이스 안쓰고 이렇게 한 이유는 뭘까 생각해보면, 자바스크립트 변환되었을때 남아있다는 점도 있고 타입 가드 쓰기 좋다는 이유도 있음

### Axios 타입 직접 만들기
- 실제 사용하는 코드 가져와서 타입 만들어보기

```tsx
// 기본 구조 잡기
interface Axios {
	get: () => {};  // axios.get<Post, AxiosResponse<Post>>('~~~'); 을 대응
	post: () => {};  // axios.post<Created, AxiosResponse<Created>, Data>('~~~', {}); 을 대응
	(config: {}): void;  // axios({ method: 'post', url: '~~~', data: {} ) 을 대응
	(url: string, config: {}): void;  // axios('~~~', { method: 'post', data: {} }) 을 대응
}

// 매개변수 타이핑
interface Axios {
	get: (url: string) => {};
	post: (url: string, data: any) => {};
	(config: {}): void;
	(url: string, config: {}): void;
	isAxiosError: () => void;  // axios.isAxiosError(error) 을 대응
}

// isAxiosError의 커스텀 타입 가드
interface Axios {
	...
	isAxiosError: <T>(error: T) => error is T;
}

// await 붙을 수 있는 애들 처리. 리턴 타입이 Promise 이다.
interface Axios {
	get: (url: string) => Promise;  // await axios.get<Post, AxiosResponse<Post>>('~~~'); 가 될 수 있다.
	post: (url: string, data: any) => Promise;
	...
}

// Promise 추가에 대한 제네릭 작성
interface Axios {
	get: <T, R = AxiosResponse<T>>(url: string) => Promise<R>;  // 헷갈릴 수 있는지점: response.data가 T이다. 리턴 타입 자체는 AxiosResponse 임
	post: <T, R = AxiosResponse<T>, D>(url: string, data: D) => Promise<R>;  // data에 대한 타입도 D로 수정
	...
}

// 제네릭에서 필수와 옵셔널에 대한 타입 처리
// ㄴ> 제네릭에 옵셔널로 넣을수 있게 할 값들은 "변수명=any"로라도 처리하자. 제네릭 모든 값을 옵셔널 가능하게 처리
interface Axios {
	get: <T = any, R = AxiosResponse<T>>(url: string) => Promise<R>;
	post: <T = any, R = AxiosResponse<T>, D = any>(url: string, data: any) => Promise<R>;
	...
}

// isAxiosError 타이핑 수정
interface Axios {
	...
	isAxiosError: (error: unknown) => error is AxiosError;  // error가 AxiosError라고 명시
}
```

<br/><br/>
---

<br/><br/>

# 섹션4: React 타입 분석



<br/><br/>

### UMD 모듈과 tsconfig.json

- 리액트에서 컴포넌트는 함수! props를 받아 JSX를 리턴하는 함수다: (props) ⇒ JSX


<br/><br/>


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


<br/><br/>


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




<br/><br/>

### 브랜딩 기법

- 필요에 의해 커스텀으로 타입을 새로 정의하는 방법

```tsx
// 모든 number 를 허용하는 것이 아닌, 단위에 맞는 값만 받을 수 있도록 허용하고 싶다.
// Brand로 number 중에서도 특정 값만 허용하는 타입을 만들자.

type Brand<K, T> = K & { _brand: T }

type USD = brand<number, 'USD'>;
type EUR = brand<number, 'EUR'>;
type KRW = brand<number, 'KRW'>;

const usd = 10 as USD  // number 는 원시값이라서 브랜딩 쓰려면 사용할 떄 마다 어쩔 수 없이 강제 타입 변환을 해줘야함
const eur = 10 as EUR
const krw = 2000 as KRW

function euroToUsd(euro: EUR): number {
    return (euro * 1.18)
}

euroToUsd(usd)  // 에러. 인자로 EUR 타입만 넣을 수 있게 했으므로 에러가 난다.
```


<br/><br/>



### useCallback, useRef 타이핑

**useCallback**

- useCallback은 17버전과 다르게 18버전에서는 직접 타이핑 해줘야 한다.

```tsx
// v17
function useCallback<T extends (...args: any[]) => any>(callback: T, deps: DependencyList): T;

// v18
function useCallback<T extends function>(callback: T, deps: DependencyList): T;

// 위와 같이 18버전에서는 function으로만 정의되어 있어서 매개변수랑 리턴값을 적어줘야함. (17버전은 e는 any로 추론 되었다)
```

- 이벤트 타입을 찾는 팁

```tsx

const onSubmitForm = useCallback((e: FormEvent) => {
    ...
}, []);

// 이 같이 이미 작성된 타입 있으면 F12로 타고 들어가서 타입 파일에서 동작에 맞는 타입을 찾아서 적어주자 (임포트는 'react'에서 되어야 하는것 주의!)

const onClick = useCallback((e: MouseEvent<HTMLButtonElement>) => {
    ...
}, []);

const onChange = useCallback((e: ChangeEvent<HTMLInputElement>) => {
    ...
}, []);
```

**useRef**

- useRef는 총 3가지 타입이 있어서 원하는 타입에 걸리도록 해줘야함
- (1번 타입) 그 중 첫번째인 mutableRef는 JSX와 연결해주기 위한게 아닌, 데이터 저장하는 useState같은 역할을 해주고 싶은데 리렌더링은 안하고 싶을 때 사용하는 애임
- (2번 타입) 그럼 JSX랑 연결해주는 RefObject는 어떻게 연결시키나?

```tsx
// 타입 파일에 정의된 3가지 타입 유형
function useRef<T>(initialValue: T): MutableRefObject<T>;  // 값만 저장하려고 쓰는 ref
function useRef<T>(initialValue: T|null): RefObject<T>;  // JSX랑 연결하려는 ref
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

// 사용할 때 예시
const inputEl = useRef<HTMLInputElement>(null);  // 이렇게 타이핑해주면 2번째 타입으로 연결된다. HTMLInputElement 를 안쓰거나 null을 안쓰면 2번째로 연결이 안된다.
const mutaRef = useRef(0);

useEffect(() => {
    mutaRef.current += 1;  // 얘는 이런 용도로 사용.
})

// 위와 같이 1, 2, 3번 타입으로 추론되는 이유?
// 2번 타입은 제네릭으로 T를 받는데, 그 T가 실제 초기에 할당하는 null과 달라야 한다고 정의되어 있다.
// 그렇기 때문에 <HTMLInputElement>을 주고, 초기값을 null로 할당하면 2번째 타입으로 연결되는 것이다.
// HTMLInputElement를 안쓰면 초기값으로 할당한 null이 제네릭인 T로 추론되기 때문에 1번째 타입으로 연결되고
// 초기값 null을 안넣어주면 T = undefined 인 3번째 타입으로 연결되는 것.
```


<br/><br/>

### 클래스 컴포넌트 타이핑

- ReactElement 랑 ReactNode 는 다르다.
    - **`ReactElement`** : JSX로 작성된 요소
    - **`ReactNode` :**  React에서 렌더링할 수 있는 모든 타입. 문자열, 숫자, 배열, null, undefined, **`ReactElement`** 포함
    - ⇒ FunctionComponent 타입에는 `ReactElement` 인데, 클래스컴포넌트 타입에는 render()가 `ReactNode` 임 (=문자열로 리턴은 클래스만 가능하다)
- 네임스페이스 방식 vs 임포트 방식(모듈)
    - 둘 다 코드를 조직화 & 재사용하는 방법
    - 네임스페이스: Utils.add() 같이 사용. 주로 타입스크립트에서 사용. 브라우저orNode.js에서 사용하려면 트랜스파일 필요
    - 모듈: 파일로 분리해서 import 해와서 사용. 브라우저, Node.js 모두 지원. 파일로 관리해서 파일 개수는 많아진다는 단점
    - * 임포트 방식을 추천한다. (왜냐면 서로 타입 안겹치려고 하는건데 네임스페이스는 겹쳐버릴 수 있어서)


<br/><br/>


