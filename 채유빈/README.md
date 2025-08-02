## 비동기와 Promise

- 비동기 처리: 특정 작업이 완료될 때까지 **기다리지 않고** 다음 작업을 수행하는 방식
- 비동기 처리 진화 과정
    - 콜백 함수로 비동기 처리 → 콜백 지옥→ Promise 등장 → Promise 지옥 → Promise를 더욱 간단하게 작성할 수 있는 async, await

<br/>

## Promise

```jsx
const myPromise = new Promise((resolve, reject) => {
	// 비동기 작업 수행
	const data = fetch('url');
	
	if (data) {
		resolve(data);  // 요청이 성공하여 데이터가 있을 경우 resolve 호출
	} else {
		reject("Error");  // 요청이 실패하여 데이터가 없을 경우 reject 호출
	}
});
```

<br/>

### Promise의 상태와 핸들러

| 상태 | 설명 |
| --- | --- |
| `Pending` | 프로미스가 생성되었지만 아직 결과가 결정되지 않은 초기 상태 |
| `Fulfilled` | 비동기 작업이 성공적으로 완료되어 `resolve`가 호출된 상태 |
| `Rejected` | 비동기 작업이 실패하여 `reject`가 호출된 상태 |

| 메서드 | 동작 시점 | 설명 |
| --- | --- | --- |
| `.then()` | `Fulfilled` 상태일 때 | 성공 시 실행할 콜백 함수를 등록하고, **새로운 프로미스**를 반환 |
| `.catch()` | `Rejected` 상태일 때 | 실패 시 실행할 콜백 함수를 등록하고, **새로운 프로미스**를 반환 |
| `.finally()` | `Fulfilled` 또는 `Rejected` 상태일 때 | 성공/실패와 상관없이 항상 실행되는 콜백을 등록하고, **새로운 프로미스**를 반환 |

```jsx
new Promise((resolve, reject) => {
  // 비동기 작업 수행
  if (success) resolve("성공");
  else reject("실패");
})
.then(result => {
  console.log("이행됨:", result);
})
.catch(error => {
  console.log("거부됨:", error);
})
.finally(() => {
  console.log("작업 종료");
});
```

<br/>

### Promise Chaining

- 여러 비동기 작업을 순차적으로 수행하기 위해 프로미스를 연결하는 방법
    
    ```jsx
    function doSomething() {
      return new Promise((resolve) => {
        resolve(100);
      });
    }
    
    doSomething()
      .then((value1) => {
        const data1 = value1 + 50;
        return data1; // 150 반환
      })
      .then((value2) => {
        const data2 = value2 + 50;
        return data2; // 200 반환
      })
      .then((value3) => {
        console.log(value3); // 최종 값 200 출력
      });
    ```
    
- `catch` 이후 `then` 핸들러 사용하기
    - **`catch` 핸들러에서 오류가 처리되면, 이후 `then` 핸들러들이 정상적으로 실행된다.**
    
    ```jsx
    function fetchUserProfile() {
      return new Promise((resolve, reject) => {
        // 서버 요청 실패 시나리오 가정
        setTimeout(() => reject("서버 오류: 사용자 정보를 가져오지 못했어요"), 1000);
      });
    }
    
    function renderProfile(profile) {
      console.log("👤 사용자 정보:", profile);
    }
    
    fetchUserProfile()
      .catch(error => {
        console.warn("⚠️ 오류 발생:", error);
        
        **// 기본 사용자 정보로 복구
        return {
          name: "게스트",
          age: null,
          isGuest: true
        };**
      })
      .then(profile => {
        // 오류 복구 후에도 정상적으로 사용자 정보 렌더링 가능
        renderProfile(profile);
      });
    ```
    
    ```jsx
    // 실행 결과
    ⚠️ 오류 발생: 서버 오류: 사용자 정보를 가져오지 못했어요
    👤 사용자 정보: { name: "게스트", age: null, isGuest: true }
    ```
    

<br/>

### Promise 정적 메소드

