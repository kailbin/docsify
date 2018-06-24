# OPERATORS

## Binary operators

Prometheus 的查询语言支持基本的逻辑和算术运算符。对于两个瞬时矢量之间的操作，可以修改匹配行为。

### 算术二元运算符（Arithmetic binary operators）

Prometheus 有以下几种二元算术运算符：

- `+` 加(addition)
- `-` 减(subtraction)
- `*` 乘(multiplication)
- `/` 除(division)
- `%` 模(modulo)
- `^` 次方/幂(power/exponentiation)

二元算术运算符可以用在 **标量/标量**、**矢量/标量**、**矢量/矢量** 值之间进行计算。

**在两个标量之间**，行为是显而易见的：两个标量的操作结果还是标量。

**在瞬时矢量和标量之间**，运算符被应用于矢量中每个数据样本的值。例如：如果时间序列瞬时矢量乘以2，则结果是原始矢量的每个样本值乘以2。

**在两个瞬时矢量之间**，将二元算术运算符应用于左侧矢量和右侧矢量中匹配的元素。操作之后的结果矢量中，度量指标名称被删除。没有找到右侧矢量中的匹配条目的条目不是结果的一部分。


### Comparison binary operators

Prometheus 有以下二元比较运算符：

- `==` 相等(equal)
- `!=` 不等(not-equal)
- `>` 大于(greater-than)
- `<` 小于(less-than)
- `>=` 大于等于(greater-or-equal)
- `<=` 小于等于(less-or-equal)


比较运算符可以在**标量/标量**，**矢量/标量**和**矢量/矢量**之间进行比较。比较运算符的默认行为是进行过滤。当在运算符后面提供bool值时，结果将返回0或1，而不是过滤。

Between two scalars, the bool modifier must be provided and these operators result in another scalar that is either 0 (false) or 1 (true), depending on the comparison result.
在两个标量之间，必须提供bool修饰符，并根据比较结果，这些运算符产生另一个标量0或者1（真）。

Between an instant vector and a scalar, these operators are applied to the value of every data sample in the vector, and vector elements between which the comparison result is false get dropped from the result vector. If the bool modifier is provided, vector elements that would be dropped instead have the value 0 and vector elements that would be kept have the value 1.
在瞬时矢量和标量之间，将这些运算符应用于矢量中的每个数据样本的值，并将比较结果为假的矢量元素从结果矢量中删除。, 如果提供了bool修饰符，则将被删除的矢量元素的值为0，并且将保留的矢量元素的值为1。

Between two instant vectors, these operators behave as a filter by default, applied to matching entries. Vector elements for which the expression is not true or which do not find a match on the other side of the expression get dropped from the result, while the others are propagated into a result vector with their original (left-hand side) metric names and label values. If the bool modifier is provided, vector elements that would have been dropped instead have the value 0 and vector elements that would be kept have the value 1 with the left-hand side metric names and label values.
在两个即时矢量之间，这些运算符默认用作过滤器，用于匹配条目。, 表达式不正确或者在表达式的另一端找不到匹配的矢量元素将从结果中删除，而其他元素将传播到其原始（左侧）度量标准名称的结果矢量中，, 标签值。, 如果提供了bool修饰符，那么将被删除的矢量元素的值将为0，并且要保留的矢量元素的值为1，并使用左侧的度量标准名称和标签值。

### Logical/set binary operators

这些逻辑二元运算符仅可以在瞬时矢量之间使用：

- `and` (intersection)
- `or` (union)
- `unless` (complement)

vector1 and vector2 results in a vector consisting of the elements of vector1 for which there are elements in vector2 with exactly matching label sets. Other elements are dropped. The metric name and values are carried over from the left-hand side vector.
`vector1 and vector2`会生成一个由vector1的元素组成的矢量，vector1中的元素与vector2中的元素具有完全匹配的标签集合。其他元素被丢弃， 度量标准名称和值从左侧矢量中继承。

vector1 or vector2 results in a vector that contains all original elements (label sets + values) of vector1 and additionally all elements of vector2 which do not have matching label sets in vector1.
`vector1 or vector2`将生成一个矢量，该矢量包含vector1的所有原始元素（标记集+值），另外还包含vector1中没有匹配的标记集的vector2的所有元素。

vector1 unless vector2 results in a vector consisting of the elements of vector1 for which there are no elements in vector2 with exactly matching label sets. All matching elements in both vectors are dropped.
`vector1 unless vector2`产生由vector1的元素组成的矢量，vector1中元素中没有元素与精确匹配的标签集合。, 两个矢量中的所有匹配元素都被删除。

## Vector matching
Operations between vectors attempt to find a matching element in the right-hand side vector for each entry in the left-hand side. There are two basic types of matching behavior: One-to-one and many-to-one/one-to-many.
矢量之间的操作试图在左侧的每个条目的右侧矢量中找到匹配元素。, 匹配行为有两种基本类型：一对一和多对一/一对多。

### One-to-one vector matches

One-to-one finds a unique pair of entries from each side of the operation. In the default case, that is an operation following the format vector1 <operator> vector2. Two entries match if they have the exact same set of labels and corresponding values. The ignoring keyword allows ignoring certain labels when matching, while the on keyword allows reducing the set of considered labels to a provided list:
一对一从操作的每一边找到一对独特的条目。, 在默认情况下，这是一个格式为vector1 <operator> vector2的操作。, 如果两个条目具有完全相同的一组标签和相应的值，则两个条目匹配。,`ignoring`关键字允许在匹配时忽略某些标签，而`on`关键字允许将所考虑的标签集合减少到提供的列表：

