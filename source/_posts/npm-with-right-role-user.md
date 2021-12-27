---
title: news-pubish-management(2)--right, role and user
date: 2021-12-26 11:31:31
tags:
---
Today let back to perform the right and role in news-publish-management
### 1. Use the [Table](https://ant-design.gitee.io/components/table-cn/) component in RightList.js
```javascript
import { Table } from 'antd';
import axios from 'axios';
import React, { useEffect, useState } from 'react'

export default function RightList() {
    const [dataSource, setDataSource] = useState([]);
    const columns = [
        {
            title: 'ID',
            dataIndex: 'id',
        },
        {
            title: '权限名称',
            dataIndex: 'title',
        },
        {
            title: '权限名称',
            dataIndex: 'key',
        },
    ];

    useEffect(() => {
        axios.get("/api/rights").then(res => {
            // console.log(res.data);
            setDataSource(res.data);
        })
    }, []);

    return (
        <div>
            <Table dataSource={dataSource} columns={columns} />;
        </div>
    )
}
```
### 2.Custom some column, we can use render property, render proeprty must a function. function first in Ant
```javascript
const columns = [
        {
            title: 'ID',
            dataIndex: 'id',
            render: (id)=> <b>{id}</b>
        },
        {
            title: '权限名称',
            dataIndex: 'title',
        },
        {
            title: '权限路径',
            dataIndex: 'key',
            render: (key)=>{
                return <Tag color='orange'>{key}</Tag>
            }
        },
    ];
```
### 3. Add operation column in the table component
```javascript
import {
    DeleteOutlined,
    EditOutlined,
} from '@ant-design/icons';
...
    {
            title: '操作',
            render: () => {
                return <div>
                    <Button danger shape='circle' icon={<DeleteOutlined />} />
                    <Button type='primary' shape='circle' icon={<EditOutlined />} />
                </div>
            }
        }
```
### 4. Add pagenition for Table
```javascript
<Table dataSource={dataSource} columns={columns} pagination={{ pageSize: 5 }} />;
```

### 5. use Tree to display children, it is pretty easy, change the useEffect to
```javascript
    useEffect(() => {
        axios.get("/api/rights?_embed=children").then(res => {
            // console.log(res.data);
            res.data.forEach((item) => item.children?.length === 0 ? item.children = "" : item.children);
            setDataSource(res.data);
        })
    }, []);
```

