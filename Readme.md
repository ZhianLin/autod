
# autod

[![NPM version][npm-image]][npm-url]
[![David deps][david-image]][david-url]
[![node version][node-image]][node-url]
[![Gittip][gittip-image]][gittip-url]

[npm-image]: https://img.shields.io/npm/v/autod.svg?style=flat-square
[npm-url]: https://npmjs.org/package/autod
[david-image]: https://img.shields.io/david/node-modules/autod.svg?style=flat-square
[david-url]: https://david-dm.org/node-modules/autod
[node-image]: https://img.shields.io/badge/node.js-%3E=_0.10-green.svg?style=flat-square
[node-url]: http://nodejs.org/download/
[gittip-image]: https://img.shields.io/gittip/dead-horse.svg?style=flat-square
[gittip-url]: https://www.gittip.com/dead-horse/

Auto generate dependencies and devDependencies by parse the project file.
`autod` will parse all the js files in `path`, and get the latest dependencies version from [registry.npmjs.org](https://registry.npmjs.org) or other registries by `-r`.
一个自动分析项目所有的文件，获取所有的项目依赖和它们的版本的工具。通过这个工具，我们可以很轻松的跟踪到所有依赖的最新版本，同时可以自动更新我们的 package.json 文件，新引入的模块也不需要手动去更新 package.json 文件了。
## install（必须全局安装）

```bash
$ npm install -g autod
```

## usage（使用方式）

```bash
$ bin/autod -h

 Usage: autod [options]

  Options:

    -h, --help                                           output usage information
    -V, --version                                        output the version number
    -p, --path [root path]                               the root path to be parse.
    -t, --test <test/benchmark/example directory paths>  modules in these paths will be tread as devDependencies
    -e, --exclude <exclude directory path>               exclude parse directory, split by `,`.排除不需要分析的目录。
    -r, --registry <remote registry>                     get latest version from which registry.设置模块的resource。
    -f, --prefix [version prefix]                        version prefix, can be `~` or `^`
    -F, --devprefix [dev dependencies version prefix]    develop dependencies version prefix, can be `~` or `^`
    -w, --write                                          write dependencies into package.json
    -i, --ignore                                         ignore errors, display the dependencies or write the dependencies.忽略错误。
    -m, --map                                            display all the dependencies require by which file.按文件列出依赖的包，当依赖结果的map大于500Kb时该命令会报错。
    -d, --dep <dependently modules>                      modules that not require in source file, but you need them as dependencies.把不是依赖的包手动加入到依赖列表中。
    -D, --devdep <dev dependently modules>               modules that not require in source file, but you need them in as devDependencies
    -k, --keep <dependently modules>                     modules that you want to keep version in package.json file.指定某个模块不更新版本号。
    -s, --semver <dependencies@version>                  auto update these modules within the specified semver
```

* Autod will parse all the js files in `path`, and you can exclude folder by `-e, --exclude`.
* Autod 会分析所有的 js 文件 in `path`, 你可以通过 `-e, --exclude`来把不想分析的目录排除。如assets、views、model。
* All the modules in test folder (can be alter by `-t, --text`) will parsed as devDependencies.
* `-t, --text` 指明的模块，将会作为开发环境依赖来分析。
* If you set `-w, --write`, `autod` will write the dependencies into package.json file. `dependencies` will replace `dependencies` in package.json, and `devDependencies` will merge with `devDependencies` in package.json, then write into package file.
* 当使用 `-w, --write`命令`autod` 会把生产环境依赖写入到`package.json file`。 分析出来的`dependencies` 会替换package.json原有的 `dependencies` , 同时把 `devDependencies` 合并到 package.json。
* `-f, --prefix` will add prefix to each dependencies' version.
* `-F, --devprefix` will add prefix to each dev dependencies' version.
* `-i, --ignore` will display or wrtie the dependencies even some error happened.
* `-d --dep` will add modules to dependencies even not require by any file.
* `-D --devdep` will add modules to devDependencies even not require by any file.
* `-k --keep` will keep the modules' version in package.json not change by autod.
* `-s, --semver` will update these modules within the specified semver.
* `-n, --notransform` disable transfrom es next, don't support es6 modules.

a simple example of autod:

```
autod -w --prefix="~" -d connect -D mocha,should -k express -s connect@2
```

## Maintains your dependencies in Makefile（集成到项目的Makefile中）

add a command in your Makefile  
在Makefile文件中添加以下的命令

```sh
autod:
    @./node_modules/.bin/autod -w
    @npm install

```

then run `make autod`, it will find all the dependencies and devDependencies in your project,
add / remove dependencies, and bump the versions.
然后执行命令 `make autod`,`dependencies `和 `devDependencies`会构建到项目中。

check out some examples:
可以查看下面这两个示例中的使用效果：

 - [cnpmjs.org](https://github.com/cnpm/cnpmjs.org/blob/master/Makefile#L95)
 - [koa-generic-session](https://github.com/koajs/generic-session/blob/master/Makefile#L40)

## es6 modules support（支持ES6模块）

All files will compiled by `babel` with `babel-preset-react`, `babel-preset-es2015`, `babel-preset-stage-0`.

## Plugin support（插件引入）

You can write a plugin for autod to decide how to parse dependencies.

- Write a plugin, and publish it to NPM.

```js
module.exports = function (filepath, content) {
  // find the dependencies
  return []; // return dependencies
};
```

- Use plugin:

install plugin from NPM, then use it with `autod`:

```
autod -P pluginName
```

## Config support

You can put all the options in `${cwd}/.autod.conf.js`, when you run `autod`, it will load this file as input options. It can be both a `js` file or a `json` file.

```js
module.exports = {
  write: true,
  prefix: '~',
  devprefix: '^',
  dep: [
    'bluebird'
  ],
  semver: [
    'koa-router@4',
    'koa-compose@2'
  ],
};
```

## License（版权声明）

(The MIT License)

Copyright (c) 2013~2015 dead_horse <dead_horse@qq.com>;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
