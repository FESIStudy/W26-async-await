## 1. 자바스크립트 엔진의 구조

**자바스크립트 엔진은 단순히 힙 메모리와 콜스택으로 구성되어 있다.** 콜스택에 쌓여있는 요청들을 순차적으로 처리할 뿐이다.

```javascript
function first() {
  console.log('첫 번째');
}

function second() {
  first();
  console.log('두 번째');
}

second();
// 콜스택: second() -> first() -> console.log() 순으로 실행
```

그런데 여기서 중요한 질문이 생긴다. **그럼 비동기 작업 스케줄링은 누가 담당하는가?**

답은 간단하다. **소스코드 평가, 실행을 제외한 모든 처리는 브라우저나 Node.js가 담당한다.** 스케줄링, 콜백함수 등록 등 모든 비동기 관련 작업 말이다.

## 2. 이벤트 루프의 동작 원리

이벤트 루프를 이해하려면 먼저 태스크 큐의 구조를 알아야 한다.

### 태스크 큐의 구분

! **태스크 큐에 들어간 건 콜스택이 비워져야 실행된다**

태스크 큐에도 **마이크로 태스크 큐**와 **매크로 태스크 큐**로 분리되는데, 이때는 **마이크로가 전부 다 실행되어야 나머지가 실행된다**

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0); // 매크로 태스크

Promise.resolve().then(() => console.log('3')); // 마이크로 태스크

console.log('4');

// 출력 순서: 1 -> 4 -> 3 -> 2
```

### 마이크로 태스크 vs 매크로 태스크

**마이크로 태스크:**

- Promise.then/catch/finally
- queueMicrotask()
- MutationObserver

**매크로 태스크:**

- setTimeout/setInterval
- setImmediate (Node.js)
- I/O 작업
- UI 렌더링

실행 우선순위를 보면 이렇다:

1. 콜스택의 모든 동기 코드 실행
2. 마이크로 태스크 큐의 **모든** 작업 실행
3. 매크로 태스크 큐에서 **하나씩** 실행
4. 2-3 반복

```javascript
setTimeout(() => console.log('매크로 1'), 0);

Promise.resolve()
  .then(() => {
    console.log('마이크로 1');
    return Promise.resolve();
  })
  .then(() => console.log('마이크로 2'));

setTimeout(() => console.log('매크로 2'), 0);

Promise.resolve().then(() => console.log('마이크로 3'));

// 출력: 마이크로 1 -> 마이크로 2 -> 마이크로 3 -> 매크로 1 -> 매크로 2
```

이 차이를 모르면 코드 실행 순서를 예측하기 어렵다. 특히 Promise 체이닝과 setTimeout이 섞여있을 때 말이다.

이를 통해서 알아야할 것.
자바스크립트는 싱글스레드로 동작하지만 브라우저는 멀티스레드임.
자바스크립트 엔진은 싱글스레드 <-> 브라우저는 멀티스레드(web api나 이벤트 루프를 통한 작업 스케쥴링)

```js
function foo() {
  console.log('foo');
}

function bar() {
  console.log('bar');
}

setTimeout(foo, 0);
bar();
```

실행 순서를 살펴보자

1. 전역 코드 평가, 전역 실행 컨텍스트 생성, 콜스택에 푸시
2. 전역 코드가 실행되며 setTimeout이 호출됨.이떄 setTimeout의 함수 실행 컨텍스트가 생성 + 콜 스택에 푸시되며 현재 실행중인 실행 컨택스트가 됨
3. setTimeout이 실행되면 콜백함수(`foo`)를 호출 스케줄링하고 콜 스택에서 팝됨.
   - 이때 호출 스케줄링, 타이머 설정 및 타이머 만료 후 콜백함수를 콜 `태스트 큐`에 밀어넣는건 브라우저가
   - 여기서 대기 시간이 0ms로 해놨지만 자체적으로 4ms미만은 고정적으로 4ms의 대기시간을 가지도록 하게함.
   - 또 알아야할게 내가 40ms로 설정해놨다고 해서 정확하게 40ms뒤에 시작되는건 아님
     - 왜? 태스크 큐에 쌓인 콜백함수는 콜스택에 쌓인 함수가 다 처리된 이후에야 이벤트루프 이놈이 옮겨주거든..
4. bar함수 콜스택에 들어감, 실행, 그 후 콜 스택에서 팝됨.
5. 이제 콜 스택엔 암것도 없음. 이제 이벤트 루프가 `foo`함수를 콜 스택으로 옮겨주고 foo가 실행되고 끝.

## 3. Promise 고급 API 활용하기

Promise의 기본적인 then/catch는 다들 알고 있을 테니, 실무에서 정말 유용한 고급 API들을 살펴보자.

### Promise.any()

여러 Promise 중 **가장 먼저 성공하는 하나**만 기다린다. 모든 Promise가 실패해야 AggregateError를 던진다.

```javascript
const fastAPI = fetch('/api/fast').then((res) => res.json());
const slowAPI = fetch('/api/slow').then((res) => res.json());
const backupAPI = fetch('/api/backup').then((res) => res.json());

