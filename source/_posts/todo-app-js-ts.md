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
