

# Commit Message 格式

目前规范使用较多的是 [Angular 团队的规范](https://link.zhihu.com/?target=https%3A//github.com/angular/angular.js/blob/master/DEVELOPERS.md%23-git-commit-guidelines), 继而衍生了 [Conventional Commits specification](https://link.zhihu.com/?target=https%3A//conventionalcommits.org/). 很多工具也是基于此规范, 它的 message 格式如下:

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
复制代码
```

我们通过 git commit 命令带出的 vim 界面填写的最终结果应该类似如上这个结构, 大致分为三个部分(使用空行分割):

- 标题行: 必填, 描述主要修改类型和内容
- 主题内容: 描述为什么修改, 做了什么样的修改, 以及开发的思路等等
- 页脚注释: 放 Breaking Changes 或 Closed Issues

分别由如下部分构成:

- type: commit 的类型
    - feat: 新特性
    - fix: 修改问题
    - refactor: 代码重构
    - docs: 文档修改
    - style: 代码格式修改, 注意不是 css 修改
    - test: 测试用例修改
    - chore: 其他修改, 比如构建流程, 依赖管理.
- scope: commit 影响的范围, 比如: route, component, utils, build...
- subject: commit 的概述, 建议符合  [50/72 formatting](https://link.zhihu.com/?target=https%3A//stackoverflow.com/questions/2290016/git-commit-messages-50-72-formatting)
- body: commit 具体修改内容, 可以分为多行, 建议符合 [50/72 formatting](https://link.zhihu.com/?target=https%3A//stackoverflow.com/questions/2290016/git-commit-messages-50-72-formatting)
- footer: 一些备注, 通常是 BREAKING CHANGE 或修复的 bug 的链接.



注：50/72 格式简记为：

- 第一行不超过50个字符
- 然后是空白行
- 剩余文字应以72个字符换行

## git commit 模板

如果你只是个人的项目, 或者想尝试一下这样的规范格式, 那么你可以为 git 设置 commit template, 每次 git commit 的时候在 vim 中带出, 时刻提醒自己:

修改 ~/.gitconfig, 添加:

```
[commit]
template = ~/.gitmessage
复制代码
```

新建 ~/.gitmessage 内容可以如下:

```
# head: <type>(<scope>): <subject>
# - type: feat, fix, docs, style, refactor, test, chore
# - scope: can be empty (eg. if the change is a global or difficult to assign to a single component)
# - subject: start with verb (such as 'change'), 50-character line
#
# body: 72-character wrapped. This should answer:
# * Why was this change necessary?
# * How does it address the problem?
# * Are there any side effects?
#
# footer: 
# - Include a link to the ticket, if any.
# - BREAKING CHANGE
#
复制代码
```

