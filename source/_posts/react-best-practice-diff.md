---
title: react best practice (2)-- diffing, key
date: 2022-01-07 21:08:58
tags:
---
Today we are going to learning React diffing algorithm and key in React.  Let's understand with below example.
### 1. Diffing algorithm
```bash
mkdir src/components/diff
touch src/components/diff/DiffPage.tsx
```

```typescript
import React, { FC, ReactElement, useEffect, useState } from 'react'

interface Person {
    id: number,
    name: string,
    age: number
}

const DiffPage: FC = (): ReactElement => {
    const [dataValue, setDataValue] = useState(new Date())
    const [personList, setPersonList] = useState([
        { id: 1, name: 'Alex', age: 18 },
        { id: 2, name: 'Allen', age: 19 },
    ])

    useEffect(() => {
        setInterval(() => setDataValue(new Date()), 1000)
    }, [])

    const addOne = () => {
        const person: Person = { id: personList.length + 1, name: `people ${personList.length}`, age: 20 + personList.length }
        setPersonList([person, ...personList])
    }

    return (
        <div>
            <div className='diff'>
                <h2>hello</h2>
                <input type="text" />
                <span>
                    now:{dataValue.toTimeString()}
                    <input type="text" />
                </span>
            </div>
            <div className='key'>
                <h2>Person Information List</h2>
                <button onClick={addOne}>Add one people</button>
                <h3>using index as key</h3>
                <ul>
                    {
                        personList.map((personObj: Person, index: number) => {
                            return <li key={index}>{personObj.name}---{personObj.age}<input type="text" /></li>
                        })
                    }
                </ul>
                <hr />
                <hr />
                <h3>using id as key</h3>
                <ul>
                    {
                        personList.map((personObj: Person) => {
                            return <li key={personObj.id}>{personObj.name}---{personObj.age}<input type="text" /></li>
                        })
                    }
                </ul>
            </div>
        </div>
    )
};

export default DiffPage;

```

### 2. DOM vs Virtual DOM
`DOM` stands for Document Object Model. It is the hierarchical representation of the web page(UI).
`Virtual DOM` is just a copy of the original DOM kept in the memory and synced with the real DOM by libraries such as `ReactDOM`. 
Because DOM manipulation is very slower and expensive in terms of time complexity.

When there is a update in the virtual DOM, react compares the virtual DOM with a snapshot of the virtual DOM taken right before the update of the virtual DOM.

With the help of this comparison React figures out which components in the UI needs to be updated. This process is called diffing. The algorithm that is used for the diffing process is called as the diffing algorithm.

Above source code will render to 
```html
<div>
    <div class="diff">
        <h2>hello</h2><input type="text"><span>now:21:58:08 GMT+0800 (China Standard Time)<input type="text"></span>
    </div>
    <div class="key">
        <h2>Person Information List</h2><button>Add one people</button>
        <h3>using index as key</h3>
        <ul>
            <li>Alex---18<input type="text"></li>
            <li>Allen---19<input type="text"></li>
        </ul>
        <hr>
        <hr>
        <h3>using id as key</h3>
        <ul>
            <li>Alex---18<input type="text"></li>
            <li>Allen---19<input type="text"></li>
        </ul>
    </div>
</div>
```
- The `span` in `diff` class will change every 1 second, but another html element won't change in this class section, that is the help from `diffing algorithm`.
- The `key` section, if use index as key, it can't get benifit from `diffing` because the index was changed and will force evey `li` to render it again and again. If use the id as key, the performace will revert, because the id is unique and stable for every `li`. 

> - Frequent DOM manipulations are expensive.
> - Virtual DOM is a virtual representation of DOM in memory.
> - Virtual DOM is synced with real DOM with ReactDOM library. This process is called Reconciliation.
> - React compares the Virtual DOM and pre-updated Virtual DOM and only marks the sub-tree of components that are updated. This process is called diffing.
> - The algorithm behind diffing is called Diffing algorithm.React uses keys to avoid unnecessary re-renders.

### 3. React event parameters
Compare bind parameter in React.
```bash
mkdir src/components/event
touch src/components/event/ClickButton.tsx
```
```typescript
import React, { FC, ReactElement } from 'react'

const ClickButton: FC = (): ReactElement => {

    const handleClick = (e: any) => {
        console.log(`this is passed any parameter:${e}`);
        console.log(`this is from e.id:${e.target.id}`);
        console.log(`this is e.data:${e.currentTarget.getAttribute('data-xxx')}`);
    }

    // const handleClick2 = (e: string) => {
    //     console.log(`this is passed string parameter:${e}`);
    // }

    const handleClick3 = (e: any, id: string) => {
        console.log(`this is passed id:${id}`);
        console.log(`this is from e.id:${e.target.id}`);
        console.log(`this is e.data:${e.currentTarget.getAttribute('data-xxx')}`);
    }

    return (
        <div>
            <button id="123" data-xxx="789" onClick={handleClick}>
                Click me1
            </button>

            {/* cannot write like this: */}
            {/* <button id="1234" data-xxx="7890" onClick={handleClick2('123')}>
                Click me2
            </button> */}


            <button id="456" data-xxx="78900" onClick={(e) => handleClick3(e, '123456')}>
                Click me3
            </button>
        </div>
    )
}

export default ClickButton;
```