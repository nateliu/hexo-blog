---
title: news-pubish-management(3)--login
date: 2021-12-27 08:39:53
tags:
---
It's time to implement a Login componnet.
### 1. Login out in TopHeader.js
```javascript
import { useNavigate } from 'react-router';

const navigate = useNavigate();

<Menu.Item danger key='loginOutMenu' onClick={()=>{
            localStorage.removeItem('token');
            navigate('/login');
}}>Login out</Menu.Item>

```
### 2. create Login component in src/views/login/Login.js
we use for performing [particles](https://github.com/matteobruni/tsparticles/tree/main/components/react)
```bash
yarn add react-tsparticles

touch src/views/login/Login.css
```
The css looks like below:
```css
.formContainer {
    position: fixed;
    background-color: rgba(0,0,0,0.7);
    width: 500px;
    height: 300px;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    padding: 20px;
    z-index: 100;
}
.loginTitle {
    text-align: center;
    height: 80px;
    line-height: 80px;
    font-size: 30px;
    color:white;
}
```
We use localStorage to store token for verifing and retrieving.
```javascript
    const navigate = useNavigate();

    const onFormFinish = (values) => {
        // console.log(values);

        //Because the limition of Json-server, here we chose verb get. we should use post in real db.
        axios.get(`/api/users?_expand=role&username=${values.username}&password=${values.password}&roleState=true`).then(res => {
            // console.log(res.data);
            if(res.data.length === 0) {
                message.error(`UserName or Password is wrong!`)
            } else {
                localStorage.setItem('token',JSON.stringify(res.data[0]));
                navigate('/');
            }
        })
    };
```

### 3. Back to TopHeader.js to perform and add real information in there.
```javascript
const { role: { roleName }, username } = JSON.parse(localStorage.getItem('token'));
```
### 4. Checking the SideMenu, we should guaratee difference user has difference menu.
```javascript
    const { role: { rights } } = JSON.parse(localStorage.getItem('token'));

    const checkPagePermission = (item) => {
        return item.pagepermission === 1 && rights.includes(item.key);
    }
```

### 5. locale admin only see his region user and himself, perform in UserList.js
```javascript
    const { roleId, region, username } = JSON.parse(localStorage.getItem('token'));
    const roleObj = {
        "1": "superadmin",
        "2": "admin",
        "3": "editor",
    }
    ...
    useEffect(() => {
        axios.get(`/api/users?_expand=role`).then(res => {
            // console.log(res.data);
            // setDataSource(res.data);
            const list = res.data;
            setDataSource(roleObj[roleId] === 'superadmin' ? list : [
                ...list.filter(item => item.username === username),
                ...list.filter(item => item.region === region && roleObj[item.roleId] === 'editor')
            ]);
        });
    }, []);
```

### 6. Difference use has difference permission in add and edit user form.
```javascript
const { roleId, region } = JSON.parse(localStorage.getItem('token'));
    const roleObj = {
        "1": "superadmin",
        "2": "admin",
        "3": "editor",
    }

    const checkRegionDisabled = item => {
        if (roleObj[roleId] === 'superadmin') {
            return false;
        }

        if (props.isUpdate) {
            return true;
        }

        return item.value !== region;
    }

    const checkRoleDisabled = item => {
        if (roleObj[roleId] === 'superadmin') {
            return false;
        }

        if (props.isUpdate) {
            return true;
        }

        return roleObj[item.id] !== 'editor';
    }
```
### 7. guratee deep link for difference user.
```bash
touch src/components/sandbox/NewsRouter.js
```
This is a second level router, the content we can copy from NewsSandBox.js
```javascript
import React from 'react'
import { Routes, Route, Navigate } from 'react-router-dom'
import Home from '../../views/sandbox/home/Home'
import NoPermission from '../../views/sandbox/nopermission/NoPermission'
import RightList from '../../views/sandbox/right-manage/RightList'
import RoleList from '../../views/sandbox/right-manage/RoleList'
import UserList from '../../views/sandbox/user-manage/UserList'

export default function NewsRouter() {
    return (
        <Routes>
            <Route path="home" element={<Home />} />
            <Route path="user-manage/list" element={<UserList />} />
            <Route path="right-manage/role/list" element={<RoleList />} />
            <Route path="right-manage/right/list" element={<RightList />} />
            <Route path="/" element={<Navigate replace from="/" to="home" />} />
            <Route path="/*" element={<NoPermission />} />
        </Routes>

    )
}
```
### 8 Create route dynamiclly
```bash
mkdir src/views/sandbox/news-manage
touch src/views/sandbox/news-manage/NewsAdd.js
touch src/views/sandbox/news-manage/NewsDraft.js
touch src/views/sandbox/news-manage/NewsCategory.js
mkdir src/views/sandbox/audit-manage
touch src/views/sandbox/audit-manage/Audit.js
touch src/views/sandbox/audit-manage/AuditList.js
mkdir src/views/sandbox/publish-manage
touch src/views/sandbox/publish-manage/Unpublished.js
touch src/views/sandbox/publish-manage/Published.js
touch src/views/sandbox/publish-manage/Sunset.js
```
defined a localRouteMap and use map for crating route.
```javascript
import axios from 'axios'
import React, { useEffect, useState } from 'react'
import { Routes, Route, Navigate } from 'react-router-dom'
import Audit from '../../views/sandbox/audit-manage/Audit'
import AuditList from '../../views/sandbox/audit-manage/AuditList'
import Home from '../../views/sandbox/home/Home'
import NewsAdd from '../../views/sandbox/news-manage/NewsAdd'
import NewsCategory from '../../views/sandbox/news-manage/NewsCategory'
import NewsDraft from '../../views/sandbox/news-manage/NewsDraft'
import NoPermission from '../../views/sandbox/nopermission/NoPermission'
import Published from '../../views/sandbox/publish-manage/Published'
import Sunset from '../../views/sandbox/publish-manage/Sunset'
import Unpublished from '../../views/sandbox/publish-manage/Unpublished'
import RightList from '../../views/sandbox/right-manage/RightList'
import RoleList from '../../views/sandbox/right-manage/RoleList'
import UserList from '../../views/sandbox/user-manage/UserList'

const LocalRouterMap = {
    "/home": <Home />,
    "/user-manage/list": <UserList />,
    "/right-manage/role/list": <RoleList />,
    "/right-manage/right/list": <RightList />,
    "/news-manage/add": <NewsAdd />,
    "/news-manage/draft": <NewsDraft />,
    "/news-manage/category": <NewsCategory />,
    "/audit-manage/audit": <Audit />,
    "/audit-manage/list": <AuditList />,
    "/publish-manage/unpublished": <Unpublished />,
    "/publish-manage/published": <Published />,
    "/publish-manage/sunset": <Sunset />,
};

export default function NewsRouter() {
    const [BackRouteList, setBackRouteList] = useState([]);

    useEffect(() => {
        Promise.all([
            axios.get(`/api/rights`),
            axios.get(`/api/children`),
        ]).then((res) => {
            // console.log(res.data);
            setBackRouteList([...res[0].data, ...res[1].data]);
        });
    }, []);

    return (
        <Routes>
            {BackRouteList.map(item => (
                <Route
                    path={item.key}
                    key={item.key}
                    element={LocalRouterMap[item.key]}
                />
            ))}
            <Route path="/" element={<Navigate replace from="/" to="home" />} />
            <Route path="/*" element={<NoPermission />} />
        </Routes>

    )
}
```
add permission for route checking and user checking
```javasript
    const checkRoute = item => {
        return LocalRouterMap[item.key] && item.pagepermission;
    }

    const checkUserPermission = item => {
        return rights.includes(item.key);
    }

    const { role: { rights } } = JSON.parse(localStorage.getItem('token'));
```

### 9. Add progress in NewsSandBox
```bash
yarn add nprogress
```
we use [NProcess](https://github.com/rstacruz/nprogress) to peform this progress feature.
```javascript
import nProgress from 'nprogress'

import 'nprogress/nprogress.css'
    
    nProgress.start()
    useEffect(() => {
        nProgress.done();
    })

```