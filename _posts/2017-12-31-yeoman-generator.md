---
layout: post
title: 如何开发自己的 yeoman 脚手架
description: 脚手架能够帮我们自动生成包含本地调试、编译、打包、发布等工具的项目目录，使我们能够减少大量重复劳动的同时，遵循一定的开发规范，大大提升我们的开发、协同效率。
categories: 
- 前端开发
- 脚手架
tags: 
- 前端开发
- 脚手架
---

脚手架能够帮我们自动生成包含本地调试、编译、打包、发布等工具的项目目录，使我们能够减少大量重复劳动的同时，遵循一定的开发规范，大大提升我们的开发、协同效率。

脚手架所做的事情主要有：
* 生成规范化目录结构
* 根据用户输入配置项目信息，如项目名称、开发人员等
* 生成项目本地server、编译、单元测试、打包、发布等配置

### 开发自己的脚手架

下文中出现的 generator 统一代指 yeoman 脚手架生成器。

#### 创建一个 node 模块项目 
yeoman 脚手架生成器（generator） 本质上是一个 node 模块。创建一个 yeoman 脚手架生成器，首先我们要先创建一个文件夹，文件夹名称必须为 generator-name （name 为我们自定义脚手架生成器的名称）。这点非常重要，因为 yeoman 通过文件系统去找可用的脚手架生成器。

在刚刚创建的文件夹内，创建一个 package.json文件，手动输入如下内容，也可以通过在命令行运行 npm init 生成该文件。

```
{
  "name": "generator-name",
  "version": "0.1.0",
  "description": "",
  "files": [
    "generators"
  ],
  "keywords": ["yeoman-generator"],
  "dependencies": {
    "yeoman-generator": "^1.0.0"
  }
}
```

name 属性必须带有前缀 generator- ；keywords 属性必须含有 "yeoman-generator" , 说明字段最好有对该脚手架生成器的简约清晰的说明。此外，务必引入最新版 yeoman-generator 作为项目依赖，可以通过在命令行中运行 npm install --save yeoman-generator 来安装此依赖。

### 目录结构

yeoman 会根据脚手架生成器目录结构来完成相应操作，每个子脚手架生成器必须包含在自己的文件夹内。

当在命令行运行 yo name 命令时，yeoman 会默认生成 app 文件夹内的脚手生成器架内容。

子脚手架生成器可以通过在命令行中运行 yo name:subcommand 来执行。

项目目录示例如下：

```
|---- package.json
|---- generators/
    |---- app/
        |---- index.js
    |---- router/
        |---- index.js
```
yeoman 支持两种不同的目录结构，即 ./ 和 ./generators 两种形式。

上述示例也可以这么写:

```
|---- package.json
|---- app/
    |---- index.js
|---- router/
    |---- index.js
```

### 下一步
目录结构就位，就可以完整的脚手架生成器了。

yeoman 提供了一个 base generator 基础类 ，通过继承 base generator 可大幅减少我们生成一个脚手架生成器的工作量。

可以在 index.js 文件中，集成该基础类：

```
var Generator = require('yeoman-generator');

module.exports = class extends Generator {};
```

