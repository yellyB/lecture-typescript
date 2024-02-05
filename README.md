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



### 타입가드

```tsx
function numOfStr(a: number | string) {
    (a as number).toFixed(1);  // -> 이 경우 a는 string도 올 수 있기 때문에 js단에서 에러남
}

// 따라서 아래같이 타입 가드 해준다.
function numOfStr(a: number | string) {
    if (typeof a === 'number) {
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
