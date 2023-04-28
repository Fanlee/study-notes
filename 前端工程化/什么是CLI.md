## 什么是Shell

Shell是计算机提供给用户与其他程序进行交互的接口，Shell是一个命令解释器，当你输入命令后，由Shell进行解释后交给操作系统内核进行处理。

## 什么是Bash

Bash是一种程序，它的职责是用来进行人机交互

Bash和其他程序最大区别在于，它不是用来完成特定任务，我们通过bash shell来执行程序

## Bash有什么用

通过纯文本的控制台进行控制，它的主要交互方式通过键盘输入文本。文字反馈来实现人机交互

## 什么是CLI

命令行界面（CLI）是一种基于文本界面。用于运行程序，CLI接收键盘输入，在命令符号提示处输入命令，然后由计算机执行并返回结果

## commander命令行工具

```javascript
const {program} = require('commander')

// [other...]表示接收其他的参数
// alias表示指令别名
// description 帮助信息里面会显示具体描述
program
    .command('create <project> [other...]')
    .alias('crt')
    .description('创建项目')
    .action((project, args) => {})

program.parse(process.argv)
```

## Inquirer命令行交互工具

```javascript
const {inquirer} = require('inquirer')
// type: input 用户输入
// message: 问题描述
// type: input 用户输入
// return {username: ''}
inquirer.prompt([
    {
        type: 'input',
        name： 'username',
        message: '请输入名字'
    }
]).then((answer) => {
    console.log(answer)
})
```

## download-git-repo 拉取远程模板

```javascript
download('direct:https://gitee.com/anyueleo/vue-template.git', './tmp', {clone:true}, function (err) {
  
});
```

## ora命令行等待提示交互

```javascript
const ora = require('ora')
const spinner = ora().start()
spinner.text = '加载中...'

setTimeout(() => {
    spinner.succeed('成功')
    spinner.fail('失败')
    spinner.info('信息')
}, 3000)
```

## chalk命令行样式渲染

```javascript
const chalk = require('chalk')
console.log(chalk.blue('蓝色'))
console.log(chalk.red('红色'))
console.log(chalk.rgb(255.255.255)('颜色'))
console.log(chalk.bold('文字'))
console.log(chalk.green.bold('文字'))
```

