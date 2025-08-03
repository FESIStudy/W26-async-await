https://wikidocs.net/251896 해당 내용을 읽으며 정리한 내용입니다.

## 9. async & await

- 자바스크립트의 비동기 처리 문법
- 코드를 동기적으로 작성한 것처럼 보이게 한다.
- 사용법
    - 함수 앞에 async 키워드 사용
    - await 키워드는 async 함수 내부에서만 사용 가능
    - await 키워드는 프로미스가 처리될 때까지 함수 실행을 일시 중지
    
    ```jsx
       async function 함수명() {
           let 결과 = await 프로미스_반환_함수();
           // 프로미스가 처리된 후 실행될 코드
       }
    ```
    
- 예외 처리
    - 비동기 함수 내에서 예외를 처리하기 위해 try … catch 블록 사용 가능
    
    ```jsx
    async function logTodoTitle() {
        try {
            var user = await fetchUser();
            if (user.id === 1) {
                var todo = await fetchTodo();
                console.log(todo.title); // 결과 출력
            }
        } catch (error) {
            console.log(error); // 오류 처리
        }
    }
    ```
    

⇒ async / await 를 이용하여 비동기 코드를 동기 코드 처럼 쉽게 작성할 수 있어 가독성과 유지보수성이 높아진다.

## 10. 자바스크립트 비동기 처리와 Promise

- 비동기 처리란?
    - 특정 작업이 완료될 때까지 기다리지 않고 다음 작업을 수행하는 방식.
    - 주로 서버와의 통신, 타이머 함수 등을 비동기 방식으로 처리
- 콜백 함수로 비동기 처리 해결 → 콜백 지옥 → Promise로 해결 → async/await로 더 간단하게 작성
    
    

## 11. Promise

```jsx
 function incrementAndLog(number, callback) {
  setTimeout(() => {
    const newNumber = number + 1;
    console.log(newNumber); // 증가된 숫자를 출력
    if (callback) {
      callback(newNumber); // 콜백 함수 호출
    }
  }, 1000); // 1초 지연
}
// 0부터 시작하여 1씩 증가
incrementAndLog(0, num1 => {
  incrementAndLog(num1, num2 => {
    incrementAndLog(num2, num3 => {
      incrementAndLog(num3, num4 => {
        incrementAndLog(num4, num5 => {
          console.log('끝!'); // 최종 출력
        });
      });
    });
  });
});
```

이런 콜백 지옥을

```jsx
function incrementAsync(value) {
  return new Promise((resolve) => {
    setTimeout(() => {
      const newValue = value + 1;
      console.log(newValue); // 증가된 숫자를 출력
      resolve(newValue); // Promise 해결
    }, 1000); // 1초 지연
  });
}

// 0부터 시작하여 1씩 증가
incrementAsync(0)
  .then(incrementAsync) // 첫 번째 증가
  .then(incrementAsync) // 두 번째 증가
  .then(incrementAsync) // 세 번째 증가
  .then(incrementAsync) // 네 번째 증가
  .then(() => {
    console.log('끝!'); // 최종 출력
  });
```

이렇게 사용 가능

### 프로미스 객체 기본 사용법

1. 프로미스 객체 생성
    
    ```jsx
    const myPromise = new Promise((resolve, reject) => {
      // 비동기 작업 수행: 서버로부터 데이터를 요청
      const data = fetch('서버로부터 요청할 URL');
    
      if (data) {
          resolve(data); // 요청이 성공하여 데이터가 있을 경우 resolve 호출
      } else {
          reject("Error"); // 요청이 실패하여 데이터가 없을 경우 reject 호출
      }
    });
    ```
    
    - `new Promise` 생성자를 사용하여 프로미스 객체를 생성.
    - 두 개의 매개변수 `resolve`와 `reject`를 가진 콜백 함수 포함.
    - `resolve`는 작업이 성공했음을, `reject`는 작업이 실패했음을 나타냄.
2. 프로미스 객체 처리
    
    ```jsx
    myPromise
      .then((value) => { // 프로미스가 이행되었을 때 실행될 코드
        console.log("Data: ", value); // 성공적으로 반환된 데이터 출력
      })
      .catch((error) => { // 프로미스가 거부되었을 때 실행될 코드
        console.error(error); // 오류 메시지 출력
      })
      .finally(() => { // 성공 또는 실패 여부와 상관없이 항상 실행될 코드
        console.log('작업 완료'); // 작업 완료 메시지 출력
      });
    
    ```
    
    - `.then()`: 프로미스가 성공했을 때 실행될 콜백 함수 정의.
    - `.catch()`: 프로미스가 실패했을 때 실행될 콜백 함수 정의.
    - `.finally()`: 프로미스의 성공/실패와 관계없이 항상 실행될 코드 정의.
