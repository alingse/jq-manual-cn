# jq 中文手册

文档版本期望与 jq 版本保持一致，目前 master 版先锁定 v1.5

## 简介
jq 是一个命令行 JSON 文本处理器，性能高效，语法简洁有力，是一款文本处理利器。

项目地址 : [stedolan/jq](https://github.com/stedolan/jq)

官网 : [jq](https://stedolan.github.io/jq/)

## 目的
本项目目的是翻译出 `jq` 的中文手册，推广 `jq` 在国内的使用 (毕竟某度搜 `jq` 都是jquery)，
方便使用者处理一些JSON数据(尽管也可以使用python)，但是jq语法更优雅便捷，而且可以随时命令行下修改调试)

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


------------------------------
以下为手册正文
------------------------------

