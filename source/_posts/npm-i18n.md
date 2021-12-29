---
title: news-pubish-management(10)--Internationalization
date: 2021-12-29 08:13:13
tags:
---
I am going to localize the news-publish-management with [react-i18next](https://react.i18next.com/).

### 1. config react-i18next
```bash
yarn add react-i18next i18next
mkdir src/i18n
touch src/i18n/config.js
touch src/i18n/en.json
touch src/i18n/zh.json
```
content in config.js
```javascript
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
 
import translation_en from './en.json';
import translation_zh from './zh.json';
 
const resources = {
    en: {
        translation: translation_en,
    },
    zh: {
        translation: translation_zh,
    },
};
 
i18n.use(initReactI18next).init({
    resources,
    lng: 'zh',
    interpolation: {
        escapeValue: false,
    },
});
 
export default i18n;
```

en.json add content
```json
{
    "Login": {
        "title": "News Publish Management",
        "signin":"Sign In",
        "validateUserName": "Please input your Username!",
        "validatePassword": "Please input your Password!",
        "validateLanguage": "Please select your Language!",
        "validateErrorMessage": "UserName or Password is wrong!"
    },
    "TopHeader": {
        "languageTitle": "Language",
        "welcomeMessage": "Welcome",
        "signout":"Sign Out"
    }
}
```
zh.json add content
```json
{
    "Login": {
        "title": "全球新闻发布系统",
        "signin":"登陆",
        "validateUserName": "请输入正确用户名!",
        "validatePassword": "请输入正确的密码!",
        "validateLanguage": "请选择语言!",
        "validateErrorMessage": "用户名或密码不正确!"
    },
    "TopHeader": {
        "languageTitle": "语言",
        "welcomeMessage": "欢迎",
        "signout":"退出"
    }
}
```
import config in index.js
``` javascript
...
import './i18n/config';
...
```
### 2. Using useTranslation in Login component
Go to Login.js, import i18n and use useTranslation/Trans Hoc to do translate.
```javascript
import { useTranslation, Trans } from 'react-i18next';
import i18n from 'i18next';
...
const [language, setLanguage] = useState('zh');
const { t } = useTranslation();
...

message.error(t('Login.validateErrorMessage'))
...
<div className='loginTitle'> <Trans>login.title</Trans></div>
...

// support user to select his prefer language

    <Form.Item
        name="language"
        rules={[{ required: true, message: t('Login.validateLanguage') }]}
    >
        <Select value={language} onChange={handleLanguageChange} placeholder={<GlobalOutlined className="site-form-item-icon" />} >
            <Option value='zh'>中文</Option>
            <Option value='en'>English</Option>
        </Select>
    </Form.Item>
...
    const handleLanguageChange = value => {
        // console.log(`selected ${value}`);
        setLanguage(value);
        i18n.changeLanguage(value);
    }

```
### 3. Localize in TopHeader component
Just use useTranslation hook in this component.
```javascript
...
import { useTranslation } from 'react-i18next';
import i18n from 'i18next';
...
const { t } = useTranslation();
const handleLanguageClick = value => {
i18n.changeLanguage(value);
}
...

<Menu.SubMenu key="lngMenu" icon={<GlobalOutlined />} title={t('TopHeader.languageTitle')}>
    <Menu.Item key="lngMenuZH" onClick={() => handleLanguageClick('zh')}>中文</Menu.Item>
    <Menu.Item key="lngMenuEN" onClick={() => handleLanguageClick('en')}>English</Menu.Item>
</Menu.SubMenu>
<Menu.Item danger key='loginOutMenu' onClick={() => {
    localStorage.removeItem('token');
    navigate('/login');
}}>{t('TopHeader.signout')}</Menu.Item>
```