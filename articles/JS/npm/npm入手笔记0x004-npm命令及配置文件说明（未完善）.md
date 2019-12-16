### 0x000 概述
本篇文章承接[上文][1]，记录的`package.json`的配置和`npm`命令的详细说明。

### 0x001 `package.json`的配置
- `name`：
    - 说明：项目名称，`npm install`的时候就是使用这个`name`。
    - 案例：`lodash`、`@followwinter/lodash`
- `version`：
    - 说明：版本号，符合`npm`的版本规范的版本号，默认从`1.0.0`开始。
    - 案例：`1.0.0`，`2.0.1`
- `description `：
    - 说明：项目的简介，如果不写会默认读去`README.md`的第一样作为`npmjs`搜索时候的简介
    - 案例：这是一个好项目
- `keywords`：
    - 说明：关键词
    - 案例：`lodash`、`js`
- `homepage`：
    - 说明：项目主页
    - 案例：`http://lodashjs.com/`
- `license`：
    - 说明：协议
    - 案例：`BSD-3-Clause`
- `main`：
    - 说明：模块ID
    - 案例：如果你的模块名为`foo`,如果一个用户使用`require('foo')`，就会返回一个你`export`出来的主对象。例如之前我们`export`一个`printMsg`,我们直接`@followwinter/0x007-local-global-diff1`就得到了一个对象，是因为我们指定了`@followwinter/0x007-local-global-diff1`中的`package.json`的`mian`为`index。js`。
    
    ```
    //  @followwinter/0x007-local-global-diff1/package.json
    {
      "name": "@followwinter/0x007-local-global-diff1",
      "version": "1.0.2",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC"
    }
    //  @followwinter/0x008-local-global-diff2/index.js
    var myModule = require('@followwinter/0x007-local-global-diff1');
    console.log(myModule);
    myModule.printMsg();
    ```
- `dependencies`：
    - 说明：项目的依赖库
        - `version`： 必须等于该版本
        - `>version`：必须大于该版本
        - `>=version`： 必须大于等于该版本
        - `<version`： 必须小于该版本
        - `<=version`： 必须小于等于该版本
        - `~version`： 大约等于该版本
        - `^version`： 和该版本可兼容的版本
        - `1.2.x`：`1.2.*` 版本
        - `http://...`：`http`地址
        - `*`：任意版本
        - `""`：任意版本
        - `version1 - version2`： 在`version1`到`version2`之间，包含`version1`和`version2`
        - `range1 || range2`：在范围1或者范围2之间
        - `git...`：git地址
        - `user/repo`：`user/repo`地址
        - `tag`： 该`tag`的版本
        - `path/path/path` 本地地址
    - 案例：
    ```
    { "dependencies" :
      { "foo" : "1.0.0 - 2.9999.9999"
      , "bar" : ">=1.0.2 <2.1.2"
      , "baz" : ">1.0.2 <=2.3.4"
      , "boo" : "2.0.1"
      , "qux" : "<1.0.0 || >=2.3.1 <2.4.5 || >=2.5.2 <3.0.0"
      , "asd" : "http://asdf.com/asdf.tar.gz"
      , "til" : "~1.2"
      , "elf" : "~1.2.3"
      , "two" : "2.x"
      , "thr" : "3.3.x"
      , "lat" : "latest"
      , "dyl" : "file:../dyl"
      }
    }
    ```
    - 特殊说明
        - git地址格式：`<protocol>://[<user>[:<password>]@]<hostname>[:<port>][:][/]<path>[#<commit-ish> | #semver:<semver>]`
        - git地址示例：
        ```
        git+ssh://git@github.com:npm/npm.git#v1.0.27
        git+ssh://git@github.com:npm/npm#semver:^5.0
        git+https://isaacs@github.com/npm/npm.git
        git://github.com/npm/npm.git#v1.0.27
        //现在还可以直接这么写
        {
          "name": "foo",
          "version": "0.0.0",
          "dependencies": {
            "express": "expressjs/express",
            "mocha": "mochajs/mocha#4727d357ea",
            "module": "user/repo#feature\/branch"
          }
        } 
        ```
        - 本地地址
        ```
        ../foo/bar
        ~/foo/bar
        ./foo/bar
        /foo/bar
        // 以下写法更优
        ```
        {
          "name": "baz",
          "dependencies": {
            "bar": "file:../foo/bar"
          }
        }
        ```
        ```
- `devDependencies`：
    - 说明：开发依赖
    - 案例：同上
