# Node 脚手架开发

## 一、项目初始化

项目初始化；

执行命令：

```she
pnpm init
```

在 `package.json` 中，配置项目名称：

package.json

```json
{
  "name": "zztcli",
}
```

## 二、入口文件

创建入口文件 `index.js`

lib\index.js

```js
console.log('zzt cli code exec~')
```

## 三、执行命令

尝试执行如下命令，运行该文件。

```shell
zztcli
```

出现了错误，找不到命令；

:recycle: 解决思路：

我们知道，在执行 vite 命令时，`vite/package.json` 文件中，配置了命令：

```json
{
  "bin": {
    "vite": "bin/vite.js"
  }
}
```

> “bin” 通常指代“二进制（binary）”

在 `vite/bin/vite.js` 文件中，上方有一行代码

这是一种 bash，表示在环境变量中找 node，来执行代码。

```js
#!/usr/bin/env node
```

### 1.“bin”配置

参考 vite 脚手架执行命令（其它脚手架都类似）;

在 `package.json` 中，进行如下配置；

package.json

```json
{
  "bin": {
    "zztcli": "./lib/index.js"
  }
}
```

然后给业务代码，加上注释：

lib\index.js

```js
#!/usr/bin/env node

console.log('zzt cli code exec~')
```

再次执行命令

```shell
zztcli
```

发现仍找不到命令；

### 2.本地连接

如果还没有发布到 npm，就不能 install 安装，也就不能使用 `npx` 去执行 node_modules 下的命令。

若要在本地执行命令，要先建立连接（link）。

在环境变量中，加入了 `zztcli`，建立一个软连接。

执行命令：

```shell
npm link
```

再次执行命令，有输出结果了：

```shell
zztcli

# 输出结果
zzt cli code exec~
```

## 四、解析参数

解析命令中的参数

### 1.--version

比如:解析该命令

```shell
zztcli --version
```

> 【回顾】：在 Node 环境中，`process.argv` 可以拿到命令中的参数。

命令中的所有参数，使用原生的方式解析，过于繁琐：

推荐使用工具 *commander* 工具（TJ 编写的）。

安装该工具:

```shell
pnpm add commander
```

修改业务代码：

lib\index.js

```js
#!/usr/bin/env node

const { program } = require('commander');

const version = require('../package.json').version // 动态获取 package.json 中的 version
// 处理 --version 操作
program.version(version, '-v --version')

// 解析 process.argv 参数
program.parse(process.argv)
```

执行命令：

```shell
zztcli --version
zztcli -v

# 输出结果
# 1.0.0
```

### 2.增加参数

增强其它 options 的操作。

`program.option` 方法，用于增加命令参数，并给予命令描述；

lib\index.js

```js
#!/usr/bin/env node
const { program } = require('commander');

const version = require('../package.json').version
// 1.处理 --version 操作
program.version(version, '-v --version')

// 2.增加其他的 options 的操作
program.option('-z --zzt', 'a zzt cli program~')

// 解析 process.argv 参数
program.parse(process.argv)
```

执行命令：

使用 --help 时，会展示增加的命令参数；

```shell
zztcli -z

# 没有输出结果

zztcli --help

# 输出结果
Usage: index [options]

Options:
  -v --version  output the version number
  -z --zzt      a zzt cli program~
  -h, --help    display help for command
```

:egg: 案例理解：

增加一个命令参数，这个参数用于将一个文件，拷贝到目标文件夹，为它添加描述。

lib\index.js

```js
program.option('-d --dest <dest>', 'a destination folder；例如：-d src/components')

// 获取命令中额外传递的参数
console.log(program.opts().dest)
```

执行命令

```shell
zztcli -d src/componets/xxx

# 输出结果
src/componets/xxx
```

### 3.--help 扩展

加入了自己打印的文本。

lib\index.js

```js
// 扩展 --help
program.on('--help', () => {
  console.log('自己打印的内容~')
})
```

执行命令：

```shell
zztcli --help

# 输出内容
Usage: index [options]

Options:
  -v --version      output the version number
  -z --zzt          a zzt cli program~
  -d --dest <dest>  a destination folder；例如：-d src/components
  -h, --help        display help for command
自己打印的内容~
```

## 五、逻辑抽取

将如下代码的逻辑，进行抽取：

lib\index.js

```js
#!/usr/bin/env node
const { program } = require('commander');

const version = require('../package.json').version
// 1.处理 --version 操作
program.version(version, '-v --version')

// 2.增强其他的 options 的操作
program.option('-z --zzt', 'a zzt cli program~')
program.option('-d --dest <dest>', 'a destination folder；例如：-d src/components')

// 扩展 --help
program.on('--help', () => {
  console.log('')
  console.log('others:')
  console.log('  xxx')
  console.log('  yyy')
})

// 解析 process.argv 参数
program.parse(process.argv)

// 获取额外传递的参数
console.log(program.opts().dest)
```

