### 环境说明
#### 0x001 下载源码
从 GitHub 上下载 jquery 的源码
```
$ git clone https://github.com/jquery/jquery.git
```

### 0x002 切换分支
基于 v3.0.0 分支进行代码解析
```
$ cd jquery
$ git checkout -b v3.0.0 3.0.0
```

#### 0x003 打包
不直接解析源码，而是解析打包之后未压缩的文件
```
$ npm run build
```
执行上述命令将自动生成`dist`文件夹，包含下面结构的文件，其中`jquery.js`就是要分析的主要文件。
```
- dist
    - .eslintrc.json
    - jquery.js
    - jquery.min.js
    - jquery.min.map
    - jquery.slim.js
    - jquery.slim.min.js
    - jquery.slim.min.map
```
