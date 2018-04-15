# jq 中文手册


文档版本与jq版本保持一致，同时也是项目分支名

[jq v1.5 中文文档](../v1.5/manual.zh_CN.md)

master 的文档等 1.5 翻译完之后再更新


## 简介
jq 是一个命令行 JSON文本处理器，性能高效，语法简洁有力，是一款文本处理利器。

应该与`grep`,`awk`,`sed`并列，不过目前没有用到并行多核。

项目地址 : [stedolan/jq](https://github.com/stedolan/jq)

官网 : [jq](https://stedolan.github.io/jq/)

## 目的
本项目目的是翻译出`jq`的中文手册，推广`jq`在国内的使用(毕竟某度搜`jq`都是jquery)，

方便使用者处理一些JSON数据(尽管也可以使用python的[pyrapidjson](https://github.com/hhatto/pyrapidjson)，但是jq语法更优雅便捷，而且可以随时命令行下修改调试)


## Demo:
demo1. 

```jq
sh$echo '{"s":2,"t":3,"w":5}'|jq 
{
  "s": 2,
  "t": 3,
  "w": 5
}
```

demo2.

功能:提取一行JSON数据中p值大于0.2的id

```jq

```

窗口:
<div>
<iframe src="http://showterm.io/66cd2262111dbe29437ac" width="640" height="480">
</iframe>
</div>

如果窗口没有显示，点击[jq demo2](http://showterm.io/66cd2262111dbe29437ac)
