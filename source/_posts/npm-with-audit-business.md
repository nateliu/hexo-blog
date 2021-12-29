---
title: news-pubish-management(5)--audit business
date: 2021-12-27 20:37:52
tags:
---
The audit list should be showing date the user ownself, audiState should not equals to 0 and publishState should less than 1.
### 1. Get back data in `AuditList.js`
copy from RightList.js and mofify as below
```javascript
import { Button, Table, Tag } from 'antd';

import axios from 'axios';
import React, { useEffect, useState } from 'react'

export default function AuditList() {
    const { username } = JSON.parse(localStorage.getItem('token'));

    const [dataSource, setDataSource] = useState([]);

    const columns = [
        {
            title: '新闻标题',
            dataIndex: 'title',
            render: (title, item) => {
                return <a href={`#/news-manage/preview/${item.id}`}>{title}</a>
            }
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
            title: '审核状态',
            dataIndex: 'auditState',
            render: auditState => {
                const colorList = ['', 'orange', 'green', 'red'];
                const auditList = ['未审核', '审核中', '已通过', '未通过'];
                return <Tag color={colorList[auditState]}>{auditList[auditState]}</Tag>
            }
        },
        {
            title: '操作',
            render: item => {
                return <div>
                    {
                        item.auditState === 1 && <Button danger>撤销</Button>
                    }
                    {
                        item.auditState === 2 && <Button>发布</Button>
                    }
                    {
                        item.auditState === 3 && <Button type='primary'>更新</Button>
                    }
                </div>
            }
        },
    ];

    useEffect(() => {
        axios.get(`/api/news?_expand=category&author=${username}&auditState_ne=0&publishState_lte=1`).then(res => {
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
```javascript
const auditList = ['未审核', '审核中', '已通过', '未通过'];
```
you wan to revert from audit. it is easy, change the auditState from 2 to 0.
```javascript
    const navigate = useNavigate();

    const handleRevert = item => {
        setDataSource(dataSource.filter(data=>data.id!==item.id));
        
        axios.patch(`/api/news/${item.id}`, {
            auditStats: 0
        }).then(res=>{
            navigate('/news-manage/draft');
            notification.info({
                message: `Notification`,
                description:
                    `Please go to draft box for reviewing your news`,
                placement: 'bottomRight',
            });
        })
    }
```
update is easy, useNavigate to NewsUpdate.
```javascript
    const handleUpdate = item => {
        navigate(`/news-manage/update/${item.id}`);
    }
```
publish news, change the publishState from 1 to 2
```javascript
    const handlePublish = item => {
        axios.patch(`/api/news/${item.id}`, {
            publishStats: 2
        }).then(res => {
            navigate('/publish-manage/published');
            notification.info({
                message: `Notification`,
                description:
                    `Please go to published box for reviewing your news`,
                placement: 'bottomRight',
            });
        })
    }
```
### 2. Audit news
copy from AuditList.js and modify as below
```javascript
import { Table, Button } from 'antd';
import axios from 'axios';
import React, { useEffect, useState } from 'react'
import { useNavigate } from 'react-router'

export default function Audit() {
    const [dataSource, setDataSource] = useState([])

    const navigate = useNavigate();

    const { roleId, region, username } = JSON.parse(localStorage.getItem('token'));

    const columns = [
        {
            title: '新闻标题',
            dataIndex: 'title',
            render: (title, item) => {
                return <a href={`#/news-manage/preview/${item.id}`}>{title}</a>
            }
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
                    <Button type='primary'>通过</Button>
                    <Button >驳回</Button>
                </div>
            }
        },
    ];

    useEffect(() => {
        const roleObj = {
            "1": "superadmin",
            "2": "admin",
            "3": "editor",
        }

        axios.get(`/api/news?auditState=1&_expand=category`).then(res => {
            console.log(res.data);
            const list = res.data;
            setDataSource(roleObj[roleId] === 'superadmin' ? list : [
                ...list.filter(item => item.username === username),
                ...list.filter(item => item.region === region && roleObj[item.roleId] === 'editor')
            ]);
        });
    }, [roleId, region, username]);

    return (
        <div>
            <Table dataSource={dataSource} columns={columns} pagination={{ pageSize: 5 }} rowKey={item => item.id} />;
        </div>
    )
}

```
Audit & Reject
audit change the auditState from 1 to 2, publishState change to 1, 
reject change the auditState from 1 to 3, publishState change to 0
```javascript
    const handleAudit = (item, auditState, publishState) => {
        setDataSource(dataSource.filter(data => data.id !== item.id));

        axios.patch(`/api/news/${item.id}`, {
            auditState,
            publishState
        }).then(res=>{

        })
    }
```

### 3. Add NewsCategory
copy RightList.js and modify it as below:
```javascript
import { Button, Table, Modal } from 'antd';
import {
    DeleteOutlined,
    ExclamationCircleOutlined,
} from '@ant-design/icons';

