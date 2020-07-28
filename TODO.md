# Express API Template

Open your terminal, create a new directory, and start a Node.js project.

```sh
### create a new directory
mkdir express-api-template

### change to the newly-created directory
cd express-api-template

### initialize a new Node.js project
npm init
```

Answer the questions that follow to generate a package.json file. This file holds information about your project. Example of such information includes what dependencies it uses, the command to start the project, and so on.

Create the following files in the project folder:

- _README.md_
- _.editorconfig_

Open `.editorconfig` and paste the following code:

```js
root = true
[*]
indent_style = space
indent_size = 2
charset = utf-8
trim_trailing_whitespace = false
insert_final_newline = true
```

The `[*]` means that we want to apply the rules that come under it to every file in the project. We want an indent size of two spaces and `UTF-8` character set. We also want to trim trailing white space and insert a final empty line in our file.

Open _README.md_ and add the project name as a first-level element.

```js
# Express API template
```

Let’s add version control right away.

```sh
# initialize the project folder as a git repository
git init
```

Create a _.gitignore_ file and enter the following lines:

```sh
node_modules/
yarn-error.log
.env
.nyc_output
coverage
build/
```

These are all the files and folders we don’t want to track. We don’t have them in our project yet, but we’ll see them as we proceed.

At this point, you should have the following folder structure.

```sh
EXPRESS-API-TEMPLATE
├── .editorconfig
├── .gitignore
├── package.json
└── README.md
```

I consider this to be a good point to commit my changes and push them to GitHub.ù

# Starting A New Express Project

