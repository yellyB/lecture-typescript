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
