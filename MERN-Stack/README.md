# DEPLOYING A MERN STACK TODO APPLICATION

This project is a simple yet powerful To-Do application built using the **MERN stack** (MongoDB, Express.js, React.js, and Node.js).

The application allows users to:
- Create new tasks
- View all existing tasks
- Delete completed tasks

It’s a full-stack project showcasing how the **frontend (React)** communicates with the **backend (Express + Node)** and persists data in a **cloud-hosted MongoDB database (MongoDB Atlas)**.


## ⚙️ MERN Stack Components

This project leverages the four core technologies of the MERN stack:

**MongoDB** – A NoSQL, document-based database that stores tasks as documents.
**Express.js** – A minimal web application framework for Node.js that simplifies routing and API development.
**React.js** – A frontend JavaScript library for building dynamic and reusable UI components.
**Node.js** – A JavaScript runtime that executes code on the server side.

Together, these technologies create a modern, scalable full-stack web application.

---

## ✅ Prerequisites

1. **Launching the EC2 Instance**: Ubuntu 24.04 LTS (t3.micro Free Tier)
2. **Security Group**: Inbound rules for
    - Port 22 → SSH access ( The backdoor for maintenance)
    - Port 80 → HTTP web traffic (The front door for web traffic)
    - Port 5000 → Express backend
    - Port 3000 → React frontend
3. **SSH Key Pair**: For secure authentication
4. **MongoDB Atlas account**: For database hosting
5. **Postman**: For testing API endpoints
4. **GitHub**: Version control and documentation
5. **Visual Studio Code**: Code editor and SSH integration
6. **Web Browser**: For testing deployments
<img width="925" height="111" alt="Mern stack server" src="https://github.com/user-attachments/assets/e1e94c06-252e-417f-8e51-f898e739d359" />
---

#  Step 1: Backend Configuration

```bash
#ssh -i <Your-private-key.pem> ubuntu@<EC2-Public-IP-address>
```

1. Update and Upgrade Ubuntu to get our backend environment started:
```bash
sudo apt update

sudo apt upgrade
```

2. Introducing Node.js:
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

- Install Node.js: This command installs both Node.js and npm (Node Package Manager). NPM acts like apt in terms of managing repositories:
```bash
sudo apt-get install -y nodejs
```

- Verify the installation:
```bash
node -v

npm -v
```
<img width="1005" height="289" alt="Node js installed" src="https://github.com/user-attachments/assets/13d1044d-8c9b-4e30-8bf9-85bd14d36981" />


## 👉 Application Code Setup

- Create a project directory:
```bash
mkdir Todo

ls

cd Todo

npm init
# After initializing npm, click the Enter key continuously until all details are out
# Once done, type Yes to continue.
ls
```

**This creates a package.json file, which will be the blueprint for our project. It contains a list of all its dependencies and scripts.**

- After initializing npm, press Enter several times to accept default values.
<img width="1217" height="620" alt="npm init" src="https://github.com/user-attachments/assets/c8e45f68-56a0-413a-ba0f-7923a51b27e8" />
<img width="760" height="390" alt="npm init 2" src="https://github.com/user-attachments/assets/a61aeaf6-8a85-4de9-83cb-4589a37ddf58" />
Then type Yes to accept and write out the package.json file.
---

## 👉 Install ExpressJS

Express is a framework for Node.js. It simplifies development and abstracts many low-level details. For example, Express helps to define routes of your application based on HTTP methods and URLs.

1. Install Express using npm:
```bash
npm install express
```

- Create an Index.js file: This would be the starting point of our application:
```bash
touch index.js

ls
```

2. Install the 'dotenv' module: This module is important, as it will help us manage environment variables (keeping them secret in the app):
```bash
npm install dotenv

vim index.js
```

- Paste the code into the file and save (Take note of the Port being used):
```bash
const express = require('express');
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
});
```

**To save and close the file:**
Hit the `esc` button, type `:wq`  then hit `ENTER` to save the file.

3. Start the server to see if it works:
```bash
node index.js
```
You should see the server running on port 5000 in your terminal.

- Add an inbound rule for TCP Port 5000 in the EC2 Security Group.
<img width="1848" height="300" alt="Port 5000" src="https://github.com/user-attachments/assets/42babb6b-98f6-45ac-acab-aaa77f41d787" />

- Browser Check: http://<PublicIP-or-PublicDNS>:5000(Congratulations, your Express application is live!)
<img width="374" height="81" alt="Welcome to express" src="https://github.com/user-attachments/assets/3f143381-bc9d-416e-bda7-d1b6a16a5873" />

**If you got to this point, you’ve successfully set up the skeleton of the backend.**

---

## 👉 Routes

Now that the Express server is up and running, it's time to add some structure to our application. We are going to create routes that will handle the three actions that our To-Do application should be able to do:

1. Create a new task
2. Display a list of all tasks
3. Delete a completed task

Each task in the To-Do app will be linked to a specific endpoint and use standard HTTP methods (POST, GET, DELETE) to interact with the database. To achieve this, we’ll define these endpoints in Express by creating a dedicated routes folder.

