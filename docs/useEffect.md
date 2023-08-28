# useEffect 구조

*`useEffect(() ⇒ {}, [*someProp, someState*])`* - 첫번째 인자로 콜백 함수를 그리고 두번째 인자로 의존성 배열을 받습니다.

# useEffect() 훅은 언제 사용해주면 될까?

- useEffect는 리액트 컴포넌트가 렌더링될 때 마다 특정 작업을 수행하도록 설정할 수 있는 hook입니다.
- 그리고 react 클래스 컴포넌트의 라이프사이클 메서드 일부인 _`componentDidMount`_ 와 _`componentDidUpdate`_ 을 합친 형태로 봐도 무방합니다.
  - `componentDidMount` 메서드
    - 의존성 배열을 빈 배열로 만들어서 대체가능

      ```jsx
      React.useEffect(() => {
        console.log('Wecome to Radixweb!')
      }, [])
      ```

    - ex
      - `props` 로 받은 값을 컴포넌트의 로컬 상태로 설정
      - 외부 API 요청 (REST API 등)
      - 라이브러리 사용 (D3, Video.js 등...)
      - setInterval 을 통한 반복작업 혹은 setTimeout 을 통한 작업 예약
  - _`componentDidUpdate` 메서드_
    - 의존성 배열에 주시하고 싶은 value를 넣어서 대체 가능
      ```jsx
      React.useEffect(() => {
        fetchUser()
      }, [userId])
      ```
  - _`componentWillUnmount` 메서드_
    - 클린 업을 이용해서 컴포넌트가 화면에서 사라질 때를 인식해서 추가 작업 수행 가능
      ```jsx
      React.useEffect(() => {
        console.log('Welcome to Radixweb!')

        return () => {
          console.log('Component unmounts')
        }
      }, [])
      ```
      - ex
      - setInterval, setTimeout 을 사용하여 등록한 작업들 clear 하기 (clearInterval, clearTimeout)
      - 라이브러리 인스턴스 제거

# 🙌  의존성 배열의 형태에 따라 나눠 정리 해 봤어요!

1. 렌더링이 될 때 마다 (즉 컴포넌트가 맨 처음 화면에 그려질때 그리고 컴포넌트가 업데이트 될때) 실행시키기고 싶은 작업이 있다면


    - 두번째 인자로 의존성 배열 없이 콜백 함수만 넣어준다.
      - `useEffect(() ⇒ {})`

2. 컴포넌트가 화면에 맨 처음 렌더링이 될때(디폴트) 그리고 특정값(value)이 바뀔때 마다 실행 시켜 주고 싶은 작업이 있다면
   - 의존성 배열에 주시하고 싶은 특정값을 넣으면 됩니다.
     - `useEffect(() ⇒ {}, [value])`
3. 화면에 처음 렌더링 될 때 **한 번**만 실행시켜 주고 싶은 작업이 있다면
   - 의존성 배열 부분에 [] 빈 배열을 넣어주면 됩니다.
     - `useEffect(() ⇒ {}, [])`

# Clean up

- 만약 useEffect를 이용해서 타이머를 시작했다면 타이머를 끝내주는 작업이 필요하고, 혹은 어떤 EventListener를 등록했다면 등록한 리스너를 정리해 주는 작업이 useEffect 내부에서 필요 합니다.
- 메모리 측면에서도 좋다.
  - Clean up 하는 방법

    _`return () ⇒ { …정리 하고 싶은 내용 }`_

    - 상황1. 언마운트 될 때 한번만 cleanup 함수를 실행하고 싶을 때 두 번째 파라미터로 빈 배열을 넣어주면 됩니다.
    - 상황2. 특정값이 업데이트 되기 직전에 cleanup 함수를 실행하고 싶을 때 deps 배열 안에 검사하고 싶은 값을 넣어줍니다.

    ```jsx
    useEffect(() => {
      // 설정 ...
      return () => {
        // 정리(clean up)
      }
    })
    ```

# useEffect 사용 예시

1. 데이터 불러오기(fetching data)

   - 일반적으로 다음과 같이 axios를 통해 데이터를 가져오고, 그것을 state로 저장하여 활용합니다.

   ```jsx
   export default () => {
     const [data, setData] = useState({ hits: [] })
     useEffect(async () => {
       const result = await axios.get(
         'https://hn.algolia.com/api/v1/search?query=redux'
       )
       setData(result.data)
     }, [])
     // useEffect의 두번째 인자로 빈 배열[]을 주었습니다.
     return (
       <ul>
         {data.hits.map((item) => (
           <li key={item.objectID}>
             <a href={item.url}>{item.title}</a>
           </li>
         ))}
       </ul>
     )
   }
   ```

