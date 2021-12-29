---
title: news-pubish-management(7)--manage state with redux
date: 2021-12-28 14:11:22
tags:
---
Next we focus on the state management, here we use classic [Redux](https://redux.js.org/).
### 1. setup redux
```bash
yarn add redux react-redux
mkdir src/redux
touch src/redux/store.js
mkdir src/redux/reducers
touch src/redux/reducers/CollapsedReducer.js
``` 
> [redux principles](https://redux.js.org/understanding/thinking-in-redux/three-principles)
 . Single source of truth
 . State is read-only
 . Changes are made with pure functions

### 2. Finish the collapsed in both `TopHeader` and `SideMenu` components.
CollapsedReducer.js
```javascript
export const CollapsedReducer = (prevState = { isCollapsed: false }, action) => {
    return prevState;
}
```
```javascript
import { createStore, combineReducers } from 'redux';
import { CollapsedReducer } from './reducers/CollapsedReducer';

const reducer = combineReducers({
    CollapsedReducer
})

const store = createStore(reducer);

export default store;
```
we have a single store now, it mean if we can get back our store, then we can do store.dispatch or store.subscribe operations. but how to get store in our component?
Go to App.js, make our IndexRouter in the Provider section.
```javascript
import React from 'react'
import IndexRouter from './router/IndexRouter'
import { Provider } from 'react-redux'
import store from './redux/store'

import './App.css'

export default function App() {
  return (
    <Provider store={store}>
      <IndexRouter />
    </Provider>
  )
}

```
Our `SideMenu` and `TopHeader` will use the store. so go to `TopHeader` first, because `Topheader` is publish the state.Its response is `Dispatch` the state to store.
In `TopHeader`, we use `connect` to make `TopHeader` as child components, we can get value from parameter of `props`.
```javascript
import { connect } from 'react-redux';
...

function TopHeader(props) {
...
}

const mapStateToProps = () => {
  return {
    testValue:'testValue',
  }
}

export default connect(mapStateToProps)(TopHeader);
```
In previous version, we use component's state, now we have props. and props contain isCollapsed, so we remove the component state.
```javascript in TopHeader
import React from 'react'
...
import { connect } from 'react-redux';
...

function TopHeader(props) {
    // NOTE: changed to redux, state is not neccessary.
    // const [collapsed, setCollapsed] = useState(false)
  ...
    const changeCollapsed = () => {
    // NOTE: changed to redux, state is not neccessary.
    // setCollapsed(!collapsed);
    props.changeCollapsed();
  } 
}


const mapStateToProps = ({ CollapsedReducer: { isCollapsed } }) => {
  return {
    isCollapsed
  }
}

const mapDispatchToProps = {
  changeCollapsed() {
    return {
      type: "change_collapsed",
      // payload:''
    }
  }
}

export default connect(mapStateToProps,mapDispatchToProps)(TopHeader);
```
when user click the icon in `TopHeader`, the workflow go to `store`, and `store` find out the `CollapsedReducer`, then excute the logic to change to new state. below are the `CollapsedReducer.js` content:
```javascript
export const CollapsedReducer = (prevState = { isCollapsed: false }, action) => {
    // console.log(action);
    const { type } = action;
    switch (type) {
        case 'change_collapsed':
            let newState = { ...prevState };
            newState.isCollapsed = !newState.isCollapsed;
            return newState;
        default:
            return prevState;
    }
}
```
Back to `SideMedu`, `SideMenu` retrieve the state to collapse or expand himself. We also `connect` `SideMenu `first. It doesn't need `Dispatch` anything, so our changing in `SideMenu` looks like:
```javascript
import { connect } from 'react-redux';
...
function SideMenu(props) {
    ...
    <Sider trigger={null} collapsible collapsed={props.isCollapsed}>
}

const mapStateToProps = ({ CollapsedReducer: { isCollapsed } }) => ({ isCollapsed });

export default connect(mapStateToProps)(SideMenu);
```









### 3. Add [spin](https://ant-design.gitee.io/components/spin-cn/#header) when retrieve data from backend db.
Go to NewsRouter.js
```javascript
import { Spin } from 'antd';
...

    return (
        <Spin size='large' spinning={true}>
        ...
        </Spin>
```
if the spinning is false then the Spin is displayed, otherwise will showing.

every call will showing, we can perform it in [axios intercepter](https://axios-http.com/docs/interceptors).

Go to http.js
```javascript

// Add a request interceptor
axios.interceptors.request.use(function (config) {
    // Do something before request is sent
    // Show Spin
    return config;
  }, function (error) {
    // Do something with request error
    // Hide Spin
    return Promise.reject(error);
  });

// Add a response interceptor
axios.interceptors.response.use(function (response) {
    // Any status code that lie within the range of 2xx cause this function to trigger
    // Do something with response data
    // Hide Spin
    return response;
  }, function (error) {
    // Any status codes that falls outside the range of 2xx cause this function to trigger
    // Do something with response error
    // Hide Spin
    return Promise.reject(error);
  });
```
```bash
touch src/redux/reducers/LoadingReducer.js
```
copy CollapsedReducer.js and modified as below
```javascript
export const LoadingReducer = (prevState = { isLoading: false }, action) => {
    // console.log(action);
    const { type, payload } = action;
    switch (type) {
        case 'change_loading':
            let newState = { ...prevState };
            newState.isLoading = payload;
            return newState;
        default:
            return prevState;
    }
}
```
also we need login this reducer to store.js.
```javascript
import { createStore, combineReducers } from 'redux';
import { CollapsedReducer } from './reducers/CollapsedReducer';
import { LoadingReducer } from './reducers/LoadingReducer';

const reducer = combineReducers({
    CollapsedReducer,
    LoadingReducer
})

const store = createStore(reducer);

export default store;
```

add connect to NewsRouter.js
```javascript
import { connect } from 'react-redux'
...
function NewsRouter(props) {
    ...
    <Spin size='large' spinning={props.isLoading}>
    ...
}

const mapStateToProps = ({ LoadingReducer: { isLoading } }) => ({ isLoading });

export default connect(mapStateToProps)(NewsRouter);
```
Back to http.js to dispatch isLoading to store.
```javascript
import store from '../redux/store';
...
    store.dispatch({
        type:'change_loading',
        payload: true,
    });
```


### 4. Persist redux state
We chose [redux persist](https://github.com/rt2zz/redux-persist) to implement this feature.
```bash
yarn add redux-persist
```
we have to modify our store.js
```javascript
import { createStore, combineReducers } from 'redux';
import { CollapsedReducer } from './reducers/CollapsedReducer';
import { LoadingReducer } from './reducers/LoadingReducer';

import { persistStore, persistReducer } from 'redux-persist'
import storage from 'redux-persist/lib/storage' // defaults to localStorage for web

const reducer = combineReducers({
    CollapsedReducer,
    LoadingReducer
})

const persistConfig = {
    key: 'collapsed',
    storage,
    blacklist: ['LoadingReducer']
}

const persistedReducer = persistReducer(persistConfig, reducer)

const store = createStore(persistedReducer)
const persistor = persistStore(store)

export {
    store,
    persistor,
};
```
Go to App.js
```javascript
import { store, persistor } from './redux/store'
import { PersistGate } from 'redux-persist/integration/react'
...

export default function App() {
  return (
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        ...
      </PersistGate>
    </Provider>
  )
}
```