try {
  const result = await Promise.any([fastAPI, slowAPI, backupAPI]);
  console.log('가장 빠른 응답:', result);
} catch (error) {
  console.log('모든 API 호출 실패:', error);
}
```

이건 API 서버가 여러 개 있을 때 가장 빠른 응답을 받고 싶을 때 정말 유용하다. 예를 들어 CDN이 여러 지역에 분산되어 있거나, 백업 서버가 있을 때 말이다.

### Promise.allSettled()

Promise.all()과 달리, **모든 Promise가 완료될 때까지 기다리되, 실패해도 멈추지 않는다.**

```javascript
const promises = [
  fetch('/api/user'),
  fetch('/api/posts'),
  fetch('/api/comments'), // 이게 실패해도 다른 건 계속 진행
];

const results = await Promise.allSettled(promises);

results.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    console.log(`API ${index} 성공:`, result.value);
  } else {
    console.log(`API ${index} 실패:`, result.reason);
  }
});
```

여러 API를 동시에 호출할 때 하나가 실패해도 나머지는 처리하고 싶은 경우에는 이걸 써먹어야한다

## 4. 실제 성능 최적화 사례

만양 비동기 작업을 한번에 1000개를 동시에(최적화의 느낌으로)해야한다. 이럴때 메모리 터질 확률 매우 높음

그래서 동시 실행 개수를 제한해야 했는데, 직접 구현하려니 복잡했다. 그때 발견한 게 `@supercharge/promise-pool`이다.

## 5. @supercharge/promise-pool로 동시성 제어하기

### 설치 및 기본 사용법

```bash
npm install @supercharge/promise-pool
```

```javascript
import { PromisePool } from '@supercharge/promise-pool';

const images = ['image1.jpg', 'image2.jpg' /* ... 1000개 */];

const { results, errors } = await PromisePool.withConcurrency(5) // 최대 5개만 동시 실행
  .for(images)
  .process(async (imagePath) => {
    return await resizeImage(imagePath);
  });

console.log(`성공: ${results.length}, 실패: ${errors.length}`);
```

이렇게 하니까 시스템이 안정적으로 동작했다. 5개씩 끊어서 처리하니까 메모리도 절약되고, 전체 처리 시간도 단순 순차 처리보다 훨씬 빨랐다.

### 고급 활용 사례

실무에서는 더 복잡한 시나리오들이 많다. 예를 들어 API 호출 제한이 있거나, 에러 처리를 세밀하게 해야 하는 경우 말이다.

```javascript
// API 호출 제한 및 재시도 로직
const { results, errors } = await PromisePool.withConcurrency(3) // API 제한으로 3개만 동시 호출
  .for(userIds)
  .withTaskTimeout(5000) // 5초 타임아웃
  .process(async (userId, index, pool) => {
    try {
      const user = await fetchUserData(userId);

      // 진행률 표시
      console.log(`처리 중: ${index + 1}/${userIds.length}`);

      return user;
    } catch (error) {
      if (error.status === 429) {
        // Rate limit
        console.log('API 제한 도달, 잠시 대기...');
        await new Promise((resolve) => setTimeout(resolve, 1000));
        throw error; // 재시도를 위해 에러 다시 던지기
      }
      throw error;
    }
  });

// 에러 처리
if (errors.length > 0) {
  console.log(
    '실패한 요청들:',
    errors.map((e) => e.item),
  );

  // 실패한 항목들 재시도
  const retryResults = await PromisePool.withConcurrency(1) // 재시도는 더 보수적으로
    .for(errors.map((e) => e.item))
    .process(retryFetchUserData);
}
```

### 실시간 진행률 표시

대용량 작업을 할 때는 사용자에게 진행률을 보여주는 것이 중요하다:

```javascript
let completed = 0;
const total = images.length;

