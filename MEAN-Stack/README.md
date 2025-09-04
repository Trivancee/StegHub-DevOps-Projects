# Implementing a Simple Web Book Register Application with MEAN Stack

This project is a **Simple Web Book Register Application** built with the **MEAN stack** (MongoDB, Express, AngularJS, Node.js). It allows users to add, view, and delete books stored in a MongoDB database.

- **MongoDB** → Stores book data (name, ISBN, author, pages).  
- **Express.js** → Backend framework to handle API requests.  
- **AngularJS** → Frontend framework for dynamic user interface.  
- **Node.js** → Runtime environment for running server-side code.  

---

## 🛠️ Prerequisites

- AWS EC2 instance (Ubuntu 24.04 LTS recommended, t3.micro free tier).  
- Security Group:
  - Port **22** → SSH access  
  - Port **3300** → Application access

---

## ⚙️ Step 1: Install NodeJs
1. Connect to EC2
```bash
ssh -i <your-private-key.pem> ubuntu@<EC2-Public-IP>
```

2. Update and upgrade Ubuntu
```bash
sudo apt update

sudo apt upgrade -y
```

3. Add certificates:
```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

```bash
sudo apt install -y nodejs
# Install nodejs (includes npm)
```
---

## ⚙️ Step 2: Install and Configure MongoDB
MongoDB will serve as the database for this application, storing book records such as name, ISBN, author, and number of pages.

1. Ensure curl & gnupg are present:
```bash
sudo apt-get install -y gnupg curl
```

2. Import GPG Key:
```bash
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
  --dearmor
```

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
   sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

3. Install and start MongoDB:
```bash
sudo apt-get update

sudo apt-get install -y mongodb-org

sudo service mongod start
```

4. Verify the status of MongodDB:
```bash
sudo systemctl status mongod
```
<img width="544" height="145" alt="Mongod Running" src="https://github.com/user-attachments/assets/822ee467-7615-4bb2-9e07-90a8b2f5a48e" />

5. Install body-parser:
```bash
sudo npm install body-parser
```
---

## ⚙️ Step 3: Create the Project Folder and File

1. Create and enter the project folder
```bash
mkdir Books && cd Books

npm init
```
- Click on Enter until all details are displayed:
<img width="859" height="577" alt="books mean stack" src="https://github.com/user-attachments/assets/f9054785-4ad9-46bd-a7e2-6714a3c35927" />

2. Create the server entry file:
```bash
nano server.js
```

- Copy and paste the web server code below into the server.js file:
```bash
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3300;

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/test', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.listen(PORT, () => {
  console.log(`Server up: http://localhost:${PORT}`);
});
```
---

## ⚙️ Step 4: Install Express and set up Routes to the server
Express.js provides the backend framework for handling API requests, while Mongoose manages interactions with MongoDB and body-parser enables parsing of incoming JSON data in requests.

1. Install Dependencies:
```bash
npm install express@4 mongoose body-parser
```

2. In the 'Books' folder, create a folder named apps and a file named routes.js:
```bash
mkdir apps && cd apps

nano routes.js
```
- Copy and paste the code below into 'routes.js':
```bash
const Book = require('./models/book');
const path = require('path');

module.exports = function(app) {
  app.get('/book', async (req, res) => {
    try {
      const books = await Book.find();
      res.json(books);
    } catch (err) {
      res.status(500).json({ message: 'Error fetching books', error: err.message });
    }
  });

  app.post('/book', async (req, res) => {
    try {
      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });
      const savedBook = await book.save();
      res.status(201).json({
        message: 'Successfully added book',
        book: savedBook
      });
    } catch (err) {
      res.status(400).json({ message: 'Error adding book', error: err.message });
    }
  });

  app.delete('/book/:isbn', async (req, res) => {
    try {
      const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
      if (!result) {
        return res.status(404).json({ message: 'Book not found' });
      }
      res.json({
        message: 'Successfully deleted the book',
        book: result
      });
    } catch (err) {
      res.status(500).json({ message: 'Error deleting book', error: err.message });
    }
  });

  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, '../public', 'index.html'));
  });
};
```

3. In the 'apps' folder, create a folder named models and a file named 'book.js':
```bash
mkdir models && cd models

vi book.js
```

- Copy and paste the code below into 'book.js':
```bash
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
  name: { type: String, required: true },
  isbn: { type: String, required: true, unique: true, index: true },
  author: { type: String, required: true },
  pages: { type: Number, required: true, min: 1 }
}, {
  timestamps: true
});

module.exports = mongoose.model('Book', bookSchema);
```
---

## ⚙️ Step 5: Access the Routes with AngularJS
AngularJS powers the frontend of the application, allowing users to interact with the backend routes through a dynamic interface. It sends HTTP requests to the Express server to add, fetch, and delete books, and updates the user interface in real time without requiring a page reload.

1. Change the directory back to 'Books':
```bash
cd ../..

mkdir public && cd public

nano script.js
```

- Copy and paste the code below:
```bash
angular.module('myApp', [])
  .controller('myCtrl', function($scope, $http) {
    function fetchBooks() {
      $http.get('/book')
        .then(response => {
          $scope.books = response.data;
        })
        .catch(error => {
          console.error('Error fetching books:', error);
        });
    }

    fetchBooks();

    $scope.del_book = function(book) {
      $http.delete(`/book/${book.isbn}`)
        .then(() => {
          fetchBooks();
        })
        .catch(error => {
          console.error('Error deleting book:', error);
        });
    };

    $scope.add_book = function() {
      const newBook = {
        name: $scope.Name,
        isbn: $scope.Isbn,
        author: $scope.Author,
        pages: $scope.Pages
      };

      $http.post('/book', newBook)
        .then(() => {
          fetchBooks();
          // Clear form fields
          $scope.Name = $scope.Isbn = $scope.Author = $scope.Pages = '';
        })
        .catch(error => {
          console.error('Error adding book:', error);
        });
    };
  });
