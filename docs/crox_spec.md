#Crox 语言规范 

版本(日期): 2/12/2014 11:09 AM

## 数据类型

JSON，但不支持 数组字面量、对象字面量、null

## 词法

### 保留字

js 的 ReservedWord、FutureReservedWord 和 `null`，具体包括：

`abstract` `boolean` `break` `byte` `case` `catch` `char` `class` `const` `continue` `debugger` `default` `delete` `do` `double` `else` `enum` `export` `extends` `final` `finally` `float` `for` `function` `goto` `if` `implements` `import` `in` `instanceof` `int` `interface` `let` `long` `native` `new` `package` `private` `protected` `public` `return` `short` `static` `super` `switch` `synchronized` `this` `throw` `throws` `transient` `try` `typeof` `var` `void` `volatile` `while` `with` `yield` `null`

### 标识符(id)

- 以字母或下划线开头，后面 `0` 或多个字母、数字、下划线。

- 但 **不能** 是保留字、 布尔。

- `root` 是一个特殊的标识符，表示数据。 

### 字符串(string)

`/"(?:[^"\\]|\\[\s\S])*"|'(?:[^'\\]|\\[\s\S])*'/ `

### 数值(number)

`/\d+(?:\.\d+)?(?:e-?\d+)?/`

### 布尔(boolean)

`true` `false`

### 空白 

`/\s+/` 无意义

### 文本(text)

任意字符序列，但不能包含 `{{`

并且，从倒数第 1 个字符向后看，不能是 {{ 

### 符号 

`!` `%` `&&` `(` `)` `*` `+` `-` `.` `/` `<` `<=` `=` `>` `>=` `[` `]` `||` `===` `!==`

`{{` `}}` `{{{` `}}}` `{{#if` `{{#each` `{{/if}}` `{{/each}}` `{{else}}`

## 语法

用 `BNF` 表示，参考 [巴科斯范式](http://zh.wikipedia.org/wiki/%E5%B7%B4%E7%A7%91%E6%96%AF%E8%8C%83%E5%BC%8F)

```
#start program;

program:
    statements
;

statements:
|   statements statement
;

statement:
    "{{#if" expr "}}" statements "{{/if}}"
|   "{{#if" expr "}}" statements "{{else}}" statements "{{/if}}"
|   "{{#each" expr string string? "}}" statements "{{/each}}"
|   "{{" set id "=" expr "}}"
|   "{{" expr "}}"
|   "{{{" expr "}}}"
|   text
|   "{{" include string "}}"
|   "{{#raw}}" rawtext "{{/raw}}"
;

id:
    realId 
|   set
; 

PrimaryExpression:
    string
|   number
|   boolean
|   id
|   "(" expr ")"
;

MemberExpression:
    PrimaryExpression
|   MemberExpression "." id
|   MemberExpression "[" expr "]" 
;

UnaryExpression:
    MemberExpression
|   "!" UnaryExpression
|   "-" UnaryExpression
; 

MultiplicativeExpression:
    UnaryExpression
|   MultiplicativeExpression "*" UnaryExpression
|   MultiplicativeExpression "/" UnaryExpression
|   MultiplicativeExpression "%" UnaryExpression
;

AdditiveExpression:
    MultiplicativeExpression
|   AdditiveExpression "+" MultiplicativeExpression
|   AdditiveExpression "-" MultiplicativeExpression
;

RelationalExpression:
    AdditiveExpression
|   RelationalExpression "<" AdditiveExpression
|   RelationalExpression ">" AdditiveExpression
|   RelationalExpression "<=" AdditiveExpression
|   RelationalExpression ">=" AdditiveExpression
;

EqualityExpression:
    RelationalExpression
|   EqualityExpression eq RelationalExpression
|   EqualityExpression ne RelationalExpression
;

LogicalAndExpression:
    EqualityExpression
|   LogicalAndExpression "&&" EqualityExpression
;

LogicalOrExpression:
    LogicalAndExpression
|   LogicalOrExpression "||" LogicalAndExpression;

expr:
    LogicalOrExpression
;

```

其中，不出现在产生式头部的名字，都是 `终结符`。

## 语义

按照 `js` 语义 (包括字面量取值、运算符优先级、运算规则等)

## 部分语法详细说明

### **"{{#each" expr string string? "}}" statements "{{/each}}";**

说明：两个 string 会声明一个名字为其值的变量，第一个表示 `值`，第二个表示 `索引` (可选)。

### **"{{" set id "=" expr "}}";**

说明：该语句会声明一个变量 `id`， 并赋值。 

### **"{{" expr "}}";**

说明：输出 `html 转义` 的 expr 的值。

### **"{{{" expr "}}}";**

说明：直接输出 expr 的值。

### **"{{#raw}}" rawtext "{{/raw}}"**

说明：保留原始内容，raw 标签里可以放大段文本,只要不包含{{/raw}}