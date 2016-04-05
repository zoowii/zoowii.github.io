---
title: 一个简单的正则表达式和语法分析引擎的JavaScript实现parser.js
date: 2015-04-06 19:13:10
tags: JavaScript, 编译原理
---
前段时间为了克服拖延症,花了几天时间把以前一直想填的一个小坑填了,就是这个parser.js库了 [https://github.com/zoowii/parser.js](https://github.com/zoowii/parser.js)

懒得写,就把readme.md抄一部分过来


## Features
* 实现了一个基于NFA的正则表达式引擎实现(正则表达式字符串的解析使用自身实现的底层API和下面的语法解析引擎来实现的),支持主要正则功能已经分组捕获等
* 实现了一个语法解析引擎(支持左递归定义,语法定义简单)
* 提供一个整合了词法分析和语法分析的API,方便使用,可以直接生成最终满足要求的抽象语法树(直接生成最初定义规则对应的抽象语法树,而不是解析过程中的中间产物)
* 为了性能考虑,另外提供了一个使用内置正则引擎的快速词法分析实现,并且尽量兼容自己实现的正则API,从而可以在性能有问题时切换
* 直接浏览器和Node.js环境,且不依赖任何第三方库

## Demo

* 正则表达式引擎demo

```
    console.log('-----test regex string reader=====');
    var expr1 = RegexReader.read("\"(a{3,})(b+)(([c\\s\\.\\d\\\\\\+\\u1234])*)");
    expr1.build();
    console.log('regex build done');
    var r1 = expr1.match("aabbbc 123+556end");
    var r2 = expr1.match("\"aaaabbbc 123+556");
    // var expr2 = RegexReader.read("abc");
    console.log(expr1.toString());
    console.log(r1.toString());
    console.log(r2.toString());
    assert.equal(false, r1.matched);
    assert.equal(true, r2.matched);
    console.log('-----end test regex string reader-----');
```
    
* 语法分析demo

```
    console.log('-----test parser api-----');
    parser.clearVarCache();
    var syntaxParserAndTokener = parser.buildSyntaxTreeParser(V('json'), [
        [V('bool'),
            "(?:true|false)\\s*"],
        [V('number'),
            "(?:[+-]?(?:(0x[0-9a-fA-F]+|0[0-7]+)|((?:[0-9]+(?:\\.[0-9]*)?|\\.[0-9]+)(?:[eE][+-]?[0-9]+)?|NaN|Infinity)))\\s*"],
        [V('string'),
            "(?:(?:\"((?:\\.|[^\"])*)\"|'((?:\\.|[^'])*)'))\\s*"],
        [V('name'),
            V('string')],
        [V('value'),
            V('bool'), V('number'), V('string'), V('json-object'), V('json-array')],
        [V('json-object-pair'),
            [V('name'), ":\\s*", V('value')]],
        [V('json-object-pairs'),
            V('json-object-pair'),
            [V('json-object-pair'), ",\\s*", V('json-object-pairs')]
        ],
        [V('json-object'),
            ["\\{\\s*", V('json-object-pairs'), "\\}\\s*"]],
        [V('values'),
            V('value'),
            [V('value'), ",\\s*", V('values')]],
        [V('json-array'),
            ["\\[\\s*", V('values'), "]\\s*"]],
        [V('json'),
            V('json-object'), V('json-array'), [V("\\s+"), V('json')]]
    ]);
    var jsonParser = syntaxParserAndTokener.parser;
    var tokenPatterns = syntaxParserAndTokener.token_patterns;
    var text = '{"name": "zoowii", "age": 24, "position": {"country": "China", "city": "Nanjing"}}';
    var tokens = parser.generateTokenListUsingInnerRegex(tokenPatterns, text, console.log);
    var json = jsonParser.parse(tokens);
    console.log(json.toString());
    console.log('-----end test parser api-----');
```
    

## 目前支持的正则表达式特性

* '|'
* ( ... ) group
* \d digit
* \w alpha or digit
* \uabcd unicode char support
* a-b char range
* [abc] union
* [^abc] except union
* abc concat
* a+ repeat at least one times
* a* repeat at least zero times
* a? repeat one or zero times
* a{m[,n]} repeat at least m times [and at most n times]
* . any char
* \s space
* \ + * { [ ( | \? . - ... escape