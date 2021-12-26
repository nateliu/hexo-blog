---
title: news-pubish-management(1)--layout
date: 2021-12-25 14:44:01
tags:
---
Here is step 1 for perform the layout for news-publish-management
### 1. Setup setupProxy.js
```bash
yarn add http-proxy-middleware
touch src/setupProxy.js
```
typing below code-snippets as content.
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
every js file add code snippets like below.

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
Let say src/router/IndexRouter.js is first level router, if user already login the system, then go to src/views/sandbox/NewsSandBox.js, otherwise it will go to src/views/login/Login.js. So the IndexRouter.js will look like below
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
And in src/App.js, importing our src/router/IndexRouter.js component
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
Go to brower,https//localhost:3000 will display Login components, if you manually setup the token with command of 
```javascript
localStorage.setItem("token","token")
```
then it will display the component of NewsSandBox

### 3. We assume NewsSandBox.js as second level router, so the content change to 
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
### 4. Import AntD library and setup layout, our layout designed in src/views/NewsSandBox.js
```bash
yarn add antd
```
We chose a customized layout and copy and modify the layout.

### 5. Add click event on the collapsed/expanded icon in TopHeader, here the React hooks useState is easier for understanding than the class component state.
```javascript
    const [collapsed, setCollapsed] = useState(false)

    const changeCollapsed = () => {
        setCollapsed(!collapsed)
    }
```
Then give the icon an onClick event property. for example
```javascript
<MenuFoldOutlined onClick={changeCollapsed} />
```

### 6. Add Wellcome user back in the TopHeader.
We can copy the [Dropdown](https://ant-design.gitee.io/components/dropdown-cn/) and [Avator](https://ant-design.gitee.io/components/avatar-cn/) component from AntD official website.

### 7. Modify the SideMenu
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
In this page, will use useEffect to get data back and use useState to control returned data
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
Finished above 7 steps, the page should be display correctly, but it is weird, some menu should be in second level route, but it is in SideMenu. another thing is all the menu without icon. Let's fix those two things:
for issue 1, we can identifiy the field pagepermission, check its 1 or not.
```javascript
    const checkPagePermission = (item) => {
        return item.pagepermission === 1
    }
```
> pagepermission is very important!!
for issue 2, we can define an iconList to store icon, and use array[index] to get the icon.
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
issue 3, The Home is no children, we do not need the expansed icon in the right, so we have to add children.length to check
```javascript
item.children?.length > 0 && checkPagePermission(item)
```
issue 4, the style is egly, we add some css in App.css
```css
@import '~antd/dist/antd.css';

::-webkit-scrollbar {width: 5px;height: 5px;position:absolute}
::-webkit-scrollbar-thumb {background-color: #1890ff;}
::-webkit-scrollbar-track {background-color: #ddd;}
```

issue 5, focus the previous select menu if user refresh page.
Menu component have SelectedKeys and defaultOpenKeys, we can set those two field to fix our issue.
In Ant library, most of time, if field called defaultXXXX, that meant it is uncontrolled field, otherwise it will be useState for controlling.
```javascript
    const location = useLocation();

    const selectKeys = [location.pathname]; 
    const openKeys = ["/" + location.pathname.split("/")[1]];

    ....

    <Menu theme="dark" mode="inline" selectedKeys={selectKeys} defaultOpenKeys={openKeys}>
                        {renderMenu(menuList)}
                    </Menu>
```




