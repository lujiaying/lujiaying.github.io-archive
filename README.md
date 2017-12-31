# lujiaying.github.io 

[![Build Status](https://travis-ci.org/lujiaying/lujiaying.github.io.svg?branch=source)](https://travis-ci.org/lujiaying/lujiaying.github.io)

My Blog

```branch source``` store source codes.

```branch master``` store deploy files.

## 操作指南 V1.0

以下操作均在source分支下进行

新建一篇文章
`$ hexo new <title>`

生成静态页面
`$ hexo generate`

本地服务器预览,默认端口4000
`$ hexo server`

部署到服务器
1. 使用Travis持续集成，只需在source分支下`$ hexo new <title>`，然后push代码即可自动构建。(recommend)
2. `$hexo deploy`


## 配置指南
### hexo setup

https://hexo.io/docs/setup.html

```
$ npm install -g hexo-cli
```

### repo setup
```
$ git clone https://github.com/lujiaying/lujiaying.github.io.git
$ git checkout --track origin/source
$ npm install
$ git clone https://github.com/lujiaying/hexo-theme-next themes/next
```
