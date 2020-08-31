---
title: "SSR과 Redux Saga"
date: 2020-08-31 08:26:28 -0400
categories: SSR Redux Saga React
---

# SSR 과 Redux

서버사이드 렌더링의 경우, app 내 state 를 response에 같이 전송해야한다.

그래야 클라이언트에서 initial state를 사용할 수 있기 때문! (HTML을 generating 할 때 preload 를 할 수 있다)

그러나 클라이언트 markup 과 server markup이 다르고, 클라이언트는 데이터를 다시 로드하게 될 것이다.



데이터를 클라이언트에게 보내기 위해서는:

- 완전 새 Redux stroe instance 를 모든 요청에서 생성하고
- 몇몇 actions 를 선택적으로 dispatch 한 후
- pull the state out of store
- 그리고 state 를 클라이언트에게 패스한다.

클라이언트 측에서는, 새로운 리덕스 스토어가 생성되고 서버에서 제공받은 state로 initialized 될 것이다.

리덕스는 **오로지** app 의 initial state 만 주는 것이다!!

# Settings

리액트 바인딩을 위해 리덕스 설치

```bash
$ npm install @express react-redux
```



다음 코드를 app 부분에 추가해준다.

```js
import { Provider } from 'react-redux';


<Provider>
            <GlobalStyles />
            <App {...props} />
</Provider>
```



서버사이드 렌더링의 주요한 특징은, 클라이언트 사이드에 렌더링 전에 initial HTML을 렌더링 한다는 것이다.

이것을 하기 위해, [ReactDOMServer.renderToString()](https://facebook.github.io/react/docs/react-dom-server.html#rendertostring) 을 사용한다.



그리고 store.getState() 를 통해 리덕스 스토어에서 이니셜 스테이트를 가져오면 된다. -> renderFulllPage 로!



### next,js 와 redux-saga

먼저 nextjs에 redux를 적용시켜야 한다. 이를 위해서 [next-redux-wrapper](https://github.com/kirill-konshin/next-redux-wrapper) 라이브러리를 사용하였다. 특별히 해주는 큰 작업은 없고, SSR 환경에서 redux 스토어를 만들어주는 작업을(서버사이드일 경우 스토어를 생성해주고, 클라이언트 사이드일 경우 `window` 객체에서 스토어를 가져오는 작업) 해주어 스토어를 next의 context에 주입시켜주는 HOC를 제공해준다.



제공해주는 HOC로 [pages/_app](https://nextjs.org/docs/advanced-features/custom-app) 파일 컴포넌트를 감싸주면 된다

cf) HOC 란?

HOC는 바로 HOF에서 유래한 단어이다. 즉, 컴포넌트를 인자로 받아서 컴포넌트를 반환하는 함수를 뜻한다. 

가장 많이 쓰이는 형태가 아마 스토어와 컴포넌트를 연결시켜 주는 HOC일 것이다. 최근 가장 널리 쓰이는 react-redux의 `connect` 함수도 이런 역할을 하는데, 엄밀히 말해 HOC를 생성해주는 헬퍼 함수라고 할 수 있다. `connect` 함수는 스토어의 상태를 props로 주입시켜주는 `mapStateToProps` 와 액션 생성 함수를 스토어의 dispatch와 연결시켜 props로 주입시켜주는 `mapDispatchToProps` 를 인자로 받아서 새로운 HOC를 반환한다.

이 외에도 몇가지 HOC로 할 수 있는 중요한 기능들을 나열하면 다음과 같다.

- 생명주기 메소드 주입
- State 및 이벤트 핸들러 주입
- Props 변환 및 주입
- Render 함수 확장



```js
const createStore = initialState => {
  const sagaMiddleware = createSagaMiddleware();

  const store = createStore(
    reducer,
    initialState,
    applyMiddleware(sagaMiddleware)
  );

  sagaMiddleware.run(rootSaga);

  return store;
};

class App extends NextApp {
  static async getInitialProps(context) {
    const { Component, ctx } = context;
    const { store, isServer } = ctx; // next의 context에서 store을 받을 수 있게된다.

    const pageProps = (await Component.getInitialProps?.(ctx)) || {};
    return { pageProps };
  }

  render() {
    const { Component, pageProps, store } = this.props;
    return (
      <Provider store={store}>
        <Component {...pageProps} />
      </Provider>
    );
  }
}

export default withRedux(createStore)(App);
```



이렇게 redux와 redux-saga 설정을 간단히 끝낼 수 있다. 하지만 아직 특정 컴포넌트의 `getInitialProps`에서 어떤 saga task를 실행하는 액션을 디스패치 하여도, 그 작업이 끝난 후의 스토어가 제공되는 것이 보장되지는 않는다. 모든 saga task가 완료된 후에, `App` 컴포넌트의 `getInitialProps`가 끝나는 것을 보장하기 위해서 `createStore`와 `App`을 다음과 같이 바꿀 수있다.



```js
const createStore = initialState => {
  const sagaMiddleware = createSagaMiddleware();

  const store = createStore(
    reducer,
    initialState,
    applyMiddleware(sagaMiddleware)
  );

  store.sagaTask = sagaMiddleware.run(rootSaga); // rootSaga의 task를 store 객체에 넣어준다.

  return store;
};

class App extends NextApp {
  static async getInitialProps(context) {
    const { Component, ctx } = context;
    const { store, isServer } = ctx;

    const pageProps = (await Component.getInitialProps?.(ctx)) || {};
    if (isServer) {
      // getInitialProps를 호출하는 환경이 서버일 경우에는는 모든 sagaTask가 완료된 상태의 스토어를 주입시켜줘야 한다.
      store.dispatch(END); // redux-saga의 END 액션 이용하여 saga task가 종료되도록 한다.
      await store.sagaTask.toPromise(); // saga task가 모두 종료되면 resolve 된다.
    }
    return { pageProps };
  }

  render() {
    const { Component, pageProps, store } = this.props;
    return (
      <Provider store={store}>
        <Component {...pageProps} />
      </Provider>
    );
  }
}
```



위와 같이 코드를 바꾸면, 특정 Component의 `getInitialProps`에서 saga task를 실행하는 액션을 디스패치 한 후, `store.dispatch(END)`를 통해 해당 task를 포함한 다른 task를 종료되면 `store.sagaTask.toPromise()` 가 resolve 된다.



- 참고

 https://redux.js.org/recipes/server-rendering 

[https://godsenal.com/posts/nextjs%EC%99%80-redux-saga-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0/](https://godsenal.com/posts/nextjs와-redux-saga-사용하기/)

https://meetup.toast.com/posts/137



---- 다음 포스팅

Redux Persist

[https://medium.com/humanscape-tech/redux-persist-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-2077c9e566d9](https://medium.com/humanscape-tech/redux-persist-알아보기-2077c9e566d9)

