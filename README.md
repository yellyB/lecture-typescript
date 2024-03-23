# 타입스크립트 올인원 : Part1. 기본 문법편



<br/>

### 타입스크립트를 할 때 알아야 할 단 한가지

- ts 는 최종적으로 js로 변환된다: ts -(tsc)→js
- ts코드를 js 변환할 때의 옵션들을 tsconfig.json에 작성. tsc가 읽어서 변환해줌(es5, es6같은 특정 버전으로 만들어줘, 혹은 익플말고 최신 브라우저만 지원하게 해줘)
- tsc의 역할: 1단계: type 검사/ 2단계: ts→js로 변환
    - 근데 순서는 1→2인데 별개라서 타입 검사 실패해도 코드 변환은 된다
    - 하지만 타입 검사가 강제가 아니란걸 알려주는것일 뿐, 실제 코드 짤 때는 에러 없는 타입 짜야함

<br/>

---

<br/>

```jsx
npx tsc —noEmit  // 타입 검사 실행하는 터미널 명령어
npx tsc          // ts파일을 js파일로 변환
```

### 몇가지 주요 tsconfig.json 설정들

- allowJs: true → 타입스크립트랑 자바스크립트 동시에 사용 가능
- strict: true → 엄격한 타입 검사(true를 강력추천)
- target: “es2016” → ts를 무슨 버전 코드로 바꿀지. 필요에 따라 버전 적어주기
- module: “commonjs” → 최신 es Module 쓰고 싶으면 (es2022 = es2015) 로 설정
- forceConsistentCasingInFileNames: true → 파일이름에 대소문자 구분에서 import
- skipLibCheck: true → 라이브러리의 타입 파일인 .d.ts 를 건너뛴다.



<br/>

---

<br/>


# 섹션1

### 배열 타입 표현 법 두가지

- number[]
- Array<number>

### 타입은 정확하게

- vscode에서 커서 올리면 타입 추론해주는거 활용하자

```jsx
const foo: string = '5';
```

- 위 코드는 문제 있는 코드: const 키워드를 통해 생성되었고 ‘5’라는 정확한 타입이 있는데  
굳이 string이라는 더 넓은 범위의 타입을 정해줘버림
⇒ 타입스크립트가 어차피 ‘5’라고 추론해주기 때문에 이런걸 건들지말자. 타입스크립트가 이상하게 추론할때만 지정해주기
- 타입추론이 any로 나오게 된다면 이땐 typing해주자

### JS 변환 시 사라지는 부분 파악하자

```jsx
function add(x: number, y: number): number;  // 이건 타입 선언한거라 js로 변환하면 사라짐
function add(x, y) {
	return x + y;
}
```

- 사라지는 부분 지웠을 때 제대로 된 자바스크립트 코드여야 하기 때문에 알아야 함


### `never` 타입
- 함수 리턴값이나 변수 값이 절대로 발생하지 않을 때 사용
- 빈 배열 ([]) 타입은 `never`이기 때문에 string[] 같이 해줘야 `.push()` 같은 거 사용 가능

### 느낌표(!)

- 예를들어 어떤 값의 타입이 `string | null` 일 때, 마지막에 ! 를 붙이면 `string` 타입으로 강제
    - null 이나 undefined 가 아님을 보증함.
    
    ```tsx
    const head = document.querySelector('#head');  // Element | null
    const head = document.querySelector('#head')!;  // Element
    ```
    
    - 근데 비추천하는 이유 ⇒ 어떤것도 확실하지 않기 때문. 하지만 읽을 수 있으라고 알려주는거임
    - 차라리 아래같이 쓰자.
    
    ```tsx
    if (head) {
    	head.innerHTML = 'hello';
    }
    ```
    

### 템플릿 리터럴 타입

```tsx
type StrType = 'world' | 'hell';

type Greeting = `hello ${StrType}`;  // hello world, hello hell 로 자동 타입 추천
```

### 튜플

- 튜플 타입에 없는걸 [i]로는 추가 못함. 근데 타입스크립트는 바보라서 push 로는 추가 가능

```tsx
const tuple: [string, number] = ['1', 1];

tuple[2] = 'hello';  // 불가
tuple.push('hello');  // 가능
```

