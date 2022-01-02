---
title: todo-app-js-ts
date: 2021-12-30 21:34:38
tags:
---
Use react hook to create a Todo app, and then migrate to typescript.
### 1. create todo-app-js 
I am going to perform below componets for the app, and I am going to use hooks for implement. It will contain `useState`, `useReducer`, `useEffect`, `useCallback`
- App
- TodoList
- TodoInput
- TodoItem
- todoReducer

```bash
yarn create react-app todo-app-js
cd todo-app-js
mkdir src/components
touch src/components/TodoList.js
touch src/components/TodoInput.js
touch src/components/TodoItem.js

mkdir src/reducer
touch src/reducer/todoReducer.js
```

```javascript TodoList
import React, { useReducer, useEffect, useCallback } from 'react'
import TodoInput from './TodoInput'
import TodoItem from './TodoItem'
import todoReducer from '../reducer/todoReducer'

// comment out below line because has changed to use lazy load.
// const initialState:IState = {
//     todoList:[]
// }

function init(initTodoList) {
    return {
        todoList: initTodoList
    }
}

export default function TodoList() {
    // comment out below line because has changed to use useReducer
    // const [todoList,setTodoList] = useState([]);
    const [state, dispatch] = useReducer(todoReducer, [], init)

    useEffect(() => {
        // console.log(state.todoList)
        const list = localStorage.getItem('todoList' || '[]');
        if (list) {
            const todoLists = JSON.parse(list);
            dispatch({
                type: "INIT_TODOLIST",
                payload: todoLists
            })
        }
    }, [])

    useEffect(() => {
        localStorage.setItem('todoList', JSON.stringify(state.todoList))
    }, [state.todoList])

    const addTodo = useCallback(todo => {
         // comment out below line because has changed to use useReducer.
        // setTodoList(todoList => [...todoList,todo]);
        dispatch({
            type: "ADD_TODO",
            payload: todo
        })
    }, [])


    const removeTodo = useCallback((id) => {
        dispatch({
            type: "REMOVE_TODO",
            payload: id
        })
    }, [])

  
    const toggleTodo = useCallback((id) => {
        dispatch({
            type: "TOGGLE_TODO",
            payload: id
        })
    }, [])

    return (
        <div>
            <TodoInput
                addTodo={addTodo}
                todoList = {state.todoList}
            />
            {
                state.todoList &&
                state.todoList.map(todo => {
                    return <TodoItem
                        key={todo.id}
                        todo={todo}
                        removeTodo={removeTodo}
                        toggleTodo={toggleTodo}
                    />
                })
            }
        </div>
    )
}

```

```javascript TodoInput
import React, { useRef } from 'react'

export default function TodoInput(props) {
    const inputRef = useRef(null)
    const handleClick = ()=> {
        const val = inputRef.current.value.trim();

        if(val.length){
            const isExist = props.todoList.find(todo => todo.content === val);
            if(isExist){
                alert('already exist!');
                return;
            }

            props.addTodo({
                id: Date.now(),
                content:inputRef.current.value.trim(),
                completed:false
            })
            
            inputRef.current.value = '';
        }


    }
    return (
        <div>
            <input type='text' ref={inputRef} />
            <button onClick={()=>handleClick()}>Add</button>
        </div>
    )
}

```

```javascript TodoItem
import React from 'react'

export default function TodoItem(props) {
    const { id, content, completed } = props.todo;
    return (
        <div>
            <input type="checkbox" checked={completed} onChange={() => props.toggleTodo(id)} />
            <span style={{ textDecoration: completed ? 'line-through' : 'none' }}>{content}</span>
            <button onClick={() => props.removeTodo(id)}>Delete</button>
        </div>
    )
}

```