3. 프로미스 함수 등록
    
    ```jsx
    functionfetchData() {
    returnnewPromise((resolve, reject) => {
            // 서버로부터 데이터를 요청
    const data =fetch('서버로부터 요청할 URL');
    if (data) {
    resolve(data); // 데이터가 있으면 resolve 호출
            }else {
    reject("Error"); // 데이터가 없으면 reject 호출
            }
        });
    }
    
    fetchData()
        .then((result) => { // 프로미스가 성공했을 때 실행될 코드
            console.log(result); // 성공적으로 반환된 데이터 출력
        })
        .catch((error) => { // 프로미스가 실패했을 때 실행될 코드
            console.error(error); // 오류 메시지 출력
        });
    
    ```
    
    - 프로미스 객체를 반환하는 함수를 정의하여 코드 가독성과 재사용성을 높힌다.
    - 함수 호출 시 프로미스 객체 반환.
4. 프로미스 상태
- `Pending` 상태: 프로미스가 생성되고 아직 완료되지 않은 상태.
- `Fulfilled` 상태: 프로미스가 성공적으로 완료된 상태로, `resolve`가 호출됨.
- `Rejected` 상태: 프로미스가 실패한 상태로, `reject`가 호출됨.

## 11-5. 프로미스 체이닝

- 여러 비동기 작업을 순차적으로 수행하기 위해 프로미스를 연달아 연결하는 방법
- then 핸들러는 이전 프로미스의 반환 값을 받아 새로운 값을 반환한다.
- 반환된 값은 자동으로 **프로미스 객체로 감싸져** 다음 then 핸들러에서 사용된다.

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
    const data3 = value3 + 50;
    return data3; // 250 반환
  })
  .then((value4) => {
    console.log(value4); // 최종 값 250 출력
  });
```

## 11-6. 프로미스 정적 메서드

1. Promise.resolve(): 즉시 이행된 프로미스를 반환
2. Promise.reject(): 즉시 거부된 프로미스를 반환
3. Promise.all(): 모든 프로미스가 이행될 때까지 기다렸다가, 결과를 배열로 반환
4. Promise.allSettled(): 주어진 모든 프로미스가 처리된 후, 각각의 프로미스 상태와 값을 배열로 반환.
5. Promise.any(): 주어진 프로미스 중 하나라도 완료되면 결과를 반환.  모든 프로미스가 거부되면 `AggregateError`를 반환
6. Promise.race(): 가장 먼저 처리된 프로미스의 결과를 반환. 이행 여부와 관계없이 첫번째로 완료된 프로미스가 반환된다.

## 12-1. 자바스크립트 엔진 구동 환경

### 브라우저 내부 구성도

![image.png](attachment:bbadba19-7c4c-4daa-ae73-637408f78dfb:image.png)

- **Call Stack**: 자바스크립트 엔진이 코드 실행을 위해 사용하는 메모리 구조.
- **Heap**: 동적으로 생성된 자바스크립트 객체가 저장되는 공간.
- **Web APIs**: 비동기 작업을 처리하는 브라우저 API 모음 (AJAX 호출, 타이머 함수, DOM 조작 등).
- **Callback Queue**: 비동기 작업이 완료되면 실행되는 함수들이 대기하는 공간.
- **Event Loop**: 비동기 함수들을 적절한 시점에 실행시키는 관리자.
- **Event Table**: 특정 이벤트 발생 시 어떤 콜백 함수가 호출되는지를 알고 있는 자료구조.

### Web APIs 구분

**동기적 API**

- **DOM**: HTML 문서의 구조와 내용을 표현하고 조작할 수 있는 객체
- **Console API**: 개발자 도구에서 콘솔 기능을 제공

**비동기적 API**

- **XMLHttpRequest**: 서버와 비동기적으로 데이터를 교환할 수 있는 객체. AJAX 기술의 핵심
- **Timer API**: 일정한 시간 간격으로 함수를 실행하거나 지연시키는 메소드들을 제공
- **Canvas API**: `<canvas>` 요소를 통해 그래픽을 그리거나 애니메이션을 만들 수 있는 메소드들을 제공
- **Geolocation API**: 웹 브라우저에서 사용자의 현재 위치 정보를 얻을 수 있는 메소드들을 제공

### Callback Queue의 종류

**Task Queue (macrotask queue)**

- **역할**: 비동기적으로 처리되는 함수들의 콜백 함수가 들어가는 큐
- **예시**: `setTimeout`, `setInterval`, `fetch`, `addEventListener`

**Microtask Queue**

- **역할**: 우선적으로 비동기 처리되는 함수들의 콜백 함수가 들어가는 큐
- **예시**: `promise.then`, `process.nextTick`, `MutationObserver`

→ 우선순위: microtask queue > task queue

## 12-2. Event loop의 동작 과정

1. **동기 작업 실행**: 콜 스택에 쌓인 동기 작업을 순차적으로 실행합니다.
2. **비동기 작업 처리**: 비동기 함수 호출 시, 해당 작업은 웹 API에서 처리되고, 완료된 콜백 함수는 콜백 큐에 적재됩니다.
3. **이벤트 루프 작동**: 이벤트 루프는 콜 스택이 비어 있는지 확인한 후, 비어 있다면 콜백 큐에서 대기 중인 콜백 함수를 콜 스택으로 옮겨 실행합니다.
4. **마이크로태스크 우선 처리**: 이벤트 루프는 태스크 큐보다 마이크로태스크 큐를 우선적으로 처리합니다. 마이크로태스크 큐가 비워진 후에 태스크 큐를 처리합니다.
