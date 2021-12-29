---
title: news-pubish-management(11)--Update and Fixing
date: 2021-12-29 11:12:29
tags:
---
Will update and fix issue in this document.

## Update Section
### 1. Change HashRouter to BrowserRouter
In previous, the NPM uses HashRouter, it is fine, but has # in the Browser, If you don't like the # in urls like me, then please change to BrowserRouter. For changing to BrowserRouter, every <a href=''> should remove # from href property. I guess this is the biggest different bewteen HashRouter and BrowserRouter.

## Fixing Section
### 1. How to fix "React Hook useEffect has a missing dependency. Either include it or remove the dependency array" problem?
I have a code-snippet in Home.js
```javascript
    useEffect(() => {
        axios.get(`/api/news?publisthState=2&_expand=category`).then(res => {
            setAllList(res.data);
            const viewData = _.groupBy(res.data, item => item.category.title);
            renderBarView(viewData)
        })

        return () => {
            window.onresize = null;
        }
    }, []); // eslint-disable-line react-hooks/exhaustive-deps
```
If I did't add last line of the comment, it will occcure a  warning messge. That because the renderBarView is a function. The hook useEffect cann't do like this, it will run on every re-render and will give a loop of re-render error. So the solution to resolve this waning are 2 options: one is above I did. Another is copy the renderBarView into useEffect. like below
```javascript
const renderBarView = vieewData => {
    ....
}

```
### In Loign components, after clicked Sign In button, it had executed the navigate('/'), but didn't redirect. 
The root cause is navigate has 2 signature, if use <Link> like above, it is must required NavigateOptions. So change to below code can fix this issue.
```javscript
navigate('/', {replace:true});
```
### 3. Audit usage of navigator.userAgent, navigator.appVersion, and navigator.platform
Found there are some codes like below:
```javascript
// Mock not supported chrome.* API for Firefox and Electron
window.isElectron = window.navigator && window.navigator.userAgent.indexOf('Electron') !== -1;
var isFirefox = navigator.userAgent.indexOf('Firefox') !== -1; // Background page only

````
definitely, they are from node_modules but not sure which library. So it wont fix right now. Detail message can be found in [here](https://blog.chromium.org/2021/05/update-on-user-agent-string-reduction.html)

...To be continue....