```

2. In the 'public' folder, create a file named index.html:
```bash
nano index.html
```

- Copy and paste the code below into the index.html file.
```bash
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Book Management</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
  <script src="script.js"></script>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    table { border-collapse: collapse; width: 100%; }
    th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
    th { background-color: #f2f2f2; }
    input[type="text"], input[type="number"] { width: 100%; padding: 5px; }
    button { margin-top: 10px; padding: 5px 10px; }
  </style>
</head>
<body>
  <h1>Book Management</h1>
  
  <h2>Add New Book</h2>
  <form ng-submit="add_book()">
    <table>
      <tr>
        <td>Name:</td>
        <td><input type="text" ng-model="Name" required></td>
      </tr>
      <tr>
        <td>ISBN:</td>
        <td><input type="text" ng-model="Isbn" required></td>
      </tr>
      <tr>
        <td>Author:</td>
        <td><input type="text" ng-model="Author" required></td>
      </tr>
      <tr>
        <td>Pages:</td>
        <td><input type="number" ng-model="Pages" required></td>
      </tr>
    </table>
    <button type="submit">Add Book</button>
  </form>

  <h2>Book List</h2>
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>ISBN</th>
        <th>Author</th>
        <th>Pages</th>
        <th>Action</th>
      </tr>
    </thead>
    <tbody>
      <tr ng-repeat="book in books">
        <td>{{book.name}}</td>
        <td>{{book.isbn}}</td>
        <td>{{book.author}}</td>
        <td>{{book.pages}}</td>
        <td><button ng-click="del_book(book)">Delete</button></td>
      </tr>
    </tbody>
  </table>
</body>
</html>
```

3. Change the directory to 'Books' and start the server:
```bash
cd ..

node server.js
```
<img width="1279" height="423" alt="Server 3300" src="https://github.com/user-attachments/assets/4f20b3fd-9395-4774-9f12-881c71938d52" />
**The server is now up and running, and we can connect to it via port 3300**.

- You can launch a separate SSH console to test what the curl command returns locally:
```bash
curl -s http://localhost:3300
```
<img width="550" height="418" alt="curl result" src="https://github.com/user-attachments/assets/d328c8eb-9196-4b19-b3ac-618e58c0b2be" />

4. Open TCP port 3300 in your EC2 Instance
<img width="828" height="124" alt="inbound rule 2" src="https://github.com/user-attachments/assets/30a93325-f566-433f-b34a-9a363b3e59db" />

- Access the Book Register web application from your browser using your Public IP address or the Public DNS name:
<img width="845" height="375" alt="Book management" src="https://github.com/user-attachments/assets/6cf3e05d-668f-45e7-bd2d-81d9ac2b4d9e" />

By following these steps, we successfully built and deployed a Simple Web Book Register Application using the MEAN stack on an Ubuntu server in AWS. The application demonstrates how backend APIs (Node.js, Express, MongoDB) and a frontend framework (AngularJS) work together to create a functional full-stack solution.
---

# ❌ Trobleshooting
1. MongoDB-org failed to install
<img width="1854" height="996" alt="Debugging MEAN 1" src="https://github.com/user-attachments/assets/3fcb5e75-3609-48b5-a609-9f105da6b157" />


This installation failed because I didn’t import the MongoDB GPG key, and thus the repo wasn’t properly authenticated.

**To Fix This**:
- Import the MongoDB GPG key:
```bash
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
```

- Re-add the repository:
```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

- Update package lists and install MongoDB:
```bash
sudo apt-get update

sudo apt-get install -y mongodb-org
```
- Start and verify the status of MongodDB:
```bash
sudo service mongod start

sudo systemctl status mongod
```

---

2. 'sudo apt install -y npm', Failed
<img width="1084" height="474" alt="Install npm failed" src="https://github.com/user-attachments/assets/51353728-bcfc-43e8-8ba6-72d82b1036ca" />

**To Fix This**:
- Remove the old Node.js / npm packages and update:
```bash
sudo apt-get remove -y nodejs npm

sudo apt-get update

sudo apt-get install -y nodejs
# Install Node.js (includes npm)
```
- Check Versions:
```bash
node -v

npm -v
```

---

3. Missing Parameter after running 'node server.js'
<img width="1371" height="532" alt="Missing parameter" src="https://github.com/user-attachments/assets/6d9d85e3-5e68-40b8-9285-29d2cf12b013" />

The error occurred after running 'node server.js' because Express was installed with 'sudo npm install express mongoose', which pulled in Express 5.x by default. Express 5 uses the **path-to-regexp** package with stricter parsing rules, so one of the route paths was considered invalid and caused the application to crash.

**To Fix This**:
- Install the stable Express 4 release:
```bash
sudo npm install express@4 mongoose
```
This resolves the **path-to-regexp** error and allows the application to run without crashing.

- Run the command:
```bash
node server.js
```
<img width="1279" height="423" alt="Server 3300" src="https://github.com/user-attachments/assets/d5d6dced-3ca3-4a72-b081-d99507fc0788" />

## Lessons Learned
- MongoDB installation on Ubuntu requires GPG keys to be properly added.
- Always check Express version (stick to v4 for stability).

---

# 🏁 Conclusion

This project not only highlights the deployment process but also reinforces practical debugging and problem-solving skills essential for real-world DevOps and software development.