### 1.--version、--help

抽取 "help options" 相关代码

lib\core\help-options.js

```js
const { program } = require('commander');

function helpOptions() {
  const version = require('../../package.json').version
  // 1.处理 --version 操作
  program.version(version, '-v --version')

  // 2.增强其他的 options 的操作
  program.option('-z --zzt', 'a zzt cli program~')
  program.option('-d --dest <dest>', 'a destination folder；例如：-d src/components')

  // 扩展 --help
  program.on('--help', () => {
    console.log('')
    console.log('others:')
    console.log('  xxx')
    console.log('  yyy')
  })
}

module.exports = helpOptions
```

lib\index.js

```js
#!/usr/bin/env node
const { program } = require('commander');

const helpOptions = require('./core/help-options')

// 1.配置所有的 help options
helpOptions()

// 解析 process.argv 参数
program.parse(process.argv)

// 获取额外传递的参数
console.log(program.opts().dest)
```

## 六、功能编写

### 1.创建项目

增加一些具体的功能。

比如：执行命令，自动创建一个 vue 项目。

lib\index.js

```js
// 2.增加具体的一些功能操作。
program
  .command('create <project> [...others]')
  .description('create vue roject into a folder, 比如：zztcli create vue-aribnb')
  .action(function(project) {
    console.log('创建出来一个项目：', project)
  })
```

执行命令：

```shell
zztcli --help

# 输出结果：
Usage: index [options] [command]

Options:
  -v --version                  output the version number
  -z --zzt                      a zzt cli program~
  -d --dest <dest>              a destination folder；例如：-d src/components
  -h, --help                    display help for command

Commands:
  create <project> [...others]  create vue roject into a folder, 比如：zztcli create vue-aribnb
  help [command]                display help for command

others:
  xxx
  yyy
```

执行命令：

```shell
zztcli create xxxx

# 输出结果：
创建出来一个项目： xxxx
```

继续编写 create 命令，创建项目的功能：

安装一个库 *download-git-repo*，用于从远程 git 仓库中，拉取项目下来。

```shell
pnpm add download-git-repo
```

抽取 git 仓库地址为常量：

远程仓库中，是要创建项目的模板。

lib\config\repo-constant.js

```js
const VUE_REPO = 'direct:https://github.com/coderwhy/vue3_template.git#main'

module.exports = {
  VUE_REPO
}
```

抽取 action 函数：

lib\core\actions.js

```js
const { promisify } = require('util')
const { VUE_REPO } = require('../config/repo-constant')
// const download = require('download-git-repo')
const download = promisify(require('download-git-repo'))

async function createProjectAction(project) {
  // 1.从远程 git 仓库中，clone 下来模板。
  // 默认不支持 promise，要导入 Node 中的 util 模块，使用 promisify 方法包裹，才能使用 promise。
  /* download(VUE_REPO, project, { clone: true }, (err) => {
    if (err) console.log('发生错误：', err)
  }) */

  try {
    await download(VUE_REPO, project, { clone: true })
  } catch (err) {
    console.log('github 连接失败，请稍后重试~')
  }
}

module.exports = { createProjectAction }
```

lib\index.js

```js
const { createProjectAction } = require('./core/actions')

// 2.增加具体的一些功能操作。
program
  .command('create <project> [...others]')
  .description('create vue roject into a folder, 比如：zztcli create vue-aribnb')
  .action(createProjectAction)
```

执行命令：

```shell
zztcli create haha # 使用预设的模板，创建了 haha 项目
```

#### 1.给予提示、自动执行命令

拉取模板后，给予提示，帮助执行命令。

抽取 `execCommand` 方法：

lib\utils\exec-command.js

```js
const { spawn } = require('child_process') // 借助于 child_process 模块中的 spawn，帮助执行命令，会开启一个进程。

function execCommand(...args) {
  return new Promise((resolve, reject) => {
    // npm install / npm run dev
    // 1.开启子进程，执行命令
    const childProcess = spawn(...args)

    // 2.获取子进程的输出和错误信息。
    childProcess.stdout.pipe(process.stdout)
    childProcess.stderr.pipe(process.stderr)

    // 3.监听子进程执行结束，并关闭。
    childProcess.on('close', () => {
      resolve()
    })
  })
}

module.exports = execCommand
```

在 action 中，使用：

lib\core\actions.js

