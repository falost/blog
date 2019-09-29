
### Variables：变量

使用$name或${name:default}可以插入变量的值。当变量未赋值时（如），将插入其缺省值或空字符串。 当varibale未知（即，其名称未定义）时，将插入变量的名称，并将其转换为「Placeholder」。

可以使用的「Variable」如下：

`TM_SELECTED_TEXT`：当前选定的文本或空字符串；

***注：选定后通过在命令窗口点选「插入代码片段」插入。***

`TM_CURRENT_LINE`：当前行的内容；

`TM_CURRENT_WORD`：光标所处单词或空字符串

***注：所谓光标一般为文本输入处那条闪来闪去的竖线，该项可定制。单词使用 VSCode 选词（Word Wrap）器选择。你最好只用它选择英文单词，因为这个选择器明显没有针对宽字符优化过，它甚至无法识别宽字符的标点符号。***

`TM_LINE_INDEX`：行号（从零开始）；

`TM_LINE_NUMBER`：行号（从一开始）；

`TM_FILENAME`：当前文档的文件名；

`TM_FILENAME_BASE`：当前文档的文件名（不含后缀名）；

`TM_DIRECTORY`：当前文档所在目录；

`TM_FILEPATH`：当前文档的完整文件路径；

`CLIPBOARD`：当前剪贴板中内容。

`CURRENT_YEAR`: 当前年份；

`CURRENT_YEAR_SHORT`: 当前年份的后两位；

`CURRENT_MONTH`: 格式化为两位数字的当前月份，如 02；

`CURRENT_MONTH_NAME`: 当前月份的全称，如 July；

`CURRENT_MONTH_NAME_SHORT`: 当前月份的简称，如 Jul；

`CURRENT_DATE`: 当天月份第几天；

`CURRENT_DAY_NAME`: 当天周几，如 Monday；

`CURRENT_DAY_NAME_SHORT`: 当天周几的简称，如 Mon；

`CURRENT_HOUR`: 当前小时（24 小时制）；

`CURRENT_MINUTE`: 当前分钟；

`CURRENT_SECOND`: 当前秒数。

**注：这些都是变量名，不是宏，在实际使用的时要加上 $ 符。**

##### 使用案例代码：

```
"Add multi-line comments": {
  "scope": "javascript,typescript,vue",
  "prefix": "funz",
  "body": [
    "/**",
     " * @name ${1}",
    " * @desc $2",
    " * @author Falost",
    " * @time $CURRENT_YEAR年$CURRENT_MONTH月$CURRENT_DATE日 $CURRENT_HOUR:$CURRENT_MINUTE:$CURRENT_SECOND $CURRENT_DAY_NAME",
    " * @param {${4|Object,String,Number,Array,Function|}} {$5}",
    " * @return ${6: {*}}",
    " */$10"
  ],
  "description": "Add multi-line comments"
}
```

##### 使用效果代码：

```
/**
  * @name
  * @desc
  * @author Falost
  * @time 2019年04月20日 14:58:47 星期六
  * @param {Object} {}
  * @return  {*}
  */
```

参考资料：https://code.visualstudio.com/docs/editor/userdefinedsnippets
