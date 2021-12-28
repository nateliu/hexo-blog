---
title: news-pubish-management(7)--guest system
date: 2021-12-28 19:38:59
tags:
---
This section we are going to finish the guest system.

### 1. create a guest system inforstructor.
```bash
mkdir src/views/news
touch src/views/news/News.js
touch src/views/news/Detail.js
```
rfc create those 2 components. Go to IndexRouter.js. add 2 routes.
```javascript
import Detail from '../views/news/Detail'
import News from '../views/news/News'
...
<Route path="/news" element={<News />} />
<Route path="/detail/:id" element={<Detail />} />
```
### 2. News component
we use [PageHeader](https://ant-design.gitee.io/components/page-header-cn/#header) and [Card](https://ant-design.gitee.io/components/card-cn/#header) and [List](https://ant-design.gitee.io/components/list-cn/#header) to perform this feature.
```javascript
import { PageHeader, Card, Col, Row, List } from 'antd';
import React, { useEffect, useState } from 'react';
import _ from 'lodash';
import axios from 'axios';

export default function News() {
    const [list, setList] = useState([]);
    useEffect(() => {
        axios.get(`/api/news?publisthState=2&_expand=category`).then(res => {
            const twoDims = Object.entries(_.groupBy(res.data, item => item.category.title));
            // console.log(twoDims);
            setList(twoDims);
        })
    }, []);

    return (
        <div style={{
            width: "95%",
            margin: "0 auto"
        }}>
            <PageHeader
                className="site-page-header"
                title="全球大新闻"
                subTitle="查看新闻"
            />
            <div className="site-card-wrapper">
                <Row gutter={[16, 16]}>
                    {
                        list.map(item => {
                            return <Col span={8} key={item[0]}>
                                <Card title={item[0]} bordered={true} hoverable={true}>
                                    <List
                                        size="small"
                                        dataSource={item[1]}
                                        pagination={{
                                            pageSize: 3
                                        }}
                                        renderItem={data => <List.Item>
                                            <a href={`#/detail/${data.id}`}>{data.title}</a>
                                        </List.Item>}
                                    />
                                </Card>
                            </Col>
                        })
                    }
                </Row>
            </div>
        </div>
    )
}

```
### 3. Detail component
copy from src/views/sandbox/news-manage/NewsPreviews.js and modify as below.
```javascript
import { HeartTwoTone } from '@ant-design/icons';
import { PageHeader, Descriptions } from 'antd'
import axios from 'axios';
import moment from 'moment';
import React, { useEffect, useState } from 'react'
import { useParams } from 'react-router';

export default function Detail() {
    const [newsInfo, setNewsInfo] = useState(null);

    const params = useParams();

    const handleStar = () => {
        setNewsInfo({
            ...newsInfo,
            star: newsInfo.star + 1
        });

        axios.patch(`/api/news/${params.id}`, {
            star: newsInfo.star + 1
        })
    }

    useEffect(() => {
        // console.log(props.match.params.id)
        axios.get(`/api/news/${params.id}?_expand=category&_expand=role`).then(res => {
            // console.log(res.data);
            setNewsInfo({
                ...res.data,
                view: res.data.view + 1
            });

            return res.data;
        }).then(data => {
            axios.patch(`/api/news/${params.id}`, {
                view: data.view + 1
            })
        })
    }, [params.id]);

    return (
        newsInfo && <div>
            <PageHeader
                onBack={() => window.history.back()}
                title={newsInfo.title}
                subTitle={
                    <div>
                        {newsInfo.category?.title}
                        <HeartTwoTone twoToneColor="#eb2f96" onClick={() => handleStar()} />
                    </div>
                }>
                <Descriptions size="small" column={3}>
                    <Descriptions.Item label="创建者">{newsInfo.author}</Descriptions.Item>
                    <Descriptions.Item label="发布时间">{newsInfo.publishTime ? moment(newsInfo.publishTime).format("YYYY/MM/DD HH:mm:ss") : "-"}</Descriptions.Item>
                    <Descriptions.Item label="区域">{newsInfo.region}</Descriptions.Item>
                    <Descriptions.Item label="访问数量">{newsInfo.view}</Descriptions.Item>
                    <Descriptions.Item label="点赞数量">{newsInfo.star}</Descriptions.Item>
                    <Descriptions.Item label="评论数量">0</Descriptions.Item>
                </Descriptions>
            </PageHeader>
            <div dangerouslySetInnerHTML={{
                __html: newsInfo.content
            }} style={{
                margin: "0 24px",
                border: "1px solid gray"
            }}>
            </div>
        </div>
    )
}

```