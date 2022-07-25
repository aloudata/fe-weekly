# Hue 是如何做自动补全的

## 一、简单了解 Hue

### Hue  是什么

xHue = Hadoop User Experience

Hue 是一个开源的  Apache Hadoop UI  系统，由  Cloudera Desktop  演化而来，最后 Cloudera 公司将其贡献给  Apache  基金会的  Hadoop  社区，它是基于  Python Web  框架  Django  实现的。

通过使用 Hue，可以在浏览器端的 Web 控制台上与  Hadoop  集群进行交互。

![https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/MNpQlKak371DqDvL/img/8a9276d9-7e6f-40c7-acb0-e7a58eb00c57.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/MNpQlKak371DqDvL/img/8a9276d9-7e6f-40c7-acb0-e7a58eb00c57.png)

简单来说就是一个集   数据目录、SQL  编辑器、查询结果为一身的一个  web UI 系统

### Hue 的 SQL 自动补全

![https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/MNpQlKak371DqDvL/img/83363719-7a87-49da-a5d7-79d5b1b58d60.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/MNpQlKak371DqDvL/img/83363719-7a87-49da-a5d7-79d5b1b58d60.png)

Hue 的强大之处在于有的断句如图所示的  SELECT FROM  也能提示

![https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/MNpQlKak371DqDvL/img/c9dd4b55-ad5f-4b66-a711-722c7264232c.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/MNpQlKak371DqDvL/img/c9dd4b55-ad5f-4b66-a711-722c7264232c.png)

也能识别上下文的  CTE、别名等

## 二、编译原理相关知识

看到  Hue  的自动补全，直觉上觉得跟编译原理相关，我们了解下相关知识

下图是完整的编译流程：
[编译原理流程图](./imgs/0D4F2A15-D5DC-42F0-98E1-47B9AFD4A046.png)
#### 编译器的前端

1. 词法分析器

将输入的字符流转换为特定的单词。这一步是识别组合字符的过程，识别出数字，标识符，关键字等过程就在于此。若我们这里设计的编译器不包含预处理器，那么，在这里还需要做一些预处理的过程，如识别出注释，并将其忽略掉。这一步的输入是字符，输出是单词。

1. 语法分析

将输入的单词流转换为特定的语句。这里面需要做的就是组合单词，并按照特定的语法规则去匹配出合法的语句。如  if （ bool-expression ）{ statement }  就是简单的  if  语句，而所需要的就有关键字  “if” ，符号  “( ”，表达式，符号“ )”，  符号  “{”，  合法的语句，  符号  “}”。而这里的表达式与合法语句则又由合法的单词流组成。而我们把这一步处理完毕后，我们留下我们所需要关注的核心内容。如  if  语句，我们传递下去后，我们就不需要符号  ( )  { }  等了。而表示这样的核心内容则是抽象语法树（Abstract Syntax Tree ，简称 AST）

js AST  规范是前端无形中接触最多的

