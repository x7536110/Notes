作者学习JavaScript的目的是为了能够掌握Node.Js进行后台开发，因此文中有不正确之处欢迎指正  
本文能够以代码展示的地方不会过多用语言描述。  

# 目录
# 第一章-JS概述
- JavaScript和Java没有关系
- Js是一门面向对象的函数式风格的编程语言
- Js有弱类型的特性
- Js是语言名，标准版本是ES(ES3,ES5...)
- Js运行于浏览器的Js解释器上
# 第二章-词法结构
- JS程序使用Unicode字符集编写
- JS程序大小写敏感
- 允许使用6个ASCII字符转义Unicode内码
- 标识符以字母/下划线/美元符号开头
- 保留字列表
    ```js
    break       delete  function    return  typeof
    case        do      if          switch  var
    catch       else    in          this    void
    continue    false   instanceof  throw   while
    debugger    finally new         true    with
    default     for     null        try     class
    const       enum    export      import  super   
    /* strict模式下的保留字 */
    implements  let     private     public  yield
    interface   package protected   static
    arguments   eval 
    ```
- 应避免使用的标识符列表
    ```js
    arguments   encodeURI           Infinity    Number      RegExp
    Array       encodeURIComponent  isFinite    Object      String
    Boolean     Error               isNaN       parseFloat  SyntaxError
    Date        eval                JSON        parseInt    TypeError
    EvalError   decodeURI           Math        RangeError  undefined
    Function    decodeURIComponent  NaN         URIError    ReferenceError
    ```
- 分号是可选的


# 第三章-类型、值和变量

- JS数据类型分为两类
    - 原始类型(primitive type)
        - 数字
        - 字符串
        - 布尔
    - 对象类型(object type)
- 特殊类型:null,undefined
- JS会进行自动类型转换
- JS中所有的数字都是64位浮点数
- 支持hex与dec模式表示，ES6禁止8进制表示
- JS运算溢出时会用Infinity来表示
- JS运算除0时会用NaN来表示
- infinite和NaN无法判等
- 字符串可直接用`+`号连接
- 字符串操作是拷贝操作，返回新串
- `==`运算符比较值，`===`运算符比较值比较类型
- `null==undefined => true`,`null === undefined => false`
- 对字符串、数字等取属性时，会先生成一个"包装对象",然后调用临时对象进行操作
- 对象的比较是比较引用，引用相同则为真
- 对象直接赋值是赋值引用，复制对象需要显式的进行成员复制操作
- 类型转换  
    |值\\转换目标|字符串|数字|布尔|对象|
    |---|---|---|---|---|
    |undefined|"undefined"|NaN|false|throws TypeError|
    |null|"null"|0|false|throws TypeError|
    |true|"true"|1|true|new Bollean(true)|
    |fasle|"false"|0|false|new Bollean(false)|
    |''(空串)||0|false|new String('')|
    |'1.2'||1.2|true|new String('1.2')|
    |'one'||NaN|true|new String('one')|
    |0|"0"||false|new Number(0)|
    |-0|"0"||false|new Number(-0)|
    |NaN|"NaN"||false|new Number(NaN)|
    |Infinity|"Infinity"||true|new Number(Infinity)|
    |-Infinity|"-Infinity"||true|new Number(-Infinity)|
    |1|"1"||true|new Number(1)|
    |{}(任意对象)|||true||
    |[]\(任意数组\)|''|0|true||
    |['1']|'1'|1|true||
    |['a']|'a'(使用join())|NaN|true||
    |function(){}||NaN|true||