import axios from 'axios';
import React, { useEffect, useState } from 'react'

const { confirm } = Modal;

export default function NewsCategory() {
    const [dataSource, setDataSource] = useState([]);
    const columns = [
        {
            title: 'ID',
            dataIndex: 'id',
            render: (id) => <b>{id}</b>
        },
        {
            title: '分类名称',
            dataIndex: 'title',
        },
        {
            title: '操作',
            render: item => {
                return <div>
                    <Button danger shape='circle' icon={<DeleteOutlined />} onClick={() => confirmDelete(item)} />
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
        axios.delete(`/api/categories/${item.id}`);
    }

    useEffect(() => {
        axios.get(`/api/categories`).then(res => {
            // console.log(res.data);
            setDataSource(res.data);
        })
    }, []);

    return (
        <div>
            <Table dataSource={dataSource} columns={columns} pagination={{ pageSize: 5 }} rowKey={item => item.id} />;
        </div>
    )
}
```
use table [Table EditCell](https://ant-design.gitee.io/components/table/#components-table-demo-edit-cell) for editing title

add components proeprty in Table, the components nees EditableRow and EditableCell, it is complex to read this document but need careful, at the end the NewsCategory should be looks like
```javascript
import { Button, Table, Modal, Form, Input } from 'antd';
import {
    DeleteOutlined,
    ExclamationCircleOutlined,
} from '@ant-design/icons';

import axios from 'axios';
import React, { useContext, useState, useEffect, useRef } from 'react'

const { confirm } = Modal;

export default function NewsCategory() {
    const [dataSource, setDataSource] = useState([]);

    const EditableContext = React.createContext(null);

    const columns = [
        {
            title: 'ID',
            dataIndex: 'id',
            render: (id) => <b>{id}</b>
        },
        {
            title: '分类名称',
            dataIndex: 'title',
            onCell: (record) => ({
                record,
                editable: true,
                dataIndex: 'title',
                title: '分类名称',
                handleSave: handleCellSave,
            }),
        },
        {
            title: '操作',
            render: item => {
                return <div>
                    <Button danger shape='circle' icon={<DeleteOutlined />} onClick={() => confirmDelete(item)} />
                </div>
            }
        },
    ];

    const handleCellSave = record => {
        // console.log(record);
        setDataSource(dataSource.map(item => {
            if (item.id === record.id) {
                return {
                    id: item.id,
                    title: record.title,
                    value: record.title,
                }
            }
            return item;
        }))

        axios.patch(`/api/categories/${record.id}`, {
            title: record.title,
            value: record.title,
        })
    }

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
        axios.delete(`/api/categories/${item.id}`);
    }

    useEffect(() => {
        axios.get(`/api/categories`).then(res => {
            // console.log(res.data);
            setDataSource(res.data);
        })
    }, []);

    const EditableRow = ({ index, ...props }) => {
        const [form] = Form.useForm();
        return (
            <Form form={form} component={false}>
                <EditableContext.Provider value={form}>
                    <tr {...props} />
                </EditableContext.Provider>
            </Form>
        );
    };

    const EditableCell = ({
        title,
        editable,
        children,
        dataIndex,
        record,
        handleSave,
        ...restProps
    }) => {
        const [editing, setEditing] = useState(false);
        const inputRef = useRef(null);
        const form = useContext(EditableContext);
        useEffect(() => {
            if (editing) {
                inputRef.current.focus();
            }
        }, [editing]);

        const toggleEdit = () => {
            setEditing(!editing);
            form.setFieldsValue({
                [dataIndex]: record[dataIndex],
            });
        };

        const save = async () => {
            try {
                const values = await form.validateFields();
                toggleEdit();
                handleSave({ ...record, ...values });
            } catch (errInfo) {
                console.log('Save failed:', errInfo);
            }
        };

        let childNode = children;

        if (editable) {
            childNode = editing ? (
                <Form.Item
                    style={{
                        margin: 0,
                    }}
                    name={dataIndex}
                    rules={[
                        {
                            required: true,
                            message: `${title} is required.`,
                        },
                    ]}
                >
                    <Input ref={inputRef} onPressEnter={save} onBlur={save} />
                </Form.Item>
            ) : (
                <div
                    className="editable-cell-value-wrap"
                    style={{
                        paddingRight: 24,
                    }}
                    onClick={toggleEdit}
                >
                    {children}
                </div>
            );
        }

        return <td {...restProps}>{childNode}</td>;
    };

    return (
        <div>
            <Table dataSource={dataSource} columns={columns} pagination={{ pageSize: 5 }} rowKey={item => item.id} components={{
                body: {
                    row: EditableRow,
                    cell: EditableCell,
                },
            }} />;
        </div>
    )
}
```




