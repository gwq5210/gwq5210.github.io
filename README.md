# blog
rhettguo' blog

# 安装步骤
* 安装[node.js][https://nodejs.org/en/]

* 安装[git][https://git-scm.com/]

* 使用如下命令安装hexo
```
	npm install -g hexo-cli
```
* 安装hexo插件
```
	npm install hexo-renderer-less --save
	npm install hexo-generator-index --save
	npm install hexo-generator-archive --save
	npm install hexo-generator-category --save
	npm install hexo-generator-tag --save
	npm install hexo-server --save
	npm install hexo-deployer-git --save
	npm install hexo-deployer-heroku --save
	npm install hexo-deployer-rsync --save
	npm install hexo-deployer-openshift --save
	npm install hexo-renderer-marked@0.2 --save
	npm install hexo-renderer-stylus@0.2 --save
	npm install hexo-generator-feed@1 --save
	npm install hexo-generator-sitemap@1 --save
```
* 执行如下命令后，查看效果，一般通过浏览器访问http://localhost:4000/
```
	hexo s
```

* 使用mathjax的问题
参考[这里][http://blog.csdn.net/u014630987/article/details/78670258]

# 参考
1) [使用GitHub和Hexo搭建免费静态Blog][https://wsgzao.github.io/post/hexo-guide/]
2) [hexo官网][https://hexo.io/zh-cn/docs/]