Express is a Node.js framework for building web applications. According to the [official website](https://expressjs.com/), it is a

> Fast, unopinionated, minimalist web framework for [Node.js](https://nodejs.org/en/).

There are other great web application frameworks for Node.js, but Express is very popular, with over 47k GitHub stars at the time of this writing.

In this article, we will not be having a lot of discussions about all the parts that make up Express. For that discussion, I recommend you check out Jamie’s series. The first part is [here](https://www.smashingmagazine.com/2019/02/node-api-http-es6-javascript/), and the second part is [here](https://www.smashingmagazine.com/2019/11/express-es6-javascript-stack-mongodb-mongoose-servers/).

Install Express and start a new Express project. It’s possible to manually set up an Express server from scratch but to make our life easier we’ll use the [express-generator](https://expressjs.com/en/starter/generator.html) to set up the app skeleton.

```sh
# install the express generator globally
yarn global add express-generator

# install express
yarn add express

# generate the express project in the current folder
express -f
```

The `-f` flag forces Express to create the project in the current directory.

We’ll now perform some house-cleaning operations.

1. Delete the file _routes/users.js_.
2. Delete the folders `public/` and `views/`.
3. Rename the file _bin/www_ to _bin/www.js_.
4. Uninstall `jade` with the command `yarn remove jade`.
5. Create a new folder named `src/` and move the following inside it:
   - _app.js_ file
   - `bin/` folder
   - `routes/` folder inside.
6. Open up package.json and update the `start` script to look like below.

```json
"start": "node ./src/bin/www"
```

At this point, your project folder structure looks like below. You can see how VS Code highlights the file changes that have taken place.

```sh
EXPRESS-API-TEMPLATE
├── node_modules
├── src
|   ├── bin
│   │   ├── www.js
│   ├── routes
│   |   ├── index.js
│   └── app.js
├── .editorconfig
├── .gitignore
├── package.json
├── README.md
└── yarn.lock
```

Open _src/app.js_ and replace the content with the below code.

```js
var logger = require("morgan");
var express = require("express");
var cookieParser = require("cookie-parser");
var indexRouter = require("./routes/index");
var app = express();

app.use(logger("dev"));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());
app.use("/v1", indexRouter);

module.exports = app;
```

After requiring some libraries, we instruct Express to handle every request coming to `/v1` with `indexRouter`.

Replace the content of _routes/index.js_ with the below code:

```js
var express = require("express");
var router = express.Router();
router.get("/", function (req, res, next) {
  return res.status(200).json({ message: "Welcome to Express API template" });
});
module.exports = router;
```

We grab Express, create a router from it and serve the `/` route, which returns a status code of `200` and a JSON message.

Start the app with the below command:

```sh
# start the app
yarn start
```

If you’ve set up everything correctly you should only see `$ node ./src/bin/www` in your terminal.

Visit `http://localhost:3000/v1` in your browser. You should see the following message:

```json
{
  "message": "Welcome to Express API template"
}
```

This is a good point to commit our changes.

- _The corresponding branch in my repo is [01-install-express](https://github.com/chidimo/Express-API-Template/tree/01-install-express)_.

# Converting Our Code To ES6

The code generated by `express-generator` is in `ES5`, but in this article, we will be writing all our code in `ES6` syntax. So, let’s convert our existing code to `ES6`.

Replace the content of _routes/index.js_ with the below code:

```js
import express from "express";

const indexRouter = express.Router();

indexRouter.get("/", (req, res) =>
  res.status(200).json({ message: "Welcome to Express API template" })
);

export default indexRouter;
```

It is the same code as we saw above, but with the import statement and an arrow function in the `/` route handler.

Replace the content of _src/app.js_ with the below code:

```js
import logger from "morgan";
import express from "express";
import cookieParser from "cookie-parser";
import indexRouter from "./routes/index";
const app = express();
app.use(logger("dev"));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());
app.use("/v1", indexRouter);

export default app;
```

Let’s now take a look at the content of _src/bin/www.js_. We will build it incrementally. Delete the content of `src/bin/www.js` and paste in the below code block.

```js
#!/usr/bin/env node
/**
 * Module dependencies.
 */
import debug from "debug";
import http from "http";
import app from "../app";
/**
 * Normalize a port into a number, string, or false.
 */
const normalizePort = (val) => {
  const port = parseInt(val, 10);
  if (Number.isNaN(port)) {
    // named pipe
    return val;
  }
  if (port >= 0) {
    // port number
    return port;
  }
  return false;
};

/**
 * Get port from environment and store in Express.
 */
const port = normalizePort(process.env.PORT || "3000");
app.set("port", port);

/**
 * Create HTTP server.
 */
const server = http.createServer(app);

// next code block goes here
```

This code checks if a custom port is specified in the environment variables. If none is set the default port value of `3000` is set on the app instance, after being normalized to either a string or a number by `normalizePort`. The server is then created from the `http` module, with `app` as the callback function.

The `#!/usr/bin/env node` line is optional since we would specify node when we want to execute this file. But make sure it is on line 1 of _src/bin/www.js_ file or remove it completely.

Let’s take a look at the error handling function. Copy and paste this code block after the line where the server is created.

```js
/**
 * Event listener for HTTP server "error" event.
 */
const onError = (error) => {
  if (error.syscall !== "listen") {
    throw error;
  }
  const bind = typeof port === "string" ? `Pipe ${port}` : `Port ${port}`;
  // handle specific listen errors with friendly messages
  switch (error.code) {
    case "EACCES":
      alert(`${bind} requires elevated privileges`);
      process.exit(1);
      break;
    case "EADDRINUSE":
      alert(`${bind} is already in use`);
      process.exit(1);
      break;
    default:
      throw error;
  }
};

/**
 * Event listener for HTTP server "listening" event.
 */
const onListening = () => {
  const addr = server.address();
  const bind = typeof addr === "string" ? `pipe ${addr}` : `port ${addr.port}`;
  debug(`Listening on ${bind}`);
};
/**
 * Listen on provided port, on all network interfaces.
 */
server.listen(port);
server.on("error", onError);
server.on("listening", onListening);
```

The `onError` function listens for errors in the http server and displays appropriate error messages. The `onListening` function simply outputs the port the server is listening on to the console. Finally, the server listens for incoming requests at the specified address and port.

At this point, all our existing code is in `ES6` syntax. Stop your server (use `Ctrl + C`) and run `yarn start`. You’ll get an error `SyntaxError: Invalid or unexpected token`. This happens because Node (at the time of writing) doesn’t support some of the syntax we’ve used in our code.

We’ll now fix that in the following section.

# Configuring Development Dependencies: _babel_, _nodemon_, _eslint_, and _prettier_

It’s time to set up most of the scripts we’re going to need at this phase of the project.

Install the required libraries with the below commands. You can just copy everything and paste it in your terminal. The comment lines will be skipped.

```sh
# install babel scripts
yarn add @babel/cli @babel/core @babel/plugin-transform-runtime @babel/preset-env @babel/register @babel/runtime @babel/node --dev
```

This installs all the listed babel scripts as development dependencies. Check your _package.json_ file and you should see a `devDependencies` section. All the installed scripts will be listed there.

The babel scripts we’re using are explained below:

| Libraries                       | Description                                                                                                                                           |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| @babel/cli                      | A required install for using `babel`. It allows the use of Babel from the terminal and is available as `./node_modules/.bin/babel`.                   |
| @babel/core                     | Core Babel functionality. This is a required installation.                                                                                            |
| @babel/node                     | This works exactly like the Node.js CLI, with the added benefit of compiling with babel presets and plugins. This is required for use with `nodemon`. |
| @babel/plugin-transform-runtime | This helps to avoid duplication in the compiled output.                                                                                               |
| @babel/preset-env               | A collection of plugins that are responsible for carrying out code transformations.                                                                   |
| @babel/register                 | This compiles files on the fly and is specified as a requirement during tests.                                                                        |
| @babel/runtime                  | This works in conjunction with `@babel/plugin-transform-runtime`.                                                                                     |

Create a file named _.babelrc_ at the root of your project and add the following code:

```json
{
  "presets": ["@babel/preset-env"],
  "plugins": ["@babel/transform-runtime"]
}
```

Let’s install `nodemon`

```sh
# install nodemon
yarn add nodemon --dev
```

`nodemon` is a library that monitors our project source code and automatically restarts our server whenever it observes any changes.

Create a file named _nodemon.json_ at the root of your project and add the code below:

```json
{
  "watch": [
    "package.json",
    "nodemon.json",
    ".eslintrc.json",
    ".babelrc",
    ".prettierrc",
    "src/"
  ],
  "verbose": true,
  "ignore": ["*.test.js", "*.spec.js"]
}
```

The `watch` key tells `nodemon` which files and folders to watch for changes. So, whenever any of these files changes, nodemon restarts the server. The `ignore` key tells it the files not to watch for changes.

Now update the `scripts` section of your package.json file to look like the following:

```sh
# build the content of the src folder
"prestart": "babel ./src --out-dir build"

# start server from the build folder
"start": "node ./build/bin/www"

# start server in development mode
"startdev": "nodemon --exec babel-node ./src/bin/www"
```

1. `prestart` scripts builds the content of the `src/` folder and puts it in the `build/` folder. When you issue the `yarn start` command, this script runs first before the `start` script.
2. `start` script now serves the content of the `build/` folder instead of the `src/` folder we were serving previously. This is the script you’ll use when serving the file in production. In fact, services like Heroku automatically run this script when you deploy.
3. `yarn startdev` is used to start the server during development. From now on we will be using this script as we develop the app. Notice that we’re now using `babel-node` to run the app instead of regular `node`. The `--exec` flag forces `babel-node` to serve the `src/` folder. For the `start` script, we use `node` since the files in the `build/` folder have been compiled to ES5.

Run `yarn startdev` and visit [http://localhost:3000/v1](http://localhost:3000/v1). Your server should be up and running again.

The final step in this section is to configure `ESLint` and `prettier`. ESLint helps with enforcing syntax rules while prettier helps for formatting our code properly for readability.

Add both of them with the command below. You should run this on a separate terminal while observing the terminal where our server is running. You should see the server restarting. This is because we’re monitoring _package.json_ file for changes.

```sh
# install elsint and prettier
yarn add eslint eslint-config-airbnb-base eslint-plugin-import prettier --dev
```

Now create the _.eslintrc.json_ file in the project `root` and add the below code:

```json
{
  "env": {
    "browser": true,
    "es6": true,
    "node": true,
    "mocha": true
  },
  "extends": ["airbnb-base"],
  "globals": {
    "Atomics": "readonly",
    "SharedArrayBuffer": "readonly"
  },
  "parserOptions": {
    "ecmaVersion": 2018,
    "sourceType": "module"
  },
  "rules": {
    "indent": ["warn", 2],
    "linebreak-style": ["error", "unix"],
    "quotes": ["error", "single"],
    "semi": ["error", "always"],
    "no-console": 1,
    "comma-dangle": [0],
    "arrow-parens": [0],
    "object-curly-spacing": ["warn", "always"],
    "array-bracket-spacing": ["warn", "always"],
    "import/prefer-default-export": [0]
  }
}
```

This file mostly defines some rules against which `eslint` will check our code. You can see that we’re extending the style rules used by Airbnb.

In the `"rules"` section, we define whether `eslint` should show a warning or an error when it encounters certain violations. For instance, it shows a warning message on our terminal for any indentation that does not use 2 spaces. A value of `[0]` turns off a rule, which means that we won’t get a warning or an error if we violate that rule.

Create a file named _.prettierrc_ and add the code below:

```json
{
  "trailingComma": "es5",
  "tabWidth": 2,
  "semi": true,
  "singleQuote": true
}
```

We’re setting a tab width of `2` and enforcing the use of single quotes throughout our application. Do check the prettier [guide](https://prettier.io/docs/en/options.html) for more styling options.

Now add the following scripts to your _package.json_:

```json
# add these one after the other

"lint": "./node_modules/.bin/eslint ./src"

"pretty": "prettier --write '**/*.{js,json}' '!node_modules/**'"

"postpretty": "yarn lint --fix"
```

Run `yarn lint`. You should see a number of errors and warnings in the console.

The `pretty` command prettifies our code. The `postpretty` command is run immediately after. It runs the `lint` command with the `--fix` flag appended. This flag tells `ESLint` to automatically fix common linting issues. In this way, I mostly run the `yarn pretty` command without bothering about the `lint` command.

Run `yarn pretty`. You should see that we have only two warnings about the presence of `alert` in the _bin/www.js_ file.

Here’s what our project structure looks like at this point.

```sh
EXPRESS-API-TEMPLATE
├── build
├── node_modules
├── src
|   ├── bin
│   │   ├── www.js
│   ├── routes
│   |   ├── index.js
│   └── app.js
├── .babelrc
├── .editorconfig
├── .eslintrc.json
├── .gitignore
├── .prettierrc
├── nodemon.json
├── package.json
├── README.md
└── yarn.lock
```

You may find that you have an additional file, `yarn-error.log` in your project root. Add it to `.gitignore` file. Commit your changes.

- _The corresponding branch at this point in my repo is [02-dev-dependencies](https://github.com/chidimo/Express-API-Template/tree/02-dev-dependencies)_.
