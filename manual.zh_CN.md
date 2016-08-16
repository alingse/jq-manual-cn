---
title : jq 中文手册
---

<style type="text/css">

p code {
      color:#c7254e;
}
h3 code {
      color:#c7254e;
}

</style>

# jq 中文手册(v1.5)

开发版本的jq 中文手册请点[这里](../master/manual.zh_CN.md)

jq 程序就像一个过滤器：接收输入，并产生输出。有许多内置的过滤器，提取一个对象的特定字段、或是把数字转成字符串，或是大量的其他的标准任务。

可以通过多种方法结合这些过滤器 － 可以用管道将一个过滤器的输出连到另一个的输入上，或者是把一个过滤器的输出收集到一个数组里。

一些过滤器产生多个结果，比如就有一个可以展开输入的数组把每个元素都给出来的的过滤器，用管道把这个连到第二个过滤器上就使第二个过滤器在数组的每一个元素上作用一遍。通常在其他语言里用循环或者迭代的任务在jq中通过结合过滤器来完成。

切记每个过滤器都有一个输入和一个输出。即使像"hello"或者42这样的常量都是过滤器－他们接受输入但是只产生同样的常量作为输出罢了。操作符可以结合两个过滤器，比如 **加** , 一般是给两个过滤器同样的输入，并把结果连接起来。所以你可以实现一个求平均过滤器，即`add/length` － 把输入数组分给`add`过滤器和`length`过滤器，然后做了一个除法。

但是说这个可能有些超前了。: ),接着看一些简单的:

内容:

