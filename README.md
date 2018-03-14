# Configduino

This is a sample project that can act as a skeleton for a [configduino](https://github.com/madpilot/configduino) web app.

Cloning it as using it as a starting point is a valid strategy, but if you want to understand what is actually needed, see below:

# Setup

## Packages

First things first, install preact as a runtime dependency

```
yarn add preact
```

And configduino as a dev dependency

```
yarn add --dev configduino

```

For this sample project, I'll use webpack as a packager.

```
yarn add --dev webpack@3
```

We'll need babel to deal with ES6 and the transformation of the preact JSX

```
yarn add --dev babel-cli
yarn add --dev babel-preset-env
yarn add --dev babel-plugin-transform-react-jsx
```

So webpack knows how load different files, we add babel-loader (so ```import``` works), style-loader and css-loader gives use CSS module support.

```
yarn add --dev babel-loader
yarn add --dev style-loader
yarn add --dev css-loader
```

The Extract Text Webpack plugin allows use to output a real CSS file

```
yarn add --dev extract-text-webpack-plugin
```

The HTML plugin will generate a HTML file that includes references to the generated CSS and JS files

```
yarn add --dev html-webpack-plugin
```

Finally, we user HTML Webpack Inline Source to mash the three files together into a single HTML file that we can deliver via the Arduino.

```
yarn add --dev html-webpack-inline-source-plugin
```

## Webpack file

Import webpack, and the plugins we need

```
import webpack from 'webpack';
import ExtractTextPlugin from 'extract-text-webpack-plugin';
import HtmlWebpackPlugin from 'html-webpack-plugin';
import HtmlWebpackInlineSourcePlugin from 'html-webpack-inline-source-plugin';
import path from 'path';
```

```
const ENV = process.env.NODE_ENV || 'development';
module.exports = {
  context: path.resolve(__dirname, "src"),
  entry: './index.js',
```

Set the JavaScript output file to bundle.js in the build directory

```
  output: {
    path: path.resolve(__dirname, "build"),
    publicPath: '/',
    filename: 'bundle.js'
  },
```

Tell webpack where to find include files

```
  resolve: {
    extensions: ['.jsx', '.js' ],
    modules: [
      path.resolve(__dirname, "src/lib"),
      path.resolve(__dirname, "node_modules"),
      'node_modules'
    ]
  },
```

Tell Webpack to use babel-loader for .js and .jsx files, and css-loader for .css files

css-loader is configured to user css modules, and to use short class names in production.

```
  module: {
    loaders: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      },
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
          fallback: "style-loader",
          use: "css-loader?modules&localIdentName=" + (ENV == 'production' ? "[hash:base64:4]" : "[name]__[local]___[hash:base64:5]")
        })
      },
    ]
  },
```

Tell Extract Text Plugin to output all compiled CSS to style.css, and tell the HTML plugin to compile index.ejs

Finally, inline the CSS and JS into the HTML file.

```
	plugins: ([
		new webpack.NoEmitOnErrorsPlugin(),
		new webpack.DefinePlugin({
			'process.env.NODE_ENV': JSON.stringify(ENV)
		}),
    new ExtractTextPlugin({ filename: 'style.css', allChunks: true, disable: ENV !== 'production' }),
    new HtmlWebpackPlugin({
			template: './index.ejs',
			minify: { collapseWhitespace: true },
      inlineSource: '(.js|.css)$'
		}),
    new HtmlWebpackInlineSourcePlugin()
  ]),
```

```
	stats: { colors: true },

	node: {
		global: true,
		process: false,
		Buffer: false,
		__filename: false,
		__dirname: false,
		setImmediate: false
	}
};
```

## index.ejs

This file will be compiled into a HTML file. It gets the file names generated by the HTML plugin and injects them as script and link tags. You probably won't need to change this file too much.

```html
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<% if(process.env.NODE_ENV != 'production') { %>
<% for (var chunk in htmlWebpackPlugin.files.css) { %>
  <link rel="preload" href="<%= htmlWebpackPlugin.files.css[chunk] %>"  as="style">
<% } %>
<% for (var chunk in htmlWebpackPlugin.files.chunks) { %>
  <link rel="preload" href="<%= htmlWebpackPlugin.files.chunks[chunk].entry %>" as="script">
<% } %>
<% } %>
</head>
<body></body>
</html>
```

## index.js

This is the entry point for preact. It simply attaches to the comopnent generated by app.js to the DOM.

Attaching to document.body is not usually a good idea, but in this case there is no chance of external third party scripts messing with the DOM.

```javascript
import { h, render } from 'preact';

let root;
function init() {
	let App = require('./components/app').default;
	root = render(<App />, document.body, root);
}

init();
```

## components/app.js

This is were the interesting part of your App goes. Export a root Preact component, and the you are good to go.

# Building

To build, run ```yarn build``` which will out put all the HTML, CSS and JS. The index.html file will also have all the CSS and JS mashed in to it, so it is completely standalone - all you need to do is open it in a browser.

You can see what it looks like [here](https://htmlpreview.github.io/?https://github.com/madpilot/configduino-sample/blob/master/demo/index.html).
