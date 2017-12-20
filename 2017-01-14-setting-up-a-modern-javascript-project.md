## Setting up a modern JavaScript project
Creating a front-end JavaScript project can be a daunting task due to the sheer volume of choices available while deciding the tech stack. First, you need to decide the JavaScript framework or library for your project. Do you plan to use the latest ES2015 language features in your code? Then you need a transpiler because your browser probably doesn’t support them yet. Then you require a bundling tool to get your code loaded in the browser. You may want to minify the code for faster load time. To automate all these steps, you need a build script. You may want to deploy your project on a local web server during development. Some setup is required for that. Finally, you need to include some testing framework in your project to write unit tests.

In this post I’ll demonstrate all of the above steps by incorporating all the necessary tools in a simple `React` project. I would use `Babel` to compile the project and `Browserify` to create the bundle. I would write a simple `Grunt` script to automate these steps. I’ll use `Express`, the web application framework provided by Nodejs for local deployment. For unit tests I’ll use the `Mocha` framework, backed by the assertion libraries provided by `Chai`. Because it is a React project, I’ll use `Enzyme` library to test the React components.

While this is not the most up to date way to create a React project, you are likely to encounter many existing projects that follow a similar build process.

The complete source code for the project set up described in the rest of this post is present in [this Github repo](https://github.com/sskelkar/react-setup-example).
This post is divided into four parts: setting up a simple React project, building the project, implementing unit tests and deploying the application on a local web server.

### 1. Setting up a simple React project
**i.** Create a new folder for the project. In command prompt, run the following command to create `package.json` and provide basic project information:
```
npm init
```
**ii.** Add React dependencies in your `package.json` using following command:
```
npm install react react-dom --save-dev
```
The `--save-dev` option downloads the specified libraries in the node_modules folder in your project root folder and adds their name and version numbers in `package.json` as `devDependencies`.

**iii.** Create a new `src` folder and add `index.html` file in it with the following content:
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <link href='http://fonts.googleapis.com/css?family=Roboto:400,300,700' rel='stylesheet' type='text/css'>
    <link href="css/styles.css" rel="stylesheet" type="text/css">
</head>
  <body>
    <div id="app"></div>
    <script src="bundle.js"></script>
  </body>
</html>
```

**iv**. Create a sub-folder `js` in `src` and create our sole React component `src/js/app.js` as following:
```javascript
import React from 'react';

class App extends React.Component {
  render() {
    return (
      <div><h2>Helloooo World!!</h2></div>
    );
  }
}

export default App;
```
Here, we have created a simple React component to display a message on the web page. But we need a way to hook this component into our base `index.html`.

**v.** Create `index.js` in the `src` folder that would simply render the app component inside a div tag in `index.html`.
```js
import ReactDOM from 'react-dom';
import React from 'react';
import App from './js/app';

ReactDOM.render(<App/>, document.getElementById("app"));
```
At this point your folder structure would look like this:
```
root
|____ node_modules
|____ package.json
|____ src
      |____ index.html
      |____ index.js
      |____ js
            |____ app.js
```
### 2. Building the project
In this part we would write a `Grunt` script to automate the task of compiling the project and creating a bundle. The bundle would be saved in `build` folder at the root level. We would use grunt plugins that would provide us some inbuilt tasks, which we can configure as per our requirement. Then we would write a custom grunt task to execute the plugin tasks in correct sequence.

**i.** Add grunt to you project with the following command:
```
npm install grunt --save-dev
```

**ii.** During our build process, our first step would be to delete the `build` folder. We need to add `grunt-contrib-clean` plugin to our project that would provide us with the `clean` task.
```
npm install grunt-contrib-clean --save-dev
```

**iii.** We have the `grunt clean` task now. But we need to tell it which folders to delete. We need to load the `grunt-contrib-clean` task in the script using `grunt.loadNpmTasks` method and configure this task to delete the `build` folder using `grunt.initConfig` method. So now let's create `Gruntfile.js` in the root folder.
```js
var grunt = require('grunt');

grunt.loadNpmTasks('grunt-contrib-clean');

grunt.initConfig({
  clean: ['build']
});
```

**iv.** Next step is to copy our `index.html` as it is into the `build` folder. We do this so that later when we want to locally deploy the project, we can serve the base html file directly from the `build` folder along with the bundle. We need to add `grunt-contrib-copy` plugin to our project and configure the source and destination locations of the file to be copied.
```
npm install grunt-contrib-copy --save-dev
```

The `Gruntfile.js` would look like following:
```js
var grunt = require('grunt');

grunt.loadNpmTasks('grunt-contrib-clean');
grunt.loadNpmTasks('grunt-contrib-copy');

grunt.initConfig({
  clean: ['build'],
  copy: {
    html: {
      options: {
        flatten: true
      },
      files: {
        'build/index.html': 'src/index.html'
      }
    }
  }
});
```

**v.** Now on to the compilation step. We would use `Babel` to both compile React code into plain JavaScript, as well as, ES2015 language features into older JS syntax. Babel provides us with presets for these transformations. A Babel preset is a collection of plugins to perform a particular task. We can include the Babel presets relevant to us as following:
```
npm install babel-preset-react babel-preset-es2015 --save-dev
```
**vi.** Installing the Babel presets in our project isn’t enough. We need to enable them by adding `.babelrc` file as following in the root folder.
```js
{
  "presets": ["react", "es2015"],
}
```
**vii.** We are going to use `Browserify` as our bundling tool. It would first execute the transformation step and then it would create the `bundle.js`. Since we are using Babel in the first step, we need to get it working with Browserify. So we would add `Babelify`, a Babel transformer specifically created for Browserify, to our project.
```
npm install babelify --save-dev
```
We would also add the `grunt-browserify` plugin.
```
npm install grunt-browserify --save-dev
```
**viii.** Now we need to configure the `browserify` task in our grunt script. We would specify `babelify` with appropriate presets as the transformer. We also need to specify the starting point of our React project. That would be `index.js`, where we hook up our React component into `index.html`. We also need to give the name and desired location of the bundle file.
```js
var grunt = require('grunt');

grunt.loadNpmTasks('grunt-contrib-clean');
grunt.loadNpmTasks('grunt-contrib-copy');
grunt.loadNpmTasks('grunt-browserify');

grunt.initConfig({
  clean: ['build'],
  copy: {
    html: {
      options: {
        flatten: true
      },
      files: {
        'build/index.html': 'src/index.html'
      }
    }
  },
  browserify: {
    dist: {
      files: {
        'build/bundle.js': ['src/index.js']
      },
      options: {
        transform: [[ 'babelify', { 'presets': [ 'es2015', 'react' ] }  ]]
      }
    }
  }
});
```
**ix.** Finally, we need to create a custom build task in the grunt file that should execute clean, copy and browserify tasks, in that order. Our final `Gruntfile.js` would look like this:
```js
var grunt = require('grunt');

grunt.loadNpmTasks('grunt-contrib-clean');
grunt.loadNpmTasks('grunt-contrib-copy');
grunt.loadNpmTasks('grunt-browserify');

grunt.initConfig({
  clean: ['build'],
  copy: {
    html: {
      options: {
        flatten: true
      },
      files: {
        'build/index.html': 'src/index.html'
      }
    }
  },
  browserify: {
    dist: {
      files: {
        'build/bundle.js': ['src/index.js']
      },
      options: {
        transform: [[ 'babelify', { 'presets': [ 'es2015', 'react' ] }  ]]
      }
    }
  }
});

grunt.registerTask('build', ['clean', 'copy:html', 'browserify:dist']);
```
At this point our project structure would look like this:
```
root
|____ .babelrc
|____ build
|____ Gruntfile.js
|____ node_modules
|____ package.json
|____ src
      |____ index.html
      |____ index.js
      |____ js
            |____ app.js
```
### 3. Implementing unit tests
**i.** We would use `Mocha` as our primary test runner. For an expressive assertion library we would use `Chai`. We also need to add two other testing libraries – `Enzyme` and `react-addons-test-utils`. These are React specific. To run tests in a simulated browser environment, we need `jsdom`.
```
npm install --save-dev mocha chai enzyme jsdom react-addons-test-utils
```
**ii.** We would create a new `test` folder in our root folder. To test our components in a realistic browser environment, we need to create a setup file `browser.js` in our test folder as following:
```js
require('babel-register')();

var jsdom = require('jsdom').jsdom;

var exposedProperties = ['window', 'navigator', 'document'];

global.document = jsdom('');
global.window = document.defaultView;
Object.keys(document.defaultView).forEach((property) => {
  if (typeof global[property] === 'undefined') {
    exposedProperties.push(property);
    global[property] = document.defaultView[property];
  }
});

global.navigator = {
  userAgent: 'node.js'
};

documentRef = document;
```
**iii.** To have our `test` folder structure mirror the `src` folder, we would create `test/js` sub folder. Here, we can create an `app.spec.js` file that would contain unit tests specific to the `App` component. We can write a test case to assert the message being displayed by this component as following:
```js
import React from 'react';
import { expect } from 'chai';
import { mount, shallow } from 'enzyme';
import App from '../../src/js/app';

describe('<App />', () => {
  it('should render the hello world message', () => {
    const wrapper = shallow(<App/>);
    expect(wrapper.find('h2').text()).to.equal('Helloooo World!!');
  });
});
```
**iv.** Finally, we need to add the following `scripts` block in `package.json`. Now unit tests throughout the project can be run with `npm test` command. Alternatively, a task to run the unit tests can also be created in the grunt script.
```
"scripts": {
    "test": "mocha test/browser.js test/**/*.spec.js"
}
```
### 4. Deploying the application on a local web server
**i.** We will use `Express`, a web application framework provided by Node.js.
```
npm install express --save
```
**ii.** Then we need to create a `server.js` file in the root folder. In this file, we specify the location of the html that is to be served on the browser and the port number on which the server should run.
```js
var express = require('express');

var app = express();

app.use("/", express.static("build"));

app.get('*', function(req, res) {
    res.sendFile('index.html',  { root: "build" });
});

app.listen("3000", function() {
  console.log('Starting server..');
});
```
**iii.** Use `npm start` to start the server and open `http://localhost:3000` on the browser.

The final project structure will look like this:
```
root
|____ .babelrc
|____ build
|____ Gruntfile.js
|____ node_modules
|____ package.json
|____ server.js
|____ src
|     |____ index.html
|     |____ index.js
|     |____ js
|           |____ app.js
|____ test
      |____ browser.js
      |____ js
            |____ app.spec.js
```

