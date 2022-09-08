
# Node.js + GraphQL + Socket.io Examples

<br><br>
> node.js 환경에서 GraphQL 방식의 API를 사용하여<br>
> socket.io를 WebSocket으로 사용하는 Chatting Server 예제<br>
> express server에서는 static page, REST API, GraphQL API 방식을 모두 포함한다.<br>

### index.js

```javascript
let express = require('express');
const http = require('http');
const socketio = require('socket.io');
const { graphqlHTTP } = require('express-graphql');
const { buildSchema } = require('graphql');

const schema = buildSchema(`
  type Query {
    project: Property
    guest: Person
  }

  type Person {
    name: String
    age: Int
    sex: Boolean
  }

  type Property {
    name: String
    member: [Developer]
    spec: [Module]
  }

  type Developer {
    name: String
    field: String
  }

  type Module {
    name: String
    field: String
  }
`);

var dev1 = { name: 'park', field: 'mobile' };
var dev2 = { name: 'moon', field: 'server' };
var dev3 = { name: 'na', field: 'server' };
var mod1 = { name: 'node', field: 'backend' };
var mod2 = { name: 'express', field: 'server' };
var mod3 = { name: 'graphql', field: 'api' };
var mod4 = { name: 'react', field: 'frontend' };

const prop = { 
    name: 'Mission Impossible', 
    member: [dev1, dev2, dev3], 
    spec: [mod1, mod2, mod3, mod4] 
};

const prop2 = {name: 'park', age: 12, sex: true};

const root = { 
    project: () => prop,
    guest: () => prop2
};

const app = express();
const chat = http.createServer(app);
const io = socketio(chat);

app.use(express.static(__dirname + '/public'));

chat.listen(4000, () =>
  console.log('Server is running localhost:4000 chat')
);

app.get('/', (req, res) => { 
    var clientInfo = req.baseUrl + ':'
                   + req.body + ':'
                   + req.hostname;
    console.log('Client request : ' + clientInfo);
    res.send(prop);
});

app.get('/chat', (req, res) => { 
  res.sendFile(__dirname + '/chat.html');
});

app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: root,
  graphiql: true,
}));

io.on('connection', (socket) => {
  console.log(socket.id, ' connected!!');
  socket.emit('msg', ({name:`[<-- ${socket.id}]`, message:" connected!!"}));
  
  socket.on('msg', ({name, message}) => {
    console.log(socket.id, {name, message});
    name = '[<-- ' + name + ']'
    socket.broadcast.emit('msg', ({name, message}));
  });
}); 

```

### package.json
```json
{
  "name": "------",
  "version": "1.0.0",
  "description": "------",
  "main": "index.js",
  "mode": "commonjs",
  "scripts": {
    "test": "echo \\\"Error: no test specified\\\" && exit 1",
    "server": "nodemon index.js",
    "client": "npm start --prefix ..\\------",
    "dev": "concurrently \"npm run server\" \"npm run client\""
  },
  "keywords": [
    "node.js"
  ],
  "author": "------",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.1",
    "express-graphql": "^0.12.0",
    "graphql": "^15.8.0",
    "graphql-tool": "^1.0.0",
    "socket.io": "^4.5.1",
    "ws": "^8.8.1"
  },
  "devDependencies": {
    "concurrently": "^7.3.0"
  }
}
```
