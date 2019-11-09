# jq 中文手册

jq 是一个命令行 JSON 文本处理器，性能高效、语法简洁有力、表达能力强，是一款文本处理利器。

项目地址 [github.com/stedolan/jq](https://github.com/stedolan/jq)

官网 [stedolan.github.io/jq](https://stedolan.github.io/jq/)

原文档请往 [https://stedolan.github.io/jq/manual/](https://stedolan.github.io/jq/manual/)

版本列表

- [v1.5 中文手册](./manual/v1.5/)

## 翻译目的

项目旨在给出 `jq` 的中文使用手册，推广 `jq` 在中文社区的使用(毕竟某度搜 `jq` 都是jquery)。

方便使用者对 `jq` 常用语法的查阅

## Why jq ?

jq 长于处理 JSON 格式数据，可以进行选择、转换、合并以及一些复杂操作。

jq 可以在命令行下进行不断调试，而不必写程序文件去处理，同时代码即命令，更加的直白


## 示例

示例 A

```jq
$echo '{"s":2,"t":3,"w":5}'|jq
{
  "s": 2,
  "t": 3,
  "w": 5
}
$echo '{"s":2,"t":3,"w":5}'|jq '.t'
3
```
M
示例 B

功能:提取一行 JSON 数据中 p 值大于 0.2 的 id

```jq
echo '{"code":200,"data":{"items":[{"id":100,"p":0.3},{"id":101,"p":0.5},{"id":102,"p":0.7}]}}' > raw.json

cat raw.json|jq -r -c 'select(.code==200)|.data.items|map(select(.p>0.2))|.[]|{id}'
```

窗口:
<div>
<embed src="http://showterm.io/66cd2262111dbe29437ac" width= 640 height= 480 />
</div>

如果窗口没有显示，点击 [http://showterm.io/66cd2262111dbe29437ac ](http://showterm.io/66cd2262111dbe29437ac) 访问
