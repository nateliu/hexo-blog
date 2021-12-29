---
title: news-pubish-management(1)--layout
date: 2021-12-25 14:44:01
tags:
---
Here I am going to perform the layout first for news-publish-management
### 1. Setup setupProxy.js
```bash
yarn add http-proxy-middleware
touch src/setupProxy.js
```
copy below [code-snippets](https://create-react-app.dev/docs/proxying-api-requests-in-development/#configuring-the-proxy-manually) into <span style="color:lightblue">setupProxy</span>.
```javascript
const { createProxyMiddleware } = require('http-proxy-middleware');

module.exports = function(app) {
  app.use(
    '/api',
    createProxyMiddleware({
      target: 'https://news-publish-management.herokuapp.com',
      changeOrigin: true,
    })
  );
};
````
> Why we setup like this in our project? The main reason for doing this is avoids [CORS](https://stackoverflow.com/questions/21854516/understanding-ajax-cors-and-security-considerations) issues 
### 2. Setup router
```bash
yarn add react-router-dom

mkdir src/router                  
touch src/router/IndexRouter.js

mkdir src/views
mkdir src/views/login
touch src/views/login/Login.js
mkdir src/views/sandbox
touch src/views/sandbox/NewsSandBox.js
mkdir src/views/sandbox/home
touch src/views/sandbox/home/Home.js
mkdir src/views/sandbox/user-manage
touch src/views/sandbox/user-manage/UserList.js
mkdir src/views/sandbox/right-manage
touch src/views/sandbox/right-manage/RightList.js
touch src/views/sandbox/right-manage/RoleList.js
mkdir src/views/sandbox/nopermission
touch src/views/sandbox/nopermission/NoPermission.js

mkdir src/components 
mkdir src/components/sandbox
touch src/components/sandbox/SideMenu.js
touch src/components/sandbox/TopHeader.js

```
Every each JS file we can add below code-snippets for creating react function component quickly.

```javascript
import React from 'react'

export default function ComponentName() {
    return (
        <div>
            ComponentName
        </div>
    )
}
```
> We can install a plugin [ES7 React/Redux/GraphQL/React-Native snippets](https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets) for [VS Code](https://code.visualstudio.com/). Once install successfully, use key `rfc` for lightning generating above code-snippets.

We assume <span style="color:lightblue">src/router/IndexRouter.js</span> is the first level router, when user already logined the system,  they will be redirected to <span style="color:lightblue">src/views/sandbox/NewsSandBox.js</span>, otherwise they will be redireted to <span style="color:lightblue">src/views/login/Login.js</span>. So the <span style="color:lightblue">IndexRouter.js</span> looks like below
```javascript
import React from 'react'
import { HashRouter, Routes, Route, Navigate } from 'react-router-dom'
import Login from '../views/login/Login'
import NewsSandBox from '../views/sandbox/NewsSandBox'

export default function IndexRouter() {
    return (
        <HashRouter>
            <Routes>
                <Route path="/login" element={<Login/>} />
                <Route path="*" element={localStorage.getItem("token") ? <NewsSandBox/> : <Navigate to="/login" />} />
            </Routes>
        </HashRouter>
    )
}
```
Go to <span style="color:lightblue">src/App.js</span>, and import our <span style="color:lightblue">src/router/IndexRouter.js</span> component.
```javascript
import React from 'react'
import IndexRouter from './router/IndexRouter'

export default function App() {
  return (
    <div>
      <IndexRouter/>
    </div>
  )
}
```
Go to browser, url `https//localhost:3000` will show our `Login` component. If you manually setup the token with command of 
```javascript
localStorage.setItem("token","token")
```
then the browser will show `NewsSandBox` component.

### 3. Assume <span style="color:lightblue">NewsSandBox.js </spn> as a second level router, and change its content to 
```javascript
import React from 'react'
import { Routes, Route, Navigate } from 'react-router-dom'
import SideMenu from '../../components/sandbox/SideMenu'
import TopHeader from '../../components/sandbox/TopHeader'
import Home from './home/Home'
import NoPermission from './nopermission/NoPermission'
import RightList from './right-manage/RightList'
import RoleList from './right-manage/RoleList'
import UserList from './user-manage/UserList'

export default function NewsSandBox() {
    return (
        <div>
            <SideMenu />
            <TopHeader />
            <Routes>
                <Route path="home" element={<Home />} />
                <Route path="user-manage/list" element={<UserList />} />
                <Route path="right-manage/role/list" element={<RoleList />} />
                <Route path="right-manage/right/list" element={<RightList />} />
                <Route path="/" element={<Navigate replace from="/" to="home" />} />
                <Route path="/*" element={<NoPermission />} />
            </Routes>
        </div>
    )
}
```
### 4. Import [Ant Design](https://ant-design.gitee.io/) and setup layout, our layout was designed in <span style="color:lightblue">src/views/sandbox/NewsSandBox.js</span>. `NewsSandBox` contains <span style="color:lightblue">src/components/sandbox/TopHeader.js</span> and <span style="color:lightblue">src/componets/sandbox/SideMenu.js`
```bash
yarn add antd
```
choose the [custom trigger](https://ant-design.gitee.io/components/layout/#components-layout-demo-custom-trigger) as our layout. First thing is `show code` and copy code into `NewsSandBox` and then sperate them as below <br/>
- `Layout.Sider` was pasted to `SideMenu`
- `Layout.Header` was pasted to `TopHeader`

> The import thing is don't forget to copy the CSS into our <span style="color:lightblue">src/views/sandbox/NewsSandBox.css</span> file.

### 5. Add click event on the collapsed/expanded icon in <span style="color:lightblue">src/components/sandbox/TopHeader.js</span>.
```javascript
    const [collapsed, setCollapsed] = useState(false)

    const changeCollapsed = () => {
        setCollapsed(!collapsed)
    }
```
> [React hooks](https://reactjs.org/docs/hooks-state.html)  is important. Here we use the `useState`.

Then give the icon an onClick event property. for example
```javascript
<MenuFoldOutlined onClick={changeCollapsed} />
```

### 6. Add `Wellcome user` in the TopHeader.
We copy the [Dropdown Basic](https://ant-design.gitee.io/components/dropdown/#components-dropdown-demo-basic) and [Avator Basic](https://ant-design.gitee.io/components/avatar/#components-avatar-demo-basic) components and modify them as we expected.

### 7. Modify the SideMenu
use [axios](https://axios-http.com/docs/intro) for getting data.
```bash
yarn add axios
mkdir src/util
touch src/util/http.js
```
http.js with content
```javascript
import axios from "axios";

axios.defaults.baseURL="https://news-publish-management.herokuapp.com"

// axios.defaults.headers
// axios.interceptors.request.use
// axios.interceptors.response.use
```
In this page, will use hook `useEffect` for getting data back and use  hook `useState` to control returned data
``` javascript
import React, { useEffect, useState } from 'react'
import { Layout, Menu } from 'antd';
import {
    UserOutlined,
    HomeOutlined,
    CrownOutlined,
} from '@ant-design/icons';

import './index.css'
import { useNavigate } from 'react-router';
import axios from 'axios';

const { Sider } = Layout;
const { SubMenu } = Menu;

export default function SideMenu() {
    const navigate = useNavigate();
    const [menuList, setMenuList] = useState([]);

    const renderMenu = (menuList) => {
        return menuList.map(item => {
            if (item.children) {
                return <SubMenu key={item.key} icon={item.icon} title={item.title}>
                    {renderMenu(item.children)}
                </SubMenu>
            }
            return <Menu.Item key={item.key} icon={item.icon} onClick={() => navigate(item.key)}>
                {item.title}</Menu.Item>
        })
    }

    useEffect(() => {
        axios.get("/api/rights?_embed=children").then(res=>{
            // console.log(res.data)
            setMenuList(res.data);
        })
    }, [])

    return (
        <Sider trigger={null} collapsible collapsed={false}>
            <div className="logo">News Publish Management</div>
            <Menu theme="dark" mode="inline" defaultSelectedKeys={['1']}>
                {renderMenu(menuList)}
            </Menu>
        </Sider>
    )
}
```
### 8. Fix some issue for SideMenu
Finished above 7 steps, the page should be correctly rendering, but it is weird, some menu should be in second level route, but it is in SideMenu, another thing is each of the menu without icon. Let's fix those issues we identified so far:
- For issue 1, we can identifiy the field pagepermission, check its 1 or not.
```javascript
    const checkPagePermission = (item) => {
        return item.pagepermission === 1
    }
```
> pagepermission is very important!!
- For issue 2, we can define an iconList to store icon, and use array[index] to get the icon.
```javascript
    const iconList = {
    '/home':<HomeOutlined />,
    '/user-manage':<UserOutlined />,
    '/user-manage/list':<UserOutlined />,
    '/right-manage':<CrownOutlined />,
    '/right-manage/role/list':<CrownOutlined />,
    '/right-manage/right/list':<CrownOutlined />,
}
```
- Issue 3, The Home is no children, we do not need the expansed icon in the right, so we have to add children.length to check
```javascript
item.children?.length > 0 && checkPagePermission(item)
```
- Issue 4, the style is egly, we import `antd.css` into `App.css`
```css
@import '~antd/dist/antd.css';

::-webkit-scrollbar {width: 5px;height: 5px;position:absolute}
::-webkit-scrollbar-thumb {background-color: #1890ff;}
::-webkit-scrollbar-track {background-color: #ddd;}
```

-Issue 5, focus the previous select menu if user refresh page.
Menu component have SelectedKeys and defaultOpenKeys, we can set those two field to fix our issue.
In Ant library, most of time, if field called defaultXXXX, that meant it is uncontrolled field, otherwise it use useState for controlling.
```javascript
    const location = useLocation();

    const selectKeys = [location.pathname]; 
    const openKeys = ["/" + location.pathname.split("/")[1]];

    ....

    <Menu theme="dark" mode="inline" selectedKeys={selectKeys} defaultOpenKeys={openKeys}>
                        {renderMenu(menuList)}
                    </Menu>
```