```javascript todoReducer
export default function todoReducer(state, action) {
    const { type, payload } = action;
    switch (type) {
        case "INIT_TODOLIST":
            return {
                ...state,
                todoList: payload
            }
        case "ADD_TODO":
            return {
                ...state,
                todoList: [...state.todoList, payload]
            }
        case "REMOVE_TODO":
            return {
                ...state,
                todoList: state.todoList.filter(todo => todo.id !== payload)
            }
        case "TOGGLE_TODO":
            return {
                ...state,
                todoList: state.todoList.map(todo => {
                    return todo.id === payload ? {
                        ...todo,
                        completed: !todo.completed
                    } : {
                        ...todo
                    }
                })
            }
        default:
            return state;
    }
}

```
> !! Here I chose `useReducer` to instead of `useState`. It looks unnecessary, but if we add a new component as middle component between TodoList and TodoItem, then the `useReducer` is better than `useState`.

Pretty easy! right？ that is javascript version, because I am a Java progromer before, I feel it is weird. I can give any parameters to our function in the writing duration. For example. How do I know `action` has `type` and `payload` if I just in `todoReducer` file？If I typo the `paylaod`, it won't give any error in the writing duration. So next I am going to change to use `TypeScript`

### 2. create todo-app-ts 
The components are the same with `todo-app-js`
```bash
yarn create react-app todo-app-ts --template=typescript
cd todo-app-ts
mkdir src/components
touch src/components/TodoList.tsx
touch src/components/TodoInput.tsx
touch src/components/TodoItem.tsx

mkdir src/reducer
touch src/reducer/todoReducer.ts
```
> !! use `--template=typescript` to tell `yarn create` to create a typescript application.

- Create a interfact as types.
```bash
mkdir src/types
touch src/types/todoTypes.ts
```
```typescript
export interface ITodo{
    id: number,
    content: string,
    completed:boolean
}

export interface IState{
    todoList: ITodo[]
}

export interface IAction{
    type: ACTION_TYPE,
    payload: ITodo | ITodo[] | number  //payload can be array and number.
}

export enum ACTION_TYPE{
    ADD_TODO = 'addTodo',
    REMOVE_TODO = 'removeTodo',
    TOGGLE_TODO = 'toggleTodo',
    INIT_TODOLIST = 'initTodoList'
}
```

- Go to `todoReducer.ts`
```typescript
import { ACTION_TYPE, IAction, IState, ITodo } from '../types/todoTypes';

export default function todoReducer(state: IState, action: IAction): IState {
    const { type, payload } = action;

    switch (type) {
        case ACTION_TYPE.INIT_TODOLIST:
            return {
                ...state,
                todoList: payload as ITodo[]
            }
        case ACTION_TYPE.ADD_TODO:
            return {
                ...state,
                //payload as ITodo is important, otherwise typescript don't know it is todo or id.
                todoList: [...state.todoList, payload as ITodo]
            }
        case ACTION_TYPE.REMOVE_TODO:
            return {
                ...state,
                todoList: state.todoList.filter(todo => todo.id !== payload)
            }
        case ACTION_TYPE.TOGGLE_TODO:
            return {
                ...state,
                todoList: state.todoList.map(todo => {
                    return todo.id === payload ? {
                        ...todo,
                        completed: !todo.completed
                    } : {
                        ...todo
                    }
                })
            }
        default:
            return state;
    }
}
```

- Evey each component should return ReactElement. So we are not use `rfc` formatting, we are change to below formation as well
```typescript
interface IProps{
    ...
}
const ComponentName:FC<IProps> = ({...Iprops...}) => {
    ....
    return (
        <div>
            ComponentName
        </div>
    );
}

export default ComponentName;
```
> Here we don't need move the `IProps` to `src/types/` folder, because here the `IProps` is a spicific parameter for this component, not a common parameter. For example below `TodoList` doesn't have `Props` and its has no `IProps` interfacte.