- 이렇게 하면 `Effect callbacks are synchronous to prevent race conditions. Put the async function inside` 에러가 뜹니다.

  - `effect 콜백 함수는 race condition을 방지하기 위해 동기적이어야 합니다. async를 콜백 함수 안쪽에 넣으십시오.`
  - `race condition`
    - 다중 프로세스 환경에서 **두 개 이상의 프로세스가 동시에 수행** 될 때 발생되는 비정상적인 상태를 의미합니다. 동시 수행은 **실행 결과의 일관성을 보장하기 어렵습니다.** 여러 개의 비동기 함수를 동시에 실행한다면 무슨 정보가 먼저 들어올 지 매 실행마다 달라지기 때문입니다. 이를 방지하고자 async의 위치를 안 쪽으로 넣으라고 한 것입니다.

  ```jsx
  // 예시 1 해결 방법
  // fetchData라는 비동기적 함수를 만들어 줍니다.
  // 혹은 then을 이용해서 데이터 로드가 완료된 후에만 상태를 업데이트하게 할 수 있습니다.

  import React, { useState, useEffect } from 'react'
  import axios from 'axios'

  export default () => {
    const [data, setData] = useState({ hits: [] })
    useEffect(() => {
      // fetchData란 비동기함수 생성
      const fetchData = async () => {
        const result = await axios(
          'https://hn.algolia.com/api/v1/search?query=redux'
        )
        setData(result.data)
      }

      fetchData()
    }, [])
    return (
      <ul>
        {data.hits.map((item) => (
          <li key={item.objectID}>
            <a href={item.url}>{item.title}</a>
          </li>
        ))}
      </ul>
    )
  }
  ```

2.  로컬 스토리지에서 읽어오기(Reading from local storage)
3.  이벤트 리스너를 등록/해제 (Registering and deregistering event listeners)

    - 왜 이벤트 리스너를 useEffect() 내에서 등록해야 할까?

      1.  메모리 관리

          - 만약 useEffect 밖에서 이벤트를 등록하고 해제하게 되면 컴포넌트가 리렌더링될 때마다 window에 핸들러를 계속 추가하게 됩니다. 이러다보면 뭔가 의도치 않은 동작을 하게 되기도 하고, 메모리적으로 문제가 생기기도 한다.
                ```jsx
                function App() {
                  const [state, setState] = useState<number>(0);
                  console.log('rerender');
                  const handler = () => {
                    console.log('hi');
                  }
                  return (
                    <>
                      <p>{state}</p>
                      <button onClick={() => {
                        window.addEventListener('keydown', handler)
                        setState(state + 1);
                      }}>add</button>
                      <button onClick={() => {
                        window.removeEventListener('keydown', handler);
                        setState(state - 1);
                      }}>remove</button>
                    </>
                  );
                }
                ```

      2.  useEffect의 의존성 배열을 사용하면 캡쳐링 문제를 해결할 수 있어요.

          ```jsx
          function App() {
            const [state, setState] = useState < number > 0
            const handler = () => {
              console.log(state)
            }
            useEffect(() => {
              window.addEventListener('keydown', handler)
            }, [])
            return (
              <>
                <p>{'Current: ' + state}</p>
                <button
                  onClick={() => {
                    setState(state + 1)
                  }}
                >
                  add
                </button>
              </>
            )
          }
          ```

          - 아래처럼하면 콘솔창에 계속 0이 찍힌다.
          - 왜 그럴까? 왜냐면 `window.addEventListener` 가 호출되는 시점에서의 `handler` 로 고정이 되어버리기 때문이다.
          - 이럴 경우에 의존성 배열에 state 값을 넣어주게 하면 `state` 가 변경될 때마다 기존의 Event Listener가 사라지고 새로운 값이 반영된 Event Listener가 추가되기 때문에 우리가 의도한대로 동작할 수 있게 된다.

# **useEffect 사용시 마지막 주의사항**

> useEffect 사용 시의 무한루프를 방지하려면?

- useEffect의 dependency는 빈 배열이라도 넘겨 주기
- useEffect의 dependency가 useEffect의 내부에서 변경되지 않도록 하기

  ```jsx
  useEffect(() => {
    setCount(count + 1)
  })
  ```

  - 처음 컴포넌트가 렌더링 될 때, count의 값을, 1증가 시킵니다. 그런데 `setCount`를 통해서 증가시켰기 떄문에 해당 컴포넌트는 **다시 렌더링 되고**, 다시 렌더링 되었기 떄문에, `useEffect`안에 있는 코드가 한번 더 돌고… 훌륭한 무한 루프 코드입니다!
    - 특정 값이 변화되었을 때만 해당 코드가 실행되게 하기 (`deps` 이용)