- `Promise.resolve()`: 즉시 이행된 프로미스를 반환
- `Promise.reject()`: 즉시 거부된 프로미스를 반환
- `Promise.all()`: 모든 프로미스가 이행될 때까지 기다렸다가, 결과를 배열로 반환
    - 프로미스들을 병렬로 처리해야 할 때 유용
    - 반환된 결과는 구조 분해 할당 가능
    - 프로미스 중 하나라도 실패하면 전체가 실패로 처리됨
- `Promise.allSettled()`: 모든 프로미스가 처리된 후, 각각의 프로미스 상태와 값을 배열로 반환
    - `Promise.all`과 달리 모든 프로미스의 성공 또는 실패를 개별적으로 확인 가능
- `Promise.any()`: 주어진 프로미스 중 하나라도 이행되면 결과를 반환
    - 모든 프로미스가 거부되면 `AggregateError` 반환
    - `AggregateError`에는 각 프로미스의 실패 이유들이 담겨있음
- `Promise.race()`: 가장 먼저 처리된(성공이든 실패든) 프로미스 결과 반환

<br/>

### 📌 Promise.race()가 유용한 경우

- **타임아웃 처리**
    - API 응답이 너무 느릴 때, 일정 시간 후에 실패 처리하고 싶을 때 유용
    
    ```jsx
    function fetchWithTimeout(promise, timeoutMs) {
      const timeout = new Promise((_, reject) =>
        setTimeout(() => reject(new Error("⏰ 요청 시간 초과")), timeoutMs)
      );
    
      return Promise.race([promise, timeout]);
    }
    
    // 사용 예시
    fetchWithTimeout(fetch("/api/data"), 3000)
      .then(res => res.json())
      .then(data => console.log("✅ 데이터:", data))
      .catch(err => console.error("❌ 에러:", err.message));
    ```
    
- **여러 서버 중 빠른 응답만 사용하고 싶은 경우**
    - CDN 또는 미러 서버 중 빠른 응답 하나만 쓰고 나머지는 무시하고 싶을 때
    
    ```jsx
    const fastMirror = Promise.race([
      fetch("https://mirror1.example.com/resource"),
      fetch("https://mirror2.example.com/resource"),
      fetch("https://mirror3.example.com/resource"),
    ]);
    
    fastMirror
      .then(res => res.text())
      .then(data => console.log("📦 가장 빠른 서버 응답:", data))
      .catch(err => console.error("❌ 실패:", err));
    ```
    
- **긴급 취소 로직 (abort-like 패턴)**
    - 예) 긴 작업을 진행하는 도중, 사용자가 ‘취소’ 버튼을 눌렀다면 즉시 작업을 무시하고 취소로 간주하는 시나리오
    - 네트워크 요청 도중 취소 버튼을 눌렀을 때, 폼 제출 도중 사용자가 취소 버튼을 눌렀을 때, 게임 매칭 중 사용자가 나가기를 눌렀을 때
    
    ```jsx
    const userAction = new Promise(resolve =>
      document.querySelector("#cancel-btn")
        .addEventListener("click", () => resolve("사용자 취소"))
    );
    
    const longOperation = new Promise(resolve =>
      setTimeout(() => resolve("파일 업로드 완료"), 5000)
    );
    
    Promise.race([userAction, longOperation])
      .then(result => {
        if (result === "사용자 취소") {
          console.warn("❌ 사용자에 의해 작업이 중단되었습니다.");
        } else {
          console.log("✅ 작업 완료:", result);
        }
      });
    ```
    
    - `Promise.race()`는 누가 먼저 응답하느냐만 결정할 뿐, 느린 프로미스는 여전히 백그라운드에서 진행된다. 따라서 작업을 진짜로 중단하려면 `AbortController`나 명시적 cancel 로직이 필요하다.

<br/>

### 📌 AbortController

- Fetch, 타이머, 기타 **비동기 작업을 중간에 취소할 수 있도록 해주는 컨트롤러 객체**
- `controller.abort()`를 호출하면, 해당 controller가 연결된 작업이 중단된다.

