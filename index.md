---
marp: true
theme: customized
backgroundColor: white
paginate: true
footer: 'Tikal Academy'
size: 16:9


---
<!-- _class: lead -->

# Tikal Academy
## Working with Express

---
## Why is Express so popular?

* Simple
* Powerful
* Extensible

--------

## Getting started

```json
{
  "name": "users",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "tsc",
    "start": "nodemon ./out/index.js"
  },
  "keywords": [],
  "author": "Assaf Gannon",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1"
  },
  "devDependencies": {
    "@types/express": "^4.17.8",
    "nodemon": "^2.0.6"
  }
}
```

------

## Most basic setup 

```typescript
import express, { NextFunction, Request, Response } from 'express';

const app = express();

app.get('/', (req: Request, res: Response, next: NextFunction) => {
    res.send('hello world')
})

app.listen(3001, () => {
    console.log('app started and listenenig at http://localhost:3001');
})
```

-----

## Express Middleware

- Adds capabilities to the server
- Can be chained
- Order matters


```ts
expressApp.use((req: Request, res: Response, next: NextFunction) => {
    // middleware stuff 

    // call next middleware
    next();
})
```

-----

## Writing our first Middleware

```ts
function logOriginMiddleware(req: Request, res: Response, next: NextFunction){
    console.log(`Got request for ${req.path} from ${req.ip}`);
    next();
}

// ...

app.use(logOriginMiddleware);
```

-----

## Commonly used Middelwares

- 'cookie-parser'
- 'body-parser'
- 'static-conent'
- 'static'
  
```ts
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, '..', 'public')));
```

-----

## Routing - option 1

`app.[METHOD]('path', handlerFn)`

```ts
app.get('/resource', getHandler);
app.post('/resource', postHandler);
app.put('/resource', putHandler);
app.delete('/resource', deleteHandler);
```

-----

## Routing - option 2

```ts
app.route('path')
    .get(getHandler)
    .post(postHandler);
    .put(putHandler);
    .delete(deleteHandler);
```

-----

## Routing - option 3

### express.Router


```ts
// 'user-router.ts'
import express from 'express';
export const router = express.Router()

// middleware that is specific to this router
router.use(function timeLog (req, res, next) {
  console.log('Time: ', Date.now())
  next()
})
// define the home page route
router.get('/', getHandler);
router.post('/', postHandler);
router.put('/', putHandler);
router.delete('/', deleteHandler);

// ----
// in in main:
import { router } from './user-router';

app.use('/user', router);
```

------

## Route params

```ts
app.get('/user/:company/:id', (req, res) => {
    const { company, id } = req.params;
});
```

------

## Route handlers

### Single callback

```js
app.get('/example/a', function (req, res) {
  res.send('Hello from A!')
})
```

----

## Multiple callbacks

First methods act as middleware for the specific route

```js
app.get('/example/b', function (req, res, next) {
    console.log('the response will be sent by the next function ...')
    next()
}, function (req, res) {
    res.send('Hello from B!')
})
```

-----

## Response Methods

Method | Description
:---|:---
`res.download()` | Prompt a file to be downloaded.
`res.end()` | End the response process.
`res.json()` | Send a JSON response.
`res.jsonp()` | Send a JSON response with JSONP support.
`res.redirect()` | Redirect a request.
`res.render()` | Render a view template.
`res.send()` | Send a response of various types.
`res.sendFile()` | Send a file as an octet stream.
`res.sendStatus()` | Set the response status code and send its string representation as the response body.

------

## Exercise 1

- Create an express server
- Add routes for RESTful management of "item"
  - use "Inmemmory" storage

-------

## Exercise 2

- Seperate the service to different files:
  - `index.ts`, `controller.ts` and `service.ts`

-------

## Exercise 3

- Replace the "Inmemmory" storage with mongodb
  - make sure to pass the `monogo uri` as an environmnet variable

------

## Docerizing the service

Dockerfile

```yaml
FROM node:15.2.0-alpine3.10

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "node", "index.js" ]
```

------

## .dockeringore

```yaml
node_modules
npm-debug.log
```

------

## Building and running

```sh
docker build -t <your username>/<service-name> .

docker run -p 49160:8080 -d <your username>/<service-name>

```

for further reading see [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md)

------

## Exercise 4

- Add your service to a docker-compose with the mongodb


-----

# Final Project


Write a system that collects and displays location check-ins

* Service1
  - Accepts a geo-location as post requests (latitude, longitude) and userId, and timestamp
  - creates a BullMQ job

* Service2 
  - Processes jobs from the BullMQ:
    - enriches the data with address information (https://locationiq.com/geocoding, https://developers.google.com/maps/documentation/geocoding/start)
    - Create another BullMQ job (different queue)

---
## Cont. 
* Service3
  - Process the enriched data from Service2
    - Store in a mongoDB

* Service4
  - Expose the data with a web service
  - Allow to see locations by:
    - userId
    - userId and time frame
    - Most popular locations
    - show users in proximaty (accept a userId and show all users within a X km radius) - use last location per user
