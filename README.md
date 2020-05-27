### 安装 Hexo

所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。

```
$ npm install -g hexo-cli
```

### 进阶安装和使用

对于熟悉 npm 的进阶用户，可以仅局部安装 `hexo` 包。

```
$ npm install hexo
```

安装以后，可以使用以下两种方式执行 Hexo：

1. `npx hexo `

2. 将 Hexo 所在的目录下的 `node_modules` 添加到环境变量之中即可直接使用 `hexo `：
```
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile
```

### hexo命令

生成
```
hexo generate
```
该命令可以简写为
```
hexo g
```

启动
```
hexo server
```

### algolia搜索使用

安装
```
npm install --save hexo-algolia
```

配置环境变量
```
# windows环境使用set命令
export HEXO_ALGOLIA_INDEXING_KEY=你的API Key
```

更新index
```
hexo algolia
```