### as const

- enum을 쓸수도 있고 객체를 enum같이 쓸 수도 있음. 근데 enum은 자바스크립트로 변환하면 사라지지만, 객체+as const 로 쓰면 계속 남아있다. ⇒ 그래서 객체 방식을 추천
    - 남겨두는걸 추천하는 이유: 런타임에서 한번 더 사용해서 검사할 수 있기 때문. 사용자 입력은 컴파일 단계에선 검사하기 힘들다.
      
```tsx
const Direction = {
	Up: 0,
	Down: 1,
	Left: 2,
	Right: 3
} as const;  // 아래랑 동일한 코드

const Direction: {Up: 0, Down: 1, Left: 2, Right: 3} = {
	Up: 0,
	Down: 1,
	Left: 2,
	Right: 3
} as const;
```

- as const 를 안쓰면 타입스크립트는 값들을 number 형으로 추론
⇒ as const 로 의도대로 명확한 타입으로 지정해주자
⇒ 이렇게 쓰면 `readonly Up: 0` 같이 타입 정해짐

- 근데 enum 안쓰면 조금 불편한 점: 아래같이 써야함
    
    ```tsx
    type DirectionType = typeof Direction[keyof typeof Direction];
    ```
    
    - 위 코드 해석
        1. `type DirectionType = keyof Direction;` <- 이게 원형
        2. 근데 이게 타입이 아님 => `typeof` 를 앞에 붙여줌
        ⇒ `type DirectionType = keyof typeof Direction`
        3. 근데 또 여기서 키 말고 값만 뽑아내서 그거 사용해서 타입 정의하고 싶음
        ⇒ 이래서 최종적으로 저 코드가 된 것.

### union(|) intersection(&)

- 간단한거는 type, 복잡한 객체 지향은 interface 사용하
- 타입을 한 번 잘못 설정하면 줄줄이 문제 생김. 마치 아래 같이!
    
    ```jsx
    function add(x: string | number, y: string | number): string | number { return x + y}
    // 위 add 함수 파라미터 타입은 문제가 없으나..
    
    const result: string | number = add(1,2);
    // 이 result 는 분명 number타입인데, 잘못된 타입 선언으로 string타입이 될 수도 있는 상태가 되어버림
    ```

### 타입 앨리어스와 인터페이스

- 인터페이스는 같은 이름을 계속 선언할 수 있음 → 계속 확장됨. 때문에 라이브러리는 보통 인터페이스 씀

### 잉여 속성 검사

- 객체 리터럴을 타입 정의한 변수에 대입할때는 잉여 속성 검사를 한다.
    
    ```tsx
    interface Human { name: string }
    
    const oto: Human = { name: 'bomi', height: 160};  // 잉여 속성 검사를 하기 때문에 height 에 대한 에러 남
    ```
    
    ⇒ 이때는 객체 리터럴을 임시 변수에 한번 담은 뒤, 임시 변수를 다시 변수에 넣으면 문제가 없다.
    
    ```tsx
    const temp = { name: 'bomi', height: 160};
    const oto: Human = temp;  // 이때는 타입 문제 없음
    ```
    

### void

- void 타입 정의하는 세가지 경우가 있다.
    - 매개변수 콜백
    - 함수의 직접적인 리턴 값
    - 함수 메서드 자체
    
    ```tsx
    function a(callback: () => void) {}  // 매개변수 콜백
    function a(): void {}  // 함수 리턴 값
    interface Human { talk: () => void }  // 메서드
    ```
    
    ⇒ 이 중에 유효한 리턴값이 있으면 에러나는 경우는: 2번, 직접적인 리턴값이 void로 설정된 경우
    
    ⇒ 나머지 두 경우는 “리턴값을 사용하지 않겠다!”는 뜻일 뿐
    
