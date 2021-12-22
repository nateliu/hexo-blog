---
title: publish news-publish-management to netlify and heroku
date: 2021-12-22 16:01:28
tags:
---
I am learning react at home duraing the annual leave vacation. I have created a [news-publish-management](https://github.com/nateliu/news-publish-management) at GitHub.com

The first commit is doning following things:
1. Using yarn creat a new react app 
```bash
yarn create react-app news-publish-manaments
```

2. Installing json-server for deploying the db.json to hereku
```bash
yarn add json-server
```

3. Modifing the package.json, appending below lines in <span style={color:red}>scripts</span> selction
```javascript
    "json-server-dev": "nodemon mock-api/server.js",
    "json-server-prod": "node mock-api/server.js"
```

4. Adding <span style={color:red}>netlify.toml</span>
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

5. Adding <span style={color:red}>Procfile</span>
```javascript
web: npm run json-server-prod
```

6. Creating a <span style={color:red}>server.js</span> under <span style={color:red}>mock-api</span> folder
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


7. Pushing above changing to GitHub

8. Going to [Heroku](https://id.heroku.com/login) for deploying news-publish-managements
once successfully deployed. It is verified via
https://news-publish-management.herokuapp.com/api/news


9. Going to [netlify](https://app.netlify.com/) for deploying news-publish-managements
once successfully deployed. It is verified via
https://news-publish-management.netlify.app/