```typescript TodoList.tsx
import React, { ReactElement, useReducer, useEffect, useCallback } from 'react'
import todoReducer from '../reducer/todoReducer';
import { ACTION_TYPE, IState, ITodo } from '../types/todoTypes';
import TodoInput from './TodoInput';
import TodoItem from './TodoItem';

// comment out below line because has changed to use lazy load.
// const initialState:IState = {
//     todoList:[]
// }

function init(initTodoList: ITodo[]):IState{
    return {
        todoList: initTodoList
    }
}

const TodoList: FC = (): ReactElement => {
    // comment out below line because has changed to use useReducer
    // const [todoList, setTodoList] = useState<ITodo[]>([]);

    // comment out below line because has changed to use lazy load.
    // const [ state, dispatch] = useReducer(todoReducer,initialState)

    // lazy load
    const [state, dispatch] = useReducer(todoReducer, [], init)

    useEffect(() => {
        // console.log(state.todoList)
        const list = localStorage.getItem('todoList' || '[]');
        if (list) {
            const todoLists = JSON.parse(list);
            dispatch({
                type: ACTION_TYPE.INIT_TODOLIST,
                payload: todoLists
            })
        }
    }, [])

    useEffect(() => {
        localStorage.setItem('todoList', JSON.stringify(state.todoList))
    }, [state.todoList])


    const addTodo = useCallback((todo: ITodo) => {
        // comment out below line because has changed to use useReducer
        // setTodoList(todoList => [...todoList,todo]);
        dispatch({
            type: ACTION_TYPE.ADD_TODO,
            payload: todo
        })
    }, [])

    const removeTodo = useCallback((id: number) => {
        dispatch({
            type: ACTION_TYPE.REMOVE_TODO,
            payload: id
        })
    }, [])

    const toggleTodo = useCallback((id: number) => {
        dispatch({
            type: ACTION_TYPE.TOGGLE_TODO,
            payload: id
        })
    }, [])

    return (
        <div className="todo-list">
            <TodoInput
                addTodo={addTodo}
                todoList={state.todoList}
            />
            {
                state.todoList &&
                state.todoList.map(todo => {
                    return <TodoItem
                        key={todo.id}
                        todo={todo}
                        removeTodo={removeTodo}
                        toggleTodo={toggleTodo}
                    />
                })
            }
        </div>
    )
}
export default TodoList;

```
- `TodoList` did all the logics, we assume `TodoInput` and `TodoItem` are child components.
```typescript TodoInput
import React, { FC, ReactElement, useRef } from 'react'
import { ITodo } from '../types/todoTypes'

interface IProps {
    addTodo: (todo: ITodo) => void,
    todoList: ITodo[]
}

const TodoInput: FC<IProps> = ({ addTodo, todoList }): ReactElement => {

    const inputRef = useRef<HTMLInputElement>(null);

    const addItem = (): void => {

        const val: string = inputRef.current!.value.trim();

        if (val.length) {
            const isExist = todoList.find(todo => todo.content === val);
            if (isExist) {
                alert('already exist!');
                return;
            }

            addTodo({
                id: new Date().getTime(),
                content: val,
                completed: false
            })

            inputRef.current!.value = '';
        }
    }

    return (
        <div>
            <input type="text" ref={inputRef} />
            <button onClick={addItem}>Add</button>
        </div>
    )
}

export default TodoInput
```
> Because we have pointed the `props.addTodo` as `addTodo` in the definition. So here we can just use `addTodo`.

- Go to `TodoItem`
```typescript
import React, { FC, ReactElement } from 'react'
import { ITodo } from '../types/todoTypes'

interface IProps {
    todo: ITodo,
    toggleTodo: (id: number) => void,
    removeTodo: (id: number) => void
}

const TodoItem: FC<IProps> = ({ todo, toggleTodo, removeTodo }): ReactElement => {
    const { id, content, completed } = todo;

    return (
        <div>
            <input type="checkbox" checked={completed} onChange={() => toggleTodo(id)} />
            <span style={{ textDecoration: completed ? 'line-through' : 'none' }}>{content}</span>
            <button onClick={() => removeTodo(id)}>Delete</button>
        </div>
    )
}
export default TodoItem;

```

### 3. setup environtment from 0
- Initialize `todo-app-self`
```bash
mkdir todo-app-self 
cd todo-app-self
yarn init
```