如果想支持 ES5 环境，静态方法 Generator.extend() 可以用来拓展基类，并且允许你提供一个新的 prototype。该方法参照 [class-extend](https://github.com/SBoudrias/class-extend)。

### 重写构造函数

有些生成器函数只能在构造函数 constructor 内部调用，这些特殊的方法可能用于重要状态的控制或者不对构造函数外部造成影响。

重写生成器构造函数的方法如下：

```
module.exports = class extends Generator {
  // The name `constructor` is important here
  constructor(args, opts) {
    // Calling the super constructor is important so our generator is correctly set up
    super(args, opts);

    // Next, add your custom code
    this.option('babel'); // This method adds support for a `--babel` flag
  }
};
```

### 增加自定义功能
增加到 property 上的每个方法都会按照某种顺序执行，但是某些特定的方法也会有特殊的触发逻辑。

如下，增加几个方法：

```
module.exports = class extends Generator {
  method1() {
    this.log('method 1 just ran');
  }

  method2() {
    this.log('method 2 just ran');
  }
};
```

### 运行脚手架

到这里，我们的脚手架就处于可运行状态了。刚才我们本地开发了 一个脚手架，但是它还不能全局运行。我们应该使用 npm 创建一个node全局模块，并且连接到本地。

在脚手架项目根目录下（generator-name/），输入如下

```
npm link
```

这样会安装依赖，并且连接全局模块到本地文件。然后，可通过在命令行中输入 yo name ，即可运行该脚手架生成器。

## 脚手架运行时环境

脚手架生成器方法是如何运行的、运行环境是怎样的是对于开发脚手架最重要的概念之一。

### 方法即行为

每一个绑定在脚手架生成器 prototype 上的方法都可以被当做一个任务。每个任务都是按顺序在 yeoman 运行环境中运行的。

### 助手和私有方法
既然每个 prototype 方法都被当做一个任务，你可能会疑惑如何去定义一个不会被自动调用的助手或私有方法。以下有3种方式达到该目的：

1. 为方法名添加下划线前缀

```
  class extends Generator {
    method1() {
      console.log('hey 1');
    }

    _private_method() {
      console.log('private hey');
    }
  }
```

2. 使用实例方法

```  
  class extends Generator {
    constructor(args, opts) {
      // Calling the super constructor is important so our generator is correctly set up
      super(args, opts)

      this.helperMethod = function () {
        console.log('won\'t be called automatically');
      };
    }
  }
```

3. 继承一个父生成器 （parent generator）

```
  class MyBase extends Generator {
    helper() {
      console.log('methods on the parent generator won\'t be called automatically');
    }
  }

  module.exports = class extends MyBase {
    exec() {
      this.helper();
    }
  };
```
### 生成器生命周期

按顺序运行任务对于单生成器来说已经够了，但是当你把多个生成器合在一起时就有问题了。这就是 yeoman 使用生命周期的原因。

生命周期是一个带有优先权的队列系统。

生命周期执行顺序如下：

1. initializing - 初始化函数
2. prompting - 接收用户输入阶段
3. configuring - 保存配置信息和文件
4. default - 自定义功能函数名称，如 method1
5. writing - 生成项目目录结构阶段
6. conflicts - 统一处理冲突，如要生成的文件已经存在是否覆盖等处理
7. install - 安装依赖阶段
8. end - 生成器结束阶段

### 异步任务

有几种暂停生命周期知道一个异步任务结束的方法。

最简单的方式就是 return 一个 promise。

如果你依赖的异步 API 不支持 promise，我们可以使用 this.async() 方法。调用 this.async() 方法会在任务结束时返回一个函数。

```
asyncTask() {
  var done = this.async();

  getUserEmail(function (err, name) {
    done(err);
  });
}
```

如果 done 函数被调用，并且传参为一个 error 参数，那么生命周期会结束，并且异常会抛出。

## 用户交互
### 系统提示
系统提示是脚手架生成器和用户交互的主要方式。系统提示模块使用[Inquirer.js](https://github.com/SBoudrias/Inquirer.js)。

系统提示命令时异步的，会返回一个 promise。你需要在你的任务中返回一个 promise ，来达到执行完当前任务，再执行下一个任务的目的。

示例如下：

```
module.exports = class extends Generator {
  prompting() {
    return this.prompt([{
      type    : 'input',
      name    : 'name',
      message : 'Your project name',
      default : this.appname // Default to current folder name
    }, {
      type    : 'confirm',
      name    : 'cool',
      message : 'Would you like to enable the Cool feature?'
    }]).then((answers) => {
      this.log('app name', answers.name);
      this.log('cool feature', answers.cool);
    });
  }
};
```

### 记录用户参数

对于特定问题，用户往往会给出相同的答案，对于这些问题，最好记住用户的历史输入。

yeoman 拓展了 Inquirer.js，新增 store 属性，用于允许记录用户历史输入。示例如下：

```
this.prompt({
  type    : 'input',
  name    : 'username',
  message : 'What\'s your GitHub username',
  store   : true
});
```

### 参数 arguments

参数直接从命令行中获取，如：

```
yo webapp my-project
```

上述例子中，my-project 就是一个参数。

定义参数的方法如下：

使用 this.argument(name, options) 方法，其中 options 是一个多键值对对象：

* desc String - 参数说明
* required Boolean - 是否必填
* type String, Number, Array (也可以是返回字符串的自定义函数) - 类型
* default - 默认值

this.argument(name, options) 只能在构造函数内调用。另外 help 参数不可用于传入值，如 yo webapp --help。


使用示例如下：

```
module.exports = class extends Generator {
  // note: arguments and options should be defined in the constructor.
  constructor(args, opts) {
    super(args, opts);

    // This makes `appname` a required argument.
    this.argument('appname', { type: String, required: true });

    // And you can then access it later; e.g.
    this.log(this.options.appname);
  }
};
```

### 选项 options

选项和参数非常相似。通过 generator.option(name, options) 调用。options 为一个多键值对对象，属性如下：

* desc -  选项说明
* alias - 别名
* type Either Boolean, String or Number (也可以是返回纯字符串的函数) - 类型
* default - 默认值
* hide Boolean - 是否从 help 中隐藏

使用示例：

```
module.exports = class extends Generator {
  // note: arguments and options should be defined in the constructor.
  constructor(args, opts) {
    super(args, opts);

    // This method adds support for a `--coffee` flag
    this.option('coffee');

    // And you can then access it later; e.g.
    this.scriptSuffix = (this.options.coffee ? ".coffee": ".js");
  }
};
```

### 输出信息

通过 generator.log 模块输出信息。使用示例：

```
module.exports = class extends Generator {
  myAction() {
    this.log('Something has gone wrong!');
  }
};
```

## 和文件系统交互
### 位置环境和路径

yeoman 文件功能是基于硬盘上的两个地址环境的，他们分别是生成器最长操作的读取和写入文件夹。

### 目标环境

第一个环境是目标环境。这个目标环境是 yeoman 创建脚手架应用的文件夹，也就是你的项目文件夹。

目的地环境通常是当前目录或者包含 .yo-rc.json 文件的最近父文件夹。.yo-rc.json 文件定义了 yeoman 工程的根目录。这个文件允许你的用户在子目录中运行命令。

可以通过 generator.destinationRoot() 或者 generator.destinationPath('sub/path') 获取目标路径。

```
// Given destination root is ~/projects
class extends Generator {
  paths() {
    this.destinationRoot();
    // returns '~/projects'

    this.destinationPath('index.js');
    // returns '~/projects/index.js'
  }
}
```

 你可以通过 generator.destinationRoot('new/path') 手动修改目标路径。但是为了便于维护，最好不要修改默认路径。
 
 如果你想知道用户是在哪里运行  yo 命令的，你可以通过 this.contextRoot 获取该路径。
 
### 模板环境

模板环境是你存储模板文件的文件夹，这通常是你将要读取和拷贝的文件夹。

模板环境默认为 ./templates/ ，可以通过 generator.sourceRoot('new/template/path') 进行重写。

可以通过 generator.sourceRoot() 或 generator.templatePath('app/index.js') 获取模板路径。

 ```
class extends Generator {
  paths() {
    this.sourceRoot();
    // returns './templates'

    this.templatePath('index.js');
    // returns './templates/index.js'
  }
};
```

### 文件功能

生成器将所有方法都暴露于 this.fs , this.fs 是 [mem-fs editor](https://github.com/sboudrias/mem-fs-editor)的实例，可自行学习其 API 。

不过 commit 方法没任何价值，不要在你的生成器中使用。yeoman 会在生命周期的 confilcts 阶段自行调用。

#### 实例：拷贝一个模板文件

模板文件如下：

```
<html>
  <head>
    <title><%= title %></title>
  </head>
</html>
```

我们通过 copyTpl 方法拷贝文件。

```
class extends Generator {
  writing() {
    this.fs.copyTpl(
      this.templatePath('index.html'),
      this.destinationPath('public/index.html'),
      { title: 'Templating with Yeoman' }
    );
  }
}
```

运行生成器， public/index.html 将包含如下内容:

```
<html>
  <head>
    <title>Templating with Yeoman</title>
  </head>
</html>
```

#### 通过流转换输出文件

生成器系统允许你对每个文件进行自定义过滤处理。自动美化文件、格式化空白等也都是可以的。

yeoman 运行过程中，文件将被一个 vinyl 对象流（像 gulp 一样）处理，任何生成器的作者都可以注册一个流转换来修改文件路径和内容。

示例如下:

```
var beautify = require('gulp-beautify');
this.registerTransformStream(beautify({indent_size: 2 }));
```

注意每个任何类型的文件都会被流处理。要确保不被支持的文件会被流转换器过滤掉。像 gulp-if 或 gulp-filter 之类的工具会帮我们过滤非法类型的文件。

在生成文件的 writing 阶段，你可以在 yeoman 流转换中使用任何 gulp 插件。

## demo
根据以上教程，开发了一个基于 yoeman 的 vue ui 组件脚手架生成器 [generator-vueui](https://github.com/korbinzhao/generator-vueui)。


### 使用方法

```
npm install -g yo
npm install -g generator-vueui

mkdir vueui-example
cd vueui-example

yo vueui

```

