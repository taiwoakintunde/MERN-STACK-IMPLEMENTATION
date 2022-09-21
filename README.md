**MERN STACK IMPLEMENTATION**
1. Update ubuntu
`sudo apt update`
2. Upgrade ubuntu
`sudo apt upgrade`
3. To get the location of Node.js software from Ubuntu repositories
`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`
3. Install Node.js on the server
`sudo apt-get install -y nodejs`
4. Verify the node installation with the command below
`node -v`
5. Verify the node npm installation with the command below
`npm -v`

## Application Code Setup
1. Create a new directory for your To-Do project:
`mkdir todo`
2. To verify Todo directory has been created
`ls`
3. To change the directory to the newly created one
`cd Todo`
4. You will use the command npm init to initialise your project, so that a new file named package.json will be created.
`npm init`
5. Run command ls to see the package

**INSTALL EXPRESSJS**
1.  To use express, install it using npm:
`npm install express`
2. Now create a file index.js with the command below
`touch index.js`
3. Install the dotenv module
`npm install dotenv`
4. Open the index.js file with the command below
`vim index.js`
5. Copy and paste the following code
`const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});`
6. To see the server if it works
`node index.js`
7. We need to add an inbound rule on EC2 security group to open port 5000
8. Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000:
http://34.201.60.65:5000 

## There are three actions that our To-Do application needs to be able to do:
1. Create a new task
2. Display list of all tasks
3. Delete a completed task

#### For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes
`mkdir routes`
1. Change directory to routes folder.
`cd routes`
2. Create a file api.js with the command below
`touch api.js`
3. Open the file with the command below
`vim api.js`
4. Copy and paste the follwing code in the file
`const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;`

## MODELS
1. To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.

Change directory back Todo folder with cd .. and install Mongoose
`npm install mongoose`
2. Create a new folder models :
`mkdir models`
3. Change directory into the newly created ‘models’ folder with
`cd models`
4. Inside the models folder, create a file and name it todo.js
`touch todo.js`
5. Open the file created with vim todo.js then paste the code below in the file:
`vim todo.js`
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
6. Now we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model.
7. In Routes directory, open api.js with vim api.js, delete the code inside with :%d command and paste there code below into it then save and exit

const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;

## MONGODB DATABASE
1. In the index.js file, we specified process.env to access environment variables, but we have not yet created this file. So we need to do that now.

Create a file in your Todo directory and name it .env.
`touch .env
vi .env`
2. Add the connection string to access the database in it, just as below:
`DB = mongodb+srv://paparazzomnet:xKykfNTO3Bbuk0Bf@cluster0.z7qq3sp.mongodb.net/TammyTaiwo?retryWrites=true&w=majority`
3. Now we need to update the index.js to reflect the use of .env so that Node.js can connect to the database.
Simply delete existing content in the file, and update it with the entire code below.
`vim index.js`
To delete the existing file
`:%d!`
Copy and paste the following code
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

#### Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.
4. Start your server using the command:
`node index.js`

## Testing Backend Code without Frontend using RESTful API

1. In this project I will be using postman to test API

## STEP 2 – FRONTEND CREATION
1. In the same root directory as your backend code, which is the Todo directory, run:
`npx create-react-app client`
#### Running a React App
Before testing the react app, there are some dependencies that need to be installed.
1. Install concurrently. It is used to run more than one command simultaneously from the same terminal window.
`npm install concurrently --save-dev`
2. Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.
`npm install nodemon --save-dev`
3. In Todo folder open the package.json file. Change the highlighted part of the below screenshot and replace with the code below.
`"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},`
#### Configure Proxy in package.json
1. Change directory to ‘client’
`cd client`
2. Open the package.json file
`vi package.json`
3. Add the key value pair in the package.json file "proxy": "http://localhost:5000".
Now, ensure you are inside the Todo directory, and simply do:
`npm run dev`

### Creating your React Components
1. From your Todo directory run
`cd client`
2. move to the src directory
`cd src`
3. Inside your src folder create another folder called components
`mkdir components`
4. Move into the components directory with
`cd components`
5. Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js.
`touch Input.js ListTodo.js Todo.js`
6. Open Input.js file
`vi Input.js`
7. Copy and paste the following
`import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input`

8. To make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios.

9. Move to the src folder
`cd ..`
10. Move to clients folder
`cd ..`
11. Install Axios
`npm install axios`

### FRONTEND CREATION
1. Go to ‘components’ directory
`cd src/components`
2. After that open your ListTodo.js
`vi ListTodo.js`
3. in the ListTodo.js copy and paste the following code
`import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo`

4. Then in your Todo.js file you write the following code
`import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;`
5. We need to make little adjustment to our react code. Delete the logo and adjust our App.js to look like this.

Move to the src folder
`cd ..`
6. Make sure that you are in the src folder and run
`vi App.js`
7. Copy and paste the code below into it
`import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;`

8. In the src directory open the App.css
`vi App.css`
9. Then paste the following code into App.css:
`.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}`

10. In the src directory open the index.css
`vim index.css`
11. Copy and paste the code below:
`body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}`

12. Go to the Todo directory
`cd ../..`
13. When you are in the Todo directory run:
`npm run dev`




