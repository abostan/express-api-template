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
var logger = require('morgan');
var express = require('express');
var cookieParser = require('cookie-parser');
var indexRouter = require('./routes/index');
var app = express();

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());
app.use('/v1', indexRouter);

module.exports = app;
```

After requiring some libraries, we instruct Express to handle every request coming to `/v1` with `indexRouter`.

Replace the content of _routes/index.js_ with the below code:

```js
var express = require('express');
var router = express.Router();
router.get('/', function (req, res, next) {
  return res.status(200).json({ message: 'Welcome to Express API template' });
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
import express from 'express';

const indexRouter = express.Router();

indexRouter.get('/', (req, res) =>
  res.status(200).json({ message: 'Welcome to Express API template' })
);

export default indexRouter;
```

It is the same code as we saw above, but with the import statement and an arrow function in the `/` route handler.

Replace the content of _src/app.js_ with the below code:

```js
import logger from 'morgan';
import express from 'express';
import cookieParser from 'cookie-parser';
import indexRouter from './routes/index';
const app = express();
app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());
app.use('/v1', indexRouter);

export default app;
```

Let’s now take a look at the content of _src/bin/www.js_. We will build it incrementally. Delete the content of `src/bin/www.js` and paste in the below code block.

```js
#!/usr/bin/env node
/**
 * Module dependencies.
 */
import debug from 'debug';
import http from 'http';
import app from '../app';
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
const port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

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
  if (error.syscall !== 'listen') {
    throw error;
  }
  const bind = typeof port === 'string' ? `Pipe ${port}` : `Port ${port}`;
  // handle specific listen errors with friendly messages
  switch (error.code) {
    case 'EACCES':
      alert(`${bind} requires elevated privileges`);
      process.exit(1);
      break;
    case 'EADDRINUSE':
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
  const bind = typeof addr === 'string' ? `pipe ${addr}` : `port ${addr.port}`;
  debug(`Listening on ${bind}`);
};
/**
 * Listen on provided port, on all network interfaces.
 */
server.listen(port);
server.on('error', onError);
server.on('listening', onListening);
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

# Settings And Environment Variables In Our _.Env_ File

In nearly every project, you’ll need somewhere to store settings that will be used throughout your app e.g. an AWS secret key. We store such settings as environment variables. This keeps them away from prying eyes, and we can use them within our application as needed.

I like having a _settings.js_ file with which I read all my environment variables. Then, I can refer to the settings file from anywhere within my app. You’re at liberty to name this file whatever you want, but there’s some kind of consensus about naming such files _settings.js_ or _config.js_.

For our environment variables, we’ll keep them in a `.env` file and read them into our `settings` file from there.

Create the .env file at the root of your project and enter the below line:

```js
TEST_ENV_VARIABLE = 'Environment variable is coming across';
```

To be able to read environment variables into our project, there’s a nice library, `dotenv` that reads our `.env` file and gives us access to the environment variables defined inside. Let’s install it.

```sh
# install dotenv
yarn add dotenv
```

Add the _.env_ file to the list of files being watched by `nodemon`.

Now, create the _settings.js_ file inside the `src/` folder and add the below code:

```js
import dotenv from 'dotenv';
dotenv.config();
export const testEnvironmentVariable = process.env.TEST_ENV_VARIABLE;
```

We import the `dotenv` package and call its config method. We then export the `testEnvironmentVariable` which we set in our `.env` file.

Open _src/routes/index.js_ and replace the code with the one below.

```js
import express from 'express';
import { testEnvironmentVariable } from '../settings';

const indexRouter = express.Router();

indexRouter.get('/', (req, res) =>
  res.status(200).json({ message: testEnvironmentVariable })
);

export default indexRouter;
```

The only change we’ve made here is that we import `testEnvironmentVariable` from our `settings` file and use is as the return message for a request from the `/` route.

Visit [http://localhost:3000/v1](http://localhost:3000/v1) and you should see the message, as shown below.

```json
{
  "message": "Environment variable is coming across."
}
```

Copy
And that’s it. From now on we can add as many environment variables as we want and we can export them from our _settings.js_ file.

This is a good point to commit your code. Remember to prettify and lint your code.

- _The corresponding branch on my repo is [03-env-variables](https://github.com/chidimo/Express-API-Template/tree/03-env-variables)._

# Writing Our First Test

It’s time to incorporate testing into our app. One of the things that give the developer confidence in their code is tests. I’m sure you’ve seen countless articles on the web preaching Test-Driven Development (TDD). It cannot be emphasized enough that your code needs some measure of testing. TDD is very easy to follow when you’re working with Express.js.

In our tests, we will make calls to our API endpoints and check to see if what is returned is what we expect.

Install the required dependencies:

```sh
# install dependencies
yarn add mocha chai nyc sinon-chai supertest coveralls --dev
```

Each of these libraries has its own role to play in our tests.

| Libraries  | Description                                  |
| ---------- | -------------------------------------------- |
| mocha      | test runner                                  |
| chai       | used to make assertions                      |
| nyc        | collect test coverage report                 |
| sinon-chai | extends chai’s assertions                    |
| supertest  | used to make HTTP calls to our API endpoints |
| coveralls  | for uploading test coverage to coveralls.io  |

Create a new `test/` folder at the root of your project. Create two files inside this folder:

- _test/setup.js_
- _test/index.test.js_

Mocha will find the `test/` folder automatically.

Open up _test/setup.js_ and paste the below code. This is just a helper file that helps us organize all the imports we need in our test files.

```js
import supertest from 'supertest';
import chai from 'chai';
import sinonChai from 'sinon-chai';
import app from '../src/app';

chai.use(sinonChai);
export const { expect } = chai;
export const server = supertest.agent(app);
export const BASE_URL = '/v1';
```

This is like a settings file, but for our tests. This way we don’t have to initialize everything inside each of our test files. So we import the necessary packages and export what we initialized — which we can then import in the files that need them.

Open up index.test.js and paste the following test code.

```js
import { expect, server, BASE_URL } from './setup';

describe('Index page test', () => {
  it('gets base url', (done) => {
    server
      .get(`${BASE_URL}/`)
      .expect(200)
      .end((err, res) => {
        expect(res.status).to.equal(200);
        expect(res.body.message).to.equal(
          'Environment variable is coming across.'
        );
        done();
      });
  });
});
```

Here we make a request to get the base endpoint, which is `/` and assert that the `res.body` object has a `message` key with a value of `Environment variable is coming across`.

If you’re not familiar with the `describe`, `it` pattern, I encourage you to take a quick look at Mocha’s “[Getting Started](https://mochajs.org/#getting-started)” doc.

Add the test command to the `scripts` section of package.json.

```json
"test": "nyc --reporter=html --reporter=text --reporter=lcov mocha -r @babel/register"
```

This script executes our test with `nyc` and generates three kinds of coverage report: an HTML report, outputted to the `coverage/` folder; a text report outputted to the terminal and an lcov report outputted to the `.nyc_output/` folder.

Now run `yarn test`. You should see a text report in your terminal just like the one in the below photo.

![Test coverage report](https://res.cloudinary.com/indysigner/image/fetch/f_auto,q_auto/w_1200/https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/c0a8035c-d7ad-4b6b-acb9-474495a5c84d/01-first-test-report.png)

Notice that two additional folders are generated:

- `.nyc_output/`
- `coverage/`

Look inside `.gitignore` and you’ll see that we’re already ignoring both. I encourage you to open up `coverage/index.html` in a browser and view the test report for each file.

This is a good point to commit your changes.

- _The corresponding branch in my repo is [04-first-test](https://github.com/chidimo/Express-API-Template/tree/04-first-test)._

# Continuous Integration(CD) And Badges: Travis, Coveralls, Code Climate, AppVeyor

It’s now time to configure continuous integration and deployment (CI/CD) tools. We will configure common services such as `travis-ci`, `coveralls`, `AppVeyor`, and `codeclimate` and add badges to our README file.

Let’s get started.

### TRAVIS CI

Travis CI is a tool that runs our tests automatically each time we push a commit to GitHub (and recently, Bitbucket) and each time we create a pull request. This is mostly useful when making pull requests by showing us if the our new code has broken any of our tests.

1. Visit [travis-ci.com](https://travis-ci.com/) or [travis-ci.org](https://travis-ci.org/) and create an account if you don’t have one. You have to sign up with your GitHub account.
2. Hover over the dropdown arrow next to your profile picture and click on settings.
3. Under `Repositories` tab click `Manage repositories on Github` to be redirected to Github.
4. On the GitHub page, scroll down to `Repository access` and click the checkbox next to `Only select repositories`.
5. Click the `Select repositories` dropdown and find the `express-api-template` repo. Click it to add it to the list of repositories you want to add to `travis-ci`.
6. Click `Approve and install` and wait to be redirected back to `travis-ci`.
7. At the top of the repo page, close to the repo name, click on the `build unknown` icon. From the Status Image modal, select markdown from the format dropdown.
8. Copy the resulting code and paste it in your _README.md_ file.
9. On the project page, click on `More options` > `Settings`. Under `Environment Variables` section, add the `TEST_ENV_VARIABLE` env variable. When entering its value, be sure to have it within double quotes like this `"Environment variable is coming across"`.
10. Create .travis.yml file at the root of your project and paste in the below code (We’ll set the value of `CC_TEST_REPORTER_ID` in the [Code Climate section](#code-climate)).

```sh
language: node_js
env:
  global:
    - CC_TEST_REPORTER_ID=get-this-from-code-climate-repo-page
matrix:
  include:
  - node_js: '12'
cache:
  directories: [node_modules]
install:
  yarn
after_success: yarn coverage
before_script:
  - curl -L https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64 > ./cc-test-reporter
  - chmod +x ./cc-test-reporter
  - ./cc-test-reporter before-build
script:
  - yarn test
after_script:
  - ./cc-test-reporter after-build --exit-code $TRAVIS_TEST_RESUL
```

First, we tell Travis to run our test with Node.js, then set the `CC_TEST_REPORTER_ID` global environment variable (we’ll get to this in the [Code Climate section](#code-climate)). In the `matrix` section, we tell Travis to run our tests with Node.js v12. We also want to cache the `node_modules/` directory so it doesn’t have to be regenerated every time.

We install our dependencies using the `yarn` command which is a shorthand for `yarn install`. The `before_script` and `after_script` commands are used to upload coverage results to `codeclimate`. We’ll configure codeclimate shortly. After `yarn test` runs successfully, we want to also run `yarn coverage` which will upload our coverage report to [coveralls.io](https://coveralls.io/).

### COVERALLS

Coveralls uploads test coverage data for easy visualization. We can view the test coverage on our local machine from the coverage folder, but Coveralls makes it available outside our local machine.

1. Visit [coveralls.io](https://coveralls.io/) and either sign in or sign up with your Github account.
2. Hover over the left-hand side of the screen to reveal the navigation menu. Click on `ADD REPOS`.
3. Search for the `express-api-template` repo and turn on coverage using the toggle button on the left-hand side. If you can’t find it, click on `SYNC REPOS` on the upper right-hand corner and try again. Note that your repo has to be public, unless you have a PRO account.
4. Click details to go to the repo details page.
5. Create the _.coveralls.yml_ file at the root of your project and enter the below code. To get the `repo_token`, click on the repo details. You will find it easily on that page. You could just do a browser search for `repo_token`.

```sh
repo_token: get-this-from-repo-settings-on-coveralls.io
```

This token maps your coverage data to a repo on Coveralls. Now, add the `coverage` command to the `scripts` section of your _package.json_ file:

```json
"coverage": "nyc report --reporter=text-lcov | coveralls"
```

This command uploads the coverage report in the `.nyc_output` folder to [coveralls.io](https://coveralls.io/). Turn on your Internet connection and run:

```sh
yarn coverage
```

This should upload the existing coverage report to coveralls. Refresh the repo page on coveralls to see the full report.

On the details page, scroll down to find the `BADGE YOUR REPO` section. Click on the `EMBED` dropdown and copy the markdown code and paste it into your _README_ file.

### Code Climate

Code Climate is a tool that helps us measure code quality. It shows us maintenance metrics by checking our code against some defined patterns. It detects things such as unnecessary repetition and deeply nested for loops. It also collects test coverage data just like coveralls.io.

1. Visit [codeclimate.com](https://codeclimate.com/login/github/join?return_to=%2Foss%2Fdashboard) and click on ‘Sign up with GitHub’. Log in if you already have an account.
2. Once in your dashboard, click on `Add a repository`.
3. Find the `express-api-template` repo from the list and click on `Add Repo`.
4. Wait for the build to complete and redirect to the repo dashboard.
5. Under `Codebase Summary`, click on `Test Coverage`. Under the `Test coverage` menu, copy the `TEST REPORTER ID` and paste it in your .travis.yml as the value of `CC_TEST_REPORTER_ID`.
6. Still on the same page, on the left-hand navigation, under `EXTRAS`, click on Badges. Copy the `maintainability` and `test coverage` badges in markdown format and paste them into your _README.md_ file.

It’s important to note that there are two ways of configuring maintainability checks. There are the default settings that are applied to every repo, but if you like, you could provide a _.codeclimate.yml_ file at the root of your project. I’ll be using the default settings, which you can find under the `Maintainability` tab of the repo settings page. I encourage you to take a look at least. If you still want to configure your own settings, this [guide](https://docs.codeclimate.com/docs/advanced-configuration) will give you all the information you need.

### APPVEYOR

AppVeyor and Travis CI are both automated test runners. The main difference is that travis-ci runs tests in a Linux environment while AppVeyor runs tests in a Windows environment. This section is included to show how to get started with AppVeyor.

- Visit [AppVeyor](https://ci.appveyor.com/) and log in or sign up.
- On the next page, click on `NEW PROJECT`.
- From the repo list, find the `express-api-template` repo. Hover over it and click `ADD`.
- Click on the `Settings` tab. Click on `Environment` on the left navigation. Add `TEST_ENV_VARIABLE` and its value. Click ‘Save’ at the bottom of the page.
- Create the _appveyor.yml_ file at the root of your project and paste in the below code.

```sh
environment:
  matrix:
  - nodejs_version: "12"
install:
  - yarn
test_script:
  - yarn test
build: off
```

This code instructs AppVeyor to run our tests using Node.js v12. We then install our project dependencies with the `yarn` command. `test_script` specifies the command to run our test. The last line tells AppVeyor not to create a build folder.

Click on the `Settings` tab. On the left-hand navigation, click on badges. Copy the markdown code and paste it in your _README.md_ file.

Commit your code and push to GitHub. If you have done everything as instructed all tests should pass and you should see your shiny new badges as shown below. Check again that you have set the environment variables on Travis and AppVeyor.

![Repo CI/CD badges](https://res.cloudinary.com/indysigner/image/fetch/f_auto,q_auto/w_1200/https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/4daf736a-f918-48a9-af01-04a785aedab7/02-express-api-backend-project-with-postgresql.png)

Now is a good time to commit our changes.

- _The corresponding branch in my repo is [05-ci](https://github.com/chidimo/Express-API-Template/tree/05-ci)._

# Adding A Controller

Currently, we’re handling the `GET` request to the root URL, `/v1`, inside the *src/routes/index.js*. This works as expected and there is nothing wrong with it. However, as your application grows, you want to keep things tidy. You want concerns to be separated — you want a clear separation between the code that handles the request and the code that generates the response that will be sent back to the client. To achieve this, we write `controllers`. Controllers are simply functions that handle requests coming through a particular URL.

To get started, create a `controllers/` folder inside the `src/` folder. Inside `controllers` create two files: *index.js* and *home.js*. We would export our functions from within *index.js*. You could name *home.js* anything you want, but typically you want to name controllers after what they control. For example, you might have a file *usersController.js* to hold every function related to users in your app.

Open *src/controllers/home.js* and enter the code below:

```js
import { testEnvironmentVariable } from '../settings';

export const indexPage = (req, res) =>
  res.status(200).json({ message: testEnvironmentVariable });
```

You will notice that we only moved the function that handles the request for the `/` route.

Open *src/controllers/index.js* and enter the below code.
```js
// export everything from home.js
export * from './home';
```
We export everything from the home.js file. This allows us shorten our `import statements to import { indexPage } from '../controllers';`

Open *src/routes/index.js* and replace the code there with the one below:
```js
import express from 'express';
import { indexPage } from '../controllers';
const indexRouter = express.Router();

indexRouter.get('/', indexPage);

export default indexRouter;
```
The only change here is that we’ve provided a function to handle the request to the `/` route.

You just successfully wrote your first controller. From here it’s a matter of adding more files and functions as needed.

Go ahead and play with the app by adding a few more routes and controllers. You could add a route and a controller for the about page. Remember to update your test, though.

Run `yarn test` to confirm that we’ve not broken anything. Does your test pass? That’s cool.

This is a good point to commit our changes.

*The corresponding branch in my repo is [06-controllers](https://github.com/chidimo/Express-API-Template/tree/06-controllers).*