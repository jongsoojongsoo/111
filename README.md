# async/await는 결국 Promise다!

async/await 처음 배울 때 "아 이제 비동기 끝났다" 싶었는데
쓰면 쓸수록 이상한 상황이 자꾸 생김.

- await 붙였는데 왜 에러가 안 잡히지?
- 분명히 기다렸는데 왜 undefined가 나오지?
- 왜 이게 느리지?

알고보니 async/await가 완전히 새로운 개념이 아니라
**Promise 위에 올라탄 문법 설탕**이라는 걸 알게 됨
그걸 모르고 쓰니까 계속 함정에 빠진 것

---

## 1. JavaScript가 왜 비동기인가

JavaScript는 싱글 스레드 언어 → 한 번에 하나의 작업만 처리 가능

만약 서버 응답 올 때까지 그냥 기다리면?
→ 버튼도 못 누르고 스크롤도 안 되고 화면이 얼어버림

그래서 나온 게 비동기
> "일단 요청 보내놓고 다른 거 하다가 응답 오면 그때 처리해"

---

## 2. 콜백 → Promise → async/await 흐름

### 콜백 (처음엔 이랬음)

요청하고 응답 왔을 때 실행할 함수를 미리 넘겨두는 방식

```js
getData(function(a) {
  getMoreData(a, function(b) {
    getEvenMoreData(b, function(c) {
      console.log(c); // 콜백 지옥
    });
  });
});
```

요청이 중첩될수록 코드가 오른쪽으로 밀려남
읽기도 힘들고 에러 처리도 제각각 → 유지보수 불가 수준

---

### Promise (콜백 지옥 해결)

"작업이 끝나면 이 객체를 통해 결과 알려줄게" 라는 약속을 객체로 만든 것

```js
fetch('/api/data')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

**Promise의 세 가지 상태**

| 상태 | 의미 |
|---|---|
| pending | 아직 결과 모름 (대기 중) |
| fulfilled | 성공 |
| rejected | 실패 |

콜백 지옥은 해결됐는데
then().then().then() 체인이 길어지면 또 읽기 불편해짐

---

### async/await (Promise를 더 읽기 좋게)

Promise를 동기 코드처럼 읽히게 해주는 문법

```js
async function getData() {
  const res = await fetch('/api/data');
  const data = await res.json();
  console.log(data);
}
```

await 앞에서 잠깐 멈추고 결과 오면 다음 줄로 넘어감
위에서 아래로 읽히니까 훨씬 직관적

근데 여기서 착각 시작

---

## 3. 핵심 — async/await는 동기가 아님

await가 "멈춘다"는 표현을 쓰지만
JavaScript 전체가 멈추는 게 아님

```
await를 만나면
→ 해당 함수의 실행만 잠깐 중단
→ JS 엔진은 다른 작업 하러 감
→ 결과 오면 그때 돌아와서 다음 줄 실행
```

즉 await는 **"나 여기서 기다릴게, 그동안 다른 거 해"**
이게 Promise 동작 방식이랑 완전히 똑같음

async/await는 Promise를 없앤 게 아니라 읽기 좋게 포장한 것

---

## 4. 그래서 생기는 함정들

### 함정 1 — await 순서대로 쓰면 느려짐

```js
// 이렇게 하면 첫 번째 끝나야 두 번째 시작됨 (순차 실행)
const user = await fetchUser();
const posts = await fetchPosts(); // user 안 기다려도 되는데 기다림
```

```js
// 관계없는 요청은 동시에 보내야 함
const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
```

async/await만 알면 이 발상 자체가 안 나올 수 있음
Promise.all을 알아야 함

---

### 함정 2 — try/catch가 항상 에러를 잡는 건 아님

```js
async function bad() {
  try {
    fetchData(); // await 빠뜨림
  } catch (e) {
    console.log('잡힘?'); // 안 잡힘
  }
}
```

await 없이 Promise를 그냥 반환만 하면
try/catch 밖에서 에러가 터짐
잡힐 거라고 생각했는데 안 잡히는 상황 발생

---

### 함정 3 — async 함수는 항상 Promise를 반환함

```js
async function getNumber() {
  return 1;
}

const result = getNumber();
console.log(result); // 1이 아님, Promise { 1 }
```

async 함수에서 값을 return해도
그 값이 그대로 나오는 게 아니라 Promise로 감싸져서 나옴

처음에 이거 몰라서 콘솔에 `Promise {pending}` 보고 한참 헤맸음

```js
// 받는 쪽에서도 await 해줘야 함
const result = await getNumber(); // 이제 1
```

---

## 5. React에서 왜 더 복잡해지냐

useEffect 안에서 async를 바로 못 씀

```js
// 이렇게 하면 안 됨
useEffect(async () => {
  const data = await fetchData();
}, []);
```

왜냐면
- useEffect는 cleanup 함수(또는 undefined)를 반환해야 함
- async 함수는 항상 Promise를 반환함
- Promise랑 cleanup이 충돌

그래서 이렇게 씀

```js
// 이렇게 해야 함
useEffect(() => {
  const getData = async () => {
    const data = await fetchData();
    setData(data);
  };
  getData(); // 선언하고 바로 호출
}, []);
```

이게 왜 이렇게 하는 건지 모르고 복붙만 하다가 나중에 알게 됨

---

## 6. 결국 뭘 알아야 하냐

async/await가 편한 건 맞는데
Promise를 모르고 async/await만 쓰면 에러 상황에서 막힘

Promise가 뭔지, 상태가 어떻게 바뀌는지, 언제 then/catch를 써야 하는지
이걸 알고 async/await를 쓰는 것과 모르고 쓰는 건 차이가 남

> async/await는 Promise를 대체한 게 아니라
> Promise를 더 잘 쓰기 위한 도구임

---

## 생각해볼 것들

- await를 쓰면 "기다리는" 건 맞는데, 정확히 누가 기다리는 거?
- 에러 처리를 try/catch로 하는 게 좋은지, .catch()로 하는 게 좋을지
- 여러 요청 보낼 때 순서가 중요한 경우랑 아닌 경우를 어떻게 구분?
- useEffect에서 async를 직접 못 쓰는 게 불편한데, 왜 못쓰게 함?

