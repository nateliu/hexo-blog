---
title: react best practice (1)-- useRef, forwardRef, useImperativeHandle
date: 2022-01-05 09:16:28
tags:
---
Learning React with hooks
```bash
yarn create react-app react-best-practice --template=typescript && cd react-best-practice
mkdir src/router
touch src/router/IndexRouter.tsx
yarn add react-router-dom

mkdir src/components
mkdir src/components/forwardRef
touch src/components/forwardRef/ParentPage.tsx
touch src/components/forwardRef/ChildInput.tsx
touch src/components/forwardRef/types.tsx
```
### 1. Parent component define a ref and pass it to Child componet, Child component is used `React.forwardRef` to a real `InputWithLabel` component.

- types.tsx
```typescript types.tsx
export interface IChildInputProps {
    label: string
}

export interface IInputWithLabelProps {
    label: string,
    myRef: any
}
```
- ParentPage.tsx
```typescript ParentPage.tsx
import React, { FC, ReactElement, useRef } from 'react';
import ChildInput from './ChildInput';

const ParentPage: FC = (): ReactElement => {
    const myRef = useRef<HTMLInputElement>();

    const handleClick = () => {
        const node = myRef.current;
        console.log(node?.value);
        node?.focus()
    };

    return (
        <div>
            <ChildInput label={"Child Name"} ref={myRef} />
            <button onClick={handleClick}>Click</button>
        </div>
    );
}

export default ParentPage;
```
- ChildInput.tsx
```typescript ChildInput.tsx
import React, { forwardRef, ReactElement, useState } from "react";
import { IChildInputProps, IInputWithLabelProps } from "./types"


const InputWithLabel = ({ label, myRef }: IInputWithLabelProps): ReactElement => {
    const [value, setValue] = useState("");
    const handleChange = (e: any) => {
        console.log(`message from child.value=${e.target.value}`)
        const value = e.target.value;
        setValue(value);
    };

    return (
        <div>
            <span>{label}:</span>
            <input type="text" ref={myRef} value={value} onChange={handleChange} />
        </div>
    );
}

const ChildInput = forwardRef(({ label }: IChildInputProps, ref: any) => (
    <InputWithLabel label={label} myRef={ref} />
));

export default ChildInput;
```
- IndexRouter.tsx
```typescript IndexRouter.tsx
import React, { ReactElement } from 'react';
import { BrowserRouter, Route, Routes } from 'react-router-dom';
import ParentPage from '../components/forwardRef/ParentPage';

const IndexRouter = (): ReactElement => {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/forwardRef" element={<ParentPage />} />
            </Routes>
        </BrowserRouter>
    )
}

export default IndexRouter;
```
> This method will export the whole `ChildInput` to `ParentPage`, for security reason, we just want export some functions to `ParentPage`, that we can use `useImperativeHandler`
> Otherwise, for example user can write following code in `ParentPage`. Do you really want user write `onclick` event in `ParentPage` if you are the owner of `ChildInput`?
```typescript
    const handleClick = () => {
        const node = myRef.current;
        console.log(node?.value);
        node?.focus();
        node.onclick = function() { alert('have clicked me!') }
    };
```

### 2. Add `innerRef` and  `useImperativeHandle` in real inner component `InputWithLabel`

- update `ChildInput.tsx`
```typescript ChildInput.tsx
import React, { forwardRef, ReactElement, useState, useRef, useImperativeHandle } from "react";
...
    useImperativeHandle(myRef, () => ({
        getValue,
        focus() {
            const node = innerRef.current;
            node?.focus();
        }
    }));

    const getValue = () => {
        return value;
    }
...
 <input type="text" ref={innerRef} value={value} onChange={handleChange} />

```
- update `ParentPage`
```typescript ParentPage
    const myRef = useRef<any>();

    const handleClick = () => {
        const node = myRef.current;
        console.log(node);
        console.log(`the value '${node.getValue()}' from Child fucntion getValue()`);
        node.focus();
    };
```
> From the console, when click the `Click` button, we can see only `getValue` and `focus` were exported.

### 3. Delete `InputWithLabel` and move the content into `ChildInput`
- Delete `IInputWithLabelProps`
```typescript types.tsx
export interface IChildInputProps {
    label: string
}
```
- Update `ChildInput.tsx`
```typescript ChildInput.tsx
import React, { forwardRef, ReactElement, useState, useRef, useImperativeHandle } from "react";
import { IChildInputProps } from "./types"

const ChildInput = ({ label }: IChildInputProps, ref: any): ReactElement => {
    const [value, setValue] = useState("");

    const innerRef = useRef<HTMLInputElement>(null);

    useImperativeHandle(ref, () => ({
        getValue,
        focus() {
            const node = innerRef.current;
            node?.focus();
        }
    }));

    const getValue = () => {
        return value;
    }

    const handleChange = (e: any) => {
        e.preventDefault();
        console.log(`message from child.value=${e.target.value}`)
        const value = e.target.value;
        setValue(value);
    };

    return (
        <div>
            <span>{label}:</span>
            <input type="text" ref={innerRef} value={value} onChange={handleChange} />
        </div>
    );
}

export default forwardRef(ChildInput);
```
- No change in `ParentPage`