```json
{
  "name": "todo-app-self",
  "version": "1.0.0",
  "description": "source of typescript",
  "main": "src/index.tsx",
  "author": "Nate Liu",
  "license": "MIT"
}
```
If use `yarn init -y` then the package.json will without `author` and `description`.
```bash
mkdir src
touch src/index.ts
mkdir src/components
touch src/components/TodoList.tsx
touch src/components/TodoInput.tsx
touch src/components/TodoItem.tsx

mkdir src/reducer
touch src/reducer/todoReducer.ts

```
- Add typescript and tslint globally
```bash
yarn global add typescript tslint
tsc --init
```
this will generate a `tsconfig.json` in the root projct.
> !!`yarn global add xxx` is not equal `yarn add global xxx`
```bash
yarn add webpack webpack-cli webpack-dev-server
mkdir build
touch build/webpack.config.js
```
```javascript webpack.config.js
cconst path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
    entry: './src/index.tsx',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist'),
    },
    resolve: {
        extensions: ['.ts', '.tsx', '.js']
    },
    module: {
        rules: [
            {
                test: /\.tsx?$/,
                use: 'ts-loader',
                exclude: /node_modules/
            }
        ]
    },
    devtool: process.env.NODE_ENV === 'production' ? false : 'inline-source-map',
    devServer: {
        client: {
            overlay: {
                errors: true,
                warnings: false,
            },
        },
        static: path.join(__dirname, 'dist'),
        hot: true,
        compress: false,
        host: 'localhost',
        port: 3000
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            template: './src/template/index.html'
        })
    ]
}
```
According to this `webpack.config.js`,we also need to add files as below:
```bash
yarn add path html-webpack-plugin clean-webpack-plugin ts-loader

mkdir src/template
touch src/template/index.html

yarn add typescript
```
template file content is
```html index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Todo-App-Self</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

Go back `package.json`, and add `scripts`
```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "cross-env NODE_ENV=development webpack-dev-server --config ./build/webpack.config.js",
    "build": "cross-env NODE_ENV=production webpack --config ./build/webpack.config.js"
  }
