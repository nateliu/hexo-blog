---
title: react best practice (3)-- learning source code
date: 2022-01-15 11:06:38
tags:
---
Try to setup react soure code learning environment.

## 1. Step 1
```bash
git clone https://github.com/facebook/react.git
cd react
git branch -a
git checkout 17.0.2
yarn install
```

According the [How to contribute](https://reactjs.org/docs/how-to-contribute.html#development-workflow), next run command below
```bash
yarn build react/index,react-dom/index --type=UMD

# open fixtures/packaging/babel-standalone/dev.html and it used the compiled react.js

cd build/node_modules/react
yarn link
cd build/node_modules/react-dom
yarn link

```

Using `yarn create react-app` to create a new project and remove dependency of `react` and `react-dom` in `package.json`
```bash
yarn create react-app react-source-learning --template:typescript
cd react-source-learning
yarn link react react-dom
```

## 2. Step 2
Write JSX in maunally.
```typescript
import React from 'react';
import ReactDOM from 'react-dom';

//jsx
const element = <h1 className='title' style={{color:'red'}}> hello</h1>

//babel=>js function. React.createElement()=>vnode

const element2 = React.createElement("h1", {
    className: "title",
    style: {
      color: 'red'
    }
  }, " hello");

console.log(element);

ReactDOM.render(
    element2,
    document.getElementById('root')
);
```
> We can go to [Babel](https://babeljs.io/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=DwCwjABAxgNghgZwQOTgWwKYF4DkAXASzxgxwgTwE8SsBvWqAexkYCcAuHVjAExwF9-APgggMMFsAD04IUA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.16.9&externalPlugins=&assumptions=%7B%7D) to check the `element` mentioned above. `element` is equal to `element2`.

The process is:
1, `JSX` will be transferd to `React.createElement` method by `Babel`
2, `React.createElement` is actually returned a `virtual node`.
3, `React.Render` will render the `virtual node` to `realistic dom`

So next to implement a `createElement` in `react.js`.

And then to create a new `react-dom.js`