### 6. Get and Edit in RightList.js
Because delete is danger,so we popup a [Model](https://ant-design.gitee.io/components/modal-cn/) information for confirming.
```javascript
    //show confirm modal
    const confirmDelete = (item) => {
        confirm({
            title: 'Do you Want to delete these items?',
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

    //delete item in both frontEnd and backEnd
    const realDelete = (item) => {
        // console.log(item);
        setDataSource(dataSource.filter(data => data.id !== item.id));
        axios.delete(`/api/rights/${item.id}`);
    }
```
above command can delete level 1 menu, but if we try to delete level 2 menu, then we wil encountered an issue says the data not found. How to delete level 2 menu? Let's look back the response data. We can get the first level data according the rightId, and then filter the children, and then send delete in children table.
```javascript
    const realDelete = (item) => {
        // console.log(item);
        if (item.grade === 1) {
            setDataSource(dataSource.filter(data => data.id !== item.id));
            axios.delete(`/api/rights/${item.id}`);
        } else {
            const list = dataSource.filter(data => data.id === item.rightId);
            list[0].children = list[0].children.filter(data => data.id !== item.id);
            setDataSource([...dataSource]);
            axios.delete(`/api/children/${item.id}`);
        }

    }
```
Another button is Edit button, edit button control the page configuration. We will chose Popover to perform this feature.
```javascript
    <Popover
        content={<div style={{ textAlign: 'center' }}>
            <Switch checked={item.pagepermission} onChange={() => swithPageConfiguration(item)}></Switch>
        </div>}
        title="Page configuration" trigger={item.pagepermission === undefined ? '' : 'click'}>
        <Button type='primary' shape='circle' icon={<EditOutlined />} disabled={item.pagepermission === undefined} />
    </Popover>
    ...
    const swithPageConfiguration = item => {
        // console.log(item);
        item.pagepermission = item.pagepermission === 1 ? 0 : 1;
        setDataSource([...dataSource])
        if (item.grade === 1) {
            axios.patch(`/api/rights/${item.id}`, { pagepermission: item.pagepermission });
        } else {
            axios.patch(`/api/children/${item.id}`, { pagepermission: item.pagepermission });
        }
    }
```

### 7. Use Table to perform role in RoleList.js
We already implemented the RightList, so we can refer that implement.
```javascript
import { Button, Table, Modal } from 'antd';
import {
    DeleteOutlined,
    EditOutlined,
    ExclamationCircleOutlined,
} from '@ant-design/icons';

import axios from 'axios';
import React, { useEffect, useState } from 'react'

const { confirm } = Modal;

export default function RoleList() {
    const [dataSource, setDataSource] = useState([]);
    const columns = [
        {
            title: 'ID',
            dataIndex: 'id',
            render: (id) => <b>{id}</b>
        },
        {
            title: '角色名称',
            dataIndex: 'roleName',
        },
        {
            title: '操作',
            render: item => {
                return <div>
                    <Button danger shape='circle' icon={<DeleteOutlined />} onClick={() => confirmDelete(item)} />
                    <Button type='primary' shape='circle' icon={<EditOutlined />} />
                </div>
            }
        },
    ];

    const confirmDelete = item => {
        confirm({
            title: 'Do you Want to delete these items?',
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
        axios.delete(`/api/roles/${item.id}`);
    }

    useEffect(() => {
        axios.get("/api/roles").then(res => {
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
use [Tree](https://ant-design.gitee.io/components/tree-cn/#header) component to perform edit feature in RoleList.js

Also we use [Modal](https://ant-design.gitee.io/components/modal-cn/#header) to popup the tree.
```javascript
<Modal title="Permisson Configuration" visible={isModalVisible} onOk={handleOk} onCancel={handleCancel}>
                <Tree
                    checkable
                    checkedKeys={currentRights}
                    onCheck={handleCheck}
                    checkStrictly={true}
                    treeData={treeData}
                />
</Modal>
...
    const [treeData, setTreeData] = useState([]);
    const [currentRights, setCurrentRights] = useState([]);
    const [currentId, setCurrentId] = useState(0);
    const [isModalVisible, setIsModalVisible] = useState(false);
...    
    const editPermission = item => {
        setIsModalVisible(true);
        setCurrentRights(item.rights);
        setCurrentId(item.id);
    }

    const handleOk = () => {
        setIsModalVisible(false);
        setDataSource(dataSource.map(item=>{
            if(item.id == currentId) {
                return {
                    ...item,
                    rights: currentRights
                }
            }
            return item;
        }));

        axios.patch(`/api/roles/${currentId}`,{rights:currentRights});
    }
    const handleCancel = () => {
        setIsModalVisible(false);
    }  

    const handleCheck = (checkedKeys) => {
        // console.log(checkedKeys);
        setCurrentRights(checkedKeys);
    }
...
    useEffect(() => {
        axios.get(`/api/rights?_embed=children`).then(res => {
            // console.log(res.data);
            setTreeData(res.data)
        })
    }, []);  
```
### 8. Use Table to perform user in UserList.js
```javascript
import { Button, Table, Modal, Switch } from 'antd';
import {
    DeleteOutlined,
    EditOutlined,
    ExclamationCircleOutlined,
} from '@ant-design/icons';

import axios from 'axios';
import React, { useEffect, useState } from 'react'

const { confirm } = Modal;

export default function UserList() {
    const [dataSource, setDataSource] = useState([]);
    const columns = [
        {
            title: '区域',
            dataIndex: 'region',
            render: (region) => <b>{region === '' ? 'Global' : region}</b>
        },
        {
            title: '角色名称',
            dataIndex: 'role',
            render: role => {
                return role?.roleName
            }
        },
        {
            title: '用户名',
            dataIndex: 'username',
        },
        {
            title: '用户状态',
            dataIndex: 'roleState',
            render: (roleState, item) => {
                return <Switch checked={roleState} disabled={item.default} ></Switch>
            }
        },
        {
            title: '操作',
            render: item => {
                return <div>
                    <Button danger shape='circle' icon={<DeleteOutlined />} disabled={item.default} onClick={() => confirmDelete(item)} />

                    <Button type='primary' shape='circle' icon={<EditOutlined />} disabled={item.default} />
                </div>
            }
        },
    ];

    const confirmDelete = item => {
        confirm({
            title: 'Do you Want to delete these items?',
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
    }

    useEffect(() => {
        axios.get(`/api/users?_expand=role`).then(res => {
            // console.log(res.data);
            res.data.forEach((item) => item.children?.length === 0 ? item.children = "" : item.children);
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
### 9. CRUD user in UserList.js
use [Form](https://ant-design.gitee.io/components/form-cn/#components-form-demo-form-in-modal) component to perform this feature.
```javascript
import { Button, Table, Modal, Switch, Form, Input, Select } from 'antd';
import {
    DeleteOutlined,
    EditOutlined,
    ExclamationCircleOutlined,
} from '@ant-design/icons';

import axios from 'axios';
import React, { useEffect, useState } from 'react'

const { confirm } = Modal;
const { Option } = Select;

export default function UserList() {
    const [dataSource, setDataSource] = useState([]);
    const [regionList, setRegionList] = useState([]);
    const [roleList, setRoleList] = useState([]);
    const [addFormVisible, setAddFormVisible] = useState(false)
    const columns = [
        {
            title: '区域',
            dataIndex: 'region',
            render: (region) => <b>{region === '' ? 'Global' : region}</b>
        },
        {
            title: '角色名称',
            dataIndex: 'role',
            render: role => {
                return role?.roleName
            }
        },
        {
            title: '用户名',
            dataIndex: 'username',
        },
        {
            title: '用户状态',
            dataIndex: 'roleState',
            render: (roleState, item) => {
                return <Switch checked={roleState} disabled={item.default} ></Switch>
            }
        },
        {
            title: '操作',
            render: item => {
                return <div>
                    <Button danger shape='circle' icon={<DeleteOutlined />} disabled={item.default} onClick={() => confirmDelete(item)} />

                    <Button type='primary' shape='circle' icon={<EditOutlined />} disabled={item.default} />
                </div>
            }
        },
    ];

    const confirmDelete = item => {
        confirm({
            title: 'Do you Want to delete these items?',
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
    }

    useEffect(() => {
        axios.get(`/api/users?_expand=role`).then(res => {
            // console.log(res.data);
            setDataSource(res.data);
        })
    }, []);

    useEffect(() => {
        axios.get(`/api/regions`).then(res => {
            // console.log(res.data);
            setRegionList(res.data);
        })
    }, []);

    useEffect(() => {
        axios.get(`/api/roles`).then(res => {
            // console.log(res.data);
            setRoleList(res.data);
        })
    }, []);

    return (
        <div>
            <Button type='primary' onClick={() => setAddFormVisible(true)}>Add new user</Button>
            <Table dataSource={dataSource} columns={columns} pagination={{ pageSize: 5 }} rowKey={item => item.id} />;
            <Modal
                visible={addFormVisible}
                title="Create a new user"
                okText="Create"
                cancelText="Cancel"
                onCancel={() => setAddFormVisible(false)}
                onOk={() => {
                    setAddFormVisible(false);
                }}>
                <Form layout="vertical">
                    <Form.Item
                        name="username"
                        label="User Name"
                        rules={[{ required: true, message: 'Please input the user name!' }]}>
                        <Input />
                    </Form.Item>
                    <Form.Item
                        name="password"
                        label="Password"
                        rules={[{ required: true, message: 'Please input the password!' }]}>
                        <Input />
                    </Form.Item>
                    <Form.Item
                        name="region"
                        label="Region"
                        rules={[{ required: true, message: 'Please input the region!' }]}>
                        <Select>
                            {
                                regionList.map(item =>
                                    <Option value={item.value} key={item.id}>{item.title}</Option>
                                )
                            }
                        </Select>
                    </Form.Item>
                    <Form.Item
                        name="roleId"
                        label="Role"
                        rules={[{ required: true, message: 'Please input the role!' }]}>
                        <Select>
                            {
                                roleList.map(item =>
                                    <Option value={item.id} key={item.id}>{item.roleName}</Option>
                                )
                            }
                        </Select>
                    </Form.Item>
                </Form>
            </Modal>
        </div>
    )
}
```

### 10. Wrapper addForm as component
```bash
mkdir src/components/user-manage
touch src/components/user-manage/UserForm.js
```
Copy the Form section to this file.
> use forwardRef for exporting UserForm.js for communitcating with UserList.js

create a ref in UserList.js
```javascript
import React, { useRef } from 'react'
...
const addForm = useRef(null);
...

<Modal
    visible={addFormVisible}
    title="Create a new user"
    okText="Create"
    cancelText="Cancel"
    onCancel={() => setAddFormVisible(false)}
    onOk={() => {
        // console.log(addForm);
        addForm.current.validateFields().then(value => {
            console.log(value)
        }).catch(err => {
            console.log(err);
        })
    }}>
    <UserForm regionList={regionList} roleList={roleList} ref={addForm} />
</Modal>    
```
The UserForm.js we change the definition formation
```javascript
const UserForm = forwardRef((props, ref) => {
    return (
        <Form layout="vertical" ref={ref} >
        ...
        </Form>
    )
});
export default UserForm;        
```
we can set the modalOk as below
```javascript
const addFormOk = () => {
        // console.log(addForm);
        addForm.current.validateFields().then(value => {
            // console.log(value)
            setAddFormVisible(false);
            addForm.current.resetFields();
            axios.post(`/api/users`,{
                ...value,
                'roleState': true,
                'default': false,
            }).then(res=>{
                // console.log(res.data);
                setDataSource([...dataSource, {...res.data,
                role: roleList.filter(item=>item.id=value.roleId)[0]}])
            })
        }).catch(err => {
            console.log(err);
        })
    }
```
Delete user in UserList.js
```javascript
   const realDelete = (item) => {
        // console.log(item);
        setDataSource(dataSource.filter(data=>data.id!==item.id));
        axios.delete(`/api/users/${item.id}`);
    }
```
Update user status in UserList.js
```javascript
        {
            title: '用户状态',
            dataIndex: 'roleState',
            render: (roleState, item) => {
                return <Switch checked={roleState} disabled={item.default} onChange={() => handleUserState(item)} ></Switch>
            }
        },
        ....
    const handleUserState = (item)=> {
        // console.log(item);
        item.roleState = !item.roleState;
        setDataSource([...dataSource])
        axios.patch(`/api/users/${item.id}`,{roleState:item.roleState});
    }
```
Update user in UserList.js
It is very similar with addForm, so copy an updateForm modal.
```javascript
omit
```
Filter data in [region field](https://ant-design.gitee.io/components/table-cn/#components-table-demo-head) in UserList.js
```javascript
    const columns = [
        {
            title: '区域',
            dataIndex: 'region',
            filters: [
                ...regionList.map(item => ({
                    text: item.title,
                    value: item.value
                })),
                {
                    text: 'Global',
                    value: ''
                }
            ],
            onFilter: (value, item) => item.region === value,
            render: (region) => <b>{region === '' ? 'Global' : region}</b>
        },
        ...
        }
```