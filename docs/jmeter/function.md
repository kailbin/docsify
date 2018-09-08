函数 与 变量
-------



函数使用方式： `${__functionName(var1,var2,var3)` 

逗号转义 ：  `${__time(EEE\, d MMM yyyy)}`

变量引用语法： `${varibale}`





# 函数列表

| Type of function | Name                                                         | Comment                                                      | Since        |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ |
| Information      | [threadNum](http://jmeter.apache.org/usermanual/functions.html#__threadNum) | get thread number                                            | 1.X          |
| Information      | [samplerName](http://jmeter.apache.org/usermanual/functions.html#__samplerName) | get the sampler name (label)                                 | 2.5          |
| Information      | [machineIP](http://jmeter.apache.org/usermanual/functions.html#__machineIP) | get the local machine IP address                             | 2.6          |
| Information      | [machineName](http://jmeter.apache.org/usermanual/functions.html#__machineName) | get the local machine name                                   | 1.X          |
| Information      | [time](http://jmeter.apache.org/usermanual/functions.html#__time) | return current time in various formats                       | 2.2          |
| Information      | [timeShift](http://jmeter.apache.org/usermanual/functions.html#__timeShift) | return a date in various formats with the specified amount of seconds/minutes/hours/days added | 3.3          |
| Information      | [log](http://jmeter.apache.org/usermanual/functions.html#__log) | log (or display) a message (and return the value)            | 2.2          |
| Information      | [logn](http://jmeter.apache.org/usermanual/functions.html#__logn) | log (or display) a message (empty return value)              | 2.2          |
| Input            | [StringFromFile](http://jmeter.apache.org/usermanual/functions.html#__StringFromFile) | read a line from a file                                      | 1.9          |
| Input            | [FileToString](http://jmeter.apache.org/usermanual/functions.html#__FileToString) | read an entire file                                          | 2.4          |
| Input            | [CSVRead](http://jmeter.apache.org/usermanual/functions.html#__CSVRead) | read from CSV delimited file                                 | 1.9          |
| Input            | [XPath](http://jmeter.apache.org/usermanual/functions.html#__XPath) | Use an XPath expression to read from a file                  | 2.0.3        |
| Calculation      | [counter](http://jmeter.apache.org/usermanual/functions.html#__counter) | generate an incrementing number                              | 1.X          |
| Formatting       | [dateTimeConvert](http://jmeter.apache.org/usermanual/functions.html#__dateTimeConvert) | Convert a date or time from source to target format          | 4.0          |
| Calculation      | [digest](http://jmeter.apache.org/usermanual/functions.html#__digest) | Generate a digest (SHA-1, SHA-256, MD5...)                   | 4.0          |
| Calculation      | [intSum](http://jmeter.apache.org/usermanual/functions.html#__intSum) | add int numbers                                              | 1.8.1        |
| Calculation      | [longSum](http://jmeter.apache.org/usermanual/functions.html#__longSum) | add long numbers                                             | 2.3.2        |
| Calculation      | [Random](http://jmeter.apache.org/usermanual/functions.html#__Random) | generate a random number                                     | 1.9          |
| Calculation      | [RandomDate](http://jmeter.apache.org/usermanual/functions.html#__RandomDate) | generate random date within a specific date range            | 3.3          |
| Calculation      | [RandomFromMultipleVars](http://jmeter.apache.org/usermanual/functions.html#__RandomFromMultipleVars) | extracts an element from the values of a set of variables separated by \| | 3.1          |
| Calculation      | [RandomString](http://jmeter.apache.org/usermanual/functions.html#__RandomString) | generate a random string                                     | 2.6          |
| Calculation      | [UUID](http://jmeter.apache.org/usermanual/functions.html#__UUID) | generate a random type 4 UUID                                | 2.9          |
| Scripting        | [groovy](http://jmeter.apache.org/usermanual/functions.html#__groovy) | run a Groovy script                                          | 3.1          |
| Scripting        | [BeanShell](http://jmeter.apache.org/usermanual/functions.html#__BeanShell) | run a BeanShell script                                       | 1.X          |
| Scripting        | [javaScript](http://jmeter.apache.org/usermanual/functions.html#__javaScript) | process JavaScript (Nashorn)                                 | 1.9          |
| Scripting        | [jexl2](http://jmeter.apache.org/usermanual/functions.html#__jexl2) | evaluate a Commons Jexl2 expression                          | jexl2(2.1.1) |
| Scripting        | [jexl3](http://jmeter.apache.org/usermanual/functions.html#__jexl3) | evaluate a Commons Jexl3 expression                          | jexl3 (3.0)  |
| Properties       | [isPropDefined](http://jmeter.apache.org/usermanual/functions.html#__isPropDefined) | Test if a property exists                                    | 4.0          |
| Properties       | [property](http://jmeter.apache.org/usermanual/functions.html#__property) | 读取 JMeter 属性，`${__P(属性名,默认值)}`，命令行参数 `-J` 传入属性 | 2.0          |
| Properties       | [`P`](http://jmeter.apache.org/usermanual/functions.html#__P) | property 的缩写形式                                          | 2.0          |
| Properties       | [setProperty](http://jmeter.apache.org/usermanual/functions.html#__setProperty) | set a JMeter property                                        | 2.1          |
| Variables        | [split](http://jmeter.apache.org/usermanual/functions.html#__split) | Split a string into variables                                | 2.0.2        |
| Variables        | [eval](http://jmeter.apache.org/usermanual/functions.html#__eval) | evaluate a variable expression                               | 2.3.1        |
| Variables        | [evalVar](http://jmeter.apache.org/usermanual/functions.html#__evalVar) | evaluate an expression stored in a variable                  | 2.3.1        |
| Properties       | [isVarDefined](http://jmeter.apache.org/usermanual/functions.html#__isVarDefined) | Test if a variable exists                                    | 4.0          |
| Variables        | [`V`](http://jmeter.apache.org/usermanual/functions.html#__V) | 执行变量名表达式，如： `${__V(A${N})}`                       | 2.3RC3       |
| String           | [char](http://jmeter.apache.org/usermanual/functions.html#__char) | generate Unicode char values from a list of numbers          | 2.3.3        |
| String           | [changeCase](http://jmeter.apache.org/usermanual/functions.html#__changeCase) | Change case following different modes                        | 4.0          |
| String           | [escapeHtml](http://jmeter.apache.org/usermanual/functions.html#__escapeHtml) | Encode strings using HTML encoding                           | 2.3.3        |
| String           | [escapeOroRegexpChars](http://jmeter.apache.org/usermanual/functions.html#__escapeOroRegexpChars) | quote meta chars used by ORO regular expression              | 2.9          |
| String           | [escapeXml](http://jmeter.apache.org/usermanual/functions.html#__escapeXml) | Encode strings using XMl encoding                            | 3.2          |
| String           | [regexFunction](http://jmeter.apache.org/usermanual/functions.html#__regexFunction) | parse previous response using a regular expression           | 1.X          |
| String           | [unescape](http://jmeter.apache.org/usermanual/functions.html#__unescape) | Process strings containing Java escapes (e.g. \n & \t)       | 2.3.3        |
| String           | [unescapeHtml](http://jmeter.apache.org/usermanual/functions.html#__unescapeHtml) | Decode HTML-encoded strings                                  | 2.3.3        |
| String           | [urldecode](http://jmeter.apache.org/usermanual/functions.html#__urldecode) | Decode a application/x-www-form-urlencoded string            | 2.10         |
| String           | [urlencode](http://jmeter.apache.org/usermanual/functions.html#__urlencode) | Encode a string to a application/x-www-form-urlencoded string | 2.10         |
| String           | [TestPlanName](http://jmeter.apache.org/usermanual/functions.html#__TestPlanName) | Return name of current test plan                             | 2.6          |


# Read More
- [20. Functions and Variables](http://jmeter.apache.org/usermanual/functions.html)
- [详解JMeter函数和变量](https://www.cnblogs.com/MasterMonkInTemple/p/3442770.html)