```jsx
const controller = new AbortController();
const signal = controller.signal;

fetch("https://example.com/data", { signal })  // 요청이 해당 signal을 따르게 됨
  .then(res => res.json())
  .then(data => console.log("✅ 응답 받음:", data))
  .catch(err => {
    if (err.name === "AbortError") {
      console.warn("❌ 요청이 중단되었습니다.");
    } else {
      console.error("💥 다른 오류:", err);
    }
 });

// 3초 뒤 요청 강제 중단
setTimeout(() => {
  controller.abort(); // 여기서 fetch 요청을 중단함!
}, 3000);
```

> `axios`는 `AbortController`를 공식 지원하며, `setTimeout`은 지원하지 않아 수동으로 중단할 수 있는 형태로 감싸 사용할 수 있다.
> 

<br/>

## async & await

- 프로미스 지옥을 해결할 수 있는 방법
    - 프로미스 지옥: `then()` 메서드가 과도하게 체인되어 코드 가독성이 떨어지고, 중간 에러 처리가 어려운 형태
- `async` 함수 내에서 `await`는 **프로미스가 처리될 때까지 함수 실행을 일시 중지**한다.
    - `async`: 함수 선언 앞에 붙여 비동기 함수로 만듦
    - `await`: 비동기 처리 앞에 붙여서 해당 처리가 완료될 때까지 기다린다.
    
    ```jsx
    async function 함수명() {
    	let 결과 = await 프로미스_반환_함수();
    	// 프로미스가 처리된 후 실행될 코드
    }
    ```
    
- 예외 처리를 위해 `try ... catch` 문을 사용할 수 있다.
    
    ```jsx
    async function logTodoTitle() {
    	try {
    		let user = await fetchUser();
    		if (user.id === 1) {
    			let todo = await fetchTodo();
    			console.log(todo.title);
    		}
    	} catch (error) {
    		console.log(error);  // 오류 처리
    	}
    }
    ```
    

<br/>

## 이벤트 루프

https://github.com/FESIStudy/W17_JavaScript/tree/main/%EC%B1%84%EC%9C%A0%EB%B9%88#%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84event-loop

<br/>

### 이벤트 루프 실행 순서 문제 (한번 풀어보세요😆)

```jsx
async function foo() {
  console.log('1');
  await null;
  console.log('2');
}

console.log('3');
foo();
console.log('4');

// 3 -> 1 -> 4 -> 2
```

```jsx
setTimeout(() => {
  console.log('timeout1');

  Promise.resolve().then(() => {
    console.log('promise in timeout');
  });
}, 0);

Promise.resolve().then(() => {
  console.log('promise1');
});

setTimeout(() => {
  console.log('timeout2');
}, 0);

// promise1 -> timeout1 -> promise in timeout -> timeout2
```

```jsx
console.log('start');

setTimeout(() => {
  console.log('timeout');
}, 0);

Promise.resolve().then(() => {
  console.log('promise1');
}).then(() => {
  console.log('promise2');
});

queueMicrotask(() => {
  console.log('microtask');
});

console.log('end');

// start -> end -> promise1 -> microtask -> promise2 -> timeout
```

<br/>

### 📌 queueMicrotask

- 마이크로태스크 큐에 직접 콜백 함수를 추가하는 API
- 프로미스보다 더 직접적이고 간단한 방식으로 마이크로태스크를 스케줄링할 수 있는 저수준 API
    
    ```jsx
    queueMicrotask(() => {
    	console.log('마이크로태스크 실행');
    });
    ```
    
- 용도
    - 프로미스 체이닝과 동일한 우선순위로 작업 실행
    - 렌더링 전에 추가 코드 실행 (DOM 프로퍼티 변경 직후, 실제 화면 렌더링 전)
        
        ```jsx
        const box = document.getElementById('box');
        
        // DOM 변경
        box.style.width = '200px';
        
        // 변경 직후, 마이크로태스크에 작업 예약
        queueMicrotask(() => {
          // DOM 변경 후 바로 호출되어 스타일 계산 가능
          const width = box.offsetWidth;
          console.log('Updated width:', width);
        });
        ```
        
    - 비동기 작업의 세밀한 제어
