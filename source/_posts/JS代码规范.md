---
title: JS代码规范
date: 2016-07-05 15:41:19
tags:
---
ESlint：ECMAScript/JavaScript代码的分析工具，根据编写的规则检测代码语法和风格错误
* 规则可配置性高：可设置「警告」、「错误」两个 error 等级，或者直接禁用；
* 包含代码风格检测的规则；
* 支持插件扩展、自定义规则。

### 安装

通过npm安到本地node_modules

	cd到工程目录
	npm i --save-dev eslint eslint-plugin-react eslint-plugin-react-native

### 使用

	cd到工程目录
	创建或导入配置文件：eslint --init
	开始分析代码：eslint test.js test2.js或eslint 目录/

<!-- more -->

### 配置
主要有两种配置方式：

* 直接在代码文件中配置
* 通过配置文件配置

[http://eslint.org/docs/user-guide/configuring](http://eslint.org/docs/user-guide/configuring)

#### 1.直接在代码文件中配置
禁用 ESLint：

``` bash
/* eslint-disable */
var obj = { key: 'value', }; // I don't care about IE8  
/* eslint-enable */
```

禁用一条规则：

``` bash
/*eslint-disable no-alert */
alert('doing awful things');  
/* eslint-enable no-alert */
```

调整规则：

``` bash
/* eslint no-comma-dangle:1 */
// Make this just a warning, not an error
var obj = { key: 'value', }  
```

#### 2.通过配置文件配置
可在 .eslintrc.* 文件或package.json中添加eslintConfig模块配置
eslinttrc可以识别的文件：

1. .eslintrc.js
2. .eslintrc.yaml
3. .eslintrc.yml
4. .eslintrc.json
5. .eslintrc
6. package.json

注：当同一目录包含多个以上文件时，eslint只会使用其中一个，使用顺序是上面从上往下。当你在根目录下分析代码时，eslint会使用根目录下的.eslintrc.\*当切换到子目录下分析代码时，eslint会使用子目录下的.eslintrc.*

``` bash
/* eslint-disable */
module.exports = {

  "env": {
    "es6": true,
    "node": true
  },//定义使用环境

  // extends: 'airbnb',

  plugins: [
    'react',
    'react-native',
  ],//规则插件

  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    },//指定附加的语法特性
    "sourceType": "module"//默认为"script"，如果使用ECMAScript模块，设置为"module"
  },//指定分析器支持选项

  "globals": {
        "var1": true,
        "var2": false
    },

  "rules": {

    //react-native插件规则
    "react-native/no-unused-styles": "error",
    "react-native/split-platform-components": "error",
    "react-native/no-inline-styles": "error",
    // "react-native/no-color-literals": "error",

    //容易出错的
    "no-cond-assign": ["error", "always"],//x=0不再是有效布尔表达式
    "no-console": "error",//禁用控制台输出
    "no-constant-condition": "error",//判断和循环的布尔表达式，禁用常量表达式
    "no-dupe-args": "error",//禁止方法有相同参数名
    "no-dupe-keys": "error",//禁止对象有相同属性名
    "no-duplicate-case": "error",//禁止switch中有相同的case
    "no-empty": "error",//禁止空的块声明
    "no-extra-semi": "error",//去除多余的分号,比如“;;”
    "no-func-assign": "error",//禁止重定义方法
    "no-inner-declarations": "error",//禁止在块中定义方法
    "no-invalid-regexp": "error",//禁止错误的正则表达式
    "no-irregular-whitespace": "error",//禁止不合法空格
    "no-obj-calls": "error",//禁止Math()、JSON()等，应为它们本来就是object
    "no-prototype-builtins": "error",//禁止 var hasBarProperty = foo.hasOwnProperty("bar");应该 var hasBarProperty = {}.hasOwnProperty.call(foo, "bar");
    "no-regex-spaces": "error",//正则表达式禁止连续空格/foo   bar/应该是/foo {3}bar/
    "no-sparse-arrays": "error",//禁止[,]，[ "red",, "blue" ]这种不插入元素行为
    "no-unreachable": "error",//禁止在return, throw, break, and continue后的可能执行不到的代码，比如{x = 1;return x;},而{return x；x = 1;}可以
    "no-unsafe-finally": "error",//禁止在finally中使用return, throw, break, or continue
    "use-isnan": "error",//禁止 foo == NaN，应该 isNaN(foo)
    "valid-typeof": "error",//禁止不合法的 typeof，http://eslint.org/docs/rules/valid-typeof
    // "valid-jsdoc": "error",//强制添加JSDoc方法说明
    // "no-unexpected-multiline": "error"
    // "no-debugger": "error"//禁用debugger语句

    //好的习惯
    // "accessor-pairs": "error",//强制属性添加 getter/setter
    "array-callback-return": "error",//强制array的迭代回调函数有return
    // "block-scoped-var": "error",//禁止在局域块中声明变量
    // "complexity": ["error", 2],//限制控制语句的最大级数
    // "consistent-return": "error",//禁止空return
    "curly": "error",//禁止 if (foo) foo++;等，必须 if (foo) {foo++;}
    "default-case": "error",//switch必须要有default
    "dot-notation": "error",//禁止 foo["bar"]，通过 foo.bar或 foo[bar]
    "eqeqeq": ["error", "always"],//禁止用“==”, “!=”, 用“===”和“!==”
    "no-case-declarations": "error",//case/default后必须加“{}”构成块
    "no-div-regex": "error",//不允许有分歧的正则表达式
    "no-empty-function": "error",//禁止空方法
    "no-empty-pattern": "error",//禁止空解构赋值
    "no-eq-null": "error",//只能使用“===”判断null，因为使用 foo == null，foo为undefined也会通过
    "no-extra-bind": "error",//禁止不必要的bind()
    "no-fallthrough": "error",//不允许忘了加 break
    "no-invalid-this": "error",//不允许this关键字在class域之外出现
    "no-loop-func": "error",//禁止在循环中定义函数
    // "no-magic-numbers": "warn",//magic-numbers应该消失了
    "no-new-wrappers": "error",//禁止对 String, Number, Boolean 使用new
    "no-new-func": "error",//禁止对func使用new
    "no-param-reassign": "error",//禁止给函数参数再赋值
    "no-redeclare": "error",//不可以重定义变量
    "no-return-assign": "error",//return foo = bar + 2;赋值是多余的，要么return bar + 2;要么return foo === bar + 2;
    "no-self-assign": "error",//禁止foo = foo;
    "no-self-compare": "error",//禁止x === x
    "no-unused-expressions": "error",//不要有无用表达式

    //strict模式
    // "strict": ["error", "global"]

    //关于变量
    "init-declarations": ["error", "always"],//强制初始化变量
    "no-undef": ["warn", { "typeof": true}],//禁止未定义变量
    "no-unused-vars": "warn",//未使用变量
    // "no-use-before-define": "error",//不能先使用后定义变量

    //编程风格
    "comma-dangle": ["error", "never"],//只在元素间加逗号
    "array-bracket-spacing": ["error", "never"],//方括号内无空格
    "block-spacing": "error",//block内部空格 { return true; }
    "brace-style": "error",//http://eslint.org/docs/rules/brace-style
    "camelcase": ["warn", {properties: "never"}],//驼峰命名,除对象属性外
    "comma-spacing": ["error", { "before": false, "after": true }],//“,”后要有空
    "comma-style": ["error", "last"],//“,”只能打在末尾
    "computed-property-spacing": ["error", "never"],//属性取值无空格，不要obj[ foo ]，应该obj[foo]
    "consistent-this": ["error", "self"],//this只能赋值给名为self的变量
    "eol-last": "error",//代码结束于新的一行
    "indent": ["error", 2],//2空格缩进
    "key-spacing": ["error", { "beforeColon": false }],//键值对“:”后必须加一个空格
    "keyword-spacing": ["error", { "before": false, "after": true }],//关键字前无空格，关键字后有空格
    "object-curly-newline": ["error", "always"],//悬挂式定义对象
    "quote-props": ["error", "as-needed", { "numbers": true }],//属性名无引号
    // "quotes": ["error", "double"],//双引号
    "semi": ["error", "always"],//必须分号
    "space-before-blocks": "error",//"{"前面必须加空格
    "space-infix-ops": "error",//运算符之间加空格

    //更多规则查看：http://eslint.org/docs/rules/
  }
};
```
### 4.集成
#### atom集成
安装linter, 安装linter-eslint:是eslint到linter的接口，eslint分析代码，linter将分析结果显示在atom中