- 활용처 예시
    - 아무튼.. 이런 상황에서 아래 코드 사용할때, 여기 콜백 함수의 리턴값은 number임
    근데 위에 보듯이 콜백함수 타입이 undefined이기 때문에 number는에 할당할 수 없다.
    따라서 콜백함수 타입을 void로 맞춰줘야함
    ⇒ `void` `=/=` `undefined`
    
    ```tsx
    declare function forEach(arr: number[], callback: (el: number) => undefined): void;
    
    forEach([1, 2, 3], el => target.push(el));
    ```
    
    - (참고) 함수를 바디 없이 선언함. 그리고 구현부 적어줘야 하는데 그러기 싫다면? ⇒ declare 키워드 사용
    
    ```tsx
    function forEach(arr: number[], callback: (el: number) => undefined): void;
    function forEach() {}  // <- 이거 하기 싫음!
    
    // 그럼 선언할 때 아래처럼 declare 붙여주면 됨. 이 부분은 js 변환 시 사라짐
    declare function forEach(arr: number[], callback: (el: number) => undefined): void;
    ```
    
    - forEach가 외부에서 선언되었는데 그게 확실하고 우리는 타입만 지정해주고 싶다고 할 때 



### unknown 과 any

- any: 타입 검사 포기
- unknown: 지금 당장은 타입을 모르겠을 때

