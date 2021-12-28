---
title: news-pubish-management(5)--publish business
date: 2021-12-28 11:46:42
tags:
---
### 1, create a new component
```bash
mkdir src/components/publish-manage
touch src/components/publish-manage/NewsPublish.js
```
copy RightList.js to NewsPublish.js, modify content as below.
```javascript
import { Button, Table } from 'antd';

import React from 'react'

export default function NewsPublish(props) {
    const columns = [
        {
            title: '新闻标题',
            dataIndex: 'title',
            render: (title, item) => {
                return <a href={`#/news-manage/preview/${item.id}`}>{title}</a>
            }
        },
        {
            title: '作者',
            dataIndex: 'author',
        },
        {
            title: '新闻分类',
            dataIndex: 'category',
            render: category => {
                return <div>{category.title}</div>
            }
        },
        {
            title: '操作',
            render: item => {
                return <div>
                    <Button>Button</Button>
                </div>
            }
        },
    ];

    return (
        <div>
            <Table dataSource={props.dataSource} columns={columns} pagination={{ pageSize: 5 }} rowKey={item => item.id} />;
        </div>
    )
}
```
so the Unpublish.js looks like
```javascript
import axios from 'axios';
import React, { useEffect, useState } from 'react'
import NewsPublish from '../../../components/publish-manage/NewsPublish';

export default function Unpublished() {
    const [dataSource, setDataSource] = useState([])
    const { username } = JSON.parse(localStorage.getItem('token'));
    useEffect(() => {
        axios.get(`/api/news?author=${username}&publishState=1&_expand=category`).then(res => {
            console.log(res.data);
        })
    }, [username]);

    return (
        <div>
            <NewsPublish dataSource={dataSource}></NewsPublish>
        </div>
    )
}

```
### 2. Define a custom hook
Because Published.js and Sunset.js are similar with Unpublish.js, only the publishState is difference. So we can define a custom hook.
```bash
touch src/components/publish-manage/usePublish.js
```
content is cut from Unpublish.js
```javascript


import { useEffect, useState } from 'react'
import axios from 'axios';
import { notification } from 'antd';

export default function usePublish(type) {
    const [dataSource, setDataSource] = useState([])
    const { username } = JSON.parse(localStorage.getItem('token'));

    useEffect(() => {
        axios.get(`/api/news?author=${username}&publishState=${type}&_expand=category`).then(res => {
            // console.log(res.data);
            setDataSource(res.data);
        })
    }, [username, type]);

    const handlePublish = id => {
        // console.log(id);
        setDataSource(dataSource.filter(item => item.id !== id));

        axios.patch(`/api/news/${id}`, {
            publishStats: 2,
            publishTime: Date.now(),
        }).then(res => {
            notification.info({
                message: `Notification`,
                description:
                    `Please go to published box for reviewing your news`,
                placement: 'bottomRight',
            });
        })
    }

    const handleSunset = id => {
        setDataSource(dataSource.filter(item => item.id !== id));
        axios.patch(`/api/news/${id}`, {
            publishStats: 3,
        }).then(res => {
            notification.info({
                message: `Notification`,
                description:
                    `Please go to Sunset box for reviewing your news`,
                placement: 'bottomRight',
            });
        })
    }

    const handleDelete = id => {
        setDataSource(dataSource.filter(item => item.id !== id));

        axios.delete(`/api/news/${id}`).then(res => {
            notification.info({
                message: `Notification`,
                description:
                    `Your news was deleted`,
                placement: 'bottomRight',
            });
        })
    }

    return {
        dataSource,
        handlePublish,
        handleSunset,
        handleDelete,
    }
}

```

So the Unpublished.js change to 
```javascript
import { Button } from 'antd';
import React from 'react';
import NewsPublish from '../../../components/publish-manage/NewsPublish';
import usePublish from '../../../components/publish-manage/usePublish';

export default function Unpublished() {
    // 1 unpublished
    // 2 published
    // 3 sunset
    const { dataSource, handlePublish } = usePublish(1);
    return (
        <div>
            <NewsPublish dataSource={dataSource} button={id => <Button type='primary' onClick={() => handlePublish(id)}>发布</Button>}></NewsPublish>
        </div>
    )
}

```
and the NewsPublish.js change to
```javascript
import { Button, Table } from 'antd';

import React from 'react'

export default function NewsPublish(props) {
    const columns = [
        {
            title: '新闻标题',
            dataIndex: 'title',
            render: (title, item) => {
                return <a href={`#/news-manage/preview/${item.id}`}>{title}</a>
            }
        },
        {
            title: '作者',
            dataIndex: 'author',
        },
        {
            title: '新闻分类',
            dataIndex: 'category',
            render: category => {
                return <div>{category.title}</div>
            }
        },
        {
            title: '操作',
            render: item => {
                return <div>
                    {props.button(item.id)}
                </div>
            }
        },
    ];

    return (
        <div>
            <Table dataSource={props.dataSource} columns={columns} pagination={{ pageSize: 5 }} rowKey={item => item.id} />;
        </div>
    )
}
```
according the type, easy to peform Published.js and Sunset.js

