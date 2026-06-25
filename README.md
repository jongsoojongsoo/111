# useRef

## useRef가 뭔가?

`{ current: ... }` 객체를 반환하는 훅임

```js
const ref = useRef(초기값);
// ref = { current: 초기값 }
```

리렌더가 일어나도 같은 객체가 유지됨
→ 컴포넌트 생애 동안 ref는 항상 동일한 참조를 가리킴

---

## useState랑 뭐가 다름?

| | useState | useRef |
|---|---|---|
| 값 바뀌면 리렌더? | O | X |
| 변경 방법 | setState() | .current에 직접 대입 |
| 쓰는 경우 | 화면에 보여줄 값 | DOM 접근 or 내부 변수 |

처음엔 그냥 useState 쓰면 되는 거 아닌가 싶었는데
React가 리렌더하는 이유 자체가 "화면을 다시 그리기 위해서"잖음
근데 화면이랑 전혀 관계없는 값까지 리렌더를 유발하면 낭비

그래서 판단 기준을 하나 세움

> 이 값이 바뀔 때 화면도 바뀌어야 하냐?
> YES → useState / NO → useRef

---

## useRef의 두 가지 용도

### 1. DOM 직접 접근

포커스, 스크롤, 크기 측정 같은 건 state로 못 함
직접 DOM을 건드려야 하는 상황에서 씀

```jsx
const inputRef = useRef(null);

<input ref={inputRef} />

inputRef.current.focus();
```

주의할 게 마운트 전에는 `ref.current === null`임
useEffect 밖에서 바로 접근하면 null 에러 남
항상 마운트 이후에 접근해야 함

활용 예시
- focus / blur
- scrollIntoView
- getBoundingClientRect (크기, 위치)
- video, canvas 같은 미디어
- Chart.js, Three.js 같은 외부 라이브러리 붙일 때

### 2. 렌더 없이 값 저장

DOM이랑 전혀 상관없이 그냥 "리렌더 없이 값 유지"가 필요할 때

처음엔 그냥 컴포넌트 밖에 변수 선언하면 되는 거 아닌가 싶었는데
컴포넌트 밖 변수는 모든 인스턴스가 공유하는 값이 돼버림
같은 컴포넌트를 두 개 띄우면 값이 섞임
useRef는 인스턴스마다 독립적으로 값을 가짐

```jsx
const clickCount = useRef(0);

const handleClick = () => {
  clickCount.current += 1; // 리렌더 안 일어남
};
```

---

## 자주 쓰는 패턴들

### 타이머 ID 저장

setInterval ID를 state로 관리하면 저장할 때마다 리렌더가 일어남
근데 타이머 ID는 화면에 보여줄 값이 아니잖음
리렌더가 일어날 이유가 없음 → ref에 그냥 넣으면 됨

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

### 이전 state 값 추적

```jsx
const [count, setCount] = useState(0);
const prevCount = useRef(0);

useEffect(() => {
  prevCount.current = count;
});

// 현재: count, 이전: prevCount.current
```

useEffect가 렌더 이후에 실행되는 거 이용한 패턴임
렌더 직후에 현재 값을 ref에 저장해두면 다음 렌더 때 이전 값으로 쓸 수 있음

---

## 주의사항

`ref.current`를 JSX에 직접 넣으면 안 됨

```jsx
// 값은 바뀌는데 화면이 안 바뀌는 버그
<p>{ref.current}</p>
<button onClick={() => ref.current++}>증가</button>
```

콘솔 찍으면 값은 올라가 있는데 화면은 그대로인 상황이 생김
화면에 보여줘야 하는 값이면 무조건 useState 써야 함

렌더링 중에 ref 읽거나 쓰는 것도 안 됨
→ 이벤트 핸들러나 useEffect 안에서만 접근할 것

---

## 심화: forwardRef

일반 컴포넌트는 ref를 props로 못 받음
자식 컴포넌트의 DOM에 부모가 접근하고 싶으면 forwardRef로 감싸야 함

```jsx
const FancyInput = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// 부모에서
const inputRef = useRef(null);
<FancyInput ref={inputRef} />
inputRef.current.focus();
```

### useImperativeHandle

DOM 전체를 노출하는 게 찜찜할 때 씀
필요한 메서드만 골라서 열어줄 수 있음

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

부모한테 `focus`랑 `clear`만 열어주고 나머지 DOM API는 못 쓰게 막는 거임
캡슐화 유지하면서 필요한 것만 줄 수 있어서 좋음

---