- Keep the previous terminal open, then log into your server with another terminal and run:
```bash
mkdir routes

cd routes

touch api.js

vim api.js
```

- To set up the routes, paste the code into the file and save:
```bash
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```
<img width="868" height="522" alt="Routes" src="https://github.com/user-attachments/assets/44286b91-5602-41da-85de-2209b63cf636" />

**Now we’ve laid out the blueprint for our API routes, but these routes don't do anything yet. To bring them to life, we need to create models that will interact with our database.**

---

## 👉 Models

The Model define the structure of the data we will be working with and how it's stored in the database. For our To-Do app, we will use Mongoose to create a schema and model for our tasks.
1.
```bash
cd Todo

npm install mongoose

mkdir models

cd models

touch todo.js
# This command will create a new folder, change directory into the new folder and create a folder inside the models folder.

vim todo.js
```

- Paste this code into the file:
```bash
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
```

2. To update our routes from the file 'api.js':
```bash
cd ..

cd routes

ls

vim api.js
```
- Delete the code inside with :%d and then paste the code below:
```bash
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
```
---

## 👉 MongoDB Database

For our database, we'll be using MongoDB Atlas, a cloud-based database service that will host our To-Do items. Here's how to set it up.

1. Create a MongoDB Account using https://www.mongodb.com/cloud/atlas/register
2. To build your cluster, go to **Overview** → **Cluster** → **Build a Cluster**
    - Choose **M0 (Free)**
    - Cluster name: **todo-db**
    - Provider: Choose **AWS**
    - Region: Cape Town (af-south-1)
    - Then `Create Deployment`
3. Create a Database User by setting the:
    - Username: todo-admin
    - Password: (Set your password), and then click on `Create Database User`
<img width="1888" height="816" alt="Network access" src="https://github.com/user-attachments/assets/148e1aee-a76f-4854-af4f-1c34c755cda8" />

4.  For testing purposes, we'll allow access to the MongoDB database from anywhere (not recommended for production):
    - Click on the `Network Access` Link
    - Click on `Add IP Address` →  `Allow Access From Anywhere` → Turn on the toggle button and choose `1 Week`, then `Confirm.`
    - Go back to the Overview and click on `Cluster` → `Browse Collections` → `Create Database`
        - Database name: todo-db
        - Collection name: todo
        - Then `Create`
5. To connect the todo app to the DataBase:
- Go back to the cluster and click on `Connect` →  select `Drivers` and copy the `connection string`
<img width="1828" height="787" alt="MongoDB" src="https://github.com/user-attachments/assets/55f0e27b-7402-4acc-9546-5094061028a7" />

6. Set up environment variables:
```bash
cd Todo

ls

touch .env

vi .env
```

- Add the connection string to access the database in it, just as below (make sure to edit the password):
```bash
DB = 'mongodb+srv://todo-admin:<PASSWORD>@todo-db.higonoj.mongodb.net/?retryWrites=true&w=majority&appName=todo-db'
```

7. Update index.js:
```bash
vim index.js
```
- Delete the entire content in it and replace it with:
```bash
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
```

- Start the server:
```bash
node index.js
```
<img width="1566" height="355" alt="Server running" src="https://github.com/user-attachments/assets/60cbb446-f30a-4c1f-9198-2d5997dd28b3" />
**Database connected successfully**
---

## 👉 Test Backend Configurations Using Postman

Now that we’ve written the backend part of our To-Do application and configured a database, let’s test our code using RESTful API. We'll use Postman, a powerful API testing tool, to ensure that our endpoints are working correctly.

Our To-Do application should be able to:
- Display a list of tasks - HTTP GET request
- Add a new task to the list - HTTP POST request
- Delete an existing task from the list - HTTP DELETE request

1. Download & install Postman, then create an account: https://www.postman.com/downloads/
<img width="916" height="344" alt="Postman 1" src="https://github.com/user-attachments/assets/f82f6e25-3128-4197-b69a-ca70cc71c3bd" />

2. **Create a POST request**: This will send a new task to our To-Do list so the application can store it in the database.
    - Click on the `+` (plus)  icon to create a new request.
    - Select **POST** from the drop-down menu → put in your URL: `http://<PublicIP-or-PublicDNS>:5000/api/todos`
    - Go to Headers:
        - Key: Content-Type
        - Value: application/json
<img width="1822" height="585" alt="Post request 2" src="https://github.com/user-attachments/assets/f047583e-6f31-4016-a45f-fa18a217e303" />

- Go to Body → Choose Raw and type this command to input in our database:
```bash
{
  "action": "Finish project 8 and 9"
}
```
- Click on 'Send'
<img width="1830" height="616" alt="Post request 3" src="https://github.com/user-attachments/assets/103b3b5e-9788-4b13-97b6-8fc700090f27" />

