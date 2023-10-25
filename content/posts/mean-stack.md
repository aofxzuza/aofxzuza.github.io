+++
title = 'Getting Started for MEAN Stack Part 1'
date = 2017-02-17T10:41:07
draft = false
tags = ['coding','web', 'mean', 'mongodb', 'express', 'angular', 'nodejs']
+++

## What is MEAN Stack?
MEAN is a framework for an easy starting point with
*   MongoDB
*   Express
*   Angularjs
*   Nodejs

It's designed to give you quick and organized way to start developing MEAN based web apps with useful modules. More information [here](http://mean.io/)

## Getting Started
### Prerequisite
1.  Install [Nodejs](https://nodejs.org/en/download/)
2.  Install [Git](https://git-scm.com/downloads)
3.  Install [MongoDB](https://www.mongodb.com/download-center#community)
4.  Install Bower by typing
    ```sh
    npm install bower -g
    ```

### Installation
1.  Create project “music_app”
    ```sh
    mkdir music_app
    cd music_app
    npm init
    ```
2.  Install npm dependencies
    ```sh
    npm install express mongoose body-parser --save
    ```
    **package.json**
    ```json
      {
      "name": "music_app",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC",
      "dependencies": {
        "body-parser": "^1.18.2",
        "express": "^4.16.2",
        "mongoose": "^5.0.0-rc1"
      }
    }
    ```
3.  Create bower.json by typing
    ```sh
    bower init
    ```
4.  Install dependencies for <span style="text-decoration:underline;">**client side**</span> via bower by
    ```sh
    bower install angular-ui-router bootstrap traceur angular-modal-serice font-awesome --save
    ```
    **bower.json**
    ```json
    {
      "name": "music_app",
      "description": "",
      "main": "index.js",
      "authors": [
        "Rattapong Khamphansa <aof.kem@gmail.com>"
      ],
      "license": "ISC",
      "homepage": "",
      "ignore": [
        "**/.*",
        "node_modules",
        "bower_components",
        "test",
        "tests"
      ],
      "dependencies": {
        "angular-ui-router": "^1.0.12",
        "font-awesome": "^4.7.0",
        "angular-modal-service": "^0.13.0",
        "bootstrap": "^3.3.7",
        "traceur": "^0.0.111"
      }
    }
    ```
5.  Then we will have project structure like this
    <pre>
      music_app
      |-> bower_components
        |-> angular
        |-> angular-modal-service
        |-> angular-ui-router
        |-> bootstrap
        |-> font-awesome
        |-> jquery
        |-> traceur
      |-> node_modules
        |-> body-parser
        |-> express
        |-> mongoose
      |-> bower.json
      |-> package.json
    </pre>

### Implementation
#### Create provide web route
1.  Create `app.js` file
    ```js
    //Import and declare express
    var express = require('express');
    var app = express();

    //Provide static files for client accessible
    app.use('/public', express.static('bower_components'));
    app.use(express.static('app'));

    //Provide route to index page to app/Index.html
    app.get('/', function(request, response) {
        response.render('Index.html', {});
    });
    //listen to port 8000
    app.listen(8000, function() {
        console.log('Express server started!!!');
    });
    ```
2.  Create `Index.html` in `app` directory
    ```html
    <!DOCTYPE html>
    <html>

    <head lang="en">
    </head>

    <body>
        <h1>Hello World</h1>
    </body>

    </html>
    ```
3.  Then we will have project structure like this
    <pre>
    music_app
    |-> app
      |-> Index.html
    |-> bower_components
      |-> angular
      |-> angular-modal-service
      |-> angular-ui-router
      |-> bootstrap
      |-> font-awesome
      |-> jquery
      |-> traceur
    |-> node_modules
      |-> body-parser
      |-> express
      |-> mongoose
    |-> app.js
    |-> bower.json
    |-> package.json
    </pre>
4.  Run node server by typing
    ```sh
    node app.js
    ```
5.  Open browser and go to [http://localhost:8000/](http://localhost:8000/)
6.  See result
![MEAN Result](/imgs/mean-result.png)

## Source Code
[Github](https://github.com/example-mean-stack/example-part1)

