# 长文件名支持
[任务一](./task1.md)的“模拟文件系统”中限定了文件名为8.3模式，最大为11个字符，不支持长文件名。现在需要支持任意长度的文件名。

## 设计文档
[VFATX](./docs/vfat-long-file-names-spec.md)

## 任务描述
- 对任务二的codebase打个tag，命名为`tag-task2`
- 继续实现[VFATX](./docs/vfat-long-file-names-spec.md)以支持长文件名
- 支持`scp`命令传输文件到模拟文件系统中或从模拟文件系统中传输文件到本地
- 支持管道，并实现简易的`grep`命令以验证管道功能, 如`ls | grep ".txt"`

## 交付物
- 源代码
- 可执行程序

交付方式：同任务一的`git repo` 

## 其他说明
- 暂不需要支持权限

## 验收要求
- 相关命令操作顺畅、没有明显bug
- 时间要求：<= 5周, (超过5周还需继续完成，但当前任务记0分)
- 语言要求：同任务一
- 是否兼容8.3文件名模式
- 基础抽象是否稳定
- 代码实现是否简洁

## 功能参考
![./docs/demo3.gif](./docs/demo3.gif)

## 参考资料
- [Go进阶52:开发扩展SSH的使用领域和功能](https://mojotv.cn/golang/ssh-pty-im)
- [SCP in mina-sshd](https://github.com/apache/mina-sshd/blob/master/docs/scp.md)