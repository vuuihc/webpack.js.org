---
title: sass-loader
source: https://raw.githubusercontent.com/webpack-contrib/sass-loader/master/README.md
edit: https://github.com/webpack-contrib/sass-loader/edit/master/README.md
---
## 安装

```bash
npm install sass-loader node-sass webpack --save-dev
```
sass-loader的[`peerDependency`](https://docs.npmjs.com/files/package.json#peerdependencies)有[node-sass](https://github.com/sass/node-sass) 和 [webpack](https://github.comwebpack)，因此能够精确控制它们的版本。

## 例子
将sass-loader与[css-loader](https://github.com/webpack-contrib/css-loader)和[style-loader](https://github.com/webpack-contrib/style-loader)连接起来使用，可以将所有的样式应用于DOM元素。

```js
// webpack.config.js
module.exports = {
	...
    module: {
        rules: [{
            test: /\.scss$/,
            use: [{
                loader: "style-loader" // 从JS字符串生成样式节点
            }, {
                loader: "css-loader" // 将CSS转化成CommonJS
            }, {
                loader: "sass-loader" // 将Sass编译成CSS
            }]
        }]
    }
};
```
也可以通过指定`options`参数，向[node-sass](https://github.com/andrew/node-sass)传递选项参数。比如：

```js
// webpack.config.js
module.exports = {
	...
    module: {
        rules: [{
            test: /\.scss$/,
            use: [{
                loader: "style-loader"
            }, {
                loader: "css-loader"
            }, {
                loader: "sass-loader",
                options: {
                    includePaths: ["absolute/path/a", "absolute/path/b"]
                }
            }]
        }]
    }
};
```

Sass的更多参数见[node-sass](https://github.com/andrew/node-sass)。

### 生产环境
通常，生产环境下比较推荐的做法是使用[ExtractTextPlugin](https://github.com/webpack-contrib/extract-text-webpack-plugin)，将样式表抽离成独立文件。这样，样式表将不依赖于JS：

```js
const ExtractTextPlugin = require("extract-text-webpack-plugin");

const extractSass = new ExtractTextPlugin({
    filename: "[name].[contenthash].css",
    disable: process.env.NODE_ENV === "development"
});

module.exports = {
	...
    module: {
        rules: [{
            test: /\.scss$/,
            use: extractSass.extract({
                use: [{
                    loader: "css-loader"
                }, {
                    loader: "sass-loader"
                }],
                // use style-loader in development
                fallback: "style-loader"
            })
        }]
    },
    plugins: [
        extractSass
    ]
};
```

## 使用

### Imports

webpack provides an [advanced mechanism to resolve files](http://webpack.github.io/docs/resolving.html). The sass-loader uses node-sass' custom importer feature to pass all queries to the webpack resolving engine. Thus you can import your Sass modules from `node_modules`. Just prepend them with a `~` to tell webpack that this is not a relative import:

```css
@import "~bootstrap/css/bootstrap";
```

It's important to only prepend it with `~`, because `~/` resolves to the home directory. webpack needs to distinguish between `bootstrap` and `~bootstrap` because CSS and Sass files have no special syntax for importing relative files. Writing `@import "file"` is the same as `@import "./file";`

### Problems with `url(...)`

Since Sass/[libsass](https://github.com/sass/libsass) does not provide [url rewriting](https://github.com/sass/libsass/issues/532), all linked assets must be relative to the output.

- If you're just generating CSS without passing it to the css-loader, it must be relative to your web root.
- If you pass the generated CSS on to the css-loader, all urls must be relative to the entry-file (e.g. `main.scss`).

More likely you will be disrupted by this second issue. It is natural to expect relative references to be resolved against the `.scss` file in which they are specified (like in regular `.css` files). Thankfully there are a two solutions to this problem:

- Add the missing url rewriting using the [resolve-url-loader](https://github.com/bholloway/resolve-url-loader). Place it directly after the sass-loader in the loader chain.
- Library authors usually provide a variable to modify the asset path. [bootstrap-sass](https://github.com/twbs/bootstrap-sass) for example has an `$icon-font-path`. Check out [this working bootstrap example](https://github.com/webpack-contrib/sass-loader/tree/master/test/bootstrapSass).

### Extracting style sheets

Bundling CSS with webpack has some nice advantages like referencing images and fonts with hashed urls or [hot module replacement](https://webpack.js.org/concepts/hot-module-replacement/) in development. In production, on the other hand, it's not a good idea to apply your style sheets depending on JS execution. Rendering may be delayed or even a [FOUC](https://en.wikipedia.org/wiki/Flash_of_unstyled_content) might be visible. Thus it's often still better to have them as separate files in your final production build.

There are two possibilities to extract a style sheet from the bundle:

- [extract-loader](https://github.com/peerigon/extract-loader) (simpler, but specialized on the css-loader's output)
- [extract-text-webpack-plugin](https://github.com/webpack-contrib/extract-text-webpack-plugin) (more complex, but works in all use-cases)

### Source maps

To enable CSS source maps, you'll need to pass the `sourceMap` option to the sass-loader *and* the css-loader. Your `webpack.config.js` should look like this:

```javascript
module.exports = {
    ...
    devtool: "source-map", // any "source-map"-like devtool is possible
    module: {
        rules: [{
            test: /\.scss$/,
            use: [{
                loader: "style-loader"
            }, {
                loader: "css-loader", options: {
                    sourceMap: true
                }
            }, {
                loader: "sass-loader", options: {
                    sourceMap: true
                }
            }]
        }]
    }
};
```

If you want to edit the original Sass files inside Chrome, [there's a good blog post](https://medium.com/@toolmantim/getting-started-with-css-sourcemaps-and-in-browser-sass-editing-b4daab987fb0). Checkout [test/sourceMap](https://github.com/webpack-contrib/sass-loader/tree/master/test) for a running example.

### Environment variables

If you want to prepend Sass code before the actual entry file, you can set the `data` option. In this case, the sass-loader will not override the `data` option but just append the entry's content. This is especially useful when some of your Sass variables depend on the environment:

```javascript
{
    loader: "sass-loader",
    options: {
        data: "$env: " + process.env.NODE_ENV + ";"
    }
}
```

**Please note:** Since you're injecting code, this will break the source mappings in your entry file. Often there's a simpler solution than this, like multiple Sass entry files.

## Maintainers

<table>
    <tr>
      <td align="center">
        <a href="https://github.com/jhnns"><img width="150" height="150" src="https://avatars0.githubusercontent.com/u/781746?v=3"></a><br>
        <a href="https://github.com/jhnns">Johannes Ewald</a>
      </td>
      <td align="center">
        <a href="https://github.com/webpack-contrib"><img width="150" height="150" src="https://avatars1.githubusercontent.com/u/1243901?v=3&s=460"></a><br>
        <a href="https://github.com/webpack-contrib">Jorik Tangelder</a>
      </td>
      <td align="center">
        <a href="https://github.com/akiran"><img width="150" height="150" src="https://avatars1.githubusercontent.com/u/3403295?v=3"></a><br>
        <a href="https://github.com/akiran">Kiran</a>
      </td>
    <tr>
</table>


## License

[MIT](http://www.opensource.org/licenses/mit-license.php)

[npm]: https://img.shields.io/npm/v/sass-loader.svg
[npm-stats]: https://img.shields.io/npm/dm/sass-loader.svg
[npm-url]: https://npmjs.com/package/sass-loader

[node]: https://img.shields.io/node/v/sass-loader.svg
[node-url]: https://nodejs.org

[deps]: https://david-dm.org/webpack-contrib/sass-loader.svg
[deps-url]: https://david-dm.org/webpack-contrib/sass-loader

[travis]: http://img.shields.io/travis/webpack-contrib/sass-loader.svg
[travis-url]: https://travis-ci.org/webpack-contrib/sass-loader

[appveyor-url]: https://ci.appveyor.com/project/jhnns/sass-loader/branch/master
[appveyor]: https://ci.appveyor.com/api/projects/status/github/webpack-contrib/sass-loader?svg=true

[cover]: https://coveralls.io/repos/github/webpack-contrib/sass-loader/badge.svg
[cover-url]: https://coveralls.io/github/webpack-contrib/sass-loader

[chat]: https://badges.gitter.im/webpack-contrib/webpack.svg
[chat-url]: https://gitter.im/webpack-contrib/webpack

> 原文：https://webpack.js.org/***/