[https://github.com/estree/estree](https://github.com/estree/estree)

1. 语义分析

这一步主要是对已经生成的  AST  进行分析，使用语法树和符号表中的信息来检查源程序是否和语言定义的语义一致。它同时也收集类型信息，并把这些信息存放在语法树或者符号表中，以便在随后的代码生成过程中使用

## 三、Jison

Hue  的词法分析和语法分析是基于  [Jison](https://github.com/zaach/jison)  这个库来做的

Jison  的  API  与  Bison  的相似，因此得名。它支持  Bison  的许多主要功能，以及它自己的一些功能。如果对诸如  Bison  之类的解析器生成器和上下文无关语法不熟悉，那么在  Bison  手册中可以找到一些介绍。

[http://dinosaur.compilertools.net/bison/index.html#SEC34](http://dinosaur.compilertools.net/bison/index.html#SEC34)

#### 首先了解什么是解析器（parser）

解析器生成器？听起来是有点拗口，要理解这个名称还是要从 JSON 说起

```
JSON.parse(`{
	"Object": {
		"PI": 3.1415926
  }
}`);
```

这里使用的  JSON.parse 函数就是解析器（parse）

那么解析器和解析器生成器  (parser generator)  有什么区别和联系呢？

JSON.parse  函数只能用于解析格式为符合 JSON 规范的字符串，如果字符串格式不是 JSON，或者略有变化，比如在 JSON 中像写程序一样增加注释

```
JSON.parse(`{
	// 这是一个注释
	"Object": {
		"PI": 3.1415926
  }
}`);
```

就会报错  Uncaught SyntaxError: Unexpected token / in JSON at position 3

这是因为  JSON.parse  不能识别  “//  这是一个注释”，此时可以用 JSON5

```
cosnt JSON5 = require('json5');
JSON5.parse(`{
	// 这是一个注释
	"Object": {
		"PI": 3.1415926
  }
}`);
```

不过  JSON5  也只能解析符合  JSON5  规范的字符串。类似的解析器还有  Hjson、HOCON，都是 JSON 的变体解析器。每个解析器只能解析符合自己规范格式的语法

而  Jison  就是使用者可以通过一套规则来定义生成一个解析器，Hue  的  Parser  就是使用  Jison  生成的拥有独特解析规则的解析器

#### 简单的介绍下 Jison 的语法规则

#### 词法分析

Jison  的词法分析

```
/* lexical grammar */
%lex
%%

\s+            /* skip whitespace */
"你"           return '你';
"我"           return '我';
"头发少"        return '头发少';
"头发多"        return '头发多';
";"            return ';';
<<EOF>>        return 'EOF';

/lex
```

- %lex  代表以下为词法分析，/lex  是相应的闭合符号
- %%  是开始符号
- 左侧为匹配符号，支持正则，也支持指定字符串；有些特殊符号如  <<EOF>>  代表文本末尾
- 右边为一个代码块，也就是说是支持执行一些代码的只是最后需要  return  出识别的  token

#### 语法分析

#### 上下文无关文法

一个上下文无关文法（context-free grammar，简写 CFG）由 4 个元素组成：

1. 一个终结符号集合，他们有时也成为“词法单元”。终结符号是该文法所定义的语言的基本符号的集合。
2. 一个非终结符号集合，他们也称为“语法变量”。每个非终结符号表示一个终结符号串的集合。
3. 一个产生式集合，其中每个产生式包括一个成为产生式头或左部的非终结符号，一个箭头，和一个称为产生式体或右部的由终结符号及非终结符号组成的序列。产生式主要用来表示某个构造的某种书写形式。如果产生式头非终结符号代表一个构造，那么该生产式体就代表了该构造的一种书写方式。
4. 指定一个非终结符号为开始符号。

#### 巴克斯范式

巴克斯范式（Backus Normal Form）BNF，是一种用于表示上下文无关（Context-Free Grammar）的语言

BNF 的形式为<符号> ::= <使用符号的表达式>，这里的  <符号>  是非终结符，而表达式由一个符号序列，或用指示选择的竖杠  “|”  分隔的多个符号序列构成，每个符号序列整体都是左端的符号的一种可能的替代。从未在左端出现的符号就是终结符。

10 以内的加减乘除 BNF 表达式：

```

expression::= number + number | number - number | number * number | number / number

number::= 0|1|2|3|4|5|6|7|8|9

```

前端经常能接触到的是  JSON Schema  的  BNF：

[https://cswr.github.io/JsonSchema/spec/grammar/](https://cswr.github.io/JsonSchema/spec/grammar/)

jison 的语法分析

```
%start expressions
%% /* language grammar */

expressions
  : paragraph EOF
  ;
paragraph
  : sentence ';' paragraph
  | sentence
  ;
sentence
  : word
  | you '头发少' {console.log('你技术好');}
  | you '头发多' {console.log('你技术差');}
  | word sentence
  ;
you
  : '你'
  ;
word
  : '我'
  | '头发少'
  | '头发多'
  ;
```

- %start expressions  指定了一个开始符号
- %%  为语法分析开始符号
- 后续为 BNF 范式

#### 执行结果

将该 Jison 文件编译，未安装 Jison 的请先安装

```
npm i jison -g
```

编译以上  test.jison  文件

```
jison test.jison
```

写入一下要被编译的字符创

```
echo '你头发多;你头发少'>data
```

开始解析

```
node test.js data
你技术差
你技术好
```

## 四、Hue  基于  jison  的自动提示

### 文件结构

首先找到我们需要解析的语法包，这里就以  calcite  语法包为例子，找到位于 hue/desktop/core/src/desktop/js/parse/jison/sql/calcite  下的  structure.json  文件

里面记录了所有词法分析文件和语法分析文件

```
{
  "lexer": "../generic/sql.jisonlex",
  "autocomplete": [
    "../generic/autocomplete_header.jison",
    "../generic/alter/alter_common.jison",
    "../generic/alter/alter_table.jison",
    "../generic/alter/alter_view.jison",
    "../generic/create/create_common.jison",
    "../generic/create/create_database.jison",
    "../generic/create/create_role.jison",
    "../generic/create/create_table.jison",
    "../generic/create/create_view.jison",
    "../generic/drop/drop_common.jison",
    "../generic/drop/drop_database.jison",
    "../generic/drop/drop_role.jison",
    "../generic/drop/drop_table.jison",
    "../generic/drop/drop_view.jison",
    "../generic/insert/insert.jison",
    "../generic/select/cte_select_statement.jison",
    "../generic/select/from_clause.jison",
    "../generic/select/group_by_clause.jison",
    "../generic/select/having_clause.jison",
    "../generic/select/joins.jison",
    "../generic/select/limit_clause.jison",
    "../generic/select/order_by_clause.jison",
    "../generic/select/select.jison",
    "../generic/select/select_conditions.jison",
    "../generic/select/union_clause.jison",
    "../generic/select/where_clause.jison",
    "../generic/set/set_common.jison",
    "../generic/set/set_all.jison",
    "../generic/set/set_option.jison",
    "../generic/truncate/truncate_table.jison",
    "../generic/udf/aggregate/aggregate_common.jison",
    "../generic/udf/aggregate/avg.jison",
    "../generic/udf/aggregate/count.jison",
    "../generic/udf/aggregate/max.jison",
    "../generic/udf/aggregate/min.jison",
    "../generic/udf/aggregate/stddev_pop.jison",
    "../generic/udf/aggregate/stddev_samp.jison",
    "../generic/udf/aggregate/sum.jison",
    "../generic/udf/aggregate/var_pop.jison",
    "../generic/udf/aggregate/var_samp.jison",
    "../generic/udf/aggregate/variance.jison",
    "../generic/udf/analytic/analytic.jison",
    "../generic/udf/function/array.jison",
    "../generic/udf/function/cast.jison",
    "../generic/udf/function/if.jison",
    "../generic/udf/function/map.jison",
    "../generic/udf/function/truncate.jison",
    "../generic/udf/udf_common.jison",
    "../generic/update/update_table.jison",
    "../generic/use/use.jison",
    "../generic/sql_main.jison",
    "select/select_stream.jison",
    "../generic/sql_error.jison",
    "../generic/sql_valueExpression.jison",
    "describe/describe_table.jison",
    "../generic/autocomplete_footer.jison"
  ],
  "syntax": [
    "../generic/syntax_header.jison",
    "../generic/alter/alter_common.jison",
    "../generic/alter/alter_table.jison",
    "../generic/alter/alter_view.jison",
    "../generic/create/create_common.jison",
    "../generic/create/create_database.jison",
    "../generic/create/create_role.jison",
    "../generic/create/create_table.jison",
    "../generic/create/create_view.jison",
    "../generic/drop/drop_common.jison",
    "../generic/drop/drop_database.jison",
    "../generic/drop/drop_role.jison",
    "../generic/drop/drop_table.jison",
    "../generic/drop/drop_view.jison",
    "../generic/insert/insert.jison",
    "../generic/select/cte_select_statement.jison",
    "../generic/select/from_clause.jison",
    "../generic/select/group_by_clause.jison",
    "../generic/select/having_clause.jison",
    "../generic/select/joins.jison",
    "../generic/select/limit_clause.jison",
    "../generic/select/order_by_clause.jison",
    "../generic/select/select.jison",
    "../generic/select/select_conditions.jison",
    "../generic/select/union_clause.jison",
    "../generic/select/where_clause.jison",
    "../generic/set/set_common.jison",
    "../generic/set/set_all.jison",
    "../generic/set/set_option.jison",
    "../generic/truncate/truncate_table.jison",
    "../generic/udf/aggregate/aggregate_common.jison",
    "../generic/udf/aggregate/avg.jison",
    "../generic/udf/aggregate/count.jison",
    "../generic/udf/aggregate/max.jison",
    "../generic/udf/aggregate/min.jison",
    "../generic/udf/aggregate/stddev_pop.jison",
    "../generic/udf/aggregate/stddev_samp.jison",
    "../generic/udf/aggregate/sum.jison",
    "../generic/udf/aggregate/var_pop.jison",
    "../generic/udf/aggregate/var_samp.jison",
    "../generic/udf/aggregate/variance.jison",
    "../generic/udf/analytic/analytic.jison",
    "../generic/udf/function/array.jison",
    "../generic/udf/function/cast.jison",
    "../generic/udf/function/if.jison",
    "../generic/udf/function/map.jison",
    "../generic/udf/function/truncate.jison",
    "../generic/udf/udf_common.jison",
    "../generic/update/update_table.jison",
    "../generic/use/use.jison",
    "../generic/sql_main.jison",
    "select/select_stream.jison",
    "../generic/sql_valueExpression.jison",
    "describe/describe_table.jison",
    "../generic/syntax_footer.jison"
  ]
}

```

找到词法分析文件  sql.jisonlex  也是入口文件

```
// ...
'\u2020'                                   { parser.yy.partialCursor = false; parser.yy.cursorFound = yylloc; return 'CURSOR'; }
'\u2021'                                   { parser.yy.partialCursor = true; parser.yy.cursorFound = yylloc; return 'PARTIAL_CURSOR'; }
// ...
'FROM'                                     { parser.determineCase(yytext); return 'FROM'; }
// ...
'SELECT'                                   { parser.determineCase(yytext); parser.addStatementTypeLocation('SELECT', yylloc); return 'SELECT'; }
```

可以看到这里  Hue  是将光标作为了一个  token  进行处理，后续所有带有光标的语句都会被认为是正在编辑中，后缀会添加  _EDIT  作为添加

然后再来看  Hue  的语法分析文件，Hue  的语法分析文件非常分散，因为目前  SQL  会有好多的标准和方言，Hue  将公用的语法分析部分按不同作用分别抽出，然后各个方言的特色语法都是放在各自文件中。做到了最大程度的复用。

这里我们阅读时建议将其合并阅读，会更加方便。

### 为什么断句  SELECT FROM  还可以提示

![https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/MNpQlKak371DqDvL/img/83363719-7a87-49da-a5d7-79d5b1b58d60.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/MNpQlKak371DqDvL/img/83363719-7a87-49da-a5d7-79d5b1b58d60.png)

还记得我们在了解  Hue  时放出了一张在上面还有两句带分号的不完整的语句  SELECT FROM  的时候还正确提示的图，带着这两个疑问我们去阅读  Hue  的语法分析文件。

自动补全的语法文件入口为 SqlAutocomplete

```
SqlAutocomplete
 : SqlStatements EOF
 | SqlStatements_EDIT EOF
 ;

SqlStatements
 :
 | SqlStatement
 | SqlStatements ';' NewStatement SqlStatements
 ;

SqlStatements_EDIT
 : SqlStatement_EDIT
 | SqlStatement_EDIT ';' SqlStatements
 //SELECT;
 //WITH;
 //SELECT FROM CURSOR;
 // 我们的语句符合此推导
 | SqlStatements ';' SqlStatement_EDIT
 | SqlStatements ';' SqlStatement_EDIT ';' SqlStatements
 ;
```

可以看到语法文件中有很多带有\_EDIT，那是正在编辑中的语法意思，也就是断句

我们的 SQL 语句

```
SELECT;
WITH;
SELECT FROM CURSOR;
```

显然符合第三种推导，不受前面语句的影响。

那么 SELECT FROM  又是怎么给出提示的。

我们接下去看语法文件

```
SqlStatement_EDIT
 : AnyCursor
 | CommonTableExpression 'CURSOR'
 | DataDefinition_EDIT
 | DataManipulation_EDIT
 | QuerySpecification_EDIT
 | SetSpecification_EDIT
 ;

QuerySpecification_EDIT
 : SelectStatement_EDIT OptionalUnions
 | ...
 ;

OptionalUnions
 :
 | Unions
 ;

SelectStatement_EDIT
	: 'SELECT' OptionalAllOrDistinct SelectList TableExpression_EDIT
  | ...
  ;
  // SelectList 经过复杂的推导也可为空
 OptionalAllOrDistinct
   :
   | 'ALL'
   | 'DISTINCT'
   ;

```

SelectStatement_EDIT  已经可以推导为  'SELECT' TableExpression_EDIT

我们接下来看  TableExpression_EDIT  的推导

```
TableExpression_EDIT
 : FromClause_EDIT OptionalSelectConditions
 | ...
 ;

 OptionalSelectConditions
 // 这几个值都可以为空
 	:OptionalWhereClause OptionalGroupByClause OptionalHavingClause OptionalWindowClause OptionalOrderByClause OptionalClusterOrDistributeBy OptionalLimitClause
  | ...
  ;
```

所以就演变成了

```
SelectStatement_EDIT
	:'SELECT' FromClause_EDIT
  | ...
  ;
```

再看下  FromClause

```
FromClause_EDIT
 : 'FROM' 'CURSOR'
   {
       parser.suggestTables();
       parser.suggestDatabases({ appendDot: true });
   }
	| ...
  ;

```

真相大白，当语法分析到  'SELECT' 'FROM' 'CURSOR'  时他就会给出推荐  tables  或者  databases

## 参考资料

《编译原理》第 2 版  --  机械工业出版社

[《从零开始写一个 Jison 解析器(1/10)：Jison，不是 Json》](https://blog.csdn.net/hu_zhenghui/article/details/106046103) --  胡争辉