![읽는법: (왼쪽 열)은 (상단 행)에 대입 (가능/불가능)하다](https://github.com/yellyB/lecture-typescript/assets/50893303/b877d35a-56df-43f4-863b-0c3eff843a3f)
(왼쪽 열)은 (상단 행)에 대입 (가능/불가능)하다
- 초록 체크 표시는 strict: true 일때는 x로 간주하기



### 타입가드(타입 좁히기)

```tsx
function numOfStr(a: number | string) {
    (a as number).toFixed(1);  // -> 이 경우 a는 string도 올 수 있기 때문에 js단에서 에러남
}

// 따라서 아래같이 타입 가드 해준다.
function numOfStr(a: number | string) {
    if (typeof a === 'number') {
        a.toFixed(1);
    } else {
        a.charAt(3);  // number가 아니면 string일 것이기 때문에 else로 써줘도 타입 추론 잘 됨
    }
}

// 배열 여부는 아래 같이 구분
if (Array.isArray(a)) {}

// 클래스는 new키워드로 생성한 객체를 타입 검사할 수 있음
class classA {
    aaa() {}
}
class classB {
    bbb() {}
}
function classAOrB(param: classA | classB) {
    if (param instanceof classA) {
        param.aaa();
    }
}
classAOrB(new classA());
```

- 타입스크립트는 if 문에 대해 타입추론 잘해준다.
    - 그래서 객체같은 경우는 안의 속성 혹은 키로 구분함
    
    ```tsx
    type apple = { type: 'fruit', color: 'red' }
    type cat = { type: 'animal', fur: 'soft' }
    
    // 값으로 체크
    function typeCheck(param: apple | cat) {
       if (param.type === 'fruit') {
            param.type;
        }
    }
    
    // 속성으로 체크
    function typeCheck(param: apple | cat) {
       if ('color' in apple) {
            param.type;
        }
    }
    ```

### is: 커스텀 타입 가드

- 함수 리턴값에 is 키워드를 넣으면 커스텀하게 타입 검사 할 수 있음

```tsx
const isRejected = (input: PromiseSettledResult<unknown>): input is PromiseRejectedResult => {
    return input.status === 'rejected';
};

const Promises = await Promise.allSettled([Promise.resolve('a'), Promise.resolve('b')]);
// 바로 아래 코드는 타입스크립트가 타입을 PromiseRejectedResult 타입으로 자동 필터 못해줌. 그래서 아래아래 줄처럼 커스텀 타입 써서 추론.
const errors = promises.filter((promise) => promise.status === 'rejected');
const errors = promises.filter(isRejected);
```


### 인덱스드 시그니처

- 객체의 모든 키가 특정 타입이면 사용하면 좋음

```tsx
type A = { a: string, b: string, c: string, d: string, ... }  // (x)이렇게 하는것 보단
type A = { [key: string]: string }  // (o)이렇게 한번에 선언 가능
```

### 맵드 타입스

- 객체의 키가 미리 정의한 타입으로만 구성되었으면 좋겠다

```tsx
type B = 'Human' | 'Mammal' | 'Animal';
type A = { [key in B]: string };  // key 는 B에 있는 값들만 올 수 있음
```

### 클래스

|  | public | protected | private |
| --- | --- | --- | --- |
| 클래스내부 | o | o | o |
| 인스턴스 | o | x | x |
| 상속클래스 | o | o | x |

### 제네릭

**제네릭의 필요성**

- add(1, 2) → 3
add('1', '2') → 12
⇒ 이 두가지 결과를 모두 반환할 수 있는 add() 함수 만들고 싶음

```tsx
function add(x: string | number, y: string | number): string | number { return x + y };
// 이렇게 만들었더니 add(1, '2') 도 될 수 있는 문제가 생김

function add(x: string, y: string): string {return x + y};
function add(x: number, y: number): number {return x + y};
// 이렇게 두개 만들자니 같은 이름 두번 선언 못함.
```

- 이때문에 타입을 변수처럼 만들게됨

```tsx
function add<T>(x: T, y: T): T {return x + y};
// T가 무슨 타입이 될지는 모르지만, 아무튼 파라미터x, y랑 결과값은 모두 동일한 타입이라고 알려주기
```

- 다양하게 사용하기

```tsx
function add<T extends number | string>(x: T, y: T): T {return x + y};  // number or string으로 제한

function add<T extends number, K extends string>(x: T, y: K): T {return x + y};  // T, K 둘다 각각 제한

function add<T extends { a: string }>(x: T): T {return x};

function add<T extends string[]>(x: T): T {return x};
```

- 리액트에서는 JSX때문에 꺽쇠쓰면 ts 에러뜸

```tsx
const add = <T = unknown>(x: T, y: T) => ({ x,y });  // (보통 이렇게 사용)기본값 넣어주기
const add = <T extends unknown>(x: T, y: T) => ({ x,y });  // 이렇게도 가능
```



<br/>

---

<br/>


# 섹션2: lib.es5.d.ts 분석

### forEach 제네릭 분석

```tsx
// lib.es5.d.ts 에서 긁어온 forEach 의 타입
interface Array<T> {
    forEach(callbackfn: (value: T, index: number, array: T[]) => void, thisArg?: any): void;
}

[1, 2, 3].forEach((value) => { console.log(value}; )  // ts는 value를 number로 추론해준다.
```

- 제네릭 덕분에 forEach 콜백 함수의 value 를 number라고 알 수 있게됨

### map 제네릭 분석

```tsx
interface Array<T> {
    map<U>(callbackfn: (value: T, index: number, array: T[]) => U, thisArg?: any): U[];
}

const strings = [1, 2, 3].map((value) => value.toString());  // ['1', '2', '3']
```

- toString() 한 결과가 U(여기선 string), 그러므로 map의 리턴값은 string[]가 됨


### filter 제네릭 분석

```tsx
interface Array<T> {
    filter<S extends T>(predicate: (value: T, index: number, array: T[]) => value is S, thisArg?: any): S[];
    filter(predicate: (value: T, index: number, array: T[]) => unknown, thisArg?: any): T[];
}

const filtered1 = [1, 2, 3].filter((value) => value % 2);  // 첫번째 선언된 타입에 들어맞음
const filtered2 = ['1', 2, '3'].filter((value) => typeof value === 'string');  // 첫번째, 두번째 타입에 둘 다 적용됨.

// filtered2 는 string | number [] 로 추론됨
// 만약 filtered2 를 string[] 으로 추론하고 싶다면?

const predicate = (value: string | number): value is string => typeof value === 'string';
const filtered2 = ['1', 2, '3'].filter(predicate);  // string[] 으로 추론 잘 됨
// 위에 predicate에서 value is string으로 타입스크립트에게 알려줬기 때문에 추론 잘 된것
```


### forEach, map, filter 타입 직접 만들기

- forEach

```tsx
const nums: Arr = [1, 2, 3];

nums.forEach((item) => {
    console.log(item);
});
nums.forEach((item) => {
    console.log(item);
    return '3';
});

// 이제부터 Arr 타입만들어주기(변화과정)
interface Arr {
    forEach(): void;
    forEach(callback: () => void): void;  // forEach 첫번째 인자 콜백함수니까 추가
    forEach(callback: (item: number) => void): void;  // 일단은 nums 가 숫자밖에 없으니 number
    // ['1', '2', '3'] 도 커버하도록 하려면?
    forEach(callback: (item: string | number) => void): void;  // 하지만 콜백 내부에서 toFixed, charAt 쓰면 타입 에러 발생
    forEach<T>(callback: (item: T) => void): void;  // 이러면 실제 사용할 때 forEach<number> 같은 식으로 귀찮게 작성을 해줘야 함
}

// 최종 결과물
interface Arr<T> {
    forEach<T>(callback: (item: T) => void): void;
}
const nums: Arr<number> = [1, 2, 3];
...

```

- map

```tsx
const nums: Arr = [1, 2, 3];

nums.map((v) => v + 1);

// 타입 만들기
interface Arr<T> {
    map(): void;
    map<T>(callback: (v: T) => void): void;
    map<T>(callback: (v: T) => T): T[];  // 위 예시코드의 map 할 때 v+1도 number이니 T로 설정, 결과도 배열로 설정
}

// 여기까지 하니 문제점 있음. 아래 경우 커버 못함
const strs1 = nums.map((v) => v.toString());  // ['2', '3', '4']; string[]
const strs2 = nums.map((v) => v % 2 === 0);  // [false, true, false]; boolean[]

interface Arr<T> {
    map<S>(callback: (v: T) => S): S[];  // S를 추가해 다양한 타입 커버할 수 있도록 변경
}
```

- filter

```tsx
const nums: Arr = [1, 2, 3];
const filtered1 = nums.filter((v) => v % 2 === 0);  // [2]: number[]

// 타입 만들기
interface Arr<T> {
    filter(): void;
    filter<T>(callback: (v: T) => boolean): T[];
}

// filtered1을 보면 위 타입만으로 커버 가능하긴 함.
// 문제는 아래가 추가되었을 경우.

const numsAndStrs: Arr = [1, '2', 3];
const filtered2 = numsAndStrs.filter((v) => typeof v === 'string');  // ['2']: string[]
// filtered2 의 결과는 string[] 인데도 위에 세팅한 타입 때문에 filtered2는 string | number[]로 추론된다.

// 다시 만들기 ㄱㄱ
interface Arr<T> {
    filter<T>(callback: (v: T) => v is T): T[];  // 형식조건자
}
const filtered2 = numsAndStrs.filter((v): v is string => typeof v === 'string');  // 여기에도 형식조건자 추가. 이러면 일단 에러는 사라진다. 그러나 여전히 string | number[]로 추론됨

// 다시 만들기 ㄱㄱ 2.  string | number 가 아닌 새로운 타입이 필요하다.
interface Arr<T> {
    filter<S>(callback: (v: T) => v is S): S[];  // S는 어디서 나왔길래 v랑 연결되나? ts가 이해하지 못해서 에러
    filter<S extends T>(callback: (v: T) => v is S): S[];  // S는 어디서 나왔길래 v랑 연결되나? ts가 이해하지 못해서 에러
}
```


### 공변성과 반공변성

- 함수를 다른 함수타입 변수에 대입할 때 return 값은 대입하려는 타입이 더 넓으면 가능. 매개변수는 더 좁으면 가능

```tsx
function funcA(x: string | number): number {
    return 0;
}
type typeB(x: string): number | string;

let funcB: typeB = funcA;  // 가능
```

*참고 : 타입 넓히기, 타입 좁히기

```tsx
// 타입 넓히기
let a = 5;  // hover 해보면 a는 number로 추론됨. = 타입스크립트가 타입 넓히기

// 타입 좁히기
let a: string | number = 5;
if (typeof a === 'string') {  // 이렇게 string 으로 좁혀주는거
    a.toString();
}
```

### 오버로딩

- 타입 하나로 모든 경우 커버하지 못하겠다면 같은 타입을 여러번 선언할 수 있음
    - 타입 스크립트가 알아서 타입 추론해줄거임

### catch 문에서 에러 타입?

```tsx
// Error에는 response가 없어서 추가
interface CustomError extends Error {
    response?: {
        data: any;
    }
}

(async () => {
    try {
        await axios.get();
    } catch (err: unknown) {
        const customError = err as CustomError;  // 1. unknown을 썼기 때문에 반드시 as문 사용 / 2. as 는 1회성이라 변수로 따로 할당해서 사용
        console.log(customError.response?.data);
        customError.response?.data;
    }
})();

// 위 코드의 문제점: catch 문의 변수 customError가 CustomError 타입이 아니면 어쩔거임..? 밑에 줄줄이 에러나게됨
// 아래처럼 개선:

// interface 쓰면 자바스크립트에서 사라져서 아래에 instanceof 를 쓸수가 없게됨. 때문에 class로 변경
class CustomError extends Error {
    response?: {
        data: any;
    }
}

(async () => {
    try {
        await axios.get();
    } catch (err) {
        // 아래처럼 if문으로 방어해주기
		    if (err instanceof CustomError) {
            const customError = err as CustomError;
            console.log(customError.response?.data);
            customError.response?.data;
		    }
    }
})();
```


# 섹션3: Utility Types

### Partial 타입 분석

- 타입스크립트 내장 타입인 Partial을 직접 만들어보기
    - Partial: 타입의 모든 요소들을 옵셔널로 만들어줌

```tsx
interface Profile {
    name: string;
    age: number;
    married: boolean;
}

const person: Partial<Profile> = {
    name: 'kimbomi',
    age: 30,
}

// 위에서 쓸 수 있도록 Partial 타입 만드는 과정
type Partial<T> = {
    [Key: string]: string;
    [Key in keyof Profile]?: string;  // ?로 옵셔널값을 만들어줌
    [Key in keyof Profile]?: Profile[Key];  // value의 타입 정해주기
    [Key in keyof T]?: T[Key];  // 범용적으로 사용할 수 있게 T 선언
}
```

### Pick 타입 분석

- Partial의 단점은 객체에 아무값도 안넣어도 되는 상태로 만들어버리는 것
- Pick을 쓰면 특정 타입만 뽑기 때문에 이 단점 보완됨 / 혹은 Omit을 써서 특정 타입만 제거

```tsx
const person: Pick<Profile, 'name' | 'age'> = {
    name: 'kimbomi',
    age: 30,
} 

type Pick<T, S> = {
    // 우선 T, S 두개가 필요함
    ?? : T[S];  // 키는 모르지만 값은 이렇게 될 것 같음
    [Key in keyof S]: T[S];  // 근데 Profile['name' | 'age'] 형태는 불가능한데?
    [Key in keyof S]: S[Key];  // 근데 여기에 S랑 T의 관계를 정해줘야함
}

type Pick<T, S extends keyof T> = {
    [Key in S]: T[Key];  // extends로 S가 T의 일부인걸 알려줌
}
```


### Omit, Exclude, Extract 타입 분석

- Exclude: 유니온 타입에서 특정 타입을 ‘제거’해 새로운 유니온 타입 만듬
* 유니온 타입: 여러 개의 타입을 하나의 변수에 허용
- Exclude, Extract

```tsx
type Animal = 'cat' | 'dog' | 'human';

type Mammal = Exclude<Animal, 'human'>;  // cat, dog
type Human = Extract<Animal, 'cat' | 'dog'>;  // cat, dog

// type Exclude<T, U> = T extends U? never : T;
// type Extract<T, U> = T extends U? T : never;

// 위를 보면 Exclude는 T가 U의 확장판이면 never로 취급하니까, 확장판이면 빼는것.
```

- Omit: Exclude + Pick. 지정한 속성을 제외한 나머지들로 새로운 타입 만들어냄

```tsx
type A = Exclude<keyof Profile, 'married'>

type Omit<T, S> = Pick<T, Exclude<keyof T, S>>
const NewPerson: Omit<Profile, 'married'>> = {
    name: 'kim',
    age: 29,
}
```


### Required, Record, NonNullable 타입 분석

- Required는 옵셔널한 요소를 전부 필수 요소로 만들어줌

```tsx
interface Profile {
    name?: string;
    age?: number;
    married?: boolean;
}

type Required<T> = {
    [key in keyof T]-?: T[key];  // modifier. ?를 빼버린다는 뜻
}
```

- Readonly 붙이면 수정 못하게 할 수 있음

```tsx
const person: Readonly<Profile> = {
    name: 'kimbomi',
    age: 30,
}
person.name = 'nero';  // 변경 불가능

// "-" 붙여서 readonly를 제거할수도 있음
type R<T> = {
    -readonley [key in keyof T]-?: T[key];
}
```

- Record: 객체를 표현하는 한가지 방법

```tsx
// 아래 두개는 같은 의미
interface Obj { [key: string]: number }
const a: Record<string, number> = { a: 3, b:5, c: 7 };
```

```tsx
type Record<T extends keyof any, S> = {
    [key in T]: S;
}
// 객체의 키로는 number, string, symbol만 올 수 있기 때문에 T에 keyof any를 써서 제한조건을 걸어준다.
```

- NonNullable: 여러 타입들중에 null과 undefined을 제외하고 나머지만 취하고 싶을 때 사용
    
    * 타입들이 키에 적용되는게 있고 객체에 적용되는게 있어서 이 둘을 구별해야한다. NonNullable은 키에 적용
    

```tsx
// 사용 예시
type All = string | null | undefined | boolean | number;
type filtered = NonNullable<All>;  // 타입이 string | boolean | number로 추론됨
```

```tsx
// NonNullable 타입 만들기
type NonNullable<T> = T extends null | undefined ? never : T;
```


### infer 타입 분석

- Parameters: 변수도 타입으로 사용할 수 있다.
    - (...args: any) => any 는 T를 함수형식으로 제한하려고 사용됨
    - infer = 추론. 아래 코드에서 사용된 부분: args의 타입이 추론이 가능하면 그 타입을 쓰겠다는 뜻

```tsx
function zip(x: number, y: string, z: boolean): { x: number, y: string, z: boolean } {
    return { x, y, z };
}

type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;

// zip함수를 타입으로 사용할 수 있다.
type Params = Parameters<typeof zip>;
type FirstType = Params[0];  // number. 이런식으로 타입이 배열로 이뤄진경우는 인덱스로 접근 가능하다.
```

- ConstructorParameters: 클래스의 생성자도 타입으로 사용가능

```tsx
type ConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : never;
type InstanceType<T extends abstract new (...args: any) => any> = T extends abstract new (...args: any) => infer R ? R : any;
```

```tsx
class A {
    a: string;
    b: number;
    c: boolean;
    constructor(a: string, b: number, c: boolean) {
        this.a = a;
        this.b = b;
        this.c = c;
    }
}

type C = ConstructorParameters<typeof A>;  // typeof 클래스가 생성자
type I = InstanceType<typeof A>;

const temp = new A('a', 44, true);  // temp는 인스턴스
```

- 기타 유틸리티 타입으로는 Lowercase(문자를 소문자로 바꿔서 타입으로), Uppercase, Capitalize(첫글자 대문자화), Uncapitalize, ThisType 등이 있음. 이런것들은 타입스크립트 내부적으로 코드적으로 처리해서 타입스크립트로는 구현 할 수 없음


### Promise와 Awaited 타입 분석

```tsx
const p1 = Promise.resolve(1).then((a) => a + 1).then((a) => a + 1).then((a) => a.toString());
const p2 = Promise.resolve(2);
const p3 = new Promise((res, rej) => {
    setTimeout(res, 1000);
})

Promise.all([p1, p2, p3]).then((result) => {
    console.log(result);  // ['3', 2, undefined]  => string, number, unknown 으로 타입 잘 추론됨. 어떻게 이게 가능할까?
});
```

```tsx
const arr = [1, 2, 3] as const;
type Arr = keyof typeof arr;

const key: Arr = '0' // 가능
const key: Arr = 'length' // 가능
const key: Arr = '3' // 불가
const key: Arr = 1 // 불가

// 할당 가능한 값이 위처럼 되는 이유:
// [p1, p2, p3] 는 { '0': p1, '1': p2, '2': p3, length: 3 } 의 형태이다.
// 그래서 '0' | '1' | '2' | 'length' 의 값만 할당할 수 있음
```