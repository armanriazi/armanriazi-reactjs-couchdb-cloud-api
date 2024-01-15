Developed: All of the things that you need to work with CouchDB and ExpressJS.

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

## Step 3: LOCs of db.js

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

---

## Resolved issues
Let me show you the error on Github and Stackoverflow:

- [x] [Nodejs AssertionError [ERR_ASSERTION]: You must specify the endpoint URL when invoking this module #223](https://github.com/apache/couchdb-nano/issues/223)
- [x] [Stackoverflow](https://stackoverflow.com/questions/61770234/nodejs-assertionerror-err-assertion-you-must-specify-the-endpoint-url-when-in)

### Continue published of Failure: NodeJs AssertionError [ERR_ASSERTION]
If CouchDB exposes APIs for me why do I need to backend code?!
I made a sample project that needs authorization with JWT, so we use ExpressJS for it and you can have better management on DB.
I made a solution for the management of development production and staging environments without anything effort to publish on the Cloud.
All of the things that we need for a repository section of the structure project are:
(CRUD + Validate)

```bash
exports.validate = validateMovie;
exports.validateJustName = validateJustName;
exports.asyncDbGetMovieByName = asyncDbGetMovieByName;
exports.asyncDbListMovie = asyncDbListMovie;
exports.asyncDbGetMovie = asyncDbGetMovie;
exports.asyncDbAddMovie = asyncDbAddMovie;
exports.asyncDbRemoveMovie = asyncDbRemoveMovie;
exports.asyncDbUpdateNumberInStockMovie = asyncDbUpdateNumberInStockMovie;
```

Get a list of movies for example:

```js
const Joi = require("joi");
const { isOk, objOfResDbErrMsg, retObjSuccDbMsg } = require("../models/result");
const {db} = require("../startup/db");
const dbDebugger = require("debug")("app:db");
const PARTITION = "movies:";
const validateMovie = (movie) => {  const schema = {   
id: Joi.string().min(5).max(25).required(),    title: Joi.string().min(5).max(50).required(),    name: Joi.string().min(3).max(25).required(),    genreId: Joi.string().min(3).max(20).required(),    numberInStock: Joi.number().min(0).required(),    dailyRentalRate: Joi.number().min(0).required(),  };  return Joi.validate(movie, schema);};
//Async Functionsconst 
asyncDbListMovie = () =>  
new Promise((resolve, reject) => {    
resolve(      
db.list({ include_docs: true, partition: "movies" })        .then((result) => {
          return result;
}).catch((er) => {
dbDebugger(er);  
return objOfResDbErrMsg;
}));});
```

Router layer of the project for the exposed API route.

```js
router.get("/", async (req, res) => { 
const movie = await dbMovies.asyncDbListMovie(); 
if (movie != undefined && _.size(movie.rows) > 0) { res.json(retObjSuccDbMsg(movie)); const doc = movie.rows[0].doc; 
if (doc._id.length == 0) return resErrMsg(res, 404, "4043"); } 
else return resErrMsg(res, 404, "4043");});
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
```