```js
const { promisify } = require('util')
const { VUE_REPO } = require('../config/repo-constant')
// const download = require('download-git-repo')
const download = promisify(require('download-git-repo'))
const execCommand = require('../utils/exec-command')

async function createProjectAction(project) {
  // 1.从远程 git 仓库中，clone 下来模板。
  // 默认不支持 promise
  /* download(VUE_REPO, project, { clone: true }, (err) => {
    if (err) console.log('发生错误：', err)
  }) */

  try {
    // 1.从远程 git 仓库中，clone 下来模板。
    await download(VUE_REPO, project, { clone: true })

    // 2.很多脚手架，都是在这里给予提示
    /* console.log(`cd ${project}`)
    console.log(`npm run install`)
    console.log(`npm run dev`) */

    // windows 平台的特殊适配
    console.log('process.platform:', process.platform) // Win32    
    const commandName = process.platform === 'win32' ? 'npm.cmd' : 'npm'
    // 3.帮助执行 npm install 命令
    await execCommand(commandName, ['install'], { cwd: `./${project}`}) // 去对应目录下，执行命令
    // 4.帮助执行 npm run dev 命令
    await execCommand(commandName, ['run', 'dev'], { cwd: `./${project}`})
  } catch (err) {
    console.log('github 连接失败，请稍后重试~')
  }
}

module.exports = { createProjectAction }
```

### 2.创建组件

使用脚手架，创建 vue 组件。

抽取 action：

lib\core\actions.js

```js
async function addComponentAction(cpnname) {
  // 1.创建一个组件：编写组件的模板，根据内容，传入模板中填充的数据
  console.log('添加一个组件，到某个文件夹中~', cpnname)
}

module.exports = { addComponentAction }
```

给 `program` 中，应用 action：

lib\index.js

```js
const { addComponentAction } = require('./core/actions')

program
  .command('addcpn <cpnname> [...others]')
  .description('add vue component into a folder, 比如：zztcli addcpn NavBar -d src/components')
  .action(addComponentAction)
```

创建一个 vue 模板，但是以 .ejs 后缀名结尾，

> 【补充】：ejs 是一个模板引擎。详见[官方文档](https://ejs.co/)。
>
> 给 VSCode 安装 EJS language support 插件，给予更好的提示。

创建 ejs 模板：

lib\template\component.vue.ejs

```ejs
<script setup>
import { ref } from 'vue';

const msg = ref('哈哈哈')
</script>

<template>
  <div class="<%= name %>">
    <h1>NavBar: {{ msg }}</h1>
  </div>
</template>

<style scoped>
.<%= name %> {
  color: red;
}
</style>
```

安装一个库 ejs，用于解析 ejs 模板

```shell
pnpm add ejs
```

抽取 util，用于编译 ejs 模板文件，并注入参数。

lib\utils\compile-ejs.js

```js
const path = require('path')
const ejs = require('ejs')

function compileEjs(tempName, data) {
  new Promise((resolve, reject) => {
    // 1.获取当前模板的路径
    const tempPath = `../template/${tempName}`
    const absolutePath = path.resolve(__dirname, tempPath)

    // 2.使用 ejs 引擎编译模板
    ejs.renderFile(absolutePath, data, (err, res) => {
      if (err) {
        console.log('编译模板失败~，err:', err)
        return
      }
      resolve(res)
    })
  })
}

module.exports = compileEjs
```

封装写入文件的工具。

lib\utils\write-file.js

```js
const fs = require('fs')

function writeFile(path, content) {
  return fs.promises.writeFile(path, content)
}

module.exports = writeFile
```

完善 action，，在 action 中使用封装的两个工具：

lib\core\actions.js

```js
const compileEjs = require('../utils/compile-ejs')
const writeFile = require('../utils/write-file')

async function addComponentAction(cpnname) {
  // 1.创建一个组件：编写组件的模板，根据内容赋值模板中，待填充的数据。
  console.log('添加一个组件，到某个文件夹中~', cpnname)
  const res = await compileEjs('component.vue.ejs', { name: cpnname, lowername: cpnname.toLowerCase() })

  // 2.将 res 写入到对应的文件中。
  await writeFile(`src/components/${cpnname}.vue`, res)
  console.log('创建组件成功：', cpnname + '.vue')
}

module.exports = { addComponentAction }
```

创建的组件，不一定在 “src/components" 这个目录下，

应该从 --dest 命令参数中，动态获取路径。

使用 `program.opts().dest`

lib\core\actions.js

```js
const compileEjs = require('../utils/compile-ejs')
const writeFile = require('../utils/write-file')
const { program } = require('commander')

async function addComponentAction(cpnname) {
  // 1.创建一个组件：编写组件的模板，更具内容黑痣模板中填充的数据
  console.log('添加一个组件，到某个文件夹中~', cpnname)
  const res = await compileEjs('component.vue.ejs', { name: cpnname, lowername: cpnname.toLowerCase() })

  // 2.将 res 写入到对应的文件中。
  const dest = program.opts().dest || 'src/components'
  await writeFile(`${dest}/${cpnname}.vue`, res)
  console.log('创建组件成功：', cpnname + '.vue')
}

module.exports = { addComponentAction }
```

执行命令：

```shell
# 在 src/views/home/cpns 目录下，创建 HomeHeader 组件。
zztcli addcpn HomeHeader --dest src/views/home/cpns
```
