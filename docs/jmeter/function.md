函数 与 变量
-------



函数使用方式： `${__functionName(var1,var2,var3)` 

逗号转义 ：  `${__time(EEE\, d MMM yyyy)}`

变量引用语法： `${varibale}`





# 函数列表

| Type of function | Name                                                         | Comment                                                      | Since        |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ |
| Information      | [threadNum](http://jmeter.apache.org/usermanual/functions.html#__threadNum) | 只是简单地返回当前线程的编号                                 | 1.X          |
| Information      | [samplerName](http://jmeter.apache.org/usermanual/functions.html#__samplerName) | 获取 抽样器 名称                                             | 2.5          |
| Information      | [machineIP](http://jmeter.apache.org/usermanual/functions.html#__machineIP) | 获取当前主机 IP                                              | 2.6          |
| Information      | [machineName](http://jmeter.apache.org/usermanual/functions.html#__machineName) | 获取当前主机名                                               | 1.X          |
| Information      | [time](http://jmeter.apache.org/usermanual/functions.html#__time) | 以多种格式返回当前时间 `${__time(format)}`                   | 2.2          |
| Information      | [timeShift](http://jmeter.apache.org/usermanual/functions.html#__timeShift) | return a date in various formats with the specified amount of seconds/minutes/hours/days added | 3.3          |
| Information      | [log](http://jmeter.apache.org/usermanual/functions.html#__log) | 会记录一条日志，并返回函数的输入字符串                       | 2.2          |
| Information      | [logn](http://jmeter.apache.org/usermanual/functions.html#__logn) | 会记录一条日志                                               | 2.2          |
| Input            | [StringFromFile](http://jmeter.apache.org/usermanual/functions.html#__StringFromFile) | 每次调用函数，都会从文件中读取下一行，大部分情况使用 `CSV Data Set Config` 配置元件 会更好 | 1.9          |
| Input            | [FileToString](http://jmeter.apache.org/usermanual/functions.html#__FileToString) | 读取整个文件                                                 | 2.4          |
| Input            | [CSVRead](http://jmeter.apache.org/usermanual/functions.html#__CSVRead) | 从 CSV 文件读取数据，大部分情况使用 `CSV Data Set Config` 配置元件 会更好 | 1.9          |
| Input            | [XPath](http://jmeter.apache.org/usermanual/functions.html#__XPath) | 读取XML文件，并在文件中寻找与指定XPath相匹配的地方。每调用函数一次，就会返回下一个匹配项。到达文件末尾后，会从头开始 `${__XPath(some.xml, //target/@name)}` | 2.0.3        |
| Calculation      | [counter](http://jmeter.apache.org/usermanual/functions.html#__counter) | 每次调用计数器函数都会产生一个新值，从1开始每次加1           | 1.X          |
| Formatting       | [dateTimeConvert](http://jmeter.apache.org/usermanual/functions.html#__dateTimeConvert) | Convert a date or time from source to target format          | 4.0          |
| Calculation      | [digest](http://jmeter.apache.org/usermanual/functions.html#__digest) | Generate a digest (SHA-1, SHA-256, MD5...)                   | 4.0          |
| Calculation      | [intSum](http://jmeter.apache.org/usermanual/functions.html#__intSum) | 计算两个或者更多 int 值的和                                  | 1.8.1        |
| Calculation      | [longSum](http://jmeter.apache.org/usermanual/functions.html#__longSum) | 计算两个或者更多 long 值的和                                 | 2.3.2        |
| Calculation      | [Random](http://jmeter.apache.org/usermanual/functions.html#__Random) | 返回指定最大值和最小值之间的随机数 `${__Random(1,10)}`       | 1.9          |
| Calculation      | [RandomDate](http://jmeter.apache.org/usermanual/functions.html#__RandomDate) | generate random date within a specific date range            | 3.3          |
| Calculation      | [RandomFromMultipleVars](http://jmeter.apache.org/usermanual/functions.html#__RandomFromMultipleVars) | extracts an element from the values of a set of variables separated by | 3.1          |
| Calculation      | [RandomString](http://jmeter.apache.org/usermanual/functions.html#__RandomString) | generate a random string                                     | 2.6          |
| Calculation      | [UUID](http://jmeter.apache.org/usermanual/functions.html#__UUID) | generate a random type 4 UUID                                | 2.9          |
| Scripting        | [groovy](http://jmeter.apache.org/usermanual/functions.html#__groovy) | run a Groovy script                                          | 3.1          |
| Scripting        | [BeanShell](http://jmeter.apache.org/usermanual/functions.html#__BeanShell) | run a BeanShell script                                       | 1.X          |
| Scripting        | [javaScript](http://jmeter.apache.org/usermanual/functions.html#__javaScript) | process JavaScript (Nashorn)                                 | 1.9          |
| Scripting        | [jexl2](http://jmeter.apache.org/usermanual/functions.html#__jexl2) | evaluate a Commons Jexl2 expression                          | jexl2(2.1.1) |
| Scripting        | [jexl3](http://jmeter.apache.org/usermanual/functions.html#__jexl3) | evaluate a Commons Jexl3 expression                          | jexl3 (3.0)  |
| Properties       | [isPropDefined](http://jmeter.apache.org/usermanual/functions.html#__isPropDefined) | 判断属性是否存在                                             | 4.0          |
| Properties       | [property](http://jmeter.apache.org/usermanual/functions.html#__property) | 读取 JMeter 属性，`${__P(属性名,默认值)}`，命令行参数 `-J` 传入属性 | 2.0          |
| Properties       | [`P`](http://jmeter.apache.org/usermanual/functions.html#__P) | property 的缩写形式                                          | 2.0          |
| Properties       | [setProperty](http://jmeter.apache.org/usermanual/functions.html#__setProperty) | 设置 JMeter 属性                                             | 2.1          |
| Variables        | [split](http://jmeter.apache.org/usermanual/functions.html#__split) | 切割字符串放入变量 `${__split("a,b,c",VAR)}`，此时 VAR_1=a，VAR_2=b，VAR_3=c | 2.0.2        |
| Variables        | [eval](http://jmeter.apache.org/usermanual/functions.html#__eval) | 执行一个字符串表达式，并返回执行结果                         | 2.3.1        |
| Variables        | [evalVar](http://jmeter.apache.org/usermanual/functions.html#__evalVar) | 执行字符串表达式 并 放入变量                                 | 2.3.1        |
| Properties       | [isVarDefined](http://jmeter.apache.org/usermanual/functions.html#__isVarDefined) | 测试变量是否存在                                             | 4.0          |
| Variables        | [`V`](http://jmeter.apache.org/usermanual/functions.html#__V) | 执行变量名表达式，如： `${__V(A${N})}`                       | 2.3RC3       |
| String           | [char](http://jmeter.apache.org/usermanual/functions.html#__char) | generate Unicode char values from a list of numbers          | 2.3.3        |
| String           | [changeCase](http://jmeter.apache.org/usermanual/functions.html#__changeCase) | Change case following different modes                        | 4.0          |
| String           | [escapeHtml](http://jmeter.apache.org/usermanual/functions.html#__escapeHtml) | Encode strings using HTML encoding                           | 2.3.3        |
| String           | [escapeOroRegexpChars](http://jmeter.apache.org/usermanual/functions.html#__escapeOroRegexpChars) | quote meta chars used by ORO regular expression              | 2.9          |
| String           | [escapeXml](http://jmeter.apache.org/usermanual/functions.html#__escapeXml) | Encode strings using XMl encoding                            | 3.2          |
| String           | [regexFunction](http://jmeter.apache.org/usermanual/functions.html#__regexFunction) | 通过正则表达式匹配获取内容                                   | 1.X          |
| String           | [unescape](http://jmeter.apache.org/usermanual/functions.html#__unescape) | Process strings containing Java escapes (e.g. \n & \t)       | 2.3.3        |
| String           | [unescapeHtml](http://jmeter.apache.org/usermanual/functions.html#__unescapeHtml) | Decode HTML-encoded strings                                  | 2.3.3        |
| String           | [urldecode](http://jmeter.apache.org/usermanual/functions.html#__urldecode) | URL 解码                                                     | 2.10         |
| String           | [urlencode](http://jmeter.apache.org/usermanual/functions.html#__urlencode) | URL 编码                                                     | 2.10         |
| String           | [TestPlanName](http://jmeter.apache.org/usermanual/functions.html#__TestPlanName) | 返回测试计划名                                               | 2.6          |


# Read More
- [20. Functions and Variables](http://jmeter.apache.org/usermanual/functions.html)
- [详解JMeter函数和变量](https://www.cnblogs.com/MasterMonkInTemple/p/3442770.html)