- [调用jq](#Invokingjq)
- [基本过滤器](#Basicfilters)
- [类型和值](#TypesandValues)
- [内置操作符和函数](#Builtinoperatorsandfunctions)
- [条件和比较](#ConditionalsandComparisons)
- [正则表达式(PCRE)](#RegularexpressionsPCRE)
- [高级特性](#Advancedfeatures)
- [数学](#Math)
- [I/O](#IO)
- [流](#Streaming)
- [赋值](#Assignment)
- [模版](#Modules)


## [调用jq](#Invokingjq)
jq的过滤器运行在一个JSON数据流上.jq的输入被解析为一系列由空白符分隔的JSON数据，然后一次一个的传给提供的过滤器，过滤器的输出会被写入标准输出，也是一系列的空白符分隔的JSON数据。

注意：切记shell的引号规则。一般来说，最好一直都给jq程序加引号（用单引号）,因为需要jq中有特殊含义的字符也是shell元字符。比如，`jq "foo"`在大多数的Unix shells里会失败，因为是跟`jq foo`的效果一样，通常是因为`foo is not defined`。
当使用windows 命令行（cmd.exe）时，最好使用双引号括起你的jq程序（当不使用 `-f program-file`选项时）,但是程序里面的双引号就需要使用`\`来转义了。

可以使用下面的 命令行选项 来控制 jq 如果读写输入和输出： 

- `--version`:
  
  输出 jq 的版本并退出，退出状态为0

- `--seq`:

  Use the `application/json-seq` MIME type scheme for separating JSON texts in jq’s input and output. This means that an ASCII RS (record separator) character is printed before each value on output and an ASCII LF (line feed) is printed after every output. Input JSON texts that fail to parse are ignored (but warned about), discarding all subsequent input until the next RS. This more also parses the output of jq without the <code>--seq</code> option.

- `--stream`:

 Parse the input in streaming fashion, outputing arrays of path and leaf values (scalars and empty arrays or empty objects). For example, 

 `"a"` becomes `[[],"a"]`, and  `[[],"a",["b"]]` becomes `[[0],[]]`,`[[1],"a"]`
  , and `[[1,0],"b"]`.

	This is useful for processing very large inputs. Use this in conjunction with filtering and the `reduce` and `foreach` syntax to reduce large inputs incrementally.

- `--slurp` / `-s`:

	Instead of running the filter for each JSON object in the input, read the entire input stream into a large array and run the filter just once.
	
- `--raw-input` / `-R`:

 Don’t parse the input as JSON. Instead, each line of text is passed to the filter as a string. If combined with `--slurp`, then the entire input is passed to the filter as a single long string.

- `--null-input` / `-n`:
  
  Don’t read any input at all! Instead, the filter is run once using `null` as the input. This is useful when using jq as a simple calculator or to construct JSON data from scratch.

- `--compact-output` / `-c`:  

   By default, jq pretty-prints JSON output. Using this option will result in more compact output by instead putting each JSON object on a single line.

- `--tab`:
  
   Use a tab for each indentation level instead of two spaces.

- `--indent n`:

   Use the given number of spaces (no more than 8) for indentation.

- `--color-output` / `-C` and `--monochrome-output` / `-M`:

  By default, jq outputs colored JSON if writing to a terminal. You can force it to produce color even if writing to a pipe or a file using `-C`, and disable color with `-M`.
  
- `--ascii-output` / `-a`:

 jq usually outputs non-ASCII Unicode codepoints as UTF-8, even if the input specified them as escape sequences (like “\u03bc”). Using this option, you can force jq to produce pure ASCII output with every non-ASCII character replaced with the equivalent escape sequence.

- `--unbuffered`:

  Flush the output after each JSON object is printed (useful if you’re piping a slow data source into jq and piping jq’s output elsewhere).

- `--sort-keys` / `-S`:

  Output the fields of each object with the keys in sorted order.

- `--raw-output` / `-r`:

 With this option, if the filter’s result is a string then it will be written directly to standard output rather than being formatted as a JSON string with quotes. This can be useful for making jq filters talk to non-JSON-based systems.
 
- `--join-output` / `-j`:

 Like `-r` but jq won’t print a newline after each output.

- `-f filename` / `--from-file filename`:

 Read filter from the file rather than from a command line, like awk’s -f option. You can also use ‘#’ to make comments.

- `-Ldirectory ` / `-L directory`:

 Prepend <code>directory</code> to the search list for modules. If this option is used then no builtin search list is used. See the section on modules below. 

- `-e` / `--exit-status`:

 Sets the exit status of jq to 0 if the last output values was neither `false` nor `null`, 1 if the last output value was either `false` or `null`, or 4 if no valid result was ever produced. Normally jq exits with 2 if there was any usage problem or system error, 3 if there was a jq program compile error, or 0 if the jq program ran.

- `--arg name value`:

	This option passes a value to the jq program as a predefined variable. If you run jq with `--arg foo bar`, then `$foo` is available in the program and has the value `"bar"`. Note that `value` will be treated as a string, so `--arg foo 123` will bind `$foo` to `"123"`.

- `--argjson name JSON-text`:

 This option passes a JSON-encoded value to the jq program as a predefined variable. If you run jq with `--argjson foo 123`, then `$foo` is available in the program and has the value `123`.
 
- `--slurpfile variable-name filename`:

 This option reads all the JSON texts in the named file and binds an array of the parsed JSON values to the given global variable. If you run jq with `--argfile foo bar`, then `$foo` is available in the program and has an array whose elements correspond to the texts in the file named `bar`.

- `--argfile variable-name filename`:
 
 Do not use. Use `--slurpfile` instead.

  (This option is like `--slurpfile`, but when the file has just one text, then that is used, else an array of texts is used as in `--slurpfile`.)

- `--run-tests [filename]`:

 Runs the tests in the given file or standard input. This must be the last option given and does not honor all preceding options. The input consists of comment lines, empty lines, and program lines followed by one input line, as many lines of output as are expected (one per output), and a terminating empty line. Compilation failure tests start with a line containing only “%%FAIL”, then a line containing the program to compile, then a line containing an error message to compare to the actual.

 Be warned that this option can change backwards-incompatibly.



##  [基本过滤器](#Basicfilters)

### `.`

绝对最简单（也最平常）的过滤器是 `.｀，这是一个接收输入并原样输出的过滤器。

因为jq默认会优美打印所有的输出，这个小程序可以用来格式化一些JSON输出，比如`curl`。

[Example](#example1)

|            | jq  '.'         | 
| -----------|:---------------:| 
| **Input**  | "Hello, world!" | 
| **Output** | "Hello, world!" | 



### `.foo`,`.foo.bar`
 
  最简单的*有用的*过滤器是`.foo`. 给定一个JSON object(即字典或hash)做输入，它会给出"foo"键的值，如果没有这个key则给出null.

 如果键里含有关键字符，就要用双引号括起来，比如:."foo$".

一个形如`.foo.bar`的过滤器是`.foo|.bar`的对等写法。

[Examples](#example2)

```jq
        jq  '.foo'
--------------------
Input   {"foo": 42, "bar": "less interesting data"}
Output  42
```
```jq
        jq  '.foo'
--------------------
Input   {"notfoo": true, "alsonotfoo": false}
Output  null
```
```jq
        jq  '.["foo"]'
--------------------
Input   {"foo": 42}
Output  42
```


### `.foo?`
                 
 就跟`.foo`差不多,但是当`.`不是一个数组或一个object而报错时，不会输出。
 
[Examples](#example3)

```jq
        jq  '.foo?'
--------------------
Input   {"foo": 42, "bar": "less interesting data"}
Output  42
```
```jq
        jq  '.foo?'
--------------------
Input   {"notfoo": true, "alsonotfoo": false}
Output  null
```
```jq
        jq  '.["foo"]?'
--------------------
Input   {"foo": 42}
Output  42
```
```jq
        jq  '[.foo?]'
--------------------
Input   [1,2]
Output  []
```

### `.[<string>]`,`.[2]`,`.[10:15]`

可以用像`.["foo"]`这样的语法来查找一个object的多个域(上面的`.foo`是这种的速写版).这种语法在数组的情况下也可以用，如果key是正整数的话。数组是从0开始计数的（跟javascript类似），所以`.[2]`返回数组的第三个元素。

`.[10:15]`这样的语法可以用来返回一个数组的子数组或者一个字符串的子字符串。`.[10:15]`返回的数组长度是5，包含索引从10（包含）到15（不含）的元素。
 任一索引都可以是负的（这种情况下会从数组最后往前计数）, 也可以省略掉(这表示从数组开头开始或者一直到数组结尾)。

`.[2]`这样的语法用来返回指定索引的数组元素。负数索引也是可以的，-1 指向最后一个元素，-2指向倒数第二个元素，以此类推。

`.foo`这样的语法只在简单的key的情况下有用，比如，key都是字母和数组组合的字符。`.[<string>]`可以在key包含特殊字符如冒号和点的情况下有用，比如，`.["foo::bar"]`和`.["foo.bar"]`就能正常工作而`.foo::bar`和`.foo.bar`就不行。

`?`”操作符“也可以用在切片操作符上,如`.[10:15]?`，只在可以切片操作的输入上输出值。

[Examples](#example4)

```jq
        jq '.[0]'
--------------------
Input   [{"name":"JSON", "good":true}, {"name":"XML", "good":false}]
Output  {"name":"JSON", "good":true}
```
```jq
       jq '.[2]'
--------------------
Input  [{"name":"JSON", "good":true}, {"name":"XML", "good":false}]
Output null
```
```jq
       jq '.[2:4]'
Input  ["a","b","c","d","e"]
Output ["c","d"]
```
```jq
       jq '.[2:4]'
--------------------
Input  "abcdefghi"
Output "cd"
```
```jq
       jq '.[:3]'
--------------------
Input  ["a","b","c","d","e"]
Output ["a","b","c"]
```
```jq
       jq '.[-2:]'
--------------------
Input  ["a","b","c","d","e"]
Output ["d","e"]
```
```jq
       jq '.[-2]'
--------------------
Input  [1,2,3]
Output 2
```


### `.[]`

如果是使用`.[index]`语法,但是省略掉索引，就会返回数组的*所有*元素。对输入`[1,2,3]`运行`.[]`会产生3个分开的数。而不是单个数组。

你也可以用在一个object上，它会返回这个object的所有value。

[Examples](#example5)

```jq
       jq '.[]'
--------------------
Input  [{"name":"JSON", "good":true}, {"name":"XML", "good":false}]
Output {"name":"JSON", "good":true}
       {"name":"XML", "good":false}
```
```jq
       jq '.[]'
--------------------
Input  []
Output none
```
```jq
       jq '.[]'
--------------------
Input  {"a":1,"b":1}
Output 1
       1
```

### `.[]?`

跟`.[]`一样，但是在`.`不是数组或object的情况下不会报错。                  

### `，`
如果两个过滤器用逗号分开，那么输入会同时流向它们，并产生多个输出：首先，所有左边表达式产生的输出，然后是右边表达式产生的输出。比如，过滤器`.foo,.bar`会生成"foo"字段的值和"bar"字段的值，并分别输出。

[Examples](#example6)

```jq
       jq '.foo, .bar'
----------------------
Input  {"foo": 42, "bar": "something else", "baz":true}
Output 42
       "something else"
```
```jq
	    jq '.user, .projects[]'
-------------------------------
Input	{"user":"stedolan", "projects": ["jq","wikiflow"]}
Output "stedolan"
       "jq"
       "wikiflow"
```
```jq
       jq '.[4,2]'
--------------------
Input  ["a","b","c","d","e"]
Output "e"
       "c"
```
### `|`

`|`操作符结合两个过滤器，把左边过滤器的输出定向到右边过滤器的输入。如果你熟悉管道的话，就知道这和Unix shell的管道(pipe)简直一样。

如果左边的过滤器产生了多个结果，那么右边的过滤器就会在运行在每一个上面。所以，表达式`.[] ｜.foo`就是提取输入数组里每一个元素的"foo"字段。

[Examples](#example7)

```jq
        jq '.[] | .name'
-------------------------
Input	 [{"name":"JSON", "good":true}, {"name":"XML","good":false}]
Output  "JSON"
        "XML"
```

## [类型和值](#TypesandValues)

jq 支持JSON里的全部数据类型 - 数值(numbers),字符串(strings),布尔值(booleans),数组(arrays),object(就是JSON语中仅由string做键的hash),以及"null"。

布尔值，null，字符串和数值都和javascript中的的一样。就像jq中的其它东西一样，这些简单值也接收输入产生输出 - `42`是一个合法的jq表达式，接受输入并忽略掉，然后产生42作为输出。


### 数组结构`[]`

在JSON中，`[]` 用来构建数组,如[1,2,3]。数组的元素可以是任意的jq表达式,所有这些表达式产生的结果都被合起来组成一个大数组。
你可以用它来构建一个由已知数量的值组成的数组（比如`[.foo,.bar,.baz]`）或者是"收集"一个过滤器产生的所有输出到一个数组里(比如:`[.items[].name]`)。

一旦你理解了","操作符，就可以从不同角度看jq的数组语句语法:表达式`[1,2,3]`不适使用内建的逗号分隔数组的语法，而是把`[]`操作符应用在表达式1,2,3上面(这个表达式产生3个输出结果。)

如果你有一个过滤器 `X`生成4个输出结果，那么表达式`[X]`会生成一个由四个元素的单个数组的输出。

[Example](#example8)

```jq
       jq  '[.user, .projects[]]'
-----------------------------------
Input  {"user":"stedolan", "projects": ["jq","wikiflow"]}
Output ["stedolan", "jq", "wikiflow"]
```

### Objects`{}`

像在JSON一样，`{}`是用来构建对象(也即字典或哈希)的，比如:`{"a":42,"b":17}`.

如果key 都是"常规"的（全都是字母）,那么引号就可以省略掉。值可以是任何表达式（不过复杂的情况下，你可能需要用小括号括起来）,这些表达式的输入也是 {} 表达的输入(切记，所有的过滤器都有输入和输出)。

```jq
{foo: .bar}
```
在给定输`{"bar":42, "baz":43}`的情况下，会生成一个JSON 对象`{"foo":42}`。就可以用这个来选出对象的特定字段：如果输入是一个对象且有"user","title","id"和"content"字段，只想要"user"和"title"字段的话，可以这样写：

```jq
{user: .user, title: .title}
```

因为这种情况很常见，可以简写为这种：`{user,title}`.

如果其中有一个表达式产生了多个结果，那么就会生成多个字典。如果输入是：

```jq
{"user":"stedolan","titles":["JQ Primer", "More JQ"]}
```
那么这样的表达式：

```jq
{user, title: .titles[]}
```
会生成两个输出：

```jq
{"user":"stedolan", "title": "JQ Primer"}
{"user":"stedolan", "title": "More JQ"}
```

括号括起来的key 表明是里面要作为一个表达式执行的。同样用上面的输入

```jq
{(.user): .titles }
```
会生成

```jq
{"stedolan": ["JQ Primer", "More JQ"]}
```

[Examples](#example9)

```jq
       jq '{user, title: .titles[]}'
-------------------------------------
Input  {"user":"stedolan","titles":["JQ Primer", "More JQ"]}
Output {"user":"stedolan", "title": "JQ Primer"}
       {"user":"stedolan", "title": "More JQ"}
```

```jq
       jq '{(.user): .titles}'
-------------------------------
Input  {"user":"stedolan","titles":["JQ Primer", "More JQ"]}
Output {"stedolan": ["JQ Primer", "More JQ"]}
```


##  [内置操作符和函数](#Builtinoperatorsandfunctions)

一些jq操作符(举例：`+`) 会因为参数的类型(数组、数值以及等等)而起不同的作用。但是jq 不允许隐式的类型转换（译者：参考javascript）。如果要把一个字符串加到一个对象上只会得到错误提示而不产生任何结果。


### 加－`＋`

操作符`+`接收两个过滤器，给它们两份同样的输入，然后把得到的结果结合起来，至于"加"是什么意思，要看涉及的类型：

- **Numbers** 按算术相加
- **Arrays** 连接（concatenate）成一个大数组
- **Strings** 连接（join）成一个大字符串
- **Objects** 通过merge 来相加,即把两个Object的键值对都插入到一个单独的结合的Object中。如果两个object有同样的key，那么右边的会保留。(如果想要递归merge的话，要使用`*`操作符。)

`null`可以加到任何值上去，然后返回一个没有任何改变的值。

[Examples](#example10)

```jq
       jq '.a + 1'
--------------------
Input  {"a": 7}
Output 8
```
```jq
       jq '.a + .b'
---------------------
Input  {"a": [1,2], "b": [3,4]}
Output [1,2,3,4]
```
```jq
       jq '.a + null'
-----------------------
Input  {"a": 1}
Output 1
```
```jq
       jq '.a + 1'
--------------------
Input  {}
Output 1
```
```jq
       jq '{a: 1} + {b: 2} + {c: 3} + {a: 42}'
Input  null
Output {"a": 42, "b": 2, "c": 3}
```


### 减－`-`

就和正常的算术减法对数值的操作一样，`-`操作符能用在数组上,从第一个数组中移除所有出现在第二个数组中的元素。

[Examples](#exmaple11)

```jq
       jq '4 - .a'
--------------------
Input  {"a":3}
Output 1
```
```jq
       jq '. - ["xml", "yaml"]'
--------------------------------
Input  ["xml", "yaml", "json"]
Output ["json"]
```

###  乘,除,取模 - `*`,`/`,`%`
                  
这些中缀运算符在两个数值上就和预期的一样。除以0会抛出错误，`x % y`计算 x 模上 y。

一个字符串乘以一个数会生成这个字符串拼接多次的结果。
`"x" * 0` 产生**null**。

一个字符串除以另一个字符串会用第二个字符串来分割(split)第一个分成数份。

两个object相乘会递归地merge两者: 就和加法的情况类似，但在两个object包含同一个key并且两个value都是object的情况下，会用同样的策略来merge这两个value。

[Examples](#example12)

```jq
       jq '10 / . * 3'
-----------------------
Input  5
Output 6
```
```jq
       jq '. / ", "'
-----------------------
Input  "a, b,c,d, e"
Output ["a","b,c,d","e"]
```
```jq
       jq '{"k": {"a": 1, "b": 2}} * {"k": {"a": 0,"c": 3}}'
-----------------------------------------------------------
Input  null
Output {"k": {"a": 0, "b": 2, "c": 3}}
```
```jq
       jq '.[] | (1 / .)?'
-----------------------------
Input  [1,0,-1]
Output 1
       -1
```

### `length`


The builtin function <code>length</code> gets the length of various different types of value:</p>

<ul>
<li>
<p>The length of a <strong>string</strong> is the number of Unicode codepoints it contains (which will be the same as its JSON-encoded length in bytes if it’s pure ASCII).</p>
</li>

<li>
<p>The length of an <strong>array</strong> is the number of elements.</p>
</li>

<li>
<p>The length of an <strong>object</strong> is the number of key-value pairs.</p>
</li>

<li>
<p>The length of <strong>null</strong> is zero.</p>
</li>
</ul>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example13">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example13" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[] | length'</td></tr>
                            <tr><th>Input</th><td>[[1,2], &quot;string&quot;, {&quot;a&quot;:2}, null]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>2</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>6</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>1</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>0</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="keys,keys_unsorted">
                  <h3>
                    
<code>keys</code>, <code>keys_unsorted</code>

                    
                  </h3>
                  
<p>The builtin function <code>keys</code>, when given an object, returns its keys in an array.</p>

<p>The keys are sorted “alphabetically”, by unicode codepoint order. This is not an order that makes particular sense in any particular language, but you can count on it being the same for any two objects with the same set of keys, regardless of locale settings.</p>

<p>When <code>keys</code> is given an array, it returns the valid indices for that array: the integers from 0 to length-1.</p>

<p>The <code>keys_unsorted</code> function is just like <code>keys</code>, but if the input is an object then the keys will not be sorted, instead the keys will roughly be in insertion order.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example14">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example14" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'keys'</td></tr>
                            <tr><th>Input</th><td>{&quot;abc&quot;: 1, &quot;abcd&quot;: 2, &quot;Foo&quot;: 3}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;Foo&quot;, &quot;abc&quot;, &quot;abcd&quot;]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'keys'</td></tr>
                            <tr><th>Input</th><td>[42,3,35]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[0,1,2]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="has(key)">
                  <h3>
                    
<code>has(key)</code>

                    
                  </h3>
                  
<p>The builtin function <code>has</code> returns whether the input object has the given key, or the input array has an element at the given index.</p>

<p><code>has($key)</code> has the same effect as checking whether <code>$key</code> is a member of the array returned by <code>keys</code>, although <code>has</code> will be faster.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example15">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example15" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'map(has(&quot;foo&quot;))'</td></tr>
                            <tr><th>Input</th><td>[{&quot;foo&quot;: 42}, {}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[true, false]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'map(has(2))'</td></tr>
                            <tr><th>Input</th><td>[[0,1], [&quot;a&quot;,&quot;b&quot;,&quot;c&quot;]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[false, true]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="in">
                  <h3>
                    
<code>in</code>

                    
                  </h3>
                  
<p>The builtin function <code>in</code> returns the input key is in the given object, or the input index corresponds to an element in the given array. It is, essentially, an inversed version of <code>has</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example16">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example16" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[] | in({&quot;foo&quot;: 42})'</td></tr>
                            <tr><th>Input</th><td>[&quot;foo&quot;, &quot;bar&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'map(in([0,1]))'</td></tr>
                            <tr><th>Input</th><td>[2, 0]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[false, true]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="path(path_expression)">
                  <h3>
                    
<code>path(path_expression)</code>

                    
                  </h3>
                  
<p>Outputs array representations of the given path expression in <code>.</code>. The outputs are arrays of strings (keys in objects0 and/or numbers (array indices.</p>

<p>Path expressions are jq expressions like <code>.a</code>, but also <code>.[]</code>. There are two types of path expressions: ones that can match exactly, and ones that cannot. For example, <code>.a.b.c</code> is an exact match path expression, while <code>.a[].b</code> is not.</p>

<p><code>path(exact_path_expression)</code> will produce the array representation of the path expression even if it does not exist in <code>.</code>, if <code>.</code> is <code>null</code> or an array or an object.</p>

<p><code>path(pattern)</code> will produce array representations of the paths matching <code>pattern</code> if the paths exist in <code>.</code>.</p>

<p>Note that the path expressions are not different from normal expressions. The expression <code>path(..|select(type==&quot;boolean&quot;))</code> outputs all the paths to boolean values in <code>.</code>, and only those paths.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example17">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example17" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'path(.a[0].b)'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;a&quot;,0,&quot;b&quot;]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[path(..)]'</td></tr>
                            <tr><th>Input</th><td>{&quot;a&quot;:[{&quot;b&quot;:1}]}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[[],[&quot;a&quot;],[&quot;a&quot;,0],[&quot;a&quot;,0,&quot;b&quot;]]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="del(path_expression)">
                  <h3>
                    
<code>del(path_expression)</code>

                    
                  </h3>
                  
<p>The builtin function <code>del</code> removes a key and its corresponding value from an object.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example18">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example18" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'del(.foo)'</td></tr>
                            <tr><th>Input</th><td>{&quot;foo&quot;: 42, &quot;bar&quot;: 9001, &quot;baz&quot;: 42}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;bar&quot;: 9001, &quot;baz&quot;: 42}</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'del(.[1, 2])'</td></tr>
                            <tr><th>Input</th><td>[&quot;foo&quot;, &quot;bar&quot;, &quot;baz&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;foo&quot;]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="to_entries,from_entries,with_entries">
                  <h3>
                    
<code>to_entries</code>, <code>from_entries</code>, <code>with_entries</code>

                    
                  </h3>
                  
<p>These functions convert between an object and an array of key-value pairs. If <code>to_entries</code> is passed an object, then for each <code>k: v</code> entry in the input, the output array includes <code>{&quot;key&quot;: k, &quot;value&quot;: v}</code>.</p>

<p><code>from_entries</code> does the opposite conversion, and <code>with_entries(foo)</code> is a shorthand for <code>to_entries |
map(foo) | from_entries</code>, useful for doing some operation to all keys and values of an object. <code>from_entries</code> accepts key, Key, Name, value and Value as keys.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example19">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example19" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'to_entries'</td></tr>
                            <tr><th>Input</th><td>{&quot;a&quot;: 1, &quot;b&quot;: 2}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[{&quot;key&quot;:&quot;a&quot;, &quot;value&quot;:1}, {&quot;key&quot;:&quot;b&quot;, &quot;value&quot;:2}]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'from_entries'</td></tr>
                            <tr><th>Input</th><td>[{&quot;key&quot;:&quot;a&quot;, &quot;value&quot;:1}, {&quot;key&quot;:&quot;b&quot;, &quot;value&quot;:2}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;a&quot;: 1, &quot;b&quot;: 2}</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'with_entries(.key |= &quot;KEY_&quot; + .)'</td></tr>
                            <tr><th>Input</th><td>{&quot;a&quot;: 1, &quot;b&quot;: 2}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;KEY_a&quot;: 1, &quot;KEY_b&quot;: 2}</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="select(boolean_expression)">
                  <h3>
                    
<code>select(boolean_expression)</code>

                    
                  </h3>
                  
<p>The function <code>select(foo)</code> produces its input unchanged if <code>foo</code> returns true for that input, and produces no output otherwise.</p>

<p>It’s useful for filtering lists: <code>[1,2,3] | map(select(. &gt;= 2))</code> will give you <code>[2,3]</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example20">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example20" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'map(select(. &gt;= 2))'</td></tr>
                            <tr><th>Input</th><td>[1,5,3,0,7]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[5,3,7]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[] | select(.id == &quot;second&quot;)'</td></tr>
                            <tr><th>Input</th><td>[{&quot;id&quot;: &quot;first&quot;, &quot;val&quot;: 1}, {&quot;id&quot;: &quot;second&quot;, &quot;val&quot;: 2}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;id&quot;: &quot;second&quot;, &quot;val&quot;: 2}</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="arrays,objects,iterables,booleans,numbers,normals,finites,strings,nulls,values,scalars">
                  <h3>
                    
<code>arrays</code>, <code>objects</code>, <code>iterables</code>, <code>booleans</code>, <code>numbers</code>, <code>normals</code>, <code>finites</code>, <code>strings</code>, <code>nulls</code>, <code>values</code>, <code>scalars</code>

                    
                  </h3>
                  
<p>These built-ins select only inputs that are arrays, objects, iterables (arrays or objects), booleans, numbers, normal numbers, finite numbers, strings, null, non-null values, and non-iterables, respectively.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example21">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example21" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[]|numbers'</td></tr>
                            <tr><th>Input</th><td>[[],{},1,&quot;foo&quot;,null,true,false]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>1</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="empty">
                  <h3>
                    
<code>empty</code>

                    
                  </h3>
                  
<p><code>empty</code> returns no results. None at all. Not even <code>null</code>.</p>

<p>It’s useful on occasion. You’ll know if you need it :)</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example22">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example22" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '1, empty, 2'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>1</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>2</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[1,2,empty,3]'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1,2,3]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="error(message)">
                  <h3>
                    
<code>error(message)</code>

                    
                  </h3>
                  
<p>Produces an error, just like <code>.a</code> applied to values other than null and objects would, but with the given message as the error’s value.</p>


                  
                </section>
              
                <section id="$__loc__">
                  <h3>
                    
<code>$__loc__</code>

                    
                  </h3>
                  
<p>Produces an object with a “file” key and a “line” key, with the filename and line number where <code>$__loc__</code> occurs, as values.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example23">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example23" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'try error(&quot;\($__loc__)&quot;) catch .'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;{\&quot;file\&quot;:\&quot;&lt;top-level&gt;\&quot;,\&quot;line\&quot;:1}&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="map(x),map_values(x)">
                  <h3>
                    
<code>map(x)</code>, <code>map_values(x)</code>

                    
                  </h3>
                  
<p>For any filter <code>x</code>, <code>map(x)</code> will run that filter for each element of the input array, and produce the outputs a new array. <code>map(.+1)</code> will increment each element of an array of numbers.</p>

<p>Similarly, <code>map_values(x)</code> will run that filter for each element, but it will return an object when an object is passed.</p>

<p><code>map(x)</code> is equivalent to <code>[.[] | x]</code>. In fact, this is how it’s defined. Similarly, <code>map_values(x)</code> is defined as <code>.[] |= x</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example24">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example24" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'map(.+1)'</td></tr>
                            <tr><th>Input</th><td>[1,2,3]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[2,3,4]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'map_values(.+1)'</td></tr>
                            <tr><th>Input</th><td>{&quot;a&quot;: 1, &quot;b&quot;: 2, &quot;c&quot;: 3}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;a&quot;: 2, &quot;b&quot;: 3, &quot;c&quot;: 4}</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="paths,paths(node_filter),leaf_paths">
                  <h3>
                    
<code>paths</code>, <code>paths(node_filter)</code>, <code>leaf_paths</code>

                    
                  </h3>
                  
<p><code>paths</code> outputs the paths to all the elements in its input (except it does not output the empty list, representing . itself).</p>

<p><code>paths(f)</code> outputs the paths to any values for which <code>f</code> is true. That is, <code>paths(numbers)</code> outputs the paths to all numeric values.</p>

<p><code>leaf_paths</code> is an alias of <code>paths(scalars)</code>; <code>leaf_paths</code> is <em>deprecated</em> and will be removed in the next major release.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example25">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example25" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[paths]'</td></tr>
                            <tr><th>Input</th><td>[1,[[],{&quot;a&quot;:2}]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[[0],[1],[1,0],[1,1],[1,1,&quot;a&quot;]]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[paths(scalars)]'</td></tr>
                            <tr><th>Input</th><td>[1,[[],{&quot;a&quot;:2}]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[[0],[1,1,&quot;a&quot;]]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="add">
                  <h3>
                    
<code>add</code>

                    
                  </h3>
                  
<p>The filter <code>add</code> takes as input an array, and produces as output the elements of the array added together. This might mean summed, concatenated or merged depending on the types of the elements of the input array - the rules are the same as those for the <code>+</code> operator (described above).</p>

<p>If the input is an empty array, <code>add</code> returns <code>null</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example26">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example26" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'add'</td></tr>
                            <tr><th>Input</th><td>[&quot;a&quot;,&quot;b&quot;,&quot;c&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;abc&quot;</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'add'</td></tr>
                            <tr><th>Input</th><td>[1, 2, 3]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>6</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'add'</td></tr>
                            <tr><th>Input</th><td>[]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>null</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="any,any(condition),any(generator;condition)">
                  <h3>
                    
<code>any</code>, <code>any(condition)</code>, <code>any(generator; condition)</code>

                    
                  </h3>
                  
<p>The filter <code>any</code> takes as input an array of boolean values, and produces <code>true</code> as output if any of the elements of the array are <code>true</code>.</p>

<p>If the input is an empty array, <code>any</code> returns <code>false</code>.</p>

<p>The <code>any(condition)</code> form applies the given condition to the elements of the input array.</p>

<p>The <code>any(generator; condition)</code> form applies the given condition to all the outputs of the given generator.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example27">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example27" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'any'</td></tr>
                            <tr><th>Input</th><td>[true, false]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'any'</td></tr>
                            <tr><th>Input</th><td>[false, false]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'any'</td></tr>
                            <tr><th>Input</th><td>[]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="all,all(condition),all(generator;condition)">
                  <h3>
                    
<code>all</code>, <code>all(condition)</code>, <code>all(generator; condition)</code>

                    
                  </h3>
                  
<p>The filter <code>all</code> takes as input an array of boolean values, and produces <code>true</code> as output if all of the elements of the array are <code>true</code>.</p>

<p>The <code>all(condition)</code> form applies the given condition to the elements of the input array.</p>

<p>The <code>all(generator; condition)</code> form applies the given condition to all the outputs of the given generator.</p>

<p>If the input is an empty array, <code>all</code> returns <code>true</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example28">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example28" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'all'</td></tr>
                            <tr><th>Input</th><td>[true, false]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'all'</td></tr>
                            <tr><th>Input</th><td>[true, true]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'all'</td></tr>
                            <tr><th>Input</th><td>[]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="flatten,flatten(depth)">
                  <h3>
                    
<code>flatten</code>, <code>flatten(depth)</code>

                    
                  </h3>
                  
<p>The filter <code>flatten</code> takes as input an array of nested arrays, and produces a flat array in which all arrays inside the original array have been recursively replaced by their values. You can pass an argument to it to specify how many levels of nesting to flatten.</p>

<p><code>flatten(2)</code> is like <code>flatten</code>, but going only up to two levels deep.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example29">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example29" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'flatten'</td></tr>
                            <tr><th>Input</th><td>[1, [2], [[3]]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1, 2, 3]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'flatten(1)'</td></tr>
                            <tr><th>Input</th><td>[1, [2], [[3]]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1, 2, [3]]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'flatten'</td></tr>
                            <tr><th>Input</th><td>[[]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'flatten'</td></tr>
                            <tr><th>Input</th><td>[{&quot;foo&quot;: &quot;bar&quot;}, [{&quot;foo&quot;: &quot;baz&quot;}]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[{&quot;foo&quot;: &quot;bar&quot;}, {&quot;foo&quot;: &quot;baz&quot;}]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="range(upto),range(from;upto)range(from;upto;by)">
                  <h3>
                    
<code>range(upto)</code>, <code>range(from;upto)</code> <code>range(from;upto;by)</code>

                    
                  </h3>
                  
<p>The <code>range</code> function produces a range of numbers. <code>range(4;10)</code> produces 6 numbers, from 4 (inclusive) to 10 (exclusive). The numbers are produced as separate outputs. Use <code>[range(4;10)]</code> to get a range as an array.</p>

<p>The one argument form generates numbers from 0 to the given number, with an increment of 1.</p>

<p>The two argument form generates numbers from <code>from</code> to <code>upto</code> with an increment of 1.</p>

<p>The three argument form generates numbers <code>from</code> to <code>upto</code> with an increment of <code>by</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example30">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example30" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'range(2;4)'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>2</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>3</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[range(2;4)]'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[2,3]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[range(4)]'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[0,1,2,3]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[range(0;10;3)]'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[0,3,6,9]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[range(0;10;-1)]'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[range(0;-5;-1)]'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[0,-1,-2,-3,-4]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="floor">
                  <h3>
                    
<code>floor</code>

                    
                  </h3>
                  
<p>The <code>floor</code> function returns the floor of its numeric input.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example31">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example31" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'floor'</td></tr>
                            <tr><th>Input</th><td>3.14159</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>3</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="sqrt">
                  <h3>
                    
<code>sqrt</code>

                    
                  </h3>
                  
<p>The <code>sqrt</code> function returns the square root of its numeric input.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example32">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example32" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'sqrt'</td></tr>
                            <tr><th>Input</th><td>9</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>3</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="tonumber">
                  <h3>
                    
<code>tonumber</code>

                    
                  </h3>
                  
<p>The <code>tonumber</code> function parses its input as a number. It will convert correctly-formatted strings to their numeric equivalent, leave numbers alone, and give an error on all other input.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example33">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example33" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[] | tonumber'</td></tr>
                            <tr><th>Input</th><td>[1, &quot;1&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>1</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>1</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="tostring">
                  <h3>
                    
<code>tostring</code>

                    
                  </h3>
                  
<p>The <code>tostring</code> function prints its input as a string. Strings are left unchanged, and all other values are JSON-encoded.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example34">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example34" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[] | tostring'</td></tr>
                            <tr><th>Input</th><td>[1, &quot;1&quot;, [1]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;1&quot;</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>&quot;1&quot;</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>&quot;[1]&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="type">
                  <h3>
                    
<code>type</code>

                    
                  </h3>
                  
<p>The <code>type</code> function returns the type of its argument as a string, which is one of null, boolean, number, string, array or object.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example35">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example35" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'map(type)'</td></tr>
                            <tr><th>Input</th><td>[0, false, [], {}, null, &quot;hello&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;number&quot;, &quot;boolean&quot;, &quot;array&quot;, &quot;object&quot;, &quot;null&quot;, &quot;string&quot;]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="infinite,nan,isinfinite,isnan,isfinite,isnormal">
                  <h3>
                    
<code>infinite</code>, <code>nan</code>, <code>isinfinite</code>, <code>isnan</code>, <code>isfinite</code>, <code>isnormal</code>

                    
                  </h3>
                  
<p>Some arithmetic operations can yield infinities and “not a number” (NaN) values. The <code>isinfinite</code> builtin returns <code>true</code> if its input is infinite. The <code>isnan</code> builtin returns <code>true</code> if its input is a NaN. The <code>infinite</code> builtin returns a positive infinite value. The <code>nan</code> builtin returns a NaN. The <code>isnormal</code> builtin returns true if its input is a normal number.</p>

<p>Note that division by zero raises an error.</p>

<p>Currently most arithmetic operations operating on infinities, NaNs, and sub-normals do not raise errors.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example36">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example36" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[] | (infinite * .) &lt; 0'</td></tr>
                            <tr><th>Input</th><td>[-1, 1]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'infinite, nan | type'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;number&quot;</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>&quot;number&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="sort,sort_by(path_expression)">
                  <h3>
                    
<code>sort, sort_by(path_expression)</code>

                    
                  </h3>
                  
<p>The <code>sort</code> functions sorts its input, which must be an array. Values are sorted in the following order:</p>

<ul>
<li><code>null</code></li>

<li><code>false</code></li>

<li><code>true</code></li>

<li>numbers</li>

<li>strings, in alphabetical order (by unicode codepoint value)</li>

<li>arrays, in lexical order</li>

<li>objects</li>
</ul>

<p>The ordering for objects is a little complex: first they’re compared by comparing their sets of keys (as arrays in sorted order), and if their keys are equal then the values are compared key by key.</p>

<p><code>sort</code> may be used to sort by a particular field of an object, or by applying any jq filter.</p>

<p><code>sort_by(foo)</code> compares two elements by comparing the result of <code>foo</code> on each element.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example37">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example37" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'sort'</td></tr>
                            <tr><th>Input</th><td>[8,3,null,6]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[null,3,6,8]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'sort_by(.foo)'</td></tr>
                            <tr><th>Input</th><td>[{&quot;foo&quot;:4, &quot;bar&quot;:10}, {&quot;foo&quot;:3, &quot;bar&quot;:100}, {&quot;foo&quot;:2, &quot;bar&quot;:1}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[{&quot;foo&quot;:2, &quot;bar&quot;:1}, {&quot;foo&quot;:3, &quot;bar&quot;:100}, {&quot;foo&quot;:4, &quot;bar&quot;:10}]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="group_by(path_expression)">
                  <h3>
                    
<code>group_by(path_expression)</code>

                    
                  </h3>
                  
<p><code>group_by(.foo)</code> takes as input an array, groups the elements having the same <code>.foo</code> field into separate arrays, and produces all of these arrays as elements of a larger array, sorted by the value of the <code>.foo</code> field.</p>

<p>Any jq expression, not just a field access, may be used in place of <code>.foo</code>. The sorting order is the same as described in the <code>sort</code> function above.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example38">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example38" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'group_by(.foo)'</td></tr>
                            <tr><th>Input</th><td>[{&quot;foo&quot;:1, &quot;bar&quot;:10}, {&quot;foo&quot;:3, &quot;bar&quot;:100}, {&quot;foo&quot;:1, &quot;bar&quot;:1}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[[{&quot;foo&quot;:1, &quot;bar&quot;:10}, {&quot;foo&quot;:1, &quot;bar&quot;:1}], [{&quot;foo&quot;:3, &quot;bar&quot;:100}]]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="min,max,min_by(path_exp),max_by(path_exp)">
                  <h3>
                    
<code>min</code>, <code>max</code>, <code>min_by(path_exp)</code>, <code>max_by(path_exp)</code>

                    
                  </h3>
                  
<p>Find the minimum or maximum element of the input array.</p>

<p>The <code>min_by(path_exp)</code> and <code>max_by(path_exp)</code> functions allow you to specify a particular field or property to examine, e.g. <code>min_by(.foo)</code> finds the object with the smallest <code>foo</code> field.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example39">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example39" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'min'</td></tr>
                            <tr><th>Input</th><td>[5,4,2,7]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>2</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'max_by(.foo)'</td></tr>
                            <tr><th>Input</th><td>[{&quot;foo&quot;:1, &quot;bar&quot;:14}, {&quot;foo&quot;:2, &quot;bar&quot;:3}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;foo&quot;:2, &quot;bar&quot;:3}</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="unique,unique_by(path_exp)">
                  <h3>
                    
<code>unique</code>, <code>unique_by(path_exp)</code>

                    
                  </h3>
                  
<p>The <code>unique</code> function takes as input an array and produces an array of the same elements, in sorted order, with duplicates removed.</p>

<p>The <code>unique_by(path_exp)</code> function will keep only one element for each value obtained by applying the argument. Think of it as making an array by taking one element out of every group produced by <code>group</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example40">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example40" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'unique'</td></tr>
                            <tr><th>Input</th><td>[1,2,5,3,5,3,1,3]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1,2,3,5]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'unique_by(.foo)'</td></tr>
                            <tr><th>Input</th><td>[{&quot;foo&quot;: 1, &quot;bar&quot;: 2}, {&quot;foo&quot;: 1, &quot;bar&quot;: 3}, {&quot;foo&quot;: 4, &quot;bar&quot;: 5}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[{&quot;foo&quot;: 1, &quot;bar&quot;: 2}, {&quot;foo&quot;: 4, &quot;bar&quot;: 5}]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'unique_by(length)'</td></tr>
                            <tr><th>Input</th><td>[&quot;chunky&quot;, &quot;bacon&quot;, &quot;kitten&quot;, &quot;cicada&quot;, &quot;asparagus&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;bacon&quot;, &quot;chunky&quot;, &quot;asparagus&quot;]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="reverse">
                  <h3>
                    
<code>reverse</code>

                    
                  </h3>
                  
<p>This function reverses an array.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example41">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example41" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'reverse'</td></tr>
                            <tr><th>Input</th><td>[1,2,3,4]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[4,3,2,1]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="contains(element)">
                  <h3>
                    
<code>contains(element)</code>

                    
                  </h3>
                  
<p>The filter <code>contains(b)</code> will produce true if b is completely contained within the input. A string B is contained in a string A if B is a substring of A. An array B is contained in an array A if all elements in B are contained in any element in A. An object B is contained in object A if all of the values in B are contained in the value in A with the same key. All other types are assumed to be contained in each other if they are equal.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example42">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example42" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'contains(&quot;bar&quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;foobar&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'contains([&quot;baz&quot;, &quot;bar&quot;])'</td></tr>
                            <tr><th>Input</th><td>[&quot;foobar&quot;, &quot;foobaz&quot;, &quot;blarp&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'contains([&quot;bazzzzz&quot;, &quot;bar&quot;])'</td></tr>
                            <tr><th>Input</th><td>[&quot;foobar&quot;, &quot;foobaz&quot;, &quot;blarp&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'contains({foo: 12, bar: [{barp: 12}]})'</td></tr>
                            <tr><th>Input</th><td>{&quot;foo&quot;: 12, &quot;bar&quot;:[1,2,{&quot;barp&quot;:12, &quot;blip&quot;:13}]}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'contains({foo: 12, bar: [{barp: 15}]})'</td></tr>
                            <tr><th>Input</th><td>{&quot;foo&quot;: 12, &quot;bar&quot;:[1,2,{&quot;barp&quot;:12, &quot;blip&quot;:13}]}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="indices(s)">
                  <h3>
                    
<code>indices(s)</code>

                    
                  </h3>
                  
<p>Outputs an array containing the indices in <code>.</code> where <code>s</code> occurs. The input may be an array, in which case if <code>s</code> is an array then the indices output will be those where all elements in <code>.</code> match those of <code>s</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example43">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example43" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'indices(&quot;, &quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;a,b, cd, efg, hijk&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[3,7,12]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'indices(1)'</td></tr>
                            <tr><th>Input</th><td>[0,1,2,1,3,1,4]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1,3,5]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'indices([1,2])'</td></tr>
                            <tr><th>Input</th><td>[0,1,2,3,1,4,2,5,1,2,6,7]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1,8]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="index(s),rindex(s)">
                  <h3>
                    
<code>index(s)</code>, <code>rindex(s)</code>

                    
                  </h3>
                  
<p>Outputs the index of the first (<code>index</code>) or last (<code>rindex</code>) occurrence of <code>s</code> in the input.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example44">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example44" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'index(&quot;, &quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;a,b, cd, efg, hijk&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>3</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'rindex(&quot;, &quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;a,b, cd, efg, hijk&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>12</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="inside">
                  <h3>
                    
<code>inside</code>

                    
                  </h3>
                  
<p>The filter <code>inside(b)</code> will produce true if the input is completely contained within b. It is, essentially, an inversed version of <code>contains</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example45">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example45" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'inside(&quot;foobar&quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;bar&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'inside([&quot;foobar&quot;, &quot;foobaz&quot;, &quot;blarp&quot;])'</td></tr>
                            <tr><th>Input</th><td>[&quot;baz&quot;, &quot;bar&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'inside([&quot;foobar&quot;, &quot;foobaz&quot;, &quot;blarp&quot;])'</td></tr>
                            <tr><th>Input</th><td>[&quot;bazzzzz&quot;, &quot;bar&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'inside({&quot;foo&quot;: 12, &quot;bar&quot;:[1,2,{&quot;barp&quot;:12, &quot;blip&quot;:13}]})'</td></tr>
                            <tr><th>Input</th><td>{&quot;foo&quot;: 12, &quot;bar&quot;: [{&quot;barp&quot;: 12}]}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'inside({&quot;foo&quot;: 12, &quot;bar&quot;:[1,2,{&quot;barp&quot;:12, &quot;blip&quot;:13}]})'</td></tr>
                            <tr><th>Input</th><td>{&quot;foo&quot;: 12, &quot;bar&quot;: [{&quot;barp&quot;: 15}]}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="startswith(str)">
                  <h3>
                    
<code>startswith(str)</code>

                    
                  </h3>
                  
<p>Outputs <code>true</code> if . starts with the given string argument.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example46">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example46" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.[]|startswith(&quot;foo&quot;)]'</td></tr>
                            <tr><th>Input</th><td>[&quot;fo&quot;, &quot;foo&quot;, &quot;barfoo&quot;, &quot;foobar&quot;, &quot;barfoob&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[false, true, false, true, false]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="endswith(str)">
                  <h3>
                    
<code>endswith(str)</code>

                    
                  </h3>
                  
<p>Outputs <code>true</code> if . ends with the given string argument.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example47">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example47" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.[]|endswith(&quot;foo&quot;)]'</td></tr>
                            <tr><th>Input</th><td>[&quot;foobar&quot;, &quot;barfoo&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[false, true]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="combinations,combinations(n)">
                  <h3>
                    
<code>combinations</code>, <code>combinations(n)</code>

                    
                  </h3>
                  
<p>Outputs all combinations of the elements of the arrays in the input array. If given an argument <code>n</code>, it outputs all combinations of <code>n</code> repetitions of the input array.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example48">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example48" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'combinations'</td></tr>
                            <tr><th>Input</th><td>[[1,2], [3, 4]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1, 3]</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>[1, 4]</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>[2, 3]</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>[2, 4]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'combinations(2)'</td></tr>
                            <tr><th>Input</th><td>[0, 1]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[0, 0]</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>[0, 1]</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>[1, 0]</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>[1, 1]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="ltrimstr(str)">
                  <h3>
                    
<code>ltrimstr(str)</code>

                    
                  </h3>
                  
<p>Outputs its input with the given prefix string removed, if it starts with it.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example49">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example49" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.[]|ltrimstr(&quot;foo&quot;)]'</td></tr>
                            <tr><th>Input</th><td>[&quot;fo&quot;, &quot;foo&quot;, &quot;barfoo&quot;, &quot;foobar&quot;, &quot;afoo&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;fo&quot;,&quot;&quot;,&quot;barfoo&quot;,&quot;bar&quot;,&quot;afoo&quot;]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="rtrimstr(str)">
                  <h3>
                    
<code>rtrimstr(str)</code>

                    
                  </h3>
                  
<p>Outputs its input with the given suffix string removed, if it ends with it.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example50">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example50" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.[]|rtrimstr(&quot;foo&quot;)]'</td></tr>
                            <tr><th>Input</th><td>[&quot;fo&quot;, &quot;foo&quot;, &quot;barfoo&quot;, &quot;foobar&quot;, &quot;foob&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;fo&quot;,&quot;&quot;,&quot;bar&quot;,&quot;foobar&quot;,&quot;foob&quot;]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="explode">
                  <h3>
                    
<code>explode</code>

                    
                  </h3>
                  
<p>Converts an input string into an array of the string’s codepoint numbers.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example51">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example51" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'explode'</td></tr>
                            <tr><th>Input</th><td>&quot;foobar&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[102,111,111,98,97,114]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="implode">
                  <h3>
                    
<code>implode</code>

                    
                  </h3>
                  
<p>The inverse of explode.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example52">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example52" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'implode'</td></tr>
                            <tr><th>Input</th><td>[65, 66, 67]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;ABC&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="split">
                  <h3>
                    
<code>split</code>

                    
                  </h3>
                  
<p>Splits an input string on the separator argument.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example53">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example53" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'split(&quot;, &quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;a, b,c,d, e, &quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;a&quot;,&quot;b,c,d&quot;,&quot;e&quot;,&quot;&quot;]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="join(str)">
                  <h3>
                    
<code>join(str)</code>

                    
                  </h3>
                  
<p>Joins the array of elements given as input, using the argument as separator. It is the inverse of <code>split</code>: that is, running <code>split(&quot;foo&quot;) | join(&quot;foo&quot;)</code> over any input string returns said input string.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example54">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example54" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'join(&quot;, &quot;)'</td></tr>
                            <tr><th>Input</th><td>[&quot;a&quot;,&quot;b,c,d&quot;,&quot;e&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;a, b,c,d, e&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="ascii_downcase,ascii_upcase">
                  <h3>
                    
<code>ascii_downcase</code>, <code>ascii_upcase</code>

                    
                  </h3>
                  
<p>Emit a copy of the input string with its alphabetic characters (a-z and A-Z) converted to the specified case.</p>


                  
                </section>
              
                <section id="while(cond;update)">
                  <h3>
                    
<code>while(cond; update)</code>

                    
                  </h3>
                  
<p>The <code>while(cond; update)</code> function allows you to repeatedly apply an update to <code>.</code> until <code>cond</code> is false.</p>

<p>Note that <code>while(cond; update)</code> is internally defined as a recursive jq function. Recursive calls within <code>while</code> will not consume additional memory if <code>update</code> produces at most one output for each input. See advanced topics below.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example55">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example55" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[while(.&lt;100; .*2)]'</td></tr>
                            <tr><th>Input</th><td>1</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1,2,4,8,16,32,64]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="until(cond;next)">
                  <h3>
                    
<code>until(cond; next)</code>

                    
                  </h3>
                  
<p>The <code>until(cond; next)</code> function allows you to repeatedly apply the expression <code>next</code>, initially to <code>.</code> then to its own output, until <code>cond</code> is true. For example, this can be used to implement a factorial function (see below).</p>

<p>Note that <code>until(cond; next)</code> is internally defined as a recursive jq function. Recursive calls within <code>until()</code> will not consume additional memory if <code>next</code> produces at most one output for each input. See advanced topics below.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example56">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example56" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.,1]|until(.[0] &lt; 1; [.[0] - 1, .[1] * .[0]])|.[1]'</td></tr>
                            <tr><th>Input</th><td>4</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>24</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="recurse(f),recurse,recurse(f;condition),recurse_down">
                  <h3>
                    
<code>recurse(f)</code>, <code>recurse</code>, <code>recurse(f; condition)</code>, <code>recurse_down</code>

                    
                  </h3>
                  
<p>The <code>recurse(f)</code> function allows you to search through a recursive structure, and extract interesting data from all levels. Suppose your input represents a filesystem:</p>

<pre><code>{&quot;name&quot;: &quot;/&quot;, &quot;children&quot;: [
  {&quot;name&quot;: &quot;/bin&quot;, &quot;children&quot;: [
    {&quot;name&quot;: &quot;/bin/ls&quot;, &quot;children&quot;: []},
    {&quot;name&quot;: &quot;/bin/sh&quot;, &quot;children&quot;: []}]},
  {&quot;name&quot;: &quot;/home&quot;, &quot;children&quot;: [
    {&quot;name&quot;: &quot;/home/stephen&quot;, &quot;children&quot;: [
      {&quot;name&quot;: &quot;/home/stephen/jq&quot;, &quot;children&quot;: []}]}]}]}</code></pre>

<p>Now suppose you want to extract all of the filenames present. You need to retrieve <code>.name</code>, <code>.children[].name</code>, <code>.children[].children[].name</code>, and so on. You can do this with:</p>

<pre><code>recurse(.children[]) | .name</code></pre>

<p>When called without an argument, <code>recurse</code> is equivalent to <code>recurse(.[]?)</code>.</p>

<p><code>recurse(f)</code> is identical to <code>recurse(f; . != null)</code> and can be used without concerns about recursion depth.</p>

<p><code>recurse(f; condition)</code> is a generator which begins by emitting . and then emits in turn .|f, .|f|f, .|f|f|f, … so long as the computed value satisfies the condition. For example, to generate all the integers, at least in principle, one could write <code>recurse(.+1; true)</code>.</p>

<p>For legacy reasons, <code>recurse_down</code> exists as an alias to calling <code>recurse</code> without arguments. This alias is considered <em>deprecated</em> and will be removed in the next major release.</p>

<p>The recursive calls in <code>recurse</code> will not consume additional memory whenever <code>f</code> produces at most a single output for each input.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example57">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example57" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'recurse(.foo[])'</td></tr>
                            <tr><th>Input</th><td>{&quot;foo&quot;:[{&quot;foo&quot;: []}, {&quot;foo&quot;:[{&quot;foo&quot;:[]}]}]}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;foo&quot;:[{&quot;foo&quot;:[]},{&quot;foo&quot;:[{&quot;foo&quot;:[]}]}]}</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>{&quot;foo&quot;:[]}</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>{&quot;foo&quot;:[{&quot;foo&quot;:[]}]}</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>{&quot;foo&quot;:[]}</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'recurse'</td></tr>
                            <tr><th>Input</th><td>{&quot;a&quot;:0,&quot;b&quot;:[1]}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;a&quot;:0,&quot;b&quot;:[1]}</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>0</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>[1]</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>1</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'recurse(. * .; . &lt; 20)'</td></tr>
                            <tr><th>Input</th><td>2</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>2</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>4</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>16</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="..">
                  <h3>
                    
<code>..</code>

                    
                  </h3>
                  
<p>Short-hand for <code>recurse</code> without arguments. This is intended to resemble the XPath <code>//</code> operator. Note that <code>..a</code> does not work; use <code>..|a</code> instead. In the example below we use <code>..|.a?</code> to find all the values of object keys “a” in any object found “below” <code>.</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example58">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example58" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '..|.a?'</td></tr>
                            <tr><th>Input</th><td>[[{&quot;a&quot;:1}]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>1</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="env">
                  <h3>
                    
<code>env</code>

                    
                  </h3>
                  
<p>Outputs an object representing jq’s environment.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example59">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example59" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'env.PAGER'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;less&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="transpose">
                  <h3>
                    
<code>transpose</code>

                    
                  </h3>
                  
<p>Transpose a possibly jagged matrix (an array of arrays). Rows are padded with nulls so the result is always rectangular.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example60">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example60" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'transpose'</td></tr>
                            <tr><th>Input</th><td>[[1], [2,3]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[[1,2],[null,3]]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="bsearch(x)">
                  <h3>
                    
<code>bsearch(x)</code>

                    
                  </h3>
                  
<p>bsearch(x) conducts a binary search for x in the input array. If the input is sorted and contains x, then bsearch(x) will return its index in the array; otherwise, if the array is sorted, it will return (-1 - ix) where ix is an insertion point such that the array would still be sorted after the insertion of x at ix. If the array is not sorted, bsearch(x) will return an integer that is probably of no interest.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example61">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example61" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'bsearch(0)'</td></tr>
                            <tr><th>Input</th><td>[0,1]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>0</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'bsearch(0)'</td></tr>
                            <tr><th>Input</th><td>[1,2,3]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>-1</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'bsearch(4) as $ix | if $ix &lt; 0 then .[-(1+$ix)] = 4 else . end'</td></tr>
                            <tr><th>Input</th><td>[1,2,3]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1,2,3,4]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="Stringinterpolation-\(foo)">
                  <h3>
                    
String interpolation - <code>\(foo)</code>

                    
                  </h3>
                  
<p>Inside a string, you can put an expression inside parens after a backslash. Whatever the expression returns will be interpolated into the string.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example62">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example62" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '&quot;The input was \(.), which is one less than \(.+1)&quot;'</td></tr>
                            <tr><th>Input</th><td>42</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;The input was 42, which is one less than 43&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="Convertto/fromJSON">
                  <h3>
                    
Convert to/from JSON

                    
                  </h3>
                  
<p>The <code>tojson</code> and <code>fromjson</code> builtins dump values as JSON texts or parse JSON texts into values, respectively. The tojson builtin differs from tostring in that tostring returns strings unmodified, while tojson encodes strings as JSON strings.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example63">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example63" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.[]|tostring]'</td></tr>
                            <tr><th>Input</th><td>[1, &quot;foo&quot;, [&quot;foo&quot;]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;1&quot;,&quot;foo&quot;,&quot;[\&quot;foo\&quot;]&quot;]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.[]|tojson]'</td></tr>
                            <tr><th>Input</th><td>[1, &quot;foo&quot;, [&quot;foo&quot;]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[&quot;1&quot;,&quot;\&quot;foo\&quot;&quot;,&quot;[\&quot;foo\&quot;]&quot;]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.[]|tojson|fromjson]'</td></tr>
                            <tr><th>Input</th><td>[1, &quot;foo&quot;, [&quot;foo&quot;]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1,&quot;foo&quot;,[&quot;foo&quot;]]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="Formatstringsandescaping">
                  <h3>
                    
Format strings and escaping

                    
                  </h3>
                  
<p>The <code>@foo</code> syntax is used to format and escape strings, which is useful for building URLs, documents in a language like HTML or XML, and so forth. <code>@foo</code> can be used as a filter on its own, the possible escapings are:</p>

<ul>
<li>
<p><code>@text</code>:</p>

<p>Calls <code>tostring</code>, see that function for details.</p>
</li>

<li>
<p><code>@json</code>:</p>

<p>Serializes the input as JSON.</p>
</li>

<li>
<p><code>@html</code>:</p>

<p>Applies HTML/XML escaping, by mapping the characters <code>&lt;&gt;&amp;'&quot;</code> to their entity equivalents <code>&amp;lt;</code>, <code>&amp;gt;</code>, <code>&amp;amp;</code>, <code>&amp;apos;</code>, <code>&amp;quot;</code>.</p>
</li>

<li>
<p><code>@uri</code>:</p>

<p>Applies percent-encoding, by mapping all reserved URI characters to a <code>%XX</code> sequence.</p>
</li>

<li>
<p><code>@csv</code>:</p>

<p>The input must be an array, and it is rendered as CSV with double quotes for strings, and quotes escaped by repetition.</p>
</li>

<li>
<p><code>@tsv</code>:</p>

<p>The input must be an array, and it is rendered as TSV (tab-separated values). Each input array will be printed as a single line. Fields are separated by a single tab (ascii <code>0x09</code>). Input characters line-feed (ascii <code>0x0a</code>), carriage-return (ascii <code>0x0d</code>), tab (ascii <code>0x09</code>) and backslash (ascii <code>0x5c</code>) will be output as escape sequences <code>\n</code>, <code>\r</code>, <code>\t</code>, <code>\\</code> respectively.</p>
</li>

<li>
<p><code>@sh</code>:</p>

<p>The input is escaped suitable for use in a command-line for a POSIX shell. If the input is an array, the output will be a series of space-separated strings.</p>
</li>

<li>
<p><code>@base64</code>:</p>

<p>The input is converted to base64 as specified by RFC 4648.</p>
</li>
</ul>

<p>This syntax can be combined with string interpolation in a useful way. You can follow a <code>@foo</code> token with a string literal. The contents of the string literal will <em>not</em> be escaped. However, all interpolations made inside that string literal will be escaped. For instance,</p>

<pre><code>@uri &quot;https://www.google.com/search?q=\(.search)&quot;</code></pre>

<p>will produce the following output for the input <code>{&quot;search&quot;:&quot;what is jq?&quot;}</code>:</p>

<pre><code>&quot;https://www.google.com/search?q=what%20is%20jq%3F&quot;</code></pre>

<p>Note that the slashes, question mark, etc. in the URL are not escaped, as they were part of the string literal.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example64">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example64" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '@html'</td></tr>
                            <tr><th>Input</th><td>&quot;This works if x &lt; y&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;This works if x &amp;lt; y&quot;</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '@sh &quot;echo \(.)&quot;'</td></tr>
                            <tr><th>Input</th><td>&quot;O'Hara's Ale&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;echo 'O'\\''Hara'\\''s Ale'&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="Dates">
                  <h3>
                    
Dates

                    
                  </h3>
                  
<p>jq provides some basic date handling functionality, with some high-level and low-level builtins. In all cases these builtins deal exclusively with time in UTC.</p>

<p>The <code>fromdateiso8601</code> builtin parses datetimes in the ISO 8601 format to a number of seconds since the Unix epoch (1970-01-01T00:00:00Z). The <code>todateiso8601</code> builtin does the inverse.</p>

<p>The <code>fromdate</code> builtin parses datetime strings. Currently <code>fromdate</code> only supports ISO 8601 datetime strings, but in the future it will attempt to parse datetime strings in more formats.</p>

<p>The <code>todate</code> builtin is an alias for <code>todateiso8601</code>.</p>

<p>The <code>now</code> builtin outputs the current time, in seconds since the Unix epoch.</p>

<p>Low-level jq interfaces to the C-library time functions are also provided: <code>strptime</code>, <code>strftime</code>, <code>mktime</code>, and <code>gmtime</code>. Refer to your host operating system’s documentation for the format strings used by <code>strptime</code> and <code>strftime</code>. Note: these are not necessarily stable interfaces in jq, particularly as to their localization functionality.</p>

<p>The <code>gmtime</code> builtin consumes a number of seconds since the Unix epoch and outputs a “broken down time” representation of time as an array of numbers representing (in this order): the year, the month (zero-based), the day of the month, the hour of the day, the minute of the hour, the second of the minute, the day of the week, and the day of the year – all one-based unless otherwise stated.</p>

<p>The <code>mktime</code> builtin consumes “broken down time” representations of time output by <code>gmtime</code> and <code>strptime</code>.</p>

<p>The <code>strptime(fmt)</code> builtin parses input strings matching the <code>fmt</code> argument. The output is in the “broken down time” representation consumed by <code>gmtime</code> and output by <code>mktime</code>.</p>

<p>The <code>strftime(fmt)</code> builtin formats a time with the given format.</p>

<p>The format strings for <code>strptime</code> and <code>strftime</code> are described in typical C library documentation. The format string for ISO 8601 datetime is <code>&quot;%Y-%m-%dT%H:%M:%SZ&quot;</code>.</p>

<p>jq may not support some or all of this date functionality on some systems.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example65">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example65" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'fromdate'</td></tr>
                            <tr><th>Input</th><td>&quot;2015-03-05T23:51:47Z&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>1425599507</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'strptime(&quot;%Y-%m-%dT%H:%M:%SZ&quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;2015-03-05T23:51:47Z&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[2015,2,5,23,51,47,4,63]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'strptime(&quot;%Y-%m-%dT%H:%M:%SZ&quot;)|mktime'</td></tr>
                            <tr><th>Input</th><td>&quot;2015-03-05T23:51:47Z&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>1425599507</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
            </section>
          
            <section id="ConditionalsandComparisons">
              <h2>Conditionals and Comparisons</h2>
              
              
                <section id="==,!=">
                  <h3>
                    
<code>==</code>, <code>!=</code>

                    
                  </h3>
                  
<p>The expression ‘a == b’ will produce ‘true’ if the result of a and b are equal (that is, if they represent equivalent JSON documents) and ‘false’ otherwise. In particular, strings are never considered equal to numbers. If you’re coming from Javascript, jq’s == is like Javascript’s === - considering values equal only when they have the same type as well as the same value.</p>

<p>!= is “not equal”, and ‘a != b’ returns the opposite value of ‘a == b’</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example66">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example66" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[] == 1'</td></tr>
                            <tr><th>Input</th><td>[1, 1.0, &quot;1&quot;, &quot;banana&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>true</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>false</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="if-then-else">
                  <h3>
                    
if-then-else

                    
                  </h3>
                  
<p><code>if A then B else C end</code> will act the same as <code>B</code> if <code>A</code> produces a value other than false or null, but act the same as <code>C</code> otherwise.</p>

<p>Checking for false or null is a simpler notion of “truthiness” than is found in Javascript or Python, but it means that you’ll sometimes have to be more explicit about the condition you want: you can’t test whether, e.g. a string is empty using <code>if .name then A else B end</code>, you’ll need something more like <code>if (.name | length) &gt; 0 then A else
B end</code> instead.</p>

<p>If the condition <code>A</code> produces multiple results, then <code>B</code> is evaluated once for each result that is not false or null, and <code>C</code> is evaluated once for each false or null.</p>

<p>More cases can be added to an if using <code>elif A then B</code> syntax.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example67">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example67" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'if . == 0 then
  &quot;zero&quot;
elif . == 1 then
  &quot;one&quot;
else
  &quot;many&quot;
end'</td></tr>
                            <tr><th>Input</th><td>2</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;many&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id=">,>=,<=,<">
                  <h3>
                    
<code>&gt;, &gt;=, &lt;=, &lt;</code>

                    
                  </h3>
                  
<p>The comparison operators <code>&gt;</code>, <code>&gt;=</code>, <code>&lt;=</code>, <code>&lt;</code> return whether their left argument is greater than, greater than or equal to, less than or equal to or less than their right argument (respectively).</p>

<p>The ordering is the same as that described for <code>sort</code>, above.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example68">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example68" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '. &lt; 5'</td></tr>
                            <tr><th>Input</th><td>2</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="and/or/not">
                  <h3>
                    
and/or/not

                    
                  </h3>
                  
<p>jq supports the normal Boolean operators and/or/not. They have the same standard of truth as if expressions - false and null are considered “false values”, and anything else is a “true value”.</p>

<p>If an operand of one of these operators produces multiple results, the operator itself will produce a result for each input.</p>

<p><code>not</code> is in fact a builtin function rather than an operator, so it is called as a filter to which things can be piped rather than with special syntax, as in <code>.foo and .bar |
not</code>.</p>

<p>These three only produce the values “true” and “false”, and so are only useful for genuine Boolean operations, rather than the common Perl/Python/Ruby idiom of “value_that_may_be_null or default”. If you want to use this form of “or”, picking between two values rather than evaluating a condition, see the “//” operator below.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example69">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example69" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '42 and &quot;a string&quot;'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '(true, false) or false'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '(true, true) and (true, false)'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>false</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>true</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>false</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[true, false | not]'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[false, true]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="Alternativeoperator-//">
                  <h3>
                    
Alternative operator - <code>//</code>

                    
                  </h3>
                  
<p>A filter of the form <code>a // b</code> produces the same results as <code>a</code>, if <code>a</code> produces results other than <code>false</code> and <code>null</code>. Otherwise, <code>a // b</code> produces the same results as <code>b</code>.</p>

<p>This is useful for providing defaults: <code>.foo // 1</code> will evaluate to <code>1</code> if there’s no <code>.foo</code> element in the input. It’s similar to how <code>or</code> is sometimes used in Python (jq’s <code>or</code> operator is reserved for strictly Boolean operations).</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example70">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example70" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.foo // 42'</td></tr>
                            <tr><th>Input</th><td>{&quot;foo&quot;: 19}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>19</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.foo // 42'</td></tr>
                            <tr><th>Input</th><td>{}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>42</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="try-catch">
                  <h3>
                    
try-catch

                    
                  </h3>
                  
<p>Errors can be caught by using <code>try EXP catch EXP</code>. The first expression is executed, and if it fails then the second is executed with the error message. The output of the handler, if any, is output as if it had been the output of the expression to try.</p>

<p>The <code>try EXP</code> form uses <code>empty</code> as the exception handler.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example71">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example71" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'try .a catch &quot;. is not an object&quot;'</td></tr>
                            <tr><th>Input</th><td>true</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;. is not an object&quot;</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.[]|try .a]'</td></tr>
                            <tr><th>Input</th><td>[{}, true, {&quot;a&quot;:1}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[null, 1]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'try error(&quot;some exception&quot;) catch .'</td></tr>
                            <tr><th>Input</th><td>true</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>&quot;some exception&quot;</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="Breakingoutofcontrolstructures">
                  <h3>
                    
Breaking out of control structures

                    
                  </h3>
                  
<p>A convenient use of try/catch is to break out of control structures like <code>reduce</code>, <code>foreach</code>, <code>while</code>, and so on.</p>

<p>For example:</p>

<pre><code># Repeat an expression until it raises &quot;break&quot; as an
# error, then stop repeating without re-raising the error.
# But if the error caught is not &quot;break&quot; then re-raise it.
try repeat(exp) catch .==&quot;break&quot; then empty else error;</code></pre>

<p>jq has a syntax for named lexical labels to “break” or “go (back) to”:</p>

<pre><code>label $out | ... break $out ...</code></pre>

<p>The <code>break $label_name</code> expression will cause the program to to act as though the nearest (to the left) <code>label $label_name</code> produced <code>empty</code>.</p>

<p>The relationship between the <code>break</code> and corresponding <code>label</code> is lexical: the label has to be “visible” from the break.</p>

<p>To break out of a <code>reduce</code>, for example:</p>

<pre><code>label $out | reduce .[] as $item (null; if .==false then break $out else ... end)</code></pre>

<p>The following jq program produces a syntax error:</p>

<pre><code>break $out</code></pre>

<p>because no label <code>$out</code> is visible.</p>


                  
                </section>
              
                <section id="?operator">
                  <h3>
                    
<code>?</code> operator

                    
                  </h3>
                  
<p>The <code>?</code> operator, used as <code>EXP?</code>, is shorthand for <code>try EXP</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example72">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example72" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[.[]|(.a)?]'</td></tr>
                            <tr><th>Input</th><td>[{}, true, {&quot;a&quot;:1}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[null, 1]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
            </section>
          
            <section id="RegularexpressionsPCRE">
              <h2>Regular expressions (PCRE)</h2>
              
<p>jq uses the Oniguruma regular expression library, as do php, ruby, TextMate, Sublime Text, etc, so the description here will focus on jq specifics.</p>

<p>The jq regex filters are defined so that they can be used using one of these patterns:</p>

<pre><code>STRING | FILTER( REGEX )
STRING | FILTER( REGEX; FLAGS )
STRING | FILTER( [REGEX] )
STRING | FILTER( [REGEX, FLAGS] )</code></pre>

<p>where:</p>

<ul>
<li>STRING, REGEX and FLAGS are jq strings and subject to jq string interpolation;</li>

<li>REGEX, after string interpolation, should be a valid PCRE regex;</li>

<li>FILTER is one of <code>test</code>, <code>match</code>, or <code>capture</code>, as described below.</li>
</ul>

<p>FLAGS is a string consisting of one of more of the supported flags:</p>

<ul>
<li><code>g</code> - Global search (find all matches, not just the first)</li>

<li><code>i</code> - Case insensitive search</li>

<li><code>m</code> - Multi line mode (‘.’ will match newlines)</li>

<li><code>n</code> - Ignore empty matches</li>

<li><code>p</code> - Both s and m modes are enabled</li>

<li><code>s</code> - Single line mode (‘^’ -&gt; ‘\A’, ‘$’ -&gt; ‘\Z’)</li>

<li><code>l</code> - Find longest possible matches</li>

<li><code>x</code> - Extended regex format (ignore whitespace and comments)</li>
</ul>

<p>To match whitespace in an x pattern use an escape such as \s, e.g.</p>

<ul>
<li>test( “a\sb”, “x” ).</li>
</ul>

<p>Note that certain flags may also be specified within REGEX, e.g.</p>

<ul>
<li>jq -n ‘(“test”, “TEst”, “teST”, “TEST”) | test( “(?i)te(?-i)st” )’</li>
</ul>

<p>evaluates to: true, true, false, false.</p>

              
                <section id="test(val),test(regex;flags)">
                  <h3>
                    
<code>test(val)</code>, <code>test(regex; flags)</code>

                    
                  </h3>
                  
<p>Like <code>match</code>, but does not return match objects, only <code>true</code> or <code>false</code> for whether or not the regex matches the input.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example73">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example73" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'test(&quot;foo&quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;foo&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[] | test(&quot;a b c # spaces are ignored&quot;; &quot;ix&quot;)'</td></tr>
                            <tr><th>Input</th><td>[&quot;xabcd&quot;, &quot;ABC&quot;]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="match(val),match(regex;flags)">
                  <h3>
                    
<code>match(val)</code>, <code>match(regex; flags)</code>

                    
                  </h3>
                  
<p><strong>match</strong> outputs an object for each match it finds. Matches have the following fields:</p>

<ul>
<li><code>offset</code> - offset in UTF-8 codepoints from the beginning of the input</li>

<li><code>length</code> - length in UTF-8 codepoints of the match</li>

<li><code>string</code> - the string that it matched</li>

<li><code>captures</code> - an array of objects representing capturing groups.</li>
</ul>

<p>Capturing group objects have the following fields:</p>

<ul>
<li><code>offset</code> - offset in UTF-8 codepoints from the beginning of the input</li>

<li><code>length</code> - length in UTF-8 codepoints of this capturing group</li>

<li><code>string</code> - the string that was captured</li>

<li><code>name</code> - the name of the capturing group (or <code>null</code> if it was unnamed)</li>
</ul>

<p>Capturing groups that did not match anything return an offset of -1</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example74">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example74" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'match(&quot;(abc)+&quot;; &quot;g&quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;abc abc&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;offset&quot;: 0, &quot;length&quot;: 3, &quot;string&quot;: &quot;abc&quot;, &quot;captures&quot;: [{&quot;offset&quot;: 0, &quot;length&quot;: 3, &quot;string&quot;: &quot;abc&quot;, &quot;name&quot;: null}]}</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>{&quot;offset&quot;: 4, &quot;length&quot;: 3, &quot;string&quot;: &quot;abc&quot;, &quot;captures&quot;: [{&quot;offset&quot;: 4, &quot;length&quot;: 3, &quot;string&quot;: &quot;abc&quot;, &quot;name&quot;: null}]}</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'match(&qquot;foo&quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;foo bar foo&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;offset&quot;: 0, &quot;length&quot;: 3, &quot;string&quot;: &quot;foo&quot;, &quot;captures&quot;: []}</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'match([&quot;foo&quot;, &quot;ig&quot;])'</td></tr>
                            <tr><th>Input</th><td>&quot;foo bar FOO&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;offset&quot;: 0, &quot;length&quot;: 3, &quot;string&quot;: &quot;foo&quot;, &quot;captures&quot;: []}</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>{&quot;offset&quot;: 8, &quot;length&quot;: 3, &quot;string&quot;: &quot;FOO&quot;, &quot;captures&quot;: []}</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'match(&quot;foo (?&lt;bar123&gt;bar)? foo&quot;; &quot;ig&quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;foo bar foo foo  foo&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;offset&quot;: 0, &quot;length&quot;: 11, &quot;string&quot;: &quot;foo bar foo&quot;, &quot;captures&quot;: [{&quot;offset&quot;: 4, &quot;length&quot;: 3, &quot;string&quot;: &quot;bar&quot;, &quot;name&quot;: &quot;bar123&quot;}]}</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>{&quot;offset&quot;: 12, &quot;length&quot;: 8, &quot;string&quot;: &quot;foo  foo&quot;, &quot;captures&quot;: [{&quot;offset&quot;: -1, &quot;length&quot;: 0, &quot;string&quot;: null, &quot;name&quot;: &quot;bar123&quot;}]}</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[ match(&quot;.&quot;; &quot;g&quot;)] | length'</td></tr>
                            <tr><th>Input</th><td>&quot;abc&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>3</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="capture(val),capture(regex;flags)">
                  <h3>
                    
<code>capture(val)</code>, <code>capture(regex; flags)</code>

                    
                  </h3>
                  
<p>Collects the named captures in a JSON object, with the name of each capture as the key, and the matched string as the corresponding value.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example75">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example75" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'capture(&quot;(?&lt;a&gt;[a-z]+)-(?&lt;n&gt;[0-9]+)&quot;)'</td></tr>
                            <tr><th>Input</th><td>&quot;xyzzy-14&quot;</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{ &quot;a&quot;: &quot;xyzzy&quot;, &quot;n&quot;: &quot;14&quot; }</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="scan(regex),scan(regex;flags)">
                  <h3>
                    
<code>scan(regex)</code>, <code>scan(regex; flags)</code>

                    
                  </h3>
                  
<p>Emit a stream of the non-overlapping substrings of the input that match the regex in accordance with the flags, if any have been specified. If there is no match, the stream is empty. To capture all the matches for each input string, use the idiom <code>[ expr ]</code>, e.g. <code>[ scan(regex) ]</code>.</p>


                  
                </section>
              
                <section id="split(regex;flags)">
                  <h3>
                    
<code>split(regex; flags)</code>

                    
                  </h3>
                  
<p>For backwards compatibility, <code>split</code> splits on a string, not a regex.</p>


                  
                </section>
              
                <section id="splits(regex),splits(regex;flags)">
                  <h3>
                    
<code>splits(regex)</code>, <code>splits(regex; flags)</code>

                    
                  </h3>
                  
<p>These provide the same results as their <code>split</code> counterparts, but as a stream instead of an array.</p>


                  
                </section>
              
                <section id="sub(regex;tostring)sub(regex;string;flags)">
                  <h3>
                    
<code>sub(regex; tostring)</code> <code>sub(regex; string; flags)</code>

                    
                  </h3>
                  
<p>Emit the string obtained by replacing the first match of regex in the input string with <code>tostring</code>, after interpolation. <code>tostring</code> should be a jq string, and may contain references to named captures. The named captures are, in effect, presented as a JSON object (as constructed by <code>capture</code>) to <code>tostring</code>, so a reference to a captured variable named “x” would take the form: “(.x)”.</p>


                  
                </section>
              
                <section id="gsub(regex;string),gsub(regex;string;flags)">
                  <h3>
                    
<code>gsub(regex; string)</code>, <code>gsub(regex; string; flags)</code>

                    
                  </h3>
                  
<p><code>gsub</code> is like <code>sub</code> but all the non-overlapping occurrences of the regex are replaced by the string, after interpolation.</p>


                  
                </section>
              
            </section>
          
            <section id="Advancedfeatures">
              <h2>Advanced features</h2>
              
<p>Variables are an absolute necessity in most programming languages, but they’re relegated to an “advanced feature” in jq.</p>

<p>In most languages, variables are the only means of passing around data. If you calculate a value, and you want to use it more than once, you’ll need to store it in a variable. To pass a value to another part of the program, you’ll need that part of the program to define a variable (as a function parameter, object member, or whatever) in which to place the data.</p>

<p>It is also possible to define functions in jq, although this is is a feature whose biggest use is defining jq’s standard library (many jq functions such as <code>map</code> and <code>find</code> are in fact written in jq).</p>

<p>jq has reduction operators, which are very powerful but a bit tricky. Again, these are mostly used internally, to define some useful bits of jq’s standard library.</p>

<p>It may not be obvious at first, but jq is all about generators (yes, as often found in other languages). Some utilities are provided to help deal with generators.</p>

<p>Some minimal I/O support (besides reading JSON from standard input, and writing JSON to standard output) is available.</p>

<p>Finally, there is a module/library system.</p>

              
                <section id="Variables">
                  <h3>
                    
Variables

                    
                  </h3>
                  
<p>In jq, all filters have an input and an output, so manual plumbing is not necessary to pass a value from one part of a program to the next. Many expressions, for instance <code>a + b</code>, pass their input to two distinct subexpressions (here <code>a</code> and <code>b</code> are both passed the same input), so variables aren’t usually necessary in order to use a value twice.</p>

<p>For instance, calculating the average value of an array of numbers requires a few variables in most languages - at least one to hold the array, perhaps one for each element or for a loop counter. In jq, it’s simply <code>add / length</code> - the <code>add</code> expression is given the array and produces its sum, and the <code>length</code> expression is given the array and produces its length.</p>

<p>So, there’s generally a cleaner way to solve most problems in jq than defining variables. Still, sometimes they do make things easier, so jq lets you define variables using <code>expression as $variable</code>. All variable names start with <code>$</code>. Here’s a slightly uglier version of the array-averaging example:</p>

<pre><code>length as $array_length | add / $array_length</code></pre>

<p>We’ll need a more complicated problem to find a situation where using variables actually makes our lives easier.</p>

<p>Suppose we have an array of blog posts, with “author” and “title” fields, and another object which is used to map author usernames to real names. Our input looks like:</p>

<pre><code>{&quot;posts&quot;: [{&quot;title&quot;: &quot;Frist psot&quot;, &quot;author&quot;: &quot;anon&quot;},
           {&quot;title&quot;: &quot;A well-written article&quot;, &quot;author&quot;: &quot;person1&quot;}],
 &quot;realnames&quot;: {&quot;anon&quot;: &quot;Anonymous Coward&quot;,
               &quot;person1&quot;: &quot;Person McPherson&quot;}}</code></pre>

<p>We want to produce the posts with the author field containing a real name, as in:</p>

<pre><code>{&quot;title&quot;: &quot;Frist psot&quot;, &quot;author&quot;: &quot;Anonymous Coward&quot;}
{&quot;title&quot;: &quot;A well-written article&quot;, &quot;author&quot;: &quot;Person McPherson&quot;}</code></pre>

<p>We use a variable, $names, to store the realnames object, so that we can refer to it later when looking up author usernames:</p>

<pre><code>.realnames as $names | .posts[] | {title, author: $names[.author]}</code></pre>

<p>The expression <code>exp as $x | ...</code> means: for each value of expression <code>exp</code>, run the rest of the pipeline with the entire original input, and with <code>$x</code> set to that value. Thus <code>as</code> functions as something of a foreach loop.</p>

<p>Just as <code>{foo}</code> is a handy way of writing <code>{foo: .foo}</code>, so <code>{$foo}</code> is a handy way of writing <code>{foo:$foo}</code>.</p>

<p>Multiple variables may be declared using a single <code>as</code> expression by providing a pattern that matches the structure of the input (this is known as “destructuring”):</p>

<pre><code>. as {realnames: $names, posts: [$first, $second]} | ...</code></pre>

<p>The variable declarations in array patterns (e.g., <code>. as
[$first, $second]</code>) bind to the elements of the array in from the element at index zero on up, in order. When there is no value at the index for an array pattern element, <code>null</code> is bound to that variable.</p>

<p>Variables are scoped over the rest of the expression that defines them, so</p>

<pre><code>.realnames as $names | (.posts[] | {title, author: $names[.author]})</code></pre>

<p>will work, but</p>

<pre><code>(.realnames as $names | .posts[]) | {title, author: $names[.author]}</code></pre>

<p>won’t.</p>

<p>For programming language theorists, it’s more accurate to say that jq variables are lexically-scoped bindings. In particular there’s no way to change the value of a binding; one can only setup a new binding with the same name, but which will not be visible where the old one was.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example76">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example76" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.bar as $x | .foo | . + $x'</td></tr>
                            <tr><th>Input</th><td>{&quot;foo&quot;:10, &quot;bar&quot;:200}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>210</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '. as $i|[(.*2|. as $i| $i), $i]'</td></tr>
                            <tr><th>Input</th><td>5</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[10,5]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '. as [$a, $b, {c: $c}] | $a + $b + $c'</td></tr>
                            <tr><th>Input</th><td>[2, 3, {&quot;c&quot;: 4, &quot;d&quot;: 5}]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>9</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.[] as [$a, $b] | {a: $a, b: $b}'</td></tr>
                            <tr><th>Input</th><td>[[0], [0, 1], [2, 1, 0]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;a&quot;:0,&quot;b&quot;:null}</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>{&quot;a&quot;:0,&quot;b&quot;:1}</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>{&quot;a&quot;:2,&quot;b&quot;:1}</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="DefiningFunctions">
                  <h3>
                    
Defining Functions

                    
                  </h3>
                  
<p>You can give a filter a name using “def” syntax:</p>

<pre><code>def increment: . + 1;</code></pre>

<p>From then on, <code>increment</code> is usable as a filter just like a builtin function (in fact, this is how some of the builtins are defined). A function may take arguments:</p>

<pre><code>def map(f): [.[] | f];</code></pre>

<p>Arguments are passed as filters, not as values. The same argument may be referenced multiple times with different inputs (here <code>f</code> is run for each element of the input array). Arguments to a function work more like callbacks than like value arguments. This is important to understand. Consider:</p>

<pre><code>def foo(f): f|f;
5|foo(.*2)</code></pre>

<p>The result will be 20 because <code>f</code> is <code>.*2</code>, and during the first invocation of <code>f</code> <code>.</code> will be 5, and the second time it will be 10 (5 * 2), so the result will be 20. Function arguments are filters, and filters expect an input when invoked.</p>

<p>If you want the value-argument behaviour for defining simple functions, you can just use a variable:</p>

<pre><code>def addvalue(f): f as $f | map(. + $f);</code></pre>

<p>Or use the short-hand:</p>

<pre><code>def addvalue($f): ...;</code></pre>

<p>With either definition, <code>addvalue(.foo)</code> will add the current input’s <code>.foo</code> field to each element of the array.</p>

<p>Multiple definitions using the same function name are allowed. Each re-definition replaces the previous one for the same number of function arguments, but only for references from functions (or main program) subsequent to the re-definition.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example77">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example77" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'def addvalue(f): . + [f]; map(addvalue(.[0]))'</td></tr>
                            <tr><th>Input</th><td>[[1,2],[10,20]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[[1,2,1], [10,20,10]]</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'def addvalue(f): f as $x | map(. + $x); addvalue(.[0])'</td></tr>
                            <tr><th>Input</th><td>[[1,2],[10,20]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[[1,2,1,2], [10,20,1,2]]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="Reduce">
                  <h3>
                    
Reduce

                    
                  </h3>
                  
<p>The <code>reduce</code> syntax in jq allows you to combine all of the results of an expression by accumulating them into a single answer. As an example, we’ll pass <code>[3,2,1]</code> to this expression:</p>

<pre><code>reduce .[] as $item (0; . + $item)</code></pre>

<p>For each result that <code>.[]</code> produces, <code>. + $item</code> is run to accumulate a running total, starting from 0. In this example, <code>.[]</code> produces the results 3, 2, and 1, so the effect is similar to running something like this:</p>

<pre><code>0 | (3 as $item | . + $item) |
    (2 as $item | . + $item) |
    (1 as $item | . + $item)</code></pre>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example78">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example78" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'reduce .[] as $item (0; . + $item)'</td></tr>
                            <tr><th>Input</th><td>[10,2,5,3]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>20</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="limit(n;exp)">
                  <h3>
                    
<code>limit(n; exp)</code>

                    
                  </h3>
                  
<p>The <code>limit</code> function extracts up to <code>n</code> outputs from <code>exp</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example79">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example79" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[limit(3;.[])]'</td></tr>
                            <tr><th>Input</th><td>[0,1,2,3,4,5,6,7,8,9]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[0,1,2]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="first(expr),last(expr),nth(n;expr)">
                  <h3>
                    
<code>first(expr)</code>, <code>last(expr)</code>, <code>nth(n; expr)</code>

                    
                  </h3>
                  
<p>The <code>first(expr)</code> and <code>last(expr)</code> functions extract the first and last values from <code>expr</code>, respectively.</p>

<p>The <code>nth(n; expr)</code> function extracts the nth value output by <code>expr</code>. This can be defined as <code>def nth(n; expr):
last(limit(n + 1; expr));</code>. Note that <code>nth(n; expr)</code> doesn’t support negative values of <code>n</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example80">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example80" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[first(range(.)), last(range(.)), nth(./2; range(.))]'</td></tr>
                            <tr><th>Input</th><td>10</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[0,9,5]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="first,last,nth(n)">
                  <h3>
                    
<code>first</code>, <code>last</code>, <code>nth(n)</code>

                    
                  </h3>
                  
<p>The <code>first</code> and <code>last</code> functions extract the first and last values from any array at <code>.</code>.</p>

<p>The <code>nth(n)</code> function extracts the nth value of any array at <code>.</code>.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example81">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example81" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[range(.)]|[first, last, nth(5)]'</td></tr>
                            <tr><th>Input</th><td>10</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[0,9,5]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="foreach">
                  <h3>
                    
<code>foreach</code>

                    
                  </h3>
                  
<p>The <code>foreach</code> syntax is similar to <code>reduce</code>, but intended to allow the construction of <code>limit</code> and reducers that produce intermediate results (see example).</p>

<p>The form is <code>foreach EXP as $var (INIT; UPDATE; EXTRACT)</code>. Like <code>reduce</code>, <code>INIT</code> is evaluated once to produce a state value, then each output of <code>EXP</code> is bound to <code>$var</code>, <code>UPDATE</code> is evaluated for each output of <code>EXP</code> with the current state and with <code>$var</code> visible. Each value output by <code>UPDATE</code> replaces the previous state. Finally, <code>EXTRACT</code> is evaluated for each new state to extract an output of <code>foreach</code>.</p>

<p>This is mostly useful only for constructing <code>reduce</code>- and <code>limit</code>-like functions. But it is much more general, as it allows for partial reductions (see the example below).</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example82">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example82" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[foreach .[] as $item ([[],[]]; if $item == null then [[],.[0]] else [(.[0] + [$item]),[]] end; if $item == null then .[1] else empty end)]'</td></tr>
                            <tr><th>Input</th><td>[1,2,3,4,null,&quot;a&quot;,&quot;b&quot;,null]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[[1,2,3,4],[&quot;a&quot;,&quot;b&quot;]]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="Recursion">
                  <h3>
                    
Recursion

                    
                  </h3>
                  
<p>As described above, <code>recurse</code> uses recursion, and any jq function can be recursive. The <code>while</code> builtin is also implemented in terms of recursion.</p>

<p>Tail calls are optimized whenever the expression to the left of the recursive call outputs its last value. In practice this means that the expression to the left of the recursive call should not produce more than one output for each input.</p>

<p>For example:</p>

<pre><code>def recurse(f): def r: ., (f | select(. != null) | r); r;

def while(cond; update):
  def _while:
    if cond then ., (update | _while) else empty end;
  _while;

def repeat(exp):
  def _repeat:
    exp, _repeat;
  _repeat;</code></pre>


                  
                </section>
              
                <section id="Generatorsanditerators">
                  <h3>
                    
Generators and iterators

                    
                  </h3>
                  
<p>Some jq operators and functions are actually generators in that they can produce zero, one, or more values for each input, just as one might expect in other programming languages that have generators. For example, <code>.[]</code> generates all the values in its input (which must be an array or an object), <code>range(0; 10)</code> generates the integers between 0 and 10, and so on.</p>

<p>Even the comma operator is a generator, generating first the values generated by the expression to the left of the comma, then for each of those, the values generate by the expression on the right of the comma.</p>

<p>The <code>empty</code> builtin is the generator that produces zero outputs. The <code>empty</code> builtin backtracks to the preceding generator expression.</p>

<p>All jq functions can be generators just by using builtin generators. It is also possible to define new generators using only recursion and the comma operator. If the recursive call(s) is(are) “in tail position” then the generator will be efficient. In the example below the recursive call by <code>_range</code> to itself is in tail position. The example shows off three advanced topics: tail recursion, generator construction, and sub-functions.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example83">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Examples
                      </a>
                      <div id="example83" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'def range(init; upto; by): def _range: if (by &gt; 0 and . &lt; upto) or (by &lt; 0 and . &gt; upto) then ., ((.+by)|_range) else . end; if by == 0 then init else init|_range end | select((by &gt; 0 and . &lt; upto) or (by &lt; 0 and . &gt; upto)); range(0; 10; 3)'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>0</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>3</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>6</td>
                              </tr>
                            
                              <tr>
                                
                                  <th></th>
                                
                                <td>9</td>
                              </tr>
                            
                          </table>
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'def while(cond; update): def _while: if cond then ., (update | _while) else empty end; _while; [while(.&lt;100; .*2)]'</td></tr>
                            <tr><th>Input</th><td>1</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1,2,4,8,16,32,64]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
            </section>
          
            <section id="Math">
              <h2>Math</h2>
              
<p>jq currently only has IEEE754 double-precision (64-bit) floating point number support.</p>

<p>Besides simple arithmetic operators such as <code>+</code>, jq also has most standard math functions from the C math library. C math functions that take a single input argument (e.g., <code>sin()</code>) are available as zero-argument jq functions. C math functions that take two input arguments (e.g., <code>pow()</code>) are available as two-argument jq functions that ignore <code>.</code>.</p>

<p>Availability of standard math functions depends on the availability of the corresponding math functions in your operating system and C math library. Unavailable math functions will be defined but will raise an error.</p>

              
            </section>
          
            <section id="IO">
              <h2>I/O</h2>
              
<p>At this time jq has minimal support for I/O, mostly in the form of control over when inputs are read. Two builtins functions are provided for this, <code>input</code> and <code>inputs</code>, that read from the same sources (e.g., <code>stdin</code>, files named on the command-line) as jq itself. These two builtins, and jq’s own reading actions, can be interleaved with each other.</p>

<p>One builtin provides minimal output capabilities, <code>debug</code>. (Recall that a jq program’s output values are always output as JSON texts on <code>stdout</code>.) The <code>debug</code> builtin can have application-specific behavior, such as for executables that use the libjq C API but aren’t the jq executable itself.</p>

              
                <section id="input">
                  <h3>
                    
<code>input</code>

                    
                  </h3>
                  
<p>Outputs one new input.</p>


                  
                </section>
              
                <section id="inputs">
                  <h3>
                    
<code>inputs</code>

                    
                  </h3>
                  
<p>Outputs all remaining inputs, one by one.</p>

<p>This is primarily useful for reductions over a program’s inputs.</p>


                  
                </section>
              
                <section id="debug">
                  <h3>
                    
<code>debug</code>

                    
                  </h3>
                  
<p>Causes a debug message based on the input value to be produced. The jq executable wraps the input value with <code>[&quot;DEBUG:&quot;, &lt;input-value&gt;]</code> and prints that and a newline on stderr, compactly. This may change in the future.</p>


                  
                </section>
              
                <section id="input_filename">
                  <h3>
                    
<code>input_filename</code>

                    
                  </h3>
                  
<p>Returns the name of the file whose input is currently being filtered. Note that this will not work well unless jq is running in a UTF-8 locale.</p>


                  
                </section>
              
                <section id="input_line_number">
                  <h3>
                    
<code>input_line_number</code>

                    
                  </h3>
                  
<p>Returns the line number of the input currently being filtered.</p>


                  
                </section>
              
            </section>
          
            <section id="Streaming">
              <h2>Streaming</h2>
              
<p>With the <code>--stream</code> option jq can parse input texts in a streaming fashion, allowing jq programs to start processing large JSON texts immediately rather than after the parse completes. If you have a single JSON text that is 1GB in size, streaming it will allow you to process it much more quickly.</p>

<p>However, streaming isn’t easy to deal with as the jq program will have <code>[&lt;path&gt;, &lt;leaf-value&gt;]</code> (and a few other forms) as inputs.</p>

<p>Several builtins are provided to make handling streams easier.</p>

<p>The examples below use the streamed form of <code>[0,[1]]</code>, which is <code>[[0],0],[[1,0],1],[[1,0]],[[1]]</code>.</p>

<p>Streaming forms include <code>[&lt;path&gt;, &lt;leaf-value&gt;]</code> (to indicate any scalar value, empty array, or empty object), and <code>[&lt;path&gt;]</code> (to indicate the end of an array or object). Future versions of jq run with <code>--stream</code> and <code>-seq</code> may output additional forms such as <code>[&quot;error message&quot;]</code> when an input text fails to parse.</p>

              
                <section id="truncate_stream(stream_expression)">
                  <h3>
                    
<code>truncate_stream(stream_expression)</code>

                    
                  </h3>
                  
<p>Consumes a number as input and truncates the corresponding number of path elements from the left of the outputs of the given streaming expression.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example84">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example84" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '[1|truncate_stream([[0],1],[[1,0],2],[[1,0]],[[1]])]'</td></tr>
                            <tr><th>Input</th><td>1</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[[[0],2],[[0]]]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="fromstream(stream_expression)">
                  <h3>
                    
<code>fromstream(stream_expression)</code>

                    
                  </h3>
                  
<p>Outputs values corresponding to the stream expression’s outputs.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example85">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example85" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq 'fromstream(1|truncate_stream([[0],1],[[1,0],2],[[1,0]],[[1]]))'</td></tr>
                            <tr><th>Input</th><td>null</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[2]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="tostream">
                  <h3>
                    
<code>tostream</code>

                    
                  </h3>
                  
<p>The <code>tostream</code> builtin outputs the streamed form of its input.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example86">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example86" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '. as $dot|fromstream($dot|tostream)|.==$dot'</td></tr>
                            <tr><th>Input</th><td>[0,[1,{&quot;a&quot;:1},{&quot;b&quot;:2}]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>true</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
            </section>
          
            <section id="Assignment">
              <h2>Assignment</h2>
              
<p>Assignment works a little differently in jq than in most programming languages. jq doesn’t distinguish between references to and copies of something - two objects or arrays are either equal or not equal, without any further notion of being “the same object” or “not the same object”.</p>

<p>If an object has two fields which are arrays, <code>.foo</code> and <code>.bar</code>, and you append something to <code>.foo</code>, then <code>.bar</code> will not get bigger. Even if you’ve just set <code>.bar = .foo</code>. If you’re used to programming in languages like Python, Java, Ruby, Javascript, etc. then you can think of it as though jq does a full deep copy of every object before it does the assignment (for performance, it doesn’t actually do that, but that’s the general idea).</p>

<p>All the assignment operators in jq have path expressions on the left-hand side.</p>

              
                <section id="=">
                  <h3>
                    
<code>=</code>

                    
                  </h3>
                  
<p>The filter <code>.foo = 1</code> will take as input an object and produce as output an object with the “foo” field set to 1. There is no notion of “modifying” or “changing” something in jq - all jq values are immutable. For instance,</p>

<p>.foo = .bar | .foo.baz = 1</p>

<p>will not have the side-effect of setting .bar.baz to be set to 1, as the similar-looking program in Javascript, Python, Ruby or other languages would. Unlike these languages (but like Haskell and some other functional languages), there is no notion of two arrays or objects being “the same array” or “the same object”. They can be equal, or not equal, but if we change one of them in no circumstances will the other change behind our backs.</p>

<p>This means that it’s impossible to build circular values in jq (such as an array whose first element is itself). This is quite intentional, and ensures that anything a jq program can produce can be represented in JSON.</p>

<p>Note that the left-hand side of ‘=’ refers to a value in <code>.</code>. Thus <code>$var.foo = 1</code> won’t work as expected (<code>$var.foo</code> is not a valid or useful path expression in <code>.</code>); use <code>$var | .foo =
1</code> instead.</p>

<p>If the right-hand side of ‘=’ produces multiple values, then for each such value jq will set the paths on the left-hand side to the value and then it will output the modified <code>.</code>. For example, <code>(.a,.b)=range(2)</code> outputs <code>{&quot;a&quot;:0,&quot;b&quot;:0}</code>, then <code>{&quot;a&quot;:1,&quot;b&quot;:1}</code>. The “update” assignment forms (see below) do not do this.</p>

<p>Note too that <code>.a,.b=0</code> does not set <code>.a</code> and <code>.b</code>, but <code>(.a,.b)=0</code> sets both.</p>


                  
                </section>
              
                <section id="|=">
                  <h3>
                    
<code>|=</code>

                    
                  </h3>
                  
<p>As well as the assignment operator ‘=’, jq provides the “update” operator ‘|=’, which takes a filter on the right-hand side and works out the new value for the property of <code>.</code> being assigned to by running the old value through this expression. For instance, .foo |= .+1 will build an object with the “foo” field set to the input’s “foo” plus 1.</p>

<p>This example should show the difference between ‘=’ and ‘|=’:</p>

<p>Provide input ‘{“a”: {“b”: 10}, “b”: 20}’ to the programs:</p>

<p>.a = .b .a |= .b</p>

<p>The former will set the “a” field of the input to the “b” field of the input, and produce the output {“a”: 20}. The latter will set the “a” field of the input to the “a” field’s “b” field, producing {“a”: 10}.</p>

<p>The left-hand side can be any general path expression; see <code>path()</code>.</p>

<p>Note that the left-hand side of ‘|=’ refers to a value in <code>.</code>. Thus <code>$var.foo |= . + 1</code> won’t work as expected (<code>$var.foo</code> is not a valid or useful path expression in <code>.</code>); use <code>$var |
.foo |= . + 1</code> instead.</p>

<p>If the right-hand side outputs multiple values, only the last one will be used.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example87">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example87" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '(..|select(type==&quot;boolean&quot;)) |= if . then 1 else 0 end'</td></tr>
                            <tr><th>Input</th><td>[true,false,[5,true,[true,[false]],false]]</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>[1,0,[5,1,[1,[0]],0]]</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="+=,-=,*=,/=,%=,//=">
                  <h3>
                    
<code>+=</code>, <code>-=</code>, <code>*=</code>, <code>/=</code>, <code>%=</code>, <code>//=</code>

                    
                  </h3>
                  
<p>jq has a few operators of the form <code>a op= b</code>, which are all equivalent to <code>a |= . op b</code>. So, <code>+= 1</code> can be used to increment values.</p>


                  
                    <div>
                      
                      <a data-toggle="collapse" href="#example88">
                        <i class="glyphicon glyphicon-chevron-right"></i>
                        Example
                      </a>
                      <div id="example88" class="manual-example collapse">
                        
                          <table>
                            <tr><th></th><td class="jqprogram">jq '.foo += 1'</td></tr>
                            <tr><th>Input</th><td>{&quot;foo&quot;: 42}</td></tr>
                            
                            
                              <tr>
                                
                                  <th>Output</th>
                                
                                <td>{&quot;foo&quot;: 43}</td>
                              </tr>
                            
                          </table>
                        
                      </div>
                    </div>
                  
                </section>
              
                <section id="Complexassignments">
                  <h3>
                    
Complex assignments

                    
                  </h3>
                  
<p>Lots more things are allowed on the left-hand side of a jq assignment than in most languages. We’ve already seen simple field accesses on the left hand side, and it’s no surprise that array accesses work just as well:</p>

<pre><code>.posts[0].title = &quot;JQ Manual&quot;</code></pre>

<p>What may come as a surprise is that the expression on the left may produce multiple results, referring to different points in the input document:</p>

<pre><code>.posts[].comments |= . + [&quot;this is great&quot;]</code></pre>

<p>That example appends the string “this is great” to the “comments” array of each post in the input (where the input is an object with a field “posts” which is an array of posts).</p>

<p>When jq encounters an assignment like ‘a = b’, it records the “path” taken to select a part of the input document while executing a. This path is then used to find which part of the input to change while executing the assignment. Any filter may be used on the left-hand side of an equals - whichever paths it selects from the input will be where the assignment is performed.</p>

<p>This is a very powerful operation. Suppose we wanted to add a comment to blog posts, using the same “blog” input above. This time, we only want to comment on the posts written by “stedolan”. We can find those posts using the “select” function described earlier:</p>

<pre><code>.posts[] | select(.author == &quot;stedolan&quot;)</code></pre>

<p>The paths provided by this operation point to each of the posts that “stedolan” wrote, and we can comment on each of them in the same way that we did before:</p>

<pre><code>(.posts[] | select(.author == &quot;stedolan&quot;) | .comments) |=
    . + [&quot;terrible.&quot;]</code></pre>


                  
                </section>
              
            </section>
          
            <section id="Modules">
              <h2>Modules</h2>
              
<p>jq has a library/module system. Modules are files whose names end in <code>.jq</code>.</p>

<p>Modules imported by a program are searched for in a default search path (see below). The <code>import</code> and <code>include</code> directives allow the importer to alter this path.</p>

<p>Paths in the a search path are subject to various substitutions.</p>

<p>For paths starting with “~/”, the user’s home directory is substituted for “~”.</p>

<p>For paths starting with “$ORIGIN/”, the path of the jq executable is substituted for “$ORIGIN”.</p>

<p>For paths starting with “./” or paths that are “.”, the path of the including file is substituted for “.”. For top-level programs given on the command-line, the current directory is used.</p>

<p>Import directives can optionally specify a search path to which the default is appended.</p>

<p>The default search path is the search path given to the <code>-L</code> command-line option, else <code>[&quot;~/.jq&quot;, &quot;$ORIGIN/../lib/jq&quot;,
&quot;$ORIGIN/../lib&quot;]</code>.</p>

<p>Null and empty string path elements terminate search path processing.</p>

<p>A dependency with relative path “foo/bar” would be searched for in “foo/bar.jq” and “foo/bar/bar.jq” in the given search path. This is intended to allow modules to be placed in a directory along with, for example, version control files, README files, and so on, but also to allow for single-file modules.</p>

<p>Consecutive components with the same name are not allowed to avoid ambiguities (e.g., “foo/foo”).</p>

<p>For example, with <code>-L$HOME/.jq</code> a module <code>foo</code> can be found in <code>$HOME/.jq/foo.jq</code> and <code>$HOME/.jq/foo/foo.jq</code>.</p>

<p>If “$HOME/.jq” is a file, it is sourced into the main program.</p>

              
                <section id="importRelativePathStringasNAME[<metadata>];">
                  <h3>
                    
<code>import RelativePathString as NAME [&lt;metadata&gt;];</code>

                    
                  </h3>
                  
<p>Imports a module found at the given path relative to a directory in a search path. A “.jq” suffix will be added to the relative path string. The module’s symbols are prefixed with “NAME::”.</p>

<p>The optional metadata must be a constant jq expression. It should be an object with keys like “homepage” and so on. At this time jq only uses the “search” key/value of the metadata. The metadata is also made available to users via the <code>modulemeta</code> builtin.</p>

<p>The “search” key in the metadata, if present, should have a string or array value (array of strings); this is the search path to be prefixed to the top-level search path.</p>


                  
                </section>
              
                <section id="includeRelativePathString[<metadata>];">
                  <h3>
                    
<code>include RelativePathString [&lt;metadata&gt;];</code>

                    
                  </h3>
                  
<p>Imports a module found at the given path relative to a directory in a search path as if it were included in place. A “.jq” suffix will be added to the relative path string. The module’s symbols are imported into the caller’s namespace as if the module’s content had been included directly.</p>

<p>The optional metadata must be a constant jq expression. It should be an object with keys like “homepage” and so on. At this time jq only uses the “search” key/value of the metadata. The metadata is also made available to users via the <code>modulemeta</code> builtin.</p>


                  
                </section>
              
                <section id="importRelativePathStringas$NAME[<metadata>];">
                  <h3>
                    
<code>import RelativePathString as $NAME [&lt;metadata&gt;];</code>

                    
                  </h3>
                  
<p>Imports a JSON file found at the given path relative to a directory in a search path. A “.json” suffix will be added to the relative path string. The file’s data will be available as <code>$NAME::NAME</code>.</p>

<p>The optional metadata must be a constant jq expression. It should be an object with keys like “homepage” and so on. At this time jq only uses the “search” key/value of the metadata. The metadata is also made available to users via the <code>modulemeta</code> builtin.</p>

<p>The “search” key in the metadata, if present, should have a string or array value (array of strings); this is the search path to be prefixed to the top-level search path.</p>


                  
                </section>
              
                <section id="module<metadata>;">
                  <h3>
                    
<code>module &lt;metadata&gt;;</code>

                    
                  </h3>
                  
<p>This directive is entirely optional. It’s not required for proper operation. It serves only the purpose of providing metadata that can be read with the <code>modulemeta</code> builtin.</p>

<p>The metadata must be a constant jq expression. It should be an object with keys like “homepage”. At this time jq doesn’t use this metadata, but it is made available to users via the <code>modulemeta</code> builtin.</p>


                  
                </section>
              
                <section id="modulemeta">
                  <h3>
                    
<code>modulemeta</code>

                    
                  </h3>
                  
<p>Takes a module name as input and outputs the module’s metadata as an object, with the module’s imports (including metadata) as an array value for the “deps” key.</p>

<p>Programs can use this to query a module’s metadata, which they could then use to, for example, search for, download, and install missing dependencies.</p>


                  
                </section>
              
            </section>
          
        </div>
      </div>
    </div>
