---
title: publish news-publish-management to netlify and heroku
date: 2021-12-22 16:01:28
tags:
---
I am learning react at home duraing the annual leave and company holiday. I have created a repository named [news-publish-management](https://github.com/nateliu/news-publish-management) at [GitHub](https://github.com/). 
> The node version I used is <span style="color:lightblue">v16.13.1</span> <br/>
> The react version I used is <span style="color:lightblue">^17.0.2</span>

The first commit is doing following <span style="color:lightblue" bold>9</span> things:
1. Use yarn for creating a new react app 
```bash
yarn create react-app news-publish-manaments
```
> Yarn is a package manager for your code. It allows you to use and share code with other developers from around the world. Yarn does this quickly, securely, and reliably so you don't ever have to worry.

> Yarn allows you to use other developers' solutions to different problems, making it easier for you to develop your software. If you have problems, you can report issues or contribute back on GitHub, and when the problem is fixed, you can use Yarn to keep it all up to date.

> I will swith to [yarn](https://yarnpkg.com/cli/install) instead of npm after this learning section.

2. Install <span style="color:lightblue">json-server</span> for deploying the <span style="color:lightblue">db.json</span> to hereku
```bash
yarn add json-server
```
> [JSONServer](https://github.com/dreyacosta/JSONServer) was stored in GitHub.

3. Modify the <span style="color:lightblue">package.json</span>, and append below line in <span style="color:lightblue">scripts</span> selction
```json
  "json-server": "node mock-api/server.js"
```

4. Add <span style="color:lightblue">netlify.toml</span>
```bash
touch netlify.toml
```
paste content into the toml file
```toml
[build]
  command = "CI= npm run build"

[[redirects]]
  from = "/api/*"
  to = "https://news-publish-management.herokuapp.com/api/:splat"
  status = 200

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

5. Add <span style="color:lightblue">Procfile</span>, the detail information of ProcFile in Heroku can be foud in [here](https://devcenter.heroku.com/articles/procfile). Base on this page, we can add below line in the Procfile as well.
```javascript
web: npm run json-server-prod
```

6. Create a <span style="color:lightblue">server.js</span> under <span style="color:lightblue">mock-api</span> folder.
```bash
mkdir mock-api
touch mock-api/server.js
```
paste below content into <span style="color:lightblue">server.js</span>
```javascript
const jsonServer = require("json-server");
const server = jsonServer.create();
const router = jsonServer.router("mock-api/db.json");
const middlewares = jsonServer.defaults();
const port = process.env.PORT || 4000; 

server.use(middlewares);
server.use("/api", router);

server.listen(port);
```


7. Push above changs into to GitHub

8. Go to [Heroku](https://id.heroku.com/login) for deploying news-publish-managements.
Once it was successfully deployed. It can be verified via
https://news-publish-management.herokuapp.com/api/news


9. Go to [netlify](https://app.netlify.com/) for deploying news-publish-managements.
Once it was successfully deployed. It is can be verified via
https://news-publish-management.netlify.app/