3.  **Create a GET request to retrieve our tasks**
    - Click on the `+` (plus)  icon to create a new request.
    - Select **GET** from the drop-down menu → put in your URL: `http://<PublicIP-or-PublicDNS>:5000/api/todos`
    - Click on `Send`
<img width="1480" height="828" alt="Get request" src="https://github.com/user-attachments/assets/8d37cb5b-058b-47b4-819f-1b882017868e" />

4. **Create a DELETE request**
    - Click on the `+` (plus)  icon to create a new request.
    - Select **DELETE** from the drop-down menu → put in your URL: `http://<PublicIP-or-PublicDNS>:5000/api/todos` and attach the ID of the request you want to delete.
```bash
http://51.21.198.205:5000/api/todos/68b03748755fd0a617779757
```
- Click on **Send**
<img width="1467" height="769" alt="backend result" src="https://github.com/user-attachments/assets/795993a5-a09d-4fce-9043-295d346f48da" />

With this, we’ve been able to test the backend part of our To-Do application and ensure that it supports all three operations we wanted.


---

#  Step 2: Frontend Creation

Now that our backend and API are fully set up, let’s create a user interface for a Web client (browser) to interact with the To-Do app via the API. We’ll achieve this using React (a powerful library for building dynamic user interfaces).

In the same root directory as your backend code (Todo directory), run:
```bash
cd Todo

npx create-react-app client
```

## 👉 Running a React App

1. Install Concurrently and Nodeman:
```bash
npm install concurrently --save-dev

npm install nodemon --save-dev
```

2. In the Todo folder, open the 'package.json file', and then change the highlighted part of the screenshot with the code below:
```bash
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```
<img width="1132" height="708" alt="Script" src="https://github.com/user-attachments/assets/b70fe18c-f910-48b4-b64e-b791c26a4af8" />

3. Change directory to 'client':
```bash
cd client

vi package.json
```

- Add the key value pair in the package.json file:
```bash
"proxy": "http://localhost:5000"
```
<img width="1366" height="703" alt="Proxy" src="https://github.com/user-attachments/assets/e34c1e84-ff40-4e34-b685-6faf05d84a8c" />

4. Go back to the Todo directory:
```bash
cd ..

npm run dev
```
<img width="1836" height="940" alt="Client browser" src="https://github.com/user-attachments/assets/8591b00b-f9a5-4f83-aadd-c15bf498c42a" />
Our app is now open and running on localhost:3000

5. To access the application from the Internet, go to your EC2 Instance and open TCP Port 3000 by adding a new inbound rule in the security group.
<img width="1832" height="366" alt="Inbound rule" src="https://github.com/user-attachments/assets/baedd97e-1d6a-4111-9df9-1ef5c2a115fb" />

---

## 👉 Creating the React Components

We will built three main React components:
- **Input.js** – Handles adding new todos
- **ListTodo.js** – Displays a list of todos and allows deletion
- **Todo.js** – The main component that integrates Input and ListTodo

Finally, **App.js** renders the Todo component to display the app.

1. Go to the Todo directory and run:
```bash
cd client

cd src

mkdir components

cd components

touch Input.js ListTodo.js Todo.js

vi Input.js
```

- Copy and paste the following:
```bash
import React, { Component } from 'react';
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

export default Input
```

2. cd into 'client' to make use of Axios, which is a Promise-based HTTP client for the browser and Node.js:
```bash
cd ..
cd ..

npm install axios

cd src/components

vi ListTodo.js
```

- In the ListTodo.js, copy and paste the following code:
```bash
import React from 'react';

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

export default ListTodo
```

3. Open the Todo.js file:
```bash
nano Todo.js
```

- Paste the following:
```bash
import React, {Component} from 'react';
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

export default Todo;
```

4. Move to the src folder and replace the content in App.js (by deleting the logo) with the code below:
```bash
cd ..

vi App.js
```

- Paste the code:
```bash
import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;
```

5. In the src directory, open the App.css:
```bash
vi App.css
```

- Paste the code:
```bash
.App {
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
}
```

6. In the src directory, open the index.css:
```bash
vim index.css
```

- Paste the code:
```bash
body {
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
}
```

7. Go to the Todo directory and run the command to start both the backend and frontend concurrently:
```bash
cd ../..

npm run dev
```
- Backend runs on http://localhost:5000
- Frontend runs on http://localhost:3000

7. View the App in Your Browser: 'http://Public_IP_Address:3000'
<img width="1625" height="890" alt="My Todo(s)" src="https://github.com/user-attachments/assets/d66adcfd-6be7-49c3-907c-d0299ef860fc" />
<img width="1467" height="769" alt="backend result" src="https://github.com/user-attachments/assets/67a3e4c7-f6fd-4927-8f8a-4283e8b7d19a" />
Backend result on Postman

---

This project demonstrates a complete MERN stack workflow:
- Setting up a backend with Express & MongoDB
- Defining routes and models with Mongoose
- Building a frontend with React
- Testing with Postman
- Deploying on AWS

This project is a practical introduction to building scalable full-stack applications and forms a strong foundation for more complex projects.


