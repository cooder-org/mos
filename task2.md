# Task2：SSH支持
启动[任务一](./task1.md)的“模拟文件系统”服务后，能够从其他主机通过ssh连接到这个“模拟文件系统”，并能够使用「任务一」中的命令进行操作。 

## 任务描述
- 对任务一的codebase打个tag，命名为`tag-task1`
- 继续实现一个ssh server，能够接受ssh连接，并能执行「任务一」中的命令
- 支持多用户连接
- 新增实现以下命令

| 命令  | 说明   |
| ---- | -----  |
| rm     | 删除文件或目录|
| clear     | 清屏 |

## 提示
- `clear`的实现需要参考[ANSI_escape_code](https://en.wikipedia.org/wiki/ANSI_escape_code), 分为两步：
    - 清除屏幕内容：发送 `ESC[2J` 到ssh client
    - 移动光标到左上角：发送 `ESC[1;1H` 到ssh client  

  （其中`ESC`是`0x1B`）
- `rm`的实现需要注意目录不为空时不能删除
- ssh server不需要自己从0开始实现，可以使用第三方库，后面`参考lib`一节提供了两个参考，也可以自己找更合适的

## 交付物
- 源代码
- 可执行程序

交付方式：同任务一的`git repo` 

## 其他说明
- 暂不需要支持权限

## 验收要求
- `tag-task1`存在
- 命令操作顺畅、没有明显bug
- 时间要求：<= 4周, (超过4周还需继续完成，但当前任务记0分)
- 语言要求：同任务一
- 是否兼容非ssh模式
- 基础抽象是否稳定
- 代码实现是否简洁

## 功能参考
![./docs/demo.gif](./docs/demo.gif)

## 参考lib
ssh lib
- [Apache MINA SSHD](https://github.com/apache/mina-sshd)
- [Easy SSH servers in Golang](https://github.com/gliderlabs/ssh)

console lib
- [JLine is a Java library for handling console input.](https://github.com/jline/jline3)
  