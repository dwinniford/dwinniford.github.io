---
layout: post
title:      "Getting started with MERN"
date:       2020-07-30 21:52:04 +0000
permalink:  getting_started_with_mern
---


I'm excited to get started building a new language learning tool with MERN, Amazon Transcribe and Amazon Polly.  Here's some of my first steps in setting up the MERN stack!

### Generating the front and back end

I'm using [redux](http://redux.js.org/introduction/getting-started) so I used the following command to generate a React/Redux template:

```
npx create-react-app my-app --template redux
```

For Express i used the following [generator](http://expressjs.com/en/starter/generator.html) to build an API layout with no views:

```
npx express-generator --no-view my-app-backend
```

### Setting up the back end

In `bin/www` make sure to set the port to something different than the front end.

```
var port = normalizePort(process.env.PORT || '7777');
app.set('port', port);
```

To make sure the back end is working correctly, first delete the public folder and references to it in 'app.js'.  If you don't remove these the `express.static` [method](http://expressjs.com/en/starter/static-files.html) will cause the 'index.html' file to still be rendered.  Then create a json response for the '/' route in 'index.js':

```
router.get('/', function(req, res, next) {
  res.json({ title: 'Home Page' });
});
```

Run npm start and open 'localhost:7777' to check the response.

### Enabling CORS

To allow the React front end to access the back end run `npm install cors` to add the necessary [middleware](http://expressjs.com/en/resources/middleware/cors.html) and add the following to `app.js`:

```
var express = require('express');
var cors = require('cors') // ** 
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');
var indexRouter = require('./routes/index');
var usersRouter = require('./routes/users');
var corsOptions = {
    origin: 'http://localhost:3000', 
    optionsSuccessStatus: 200
  } // **
var app = express();
app.use(cors(corsOptions)) // **
```

This will make the back end accessible on all requests from the front end.

### Fetching from the front end with react hooks

To make sure the front and the back end are working properly together I'm going to set up a fetch using [react hooks](http://reactjs.org/docs/hooks-effect.html).  In 'app.js' add the following:

```
import React, {useState, useEffect} from 'react';
import './App.css';

function App() {
  const [title, setTitle] = useState("Waiting")

  useEffect(() => {
    fetch('http://localhost:7777')
      .then(res => res.json())
      .then(json => setTitle(json.title))
  })
  return (
    <div className="App">
      <h1>{title}</h1>
    </div>
  );
}

export default App;
```

The 'effect' hook allows you to perform side effects including data fetching.  'useEffect' will run after the first render and after every update.

Run npm start and your home page will display the title sent in json from Express.

### Connecting to the database

In MongoDB Compass create a new database and collection.

Login to MongoDB online.

Under your Cluster choose Connect >> Connect your application and copy the connection string.

In order to make my DB connection work I had to remove 'retryWrites=true' from the connection string.
See the error topic [here](http://github.com/jdesboeufs/connect-mongo/issues/292). 

Add a `.env` file to the express app and add 
`DATABASE="connection string"`

Also make sure you have a `.gitignore` file so you don't commit your database secrets to github. Instructions [here](http://docs.github.com/en/github/using-git/ignoring-files).  

```
npm install mongoose
npm install dotenv
```

In `bin/www` add the following:

```
const mongoose = require('mongoose');
// import the database variable from our variables.env file
require('dotenv').config({ path: 'variables.env' });
// Connect to the database
mongoose.connect(process.env.DATABASE);
mongoose.Promise = global.Promise; 
mongoose.connection.on('error', (err) => {
  console.error(err.message);
});
// import the model 
require('./models/Flashcard')
```

In `models/Flashcard.js` define the model:

```
const mongoose = require('mongoose')
mongoose.Promise = global.Promise;
const flashcardSchema = new mongoose.Schema({
    front: {
        type: String,
        trim: true,
        required: 'Please enter a word for the front of the card!'
    },
    back: {
        type: String,
        trim: true,
        required: 'Please enter a word for the back of the card!'
    }
})
module.exports = mongoose.model('Flashcard', flashcardSchema)
```

### Setting up a route and controller to save to the database

In `Index.js`:

```
const flashcardController = require('../controllers/flashcardController')

router.post('/flashcards', flashcardController.create )
```


In controllers/flashcardController.js:

```
const mongoose = require('mongoose')
const Flashcard = mongoose.model('Flashcard')
exports.create = async (req, res) => {
    const flashcard = await (new Flashcard(req.body)).save()
    res.json(flashcard)
}
```

Setup a form in react to post to this url and the new Flashcard should appear in your database.  