- `bin`:
    - 说明：可执行目录
    - 案例：
    ```
    //`npm installl -g <package_name>`的时候回将`cli.js`复制到`/usr/local/bin/myapp`,就可以使用`myapp`作为命令了，比如`webpack`
    { "bin" : { "myapp" : "./cli.js" } }
    ```
- `scripts`:
    - 说明：自定义命令，也可以覆盖自定义命令
    - 默认值：
        - `"start": "node server.js"`：执行`server.js`
        - `"install": "node-gyp rebuild"`：安装依赖
    - 自定义指令：
    ```
    "test":"jtest",
    "build":"npm install && npm test && npm publish --access public"
    ```
    - 执行自定义指令：
    如果是覆盖默认指令，直接使用默认指令便可，比如`npm install`、`npm start`，如果是自定义指令，则需要使用`npm run <script_name>`来调用，比如`npm run build`，会先执行`npm install`，再执行`npm test`，最后执行`npm publish --access public`.
   
### 0x002 `npm`命令
- `version`：版本
    ```
    npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease | from-git]
    
    'npm [-v | --version]' 查看`npm`版本
    'npm view <pkg> version' 查看某个包的版本
    'npm ls' 列出当前依赖的包
    ```
- `cache`：缓存
    ```
    'npm cache clean [<path>]' 清除缓存
    aliases: npm cache clear, npm cache rm
    ```
    
- `dedupe`：重构
    ```
    'npm dedupe' 重新整理依赖包架构
    'npm ddp'
    
    aliases: find-dupes, ddp
    
    // 执行前
    
    +-- b <-- depends on c@1.0.x
    |   `-- c@1.0.3
    `-- d <-- depends on c@~1.0.9
        `-- c@1.0.10
        
    //执行后
    
    +-- b
    +-- d
    `-- c@1.0.10
    ```
- `init`：初始化
    ```
    npm init [-f|--force|-y|--yes] 加了参数就不会提示任何信息了
    ```
- `install`：安装
    ```
    npm install (with no args, in package dir)
    npm install [<@scope>/]<name>
    npm install [<@scope>/]<name>@<tag>
    npm install [<@scope>/]<name>@<version>
    npm install [<@scope>/]<name>@<version range>
    npm install <git-host>:<git-user>/<repo-name>
    npm install <git repo url>
    npm install <tarball file>
    npm install <tarball url>
    npm install <folder>
    
    alias: npm i
    common options: [-P|--save-prod|-D|--save-dev|-O|--save-optional] [-E|--save-exact] [-B|--save-bundle] [--no-save] [--dry-run]

    ```
- `upinstall`：卸载
    ```
    npm uninstall [<@scope>/]<pkg>[@<version>]... [-S|--save|-D|--save-dev|-O|--save-optional|--no-save]

    aliases: remove, rm, r, un, unlink
    ```
- `update`：更新
    ```
    npm update [-g] [<pkg>...]

    aliases: up, upgrade
    ```
- `publish`：发布
    ```
    npm publish [<tarball>|<folder>] [--tag <tag>] [--access <public|restricted>]

    Publishes '.' if no argument supplied
    Sets tag 'latest' if no --tag specified
    ```
- `unpublish`：取消发布
    ```
    npm unpublish [<@scope>/]<pkg>[@<version>]

    ```
- `whoami`：包所属
    ```
    npm whoami [--registry <registry>]
    ```
- `dist-tag`：加tag
    ```
    npm dist-tag add <pkg>@<version> [<tag>]
    npm dist-tag rm <pkg> <tag>
    npm dist-tag ls [<pkg>]
    
    aliases: dist-tags
    ```
- `config`：设置配置
    ```
    npm config set <key> <value> [-g|--global]
    npm config get <key>
    npm config delete <key>
    npm config list [-l]
    npm config edit
    npm get <key>
    npm set <key> <value> [-g|--global]
    
    aliases: c
    ``` 
- `adduser`：添加用户
    ```
    npm adduser [--registry=url] [--scope=@orgname] [--always-auth] [--auth-type=legacy]

    aliases: login, add-user
    ```
- `logout`：退出用户
    ```
    npm logout [--registry=<url>] [--scope=<@scope>]
    ```

### 0x003 总结
多看官方文档才是王道，知道如何找到自己想找的资料才是真正的办法，死命去记是没有办法的。

### 0x004 资源
- [项目github][2]


  [1]: https://segmentfault.com/a/1190000011103086
  [2]: https://github.com/followWinter/npm-study.git
