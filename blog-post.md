## Preface
Webpack is incredibly useful. Below is a little post that explains a step by step process of setting up your own Angular 1.5 / Webpack environment. If you'd like to see how everything looks when you're finished checkout this lovely little repository. [https://github.com/rob-moore/meanTodo](https://github.com/rob-moore/meanTodo). I've seperated out the branches in a way that I hope is helpful for everyone. There are only two branches to pay attention to are:

###master
This is the completed project with everything working. It's a small to-do list app that I created. 

###boilerplate
This branch is just the bare minimum of what you need. This features a `webpack.config` file that is set up with a hot reloading dev-server.

Now, lets dive into using Webpack!

## Webpack
Webpack, in theory, does something that's very simple. It bundles together all of files that you throw at it in order to minify all of the fancy front-end tools that you're using into a package that is ready to deploy to the web. However, setting things up for webpack can be a stumbling block for people more familiar with older build tools such as Gulp or Grunt. We're going to be taking a look at starting a project from the ground up using Webpack as our build tool. By the end of this, we'll have a development environment for Angular 1.5, Bootstrap, jQuery and SCSS. 

One of my favorite things about Webpack is that it's kind of an all-in-one suite when it comes to do frontend development. Besides bundling everything into a nice little package, it has it's own dev-server that features the ability to hot-reload whenever a change is detected in the files that you're editing. 
## A brief rundown of the tools we'll be using
Here's a quick list of the tools that we'll be using for this project:

* npm
* webpack
* webpack dev server
* Angular 1.5
* Bootstrap
* jQuery



## Creating package.json
In order to use everything we're going to want to start by installing Webpack globally. We can do this by opening up a terminal and typing: `npm install -g webpack`.

This will give us access to webpack anywhere we want on our system. 

## Writing our webpack.config
After we've got Webpack installed and our npm package initialized we can finally dive into writing up our webpack.config file. Which is really just a large JSON object of guidelines for Webpack to use in our development environment. This will differ depending on what types of tools you're using for your project i.e. Angular, React, etc. Here's a link to the [`webpack.config`](https://github.com/rob-moore/meanTodo/blob/master/webpack.config.js) file when it's all finished. Now step through the code piece by piece so we can understand what everything does.

Also, please keep in mind that webpack has some absolutely wonderful documentation written out that should answer any questions that you might have about building your own config. You can find that [here](https://webpack.github.io/docs/)

    'use strict';

    const webpack = require('webpack');
    const path = require('path');

This first chunk requires everything we need to use webpack. 

`const webpack = require('webpack');`
Gives us access to webpack plugins. We'll use this later on for some plugins that will make working with webpack a little easier.

`const path = require('path');`
This is a node package that lets us use a file directory structure inside of our webpack configuration. 

    const config = {
    devtool: 'inline-source-map',
    entry: [
        'webpack-dev-server/client?http://127.0.0.1:8080',
        'webpack/hot/only-dev-server',
        'bootstrap-loader',
        './src',
    ],
Next we're going to declare our config object, what devtool we'll be using and the entry point for all of the stuff that we're going to bundle.

`const config = {`
Putting our config object into it's own variable will allow you to do cool things like having different configs for a development and production environment.

`devtool: 'inline-source-map',`
This is our devtool. This determines what developer tool we'll be using for debugging. This is all personal preference and there are several more options that are available than the one I'm using. You can find more about that in the wonderful [webpack documentation](https://webpack.github.io/docs/configuration.html#devtool). Keep in mind that depending on which devtool you use your project may take more or less time to build.

`entry: [`
This is our entry array that tells webpack where our files are.

`'webpack-dev-server/client?http://127.0.0.1:8080',`
This tells the webpack-dev-server what address to serve everything to. This is required for hot-reloading.

`'webpack/hot/only-dev-server',`
This will make the dev server hot reload whenever a file inside any of our entry points is modified.

`'bootstrap-loader',`
Gives us access to bootstrap for styling. 

`'./src',`
Finally, this is where our project lives.

    output: {
        path: path.join(__dirname, 'public'),
        filename: 'bundle.js',
    },
Here's our output. This tells webpack where to put our bundle (this is why we required `path` at the top of our config), and what to call our bundle. In our case it's `bundle.js`.

    resolve: {
        modulesDirectories: ['node_modules', 'src'],
        extension: ['', '.js'],
    },
This specifies how we specify what types of files webpack will process. That means in this case that we can leave off the `.js` when we're requiring modules throughout our project. This is just a quick time saver!

    module: {
        loaders: [
Finally, we're at the real meat of our webpack configuration. Our `module` object has an array of `loaders`. This is going to be where we tell webpack how to handle the different file types that it might encounter in our project. These loaders will always include some kind of regex test in order to determine the file type as well as some kind of loader which tells webpack what to do with these file types.

      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel',
        query: {
          presets: ['es2015'],
        },
      },
This is loader looks for any file within our entry point that that has a `.js` extension. I've also specified that it exclude our `node_modules` folder mainly because it's huge and would take forever to build.

We then specify that we want to run all of our `.js` files through the babel loader, using a preset of `es2015`. Which gives us access to all the fun features of es2015 including arrow functions, let, const and all that good stuff.

      {
        test: /\.html$/,
        loader: 'raw',
      },
Here's our HTML loader. We run it through the `raw` loader which will convert any HTML files that is required into a string. This lets webpack watch it for changes.

      {
        test: /\.scss$/,
        loaders: [
          'style',
          'css',
          'autoprefixer?browsers=last 3 versions',
          'sass?outputStyle=expanded',
        ],
      },
This is our CSS preprocessor. There are loaders for just about all of them so feel free to use whichever one you are comfortable with. In this case we'll be using SCSS. One thing to keep in mind is that the loaders are read from right to left, or in this case, bottom to top. So any file that ends will the the `.scss` extension will be run through our sass converter first, then our autoprefixer, then it will be translated into plain css and loaded up with our `style` loader.

      {
        test: /\.(woff2?|ttf|eot|svg)$/,
        loader: 'url?limit=10000',
      },
This handles all of our fonts and svgs. Basically it tests to see if they're under a certain size (in this case: 10000 bytes) and will then load them inline if they are small enough or require them from another source if they are too big.

      {
        test: /bootstrap-sass\/assets\/javascript?\//,
        loader: 'imports?jQuery=jquery',
      },
Last but not least we run our bootstrap library through our jQuery import. All this does is set `var $ = require("jquery");` in all of bootstrap-sass's files inside the `javascript` folder. This gives us access to bootstrap's jQuery plugins.

## Wrapping things up

## Links to repository and tools