const { results } = await PromisePool.withConcurrency(10)
  .for(images)
  .process(async (image) => {
    const result = await processImage(image);

    completed++;
    const progress = Math.round((completed / total) * 100);
    console.log(`진행률: ${progress}% (${completed}/${total})`);

    // 또는 웹에서는 프로그레스바 업데이트
    // updateProgressBar(progress);

    return result;
  });
```

## 6. 실무 팁과 주의사항

### 메모리 관리

대용량 작업을 할 때는 메모리 사용량도 고려해야 한다:

```javascript
// 큰 파일들을 처리할 때는 동시성을 낮춰야 함
const concurrency = process.env.NODE_ENV === 'production' ? 2 : 5;

const { results } = await PromisePool.withConcurrency(concurrency)
  .for(largeFiles)
  .process(async (file) => {
    const processed = await processLargeFile(file);

    // 메모리 정리를 위해 명시적으로 null 할당
    file = null;

    return processed;
  });
```

### 에러 전파 방지

Promise.all()과 달리 PromisePool은 일부 작업이 실패해도 나머지는 계속 진행한다. 하지만 때로는 치명적인 에러가 발생하면 전체를 중단해야 할 수도 있다:

```javascript
let shouldStop = false;

const { results, errors } = await PromisePool.withConcurrency(5)
  .for(tasks)
  .process(async (task) => {
    if (shouldStop) {
      throw new Error('작업 중단됨');
    }

    try {
      return await processTask(task);
    } catch (error) {
      if (error.code === 'CRITICAL_ERROR') {
        shouldStop = true;
      }
      throw error;
    }
  });
```

## 실행 컨텍스트 - 이건 지피티 선생님

실행 컨텍스트는 자바스크립트 엔진이 코드를 실행할 때 만드는 **환경**이야.

쉽게 말하면 "이 코드가 실행되는 공간"이라고 생각하면 돼.

## 실행 컨텍스트의 구성

실행 컨텍스트는 3가지로 구성되어 있어:

```javascript
// 실행 컨텍스트 = {
//   1. 변수 환경 (Variable Environment)
//   2. 렉시컬 환경 (Lexical Environment)
//   3. this 바인딩
// }
```

## 실제 동작 예시

```javascript
function outer() {
  var a = 10;

  function inner() {
    var b = 20;
    console.log(a + b); // 30
  }

  inner();
}

outer();
```

**실행 과정:**

```
1. 전역 실행 컨텍스트 생성
   콜스택: [전역 컨텍스트]

2. outer() 호출 → outer 실행 컨텍스트 생성
   콜스택: [전역 컨텍스트][outer 컨텍스트]
   - 변수: { a: 10, inner: function }
   - 스코프: 전역 스코프 참조

3. inner() 호출 → inner 실행 컨텍스트 생성
   콜스택: [전역 컨텍스트][outer 컨텍스트][inner 컨텍스트]
   - 변수: { b: 20 }
   - 스코프: outer 스코프 참조 (여기서 a에 접근 가능!)

4. inner() 완료 → inner 컨텍스트 제거
   콜스택: [전역 컨텍스트][outer 컨텍스트]

5. outer() 완료 → outer 컨텍스트 제거
   콜스택: [전역 컨텍스트]
```

## 핵심 포인트

**1. 각 함수 호출마다 새로운 실행 컨텍스트가 생성돼**

```javascript
function sayHello(name) {
  var greeting = '안녕 '; // 이 변수는 이 실행 컨텍스트에만 존재
  return greeting + name;
}

sayHello('철수'); // 새로운 실행 컨텍스트 생성
sayHello('영희'); // 또 다른 실행 컨텍스트 생성 (위와 완전히 별개)
```

**2. 스코프 체인으로 변수에 접근**

```javascript
var global = '전역변수';

function level1() {
  var local1 = '레벨1';

  function level2() {
    var local2 = '레벨2';

    // 여기서 접근 가능한 변수들:
    console.log(local2); // 현재 컨텍스트
    console.log(local1); // 상위 컨텍스트 (level1)
    console.log(global); // 전역 컨텍스트
  }

  level2();
}
```

**3. 호이스팅이 일어나는 이유**

```javascript
console.log(x); // undefined (에러 아님!)
var x = 5;

// 실행 컨텍스트 생성 단계에서:
// 1. 변수 선언부를 먼저 처리 (var x; 부분)
// 2. 그 다음에 코드 실행 (x = 5; 부분)
```

그래서 실행 컨텍스트는 **"이 함수가 실행될 때 필요한 모든 정보를 담은 상자"**라고 생각하면 돼. 변수들, 함수들, 스코프 정보 등등 말이야.
