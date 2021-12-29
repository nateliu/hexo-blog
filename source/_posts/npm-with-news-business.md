---
title: news-pubish-management(4)--news business
date: 2021-12-27 12:41:52
tags:
---
We have finished the permission, let's back to focus the busines of publish news.

### 1. performing the <span style="color:lightblue">src/views/sandbox/news-manage/NewsAdd.js</span>
use [Header Basic](https://ant-design.gitee.io/components/page-header/#components-page-header-demo-basic) to perform the title.
```javascript
import { PageHeader } from 'antd'
...
            <PageHeader
                className="site-page-header"
                onBack={() => null}
                title="Add news"
                subTitle="This is for adding news"
            />
```
and we use [Step Basic](https://ant-design.gitee.io/components/steps/#components-steps-demo-simple) for the progress.
```javascript
    <Steps current={current}>
                <Step title="基本信息" description="新闻标题，新闻分类" />
                <Step title="新闻内容" description="新闻主体内容" />
                <Step title="新闻提交" description="保存草稿或者提交审核" />
    </Steps>
```
chose [Form Basic](https://ant-design.gitee.io/components/form/#components-form-demo-basic) for the input and select button
```javascript
<div style={{ marginTop: '50px' }}>
    <Form {...layout}>
        <Form.Item name="title" label="新闻标题" rules={[{ required: true, message: "Please input news title." }]}>
            <Input />
        </Form.Item>
        <Form.Item name="categoryId" label="新闻分类" rules={[{ required: true }]}>
            <Select>
                {
                    categoryList.map(item => {
                        return <Option value={item.id} key={item.id}>{item.title}</Option>
                    })
                }
            </Select>
        </Form.Item>
    </Form>
</div>
```
above we called step 1, next to step 2

### 2. Add news content.
use [react draft editor](https://github.com/jpuri/react-draft-wysiwyg) for edit news content.
```bash
yarn add react-draft-wysiwyg draft-js draftjs-to-html html-to-draftjs
mkdir src/components/news-manage
touch src/components/news-manage/NewsEditor.js
```
```javascript
import { convertToRaw } from 'draft-js'
import draftToHtml from 'draftjs-to-html';
import React, { useState } from 'react'
import { Editor } from 'react-draft-wysiwyg'
import 'react-draft-wysiwyg/dist/react-draft-wysiwyg.css'

export default function NewsEditor() {
    const [editorState, setEditorState] = useState("")
    return (
        <div>
            <Editor editorState={editorState}
                toolbarClassName='toolbarClassName'
                wrapperClassName='wrapperClassName'
                editorClassName='editorClassName'
                onEditorStateChange={editorState => setEditorState(editorState)}
                onBlur={() => {
                    console.log(draftToHtml(convertToRaw(editorState.getCurrentContent())))
                }} />
        </div>
    )
}
```

### 3. Add save in progress 3
use [Notification Basic](https://ant-design.gitee.io/components/notification/#components-notification-demo-basic) in step 3
```javascript
 const handleSave = auditState => {
        axios.post('/api/news', {
            ...formInfo,
            "author": User.username,
            "region": User.region ? User.region : 'Global',
            "roleId": User.roleId,
            "auditState": auditState,
            "publishState": 0,
            "content": content,
            "createTime": Date.now(),
            "star": 0,
            "view": 0
        }).then(res => {
            navigate(auditState === 0 ? '/news-manage/draft' : '/audit-manage/list');
            notification.info({
                message: `Notification`,
                description:
                    `Please go to ${auditState === 0 ? "draft box" : "audit box"} for reviewing your news`,
                placement: 'bottomRight',
            });
        });
    }
```
### 4. perform the draft box
```javascript
import { Button, Table, Modal } from 'antd';
import {
    DeleteOutlined,
    EditOutlined,
    ExclamationCircleOutlined,
    UploadOutlined,
} from '@ant-design/icons';

import axios from 'axios';
import React, { useEffect, useState } from 'react'

const { confirm } = Modal;

export default function NewsDraft() {
    const [dataSource, setDataSource] = useState([]);

    const { username } = JSON.parse(localStorage.getItem('token'));

    const columns = [
        {
            title: 'ID',
            dataIndex: 'id',
            render: (id) => <b>{id}</b>
        },
        {
            title: '新闻标题',
            dataIndex: 'title',
        },
        {
            title: '新闻分类',
            dataIndex: 'category',
            render: category => {
                return category.title
            }
        },
        {
            title: '作者',
            dataIndex: 'author',
        },
        {
            title: '操作',
            render: item => {
                return <div>
                    <Button shape='circle' icon={<DeleteOutlined />} onClick={() => confirmDelete(item)} />
                    <Button type='primary' shape='circle' icon={<EditOutlined />} />
                    <Button danger shape='circle' icon={<UploadOutlined />} />

                </div>
            }
        },
    ];

    const confirmDelete = item => {
        confirm({
            title: 'Do you Want to delete this item?',
            icon: <ExclamationCircleOutlined />,
            onOk() {
                // console.log('OK');
                realDelete(item)
            },
            onCancel() {
                console.log('Cancel');
            },
        });
    }

    const realDelete = (item) => {
        // console.log(item);
        setDataSource(dataSource.filter(data => data.id !== item.id));
        axios.delete(`/api/news/${item.id}`);
    }

    useEffect(() => {
        axios.get(`/api/news?_expand=category&author=${username}&auditState=0`).then(res => {
            // console.log(res.data);
            setDataSource(res.data);
        })
    }, [username]);

    return (
        <div>
            <Table dataSource={dataSource} columns={columns} pagination={{ pageSize: 5 }} rowKey={item => item.id} />;
        </div>
    )
}
```
support news preview function.
```bash
touch src/views/sandbox/news-manage/NewsPreview.js
yarn add moment
```
moment is used for format DateTime.
```javascript
import { PageHeader, Descriptions } from 'antd'
import axios from 'axios';
import moment from 'moment';
import React, { useEffect, useState } from 'react'
import { useParams } from 'react-router';

export default function NewsPreview() {
    const [newsInfo, setNewsInfo] = useState(null);

    const params = useParams();

    useEffect(() => {
        // console.log(props.match.params.id)
        axios.get(`/api/news/${params.id}?_expand=category&_expand=role`).then(res => {
            console.log(res.data);
            setNewsInfo(res.data);
        })
    }, [params.id]);

    const auditList = ['未审核', '审核中', '已通过', '未通过'];
    const publishList = ['未发布', '待发布', '已发布', '已下线'];

    return (
        newsInfo && <div>
            <PageHeader
                onBack={() => window.history.back()}
                title={newsInfo.title}
                subTitle={newsInfo.category?.title}>
                <Descriptions size="small" column={3}>
                    <Descriptions.Item label="创建者">{newsInfo.author}</Descriptions.Item>
                    <Descriptions.Item label="创建时间">{moment(newsInfo.createTime).format("YYYY/MM/DD HH:mm:ss")}</Descriptions.Item>
                    <Descriptions.Item label="发布时间">{newsInfo.publishTime ? moment(newsInfo.publishTime).format("YYYY/MM/DD HH:mm:ss") : "-"}</Descriptions.Item>
                    <Descriptions.Item label="区域">{newsInfo.region}</Descriptions.Item>
                    <Descriptions.Item label="审核状态"><span style={{ color: "red" }}>{auditList[newsInfo.auditState]}</span></Descriptions.Item>
                    <Descriptions.Item label="发布状态"><span style={{ color: "red" }}>{publishList[newsInfo.publishState]}</span></Descriptions.Item>
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
support edit news
```bash
touch src/views/sandbox/news-manage/NewsUpdate.js
```
most of code can copy from NewsAdd.js and modify to as below:
```javascript
import { PageHeader, Steps, Button, Form, Input, Select, message, notification } from 'antd'
import axios from 'axios';
import React, { useEffect, useState, useRef } from 'react'
import { useNavigate, useParams } from 'react-router';
import NewsEditor from '../../../components/news-manage/NewsEditor';
import style from './News.module.css'
const { Step } = Steps;
const { Option } = Select;

export default function NewsUpdate() {
    const layout = {
        labelCol: { span: 4 },
        wrapperCol: { span: 20 },
    };

    const User = JSON.parse(localStorage.getItem('token'));

    const [current, setCurrent] = useState(0);
    const [categoryList, setCategoryList] = useState([]);
    const [formInfo, setFormInfo] = useState({});
    const [content, setContent] = useState('')

    const newsForm = useRef(null);
    const navigate = useNavigate();
    const params = useParams();

    const handlePrevious = () => {
        setCurrent(current - 1);
    }

    const handleNext = () => {
        if (current === 0) {
            newsForm.current.validateFields().then(res => {
                // console.log(res);
                setFormInfo(res);
                setCurrent(current + 1);
            }).catch(err => {
                console.log(err);
            })
        } else {
            // console.log(formInfo,content);
            if (content === '' || content.trim() === '<p></p>') {
                message.error(`News content can not be empty!`);
            } else {
                setCurrent(current + 1);
            }
        }
    }

    const handleSave = auditState => {
        axios.patch(`/api/news/${params.id}`, {
            ...formInfo,
            "auditState": auditState,
            "content": content,
        }).then(res => {
            navigate(auditState === 0 ? '/news-manage/draft' : '/audit-manage/list');
            notification.info({
                message: `Notification`,
                description:
                    `Please go to ${auditState === 0 ? "draft box" : "audit box"} for reviewing your news`,
                placement: 'bottomRight',
            });
        });
    }

    useEffect(() => {
        axios.get(`/api/categories`).then(res => {
            // console.log(res.data);
            setCategoryList(res.data);
        })
    }, []);

    useEffect(() => {
        // console.log(props.match.params.id)
        axios.get(`/api/news/${params.id}?_expand=category&_expand=role`).then(res => {
            // console.log(res.data);
            // setNewsInfo(res.data);
            // content
            // formInfo
            const { title, categoryId, content } = res.data;

            newsForm.current.setFieldsValue({
                title,
                categoryId
            });
            setContent(content);
        })
    }, [params.id]);

    return (
        <div>
            <PageHeader
                className="site-page-header"
                onBack={() => navigate('/news-manage/draft')}
                title="Update news"
                subTitle="This is for update news"
            />
            <Steps current={current}>
                <Step title="基本信息" description="新闻标题，新闻分类" />
                <Step title="新闻内容" description="新闻主体内容" />
                <Step title="新闻提交" description="保存草稿或者提交审核" />
            </Steps>
            <div style={{ marginTop: '50px' }}>
                <div className={current === 0 ? '' : style.active}>
                    <Form {...layout} name='basic' ref={newsForm}>
                        <Form.Item name="title" label="新闻标题" rules={[{ required: true, message: "Please input news title." }]}>
                            <Input />
                        </Form.Item>
                        <Form.Item name="categoryId" label="新闻分类" rules={[{ required: true }]}>
                            <Select>
                                {
                                    categoryList.map(item => {
                                        return <Option value={item.id} key={item.id}>{item.title}</Option>
                                    })
                                }
                            </Select>
                        </Form.Item>
                    </Form>
                </div>
            </div>

            <div className={current === 1 ? '' : style.active}>
                <NewsEditor getContent={value => {
                    // console.log(value);
                    setContent(value);
                }} content={content}></NewsEditor>
            </div>

            <div className={current === 2 ? '' : style.active}>
            </div>

            <div style={{ marginTop: '50px' }}>
                {current > 0 && <Button onClick={handlePrevious}> 上一步</Button>}
                {current < 2 && <Button type='primary' onClick={handleNext}> 下一步</Button>}
                {
                    current === 2 && <sapn>
                        <Button type='primary' onClick={() => handleSave(0)}>保存草稿箱</Button>
                        <Button danger onClick={() => handleSave(1)}>提交审核</Button>
                    </sapn>
                }
            </div>
        </div>
    )
}
```
and the NewsEditor looks like below
```javascript
import { ContentState, convertToRaw, EditorState } from 'draft-js'
import draftToHtml from 'draftjs-to-html';
import htmlToDraft from 'html-to-draftjs';
import React, { useEffect, useState } from 'react'
import { Editor } from 'react-draft-wysiwyg'
import 'react-draft-wysiwyg/dist/react-draft-wysiwyg.css'

export default function NewsEditor(props) {
    const [editorState, setEditorState] = useState("");

    useEffect(() => {
        // console.log(props.content);
        const html = props.content;

        if (html === undefined) {
            return;
        }

        const contentBlock = htmlToDraft(html);
        if (contentBlock) {
            const contentState = ContentState.createFromBlockArray(contentBlock.contentBlocks);
            const editorState = EditorState.createWithContent(contentState);
            setEditorState(editorState);
        }
    }, [props.content]);

    return (
        <div>
            <Editor editorState={editorState}
                toolbarClassName='toolbarClassName'
                wrapperClassName='wrapperClassName'
                editorClassName='editorClassName'
                onEditorStateChange={editorState => setEditorState(editorState)}
                onBlur={() => {
                    props.getContent(draftToHtml(convertToRaw(editorState.getCurrentContent())));
                }} />
        </div>
    )
}

```
support Audit feature. bind a handleAudit event then fine for supporting this feature.

```javascript
const handleAudit = id => {
        axios.patch(`/api/news/${id}`, {
            auditState: 1
        }).then(res => {
            navigate('/audit-manage/list');
            notification.info({
                message: `Notification`,
                description:
                    `Please go to audit box for checking your news`,
                placement: 'bottomRight',
            });
        });
    }
```