```
Here we used `cross-env`, so 
```bash
yarn add cross-env
```
Go to `index.tsx`
```typescript
console.log('Hello, Webpack')
```

by now, when run below command, you will see `Hello, Webpack` in console.
```bash
yarn start
```

Because we need `react`, it also need add following libraries.
```bash
yarn add react react-dom @types/react @types/react-dom
```
The `package.json` looks like as below:
```json
{
  "name": "todo-app-self",
  "version": "1.0.0",
  "description": "source of typescript",
  "main": "src/index.tsx",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "cross-env NODE_ENV=development webpack-dev-server --config ./build/webpack.config.js",
    "build": "cross-env NODE_ENV=production webpack --config ./build/webpack.config.js"
  },
  "author": "Nate Liu",
  "license": "MIT",
  "dependencies": {
    "@types/react": "^17.0.38",
    "@types/react-dom": "^17.0.11",
    "clean-webpack-plugin": "^4.0.0",
    "cross-env": "^7.0.3",
    "html-webpack-plugin": "^5.5.0",
    "path": "^0.12.7",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "ts-loader": "^9.2.6",
    "typescript": "^4.5.4",
    "webpack": "^5.65.0",
    "webpack-cli": "^4.9.1",
    "webpack-dev-server": "^4.7.2"
  }
}
```
Most of content in `tsconfig.json` doesn't need to modify, but only one jsx from `preserve` to `react`
```javascript
"jsx": "react", 
```
In my experience, it looks like
```json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig.json to read more about this file */

    /* Projects */
    // "incremental": true,                              /* Enable incremental compilation */
    // "composite": true,                                /* Enable constraints that allow a TypeScript project to be used with project references. */
    // "tsBuildInfoFile": "./",                          /* Specify the folder for .tsbuildinfo incremental compilation files. */
    // "disableSourceOfProjectReferenceRedirect": true,  /* Disable preferring source files instead of declaration files when referencing composite projects */
    // "disableSolutionSearching": true,                 /* Opt a project out of multi-project reference checking when editing. */
    // "disableReferencedProjectLoad": true,             /* Reduce the number of projects loaded automatically by TypeScript. */

    /* Language and Environment */
    "target": "es2016",                                  /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */
    // "lib": [],                                        /* Specify a set of bundled library declaration files that describe the target runtime environment. */
    "jsx": "react",                                /* Specify what JSX code is generated. */
    // "experimentalDecorators": true,                   /* Enable experimental support for TC39 stage 2 draft decorators. */
    // "emitDecoratorMetadata": true,                    /* Emit design-type metadata for decorated declarations in source files. */
    // "jsxFactory": "",                                 /* Specify the JSX factory function used when targeting React JSX emit, e.g. 'React.createElement' or 'h' */
    // "jsxFragmentFactory": "",                         /* Specify the JSX Fragment reference used for fragments when targeting React JSX emit e.g. 'React.Fragment' or 'Fragment'. */
    // "jsxImportSource": "",                            /* Specify module specifier used to import the JSX factory functions when using `jsx: react-jsx*`.` */
    // "reactNamespace": "",                             /* Specify the object invoked for `createElement`. This only applies when targeting `react` JSX emit. */
    // "noLib": true,                                    /* Disable including any library files, including the default lib.d.ts. */
    // "useDefineForClassFields": true,                  /* Emit ECMAScript-standard-compliant class fields. */

    /* Modules */
    "module": "commonjs",                                /* Specify what module code is generated. */
    // "rootDir": "./",                                  /* Specify the root folder within your source files. */
    // "moduleResolution": "node",                       /* Specify how TypeScript looks up a file from a given module specifier. */
    // "baseUrl": "./",                                  /* Specify the base directory to resolve non-relative module names. */
    // "paths": {},                                      /* Specify a set of entries that re-map imports to additional lookup locations. */
    // "rootDirs": [],                                   /* Allow multiple folders to be treated as one when resolving modules. */
    // "typeRoots": [],                                  /* Specify multiple folders that act like `./node_modules/@types`. */
    // "types": [],                                      /* Specify type package names to be included without being referenced in a source file. */
    // "allowUmdGlobalAccess": true,                     /* Allow accessing UMD globals from modules. */
    // "resolveJsonModule": true,                        /* Enable importing .json files */
    // "noResolve": true,                                /* Disallow `import`s, `require`s or `<reference>`s from expanding the number of files TypeScript should add to a project. */

    /* JavaScript Support */
    // "allowJs": true,                                  /* Allow JavaScript files to be a part of your program. Use the `checkJS` option to get errors from these files. */
    // "checkJs": true,                                  /* Enable error reporting in type-checked JavaScript files. */
    // "maxNodeModuleJsDepth": 1,                        /* Specify the maximum folder depth used for checking JavaScript files from `node_modules`. Only applicable with `allowJs`. */

    /* Emit */
    // "declaration": true,                              /* Generate .d.ts files from TypeScript and JavaScript files in your project. */
    // "declarationMap": true,                           /* Create sourcemaps for d.ts files. */
    // "emitDeclarationOnly": true,                      /* Only output d.ts files and not JavaScript files. */
    // "sourceMap": true,                                /* Create source map files for emitted JavaScript files. */
    // "outFile": "./",                                  /* Specify a file that bundles all outputs into one JavaScript file. If `declaration` is true, also designates a file that bundles all .d.ts output. */
    // "outDir": "./",                                   /* Specify an output folder for all emitted files. */
    // "removeComments": true,                           /* Disable emitting comments. */
    // "noEmit": true,                                   /* Disable emitting files from a compilation. */
    // "importHelpers": true,                            /* Allow importing helper functions from tslib once per project, instead of including them per-file. */
    // "importsNotUsedAsValues": "remove",               /* Specify emit/checking behavior for imports that are only used for types */
    // "downlevelIteration": true,                       /* Emit more compliant, but verbose and less performant JavaScript for iteration. */
    // "sourceRoot": "",                                 /* Specify the root path for debuggers to find the reference source code. */
    // "mapRoot": "",                                    /* Specify the location where debugger should locate map files instead of generated locations. */
    // "inlineSourceMap": true,                          /* Include sourcemap files inside the emitted JavaScript. */
    // "inlineSources": true,                            /* Include source code in the sourcemaps inside the emitted JavaScript. */
    // "emitBOM": true,                                  /* Emit a UTF-8 Byte Order Mark (BOM) in the beginning of output files. */
    // "newLine": "crlf",                                /* Set the newline character for emitting files. */
    // "stripInternal": true,                            /* Disable emitting declarations that have `@internal` in their JSDoc comments. */
    // "noEmitHelpers": true,                            /* Disable generating custom helper functions like `__extends` in compiled output. */
    // "noEmitOnError": true,                            /* Disable emitting files if any type checking errors are reported. */
    // "preserveConstEnums": true,                       /* Disable erasing `const enum` declarations in generated code. */
    // "declarationDir": "./",                           /* Specify the output directory for generated declaration files. */
    // "preserveValueImports": true,                     /* Preserve unused imported values in the JavaScript output that would otherwise be removed. */

    /* Interop Constraints */
    // "isolatedModules": true,                          /* Ensure that each file can be safely transpiled without relying on other imports. */
    // "allowSyntheticDefaultImports": true,             /* Allow 'import x from y' when a module doesn't have a default export. */
    "esModuleInterop": true,                             /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables `allowSyntheticDefaultImports` for type compatibility. */
    // "preserveSymlinks": true,                         /* Disable resolving symlinks to their realpath. This correlates to the same flag in node. */
    "forceConsistentCasingInFileNames": true,            /* Ensure that casing is correct in imports. */

    /* Type Checking */
    "strict": true,                                      /* Enable all strict type-checking options. */
    // "noImplicitAny": true,                            /* Enable error reporting for expressions and declarations with an implied `any` type.. */
    // "strictNullChecks": true,                         /* When type checking, take into account `null` and `undefined`. */
    // "strictFunctionTypes": true,                      /* When assigning functions, check to ensure parameters and the return values are subtype-compatible. */
    // "strictBindCallApply": true,                      /* Check that the arguments for `bind`, `call`, and `apply` methods match the original function. */
    // "strictPropertyInitialization": true,             /* Check for class properties that are declared but not set in the constructor. */
    // "noImplicitThis": true,                           /* Enable error reporting when `this` is given the type `any`. */
    // "useUnknownInCatchVariables": true,               /* Type catch clause variables as 'unknown' instead of 'any'. */
    // "alwaysStrict": true,                             /* Ensure 'use strict' is always emitted. */
    // "noUnusedLocals": true,                           /* Enable error reporting when a local variables aren't read. */
    // "noUnusedParameters": true,                       /* Raise an error when a function parameter isn't read */
    // "exactOptionalPropertyTypes": true,               /* Interpret optional property types as written, rather than adding 'undefined'. */
    // "noImplicitReturns": true,                        /* Enable error reporting for codepaths that do not explicitly return in a function. */
    // "noFallthroughCasesInSwitch": true,               /* Enable error reporting for fallthrough cases in switch statements. */
    // "noUncheckedIndexedAccess": true,                 /* Include 'undefined' in index signature results */
    // "noImplicitOverride": true,                       /* Ensure overriding members in derived classes are marked with an override modifier. */
    // "noPropertyAccessFromIndexSignature": true,       /* Enforces using indexed accessors for keys declared using an indexed type */
    // "allowUnusedLabels": true,                        /* Disable error reporting for unused labels. */
    // "allowUnreachableCode": true,                     /* Disable error reporting for unreachable code. */

    /* Completeness */
    // "skipDefaultLibCheck": true,                      /* Skip type checking .d.ts files that are included with TypeScript. */
    "skipLibCheck": true                                 /* Skip type checking all .d.ts files. */
  }
}
```
> webpack official site is [here](https://webpack.js.org/concepts/), if has any problem, the better thing is to visit official site to get guideline.

- Copy todo-app-ts files to this project. and it will has same result.