```
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>
```

输出示例:
```
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
```

查询语句:
```
method_code:http_errors:rate5m{code="500"} / ignoring(code) method:http_requests:rate5m
```

This returns a result vector containing the fraction of HTTP requests with status code of 500 for each method, as measured over the last 5 minutes. Without ignoring(code) there would have been no match as the metrics do not share the same set of labels. The entries with methods put and del have no match and will not show up in the result:
这将返回一个结果矢量，其中包含每个方法的状态码为500的HTTP请求的一小部分，如过去5分钟所测量的。, 在不忽略（代码）的情况下，将不会有任何匹配，因为度量标准不会共享同一组标签。, 方法put和del的条目不匹配，不会显示在结果中：

```
{method="get"}  0.04            //  24 / 600
{method="post"} 0.05            //   6 / 120
```

### Many-to-one and one-to-many vector matches

Many-to-one and one-to-many matchings refer to the case where each vector element on the "one"-side can match with multiple elements on the "many"-side. This has to be explicitly requested using the group_left or group_right modifier, where left/right determines which vector has the higher cardinality.
多对一和一对多匹配指的是“一”侧的每个矢量元素可以与“多”侧的多个元素匹配的情况。, 这必须使用`group_left`或`group_right`修饰符明确要求，其中左/右确定哪个矢量具有更高的基数。

```
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>
```

The label list provided with the group modifier contains additional labels from the "one"-side to be included in the result metrics. For on a label can only appear in one of the lists. Every time series of the result vector must be uniquely identifiable.

Grouping modifiers can only be used for comparison and arithmetic. Operations as and, unless and or operations match with all possible entries in the right vector by default.

Example query:
```
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m
```
In this case the left vector contains more than one entry per method label value. Thus, we indicate this using group_left. The elements from the right side are now matched with multiple elements with the same method label on the left:

```
{method="get", code="500"}  0.04            //  24 / 600
{method="get", code="404"}  0.05            //  30 / 600
{method="post", code="500"} 0.05            //   6 / 120
{method="post", code="404"} 0.175           //  21 / 120
```
Many-to-one and one-to-many matching are advanced use cases that should be carefully considered. Often a proper use of ignoring(<labels>) provides the desired outcome.



## 聚合操作（Aggregation operators）

Prometheus支持以下内置的聚合运算符，它们可用于聚合单个瞬时矢量的元素，从而生成包含聚合元素较少的新矢量：

- `sum` 计算总和(calculate sum over dimensions)
- `min` 选择最小(select minimum over dimensions)
- `max` 选择最大(select maximum over dimensions)
- `avg` 计算平均(calculate the average over dimensions)
- `stddev` 计算标准差(calculate population standard deviation over dimensions)
- `stdvar` 计算方差(calculate population standard variance over dimensions)
- `count` 计算数量(count number of elements in the vector)
- `count_values` 局算具有相同元素值的数量(count number of elements with the same value)
- `bottomk` 最小的k个样本值(smallest k elements by sample value)
- `topk` 最大的k个样本值(largest k elements by sample value)
- `quantile` 计算分位数(calculate φ-quantile (0 ≤ φ ≤ 1) over dimensions)

这些运算符可以用于聚合所有标签维度，也可以通过包含`without`或者`by`子句来保留指定的维度。

```
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
```

`parameter`只对`count_values`，`quantile`，`topk`和`bottomk`是必需的。
`without`从结果向量中删除列出的标签，而所有其他标签都保留到输出。
`by`会做相反的处理，并且会丢弃没有在`by`子句中列出的标签。

`count_values` outputs one time series per unique sample value. 
Each series has an additional label. 
The name of that label is given by the aggregation parameter, and the label value is the unique sample value. 
The value of each time series is the number of times that sample value was present.
`count_values`每个唯一样本值输出一个时间序列。
每个系列都有一个附加标签。
该标签的名称由聚合参数给出，标签值是唯一的样本值。
每个时间序列的值是样本值出现的次数。


topk and bottomk are different from other aggregators in that a subset of the input samples, including the original labels, are returned in the result vector. by and without are only used to bucket the input vector.

Example:

If the metric http_requests_total had time series that fan out by application, instance, and group labels, we could calculate the total number of seen HTTP requests per application and group over all instances via:
```
sum(http_requests_total) without (instance)
```
Which is equivalent to:
```
sum(http_requests_total) by (application, group)
```
If we are just interested in the total of HTTP requests we have seen in all applications, we could simply write:
```
sum(http_requests_total)
```
To count the number of binaries running each build version we could write:
```
count_values("version", build_version)
```
To get the 5 largest HTTP requests counts across all instances we could write:
```
topk(5, http_requests_total)
```

## Binary operator precedence
The following list shows the precedence of binary operators in Prometheus, from highest to lowest.

- `^`
- `*`, `/`, `%`
- `+`, `-`
- `==`, `!=`, `<=`, `<`, `>=`, `>`
- `and`, `unless`
- `or`

Operators on the same precedence level are left-associative. For example, 2 * 3 % 2 is equivalent to (2 * 3) % 2. However ^ is right associative, so 2 ^ 3 ^ 2 is equivalent to 2 ^ (3 ^ 2).

优先级相同的运算符是左关联的。, 例如，2 * 3％2相当于（2 * 3）％2。然而^是右结合的，所以2 ^ 3 ^ 2相当于2 ^（3 ^ 2）。

