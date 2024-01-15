
# Development

```bash
NODE_ENV=development
```

Step 1: Create a ".env.override.development" file.

```bash
NODE_ENV=development
SERVER_PORT=3050
SERVER_IP= ::
DB_HOST=192.168.1.14
DB_NAME=dbvidly
NODE_LOG=./logfile.log
```

# Production

Step 2: Create a ".env.override.production" file.

```bash
NODE_ENV=production
SERVER_PORT=3051
SERVER_IP= ::
DB_HOST=192.168.1.15
DB_NAME= dbarmanriazi
Step 3: Add to index.js
const env = require("./startup/environment");
const {createDb}=require("./startup/db");
require("./startup/config")();
//check connection
createDb();
//middelware
app.use(bodyParser.json());
// environment
env.selectEnvironmet();
Step 4: Add this content to environment.js
const winston = require("winston");
const fs = require("fs");
const result = require("dotenv").config();
if (result.error) {
throw result.error;
}
const selectEnvironmet = () => {
switch (process.env.NODE_ENV) {
case "development":
const envConfigDev = require("dotenv").parse(
fs.readFileSync(".env.override.development")
);
for (const k in envConfigDev) {
process.env[k] = envConfigDev[k];
winston.info(process.env[k]);
}
break;
case "production":
const envConfigPro = require("dotenv").parse(
fs.readFileSync(".env.override.production")
);
for (const k in envConfigPro) {
process.env[k] = envConfigPro[k];
winston.info(process.env[k]);
}
);
for (const k in envConfigStage) {
process.env[k] = envConfigStage[k];
winston.info(process.env[k]);
}
break;
}
};
exports.selectEnvironmet = selectEnvironmet;
```

## Step 3: Add this contents to db.js:

```js
const agentkeepalive = require("agentkeepalive");
const winston = require("winston");
const myagent = new agentkeepalive({
maxSockets: 100,
maxKeepAliveRequests: 0,
maxKeepAliveTime: 30000,
});
const opt_prod = {
url: "",
parseUrl: false,
requestDefaults: {
agent: myagent,
// "proxy" : "http://someproxy"
},
};
const opt_dev = {
url: "",
parseUrl: false,
// log: (id, args) => {
// console.log(id, args);
// },
};
const result = {};
if (process.env.NODE_ENV == "production") {
opt_prod.url="http://admin:server@192.168.1.11:5984";
result.nano = require("nano")(opt_prod);
exports.db = require("nano")(opt_prod).use(" dbarmanriazi");
} else if (process.env.NODE_ENV == "development") {
opt_dev.url="http://admin:server@192.168.1.11:5984";
result.nano = require("nano")(opt_dev);
exports.db = require("nano")(opt_dev).use(" dbarmanriazi");
}
exports.createDb = function () {
result.nano
.db.create(" dbarmanriazi", { partitioned: true })
.then((data) => {
winston.info("Connected to db…");
})
.catch((er) => {
winston.warn("Database is exist");
});
};
```

## Result at runtime

```md
info: development
info: ./logfile.log
info: 3050
info: ::
info: 192.168.1.11
info: dbarmanriazi
info: Listening on port 3050…
warn: Database is exist
---

## Resolved issues
Let me show you the error on Github and Stackoverflow:

- [x] [Nodejs AssertionError [ERR_ASSERTION]: You must specify the endpoint URL when invoking this module #223](https://github.com/apache/couchdb-nano/issues/223)
- [x] [Stackoverflow](https://stackoverflow.com/questions/61770234/nodejs-assertionerror-err-assertion-you-must-specify-the-endpoint-url-when-in)
