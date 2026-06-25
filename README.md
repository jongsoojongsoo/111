# useRef

## useRef가 뭔가?

useRef는 `{ current: ... }` 객체를 반환하는 훅임

```js
const ref = useRef(초기값);
// ref = { current: 초기값 }
```

리렌더가 일어나도 같은 객체를 유지함 → 즉 ref는 리렌더 사이에서도 값이 살아있음

---

## useState랑 뭐가 다름?

| | useState | useRef |
|---|---|---|
| 값 바뀌면 리렌더? | O | X |
| 변경 방법 | setState() | .current에 직접 대입 |
| 쓰는 경우 | 화면에 보여줄 값 | DOM 접근 or 내부 변수 |

> 판단 기준: "이 값 바뀔 때 화면도 바뀌어야 해?"
> - YES → useState
> - NO → useRef

---

## useRef의 두 가지 용도

### 1. DOM 직접 접근

```jsx
const inputRef = useRef(null);

// JSX에서 ref 연결
<input ref={inputRef} />

// 이후 DOM 직접 조작 가능
inputRef.current.focus();
```

마운트 되기 전엔 `null`, 마운트 후에 실제 DOM 요소가 들어옴

활용 예시
- focus / blur 제어
- 스크롤 이동
- 크기, 위치 측정 (getBoundingClientRect)
- canvas, video 같은 미디어 제어
- Chart.js 같은 외부 라이브러리 붙일 때

### 2. 렌더 없이 값 저장

```jsx
const clickCount = useRef(0);

const handleClick = () => {
  clickCount.current += 1; // 리렌더 안 일어남
};
```

화면엔 안 보여도 되는 값을 저장할 때 씀

활용 예시
- setInterval / setTimeout ID 저장
- 이전 state 값 추적
- 렌더 횟수 카운팅
- isFirstRender 같은 플래그 변수

---

## 이전 값 추적 패턴

```jsx
const [count, setCount] = useState(0);
const prevCount = useRef(0);

useEffect(() => {
  prevCount.current = count;
});

// 현재: count, 이전: prevCount.current
```

---

## 타이머 ID 저장 패턴

```jsx
const timerRef = useRef(null);

const start = () => {
  timerRef.current = setInterval(() => {
    console.log('tick');
  }, 1000);
};

const stop = () => {
  clearInterval(timerRef.current);
};
```

state로 타이머 ID 관리하면 불필요한 리렌더 생김 → ref가 적합

---

## 주의사항

ref.current를 JSX에 직접 넣으면 안 됨

```jsx
// 이렇게 하면 값은 바뀌는데 화면이 안 바뀜
<p>{ref.current}</p>
```

렌더링 중에 ref 읽거나 쓰지 말 것 → 이벤트 핸들러나 useEffect 안에서만

---

## 심화: forwardRef

일반 컴포넌트는 ref를 props로 못 받음 → forwardRef로 감싸야 함

```jsx
const FancyInput = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// 부모에서
const inputRef = useRef(null);
<FancyInput ref={inputRef} />
inputRef.current.focus(); // 자식 input에 접근 가능
```

### useImperativeHandle

DOM 전체 말고 특정 메서드만 골라서 노출할 때

```jsx
const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => { inputRef.current.value = ''; },
  }));

  return <input ref={inputRef} />;
});
```

캡슐화 유지하면서 필요한 것만 열어줄 수 있음

---

## 참고

- https://ko.react.dev/reference/react/useRef
- https://ko.react.dev/reference/react/forwardRef
- https://ko.react.dev/reference/react/useImperativeHandle
