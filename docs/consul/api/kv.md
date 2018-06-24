
# KV Store Endpoints

`/kv`接口用于进行的简单键/值存储，可以用于存储服务配置或其他元数据。

**需要注意的是，每个数据中心都有自己的KV存储，并且在数据中心之间不会进行复制**。
如果你对数据中心之间的复制感兴趣，请参看 [Consul Replicate](https://github.com/hashicorp/consul-replicate) 项目.

~> Values 最大支持 512kb 的数据.

对于多个键的更新，请参见 [transaction](api/txn.md).

## Read Key

This endpoint returns the specified key. If no key exists at the given path, a 404 is returned instead of a 200 response.

For multi-key reads, please consider using [transaction](/api/txn.html).

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `GET`  | `/kv/:key`                   | `application/json`         |

The table below shows this endpoint's support for
[blocking queries](/api/index.html#blocking-queries),
[consistency modes](/api/index.html#consistency-modes), and
[required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required |
| ---------------- | ----------------- | ------------ |
| `YES`            | `all`             | `key:read`   |

### Parameters

- `key` `(string: "")` - 指定读取 Key 的路径.
- `dc` `(string: "")` - 指定要查询的数据中心。 如果不指定，默认是`dc1`. 该参数作为url的一部分被指定。.
- `recurse` `(bool: false)` - 是否使用递归查询，如果指定， `key` 作为前缀匹配，而不是整体匹配. 该参数作为url的一部分被指定。.
- `raw` `(bool: false)` - 只响应键的原始值，没有任何编码或元数据。该参数作为url的一部分被指定。.
- `keys` `(bool: false)` - 只返回键，不返回值和元数据. 同样是前缀匹配. 该参数作为url的一部分被指定。.
- `separator` `(string: '/')` - 指定要用作递归查找的分隔符的字符。 该参数作为url的一部分被指定。.

> 使用参数recurse，Key `asd`和`asdasd`都会查出来，貌似跟分隔符没有关系

### Sample Request

```
$ curl  https://consul.rocks/v1/kv/my-key
```

### Sample Response

#### Metadata Response

```
[
  {
    "CreateIndex": 100,
    "ModifyIndex": 200,
    "LockIndex": 200,
    "Key": "zip",
    "Flags": 0,
    "Value": "dGVzdA==",
    "Session": "adf4238a-882b-9ddc-4a9d-5b6758e4159e"
  }
]
```

- `CreateIndex` 创建数据的时候内部的索引值.
- `ModifyIndex` 修改的时候该索引会增加. 它和响应头中的`X-Consul-Index`是一样的，而且可以通过加上`?index`请求参数来创建阻塞查询。甚至可以对KV存储的整个子树执行阻塞查询，如果使用了`?recurse`参数，则响应头中的 `X-Consul-Index`对应于前缀中最新的`ModifyIndex`，使用`?index`将阻塞直到该前缀内的任何键被更新。
- `LockIndex` 是这个 Key 成功被锁定的次数。如果锁被持有，`Session`键 只能被拥有锁的会话操作。
- `Key` 查询Key的完整名.
- `Flags` 是一个的无符号整数，任何条目中都有. 客户端可以使用它并赋予其意义.
- `Value` Base64编码的值.

#### Keys Response

When using the `?keys` query parameter, the response structure changes to an array of strings instead of an array of JSON objects. 
Listing `/web/` with a `/` separator may return:

```
[
  "/web/bar",
  "/web/foo",
  "/web/subdir/"
]
```

当您不需要值或标志（Flags），希望实现一个密钥空间资源管理器时，使用键列表接口可能是合适的。

#### Raw Response

当使用 `?raw` 参数时, 相应类型不是 `application/json`, 而是设置时候的类型.

```
)k������z^�-�ɑj�q����#u�-R�r��T�D��٬�Y��l,�ιK��Fm��}�#e��
```

(Yes, 这里故意用乱码来展示数据)

## Create/Update Key

This endpoint

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `PUT`  | `/kv/:key`                   | `application/json`         |

即使相应类型是 `application/json`, 在创建或者更新成功的时候也会返回 `true` 或 `false`.

The table below shows this endpoint's support for [blocking queries](/api/index.html#blocking-queries), [consistency modes](/api/index.html#consistency-modes), and [required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required |
| ---------------- | ----------------- | ------------ |
| `NO`             | `none`            | `key:write`  |

### Parameters

- `key` `(string: "")` - 指定读取 Key 的路径.
- `dc` `(string: "")` - 指定要查询的数据中心。 如果不指定，默认是`dc1`. 该参数作为url的一部分被指定。.
- `flags` `(int: 0)` - 指定一个介于 `0` 和 `(2^64)-1` 之间的无符号整数. 客户端可以使用它并赋予其意义. 该参数作为url的一部分被指定。.

- `cas` `(int: 0)` - 指定`Check-And-Set`操作. 这对于构建复杂的同步阻塞原语是非常有用的。如果cas设置为是 0，Consul只有在key不存在时候才会创建。如果cas设置的不是0，只有index与 `ModifyIndex` 一致的时候才会更新。

- `acquire` `(string: "")` - 获取锁。该功能在领导选举的时候会有用。如果锁没有被占用，并且会话是有效的，`LockIndex`属性会增加，并且绑定`Session`属性。key不存在也可以获取锁。
如果锁已经被指定的Session占用，更新Key内容的时候`LockIndex`不会增加。这使得当前的锁持有者无需放弃锁就可以更新密钥内容。


- `release` `(string: "")` - 释放锁。该参数和`?acquire=`配合使用。该操作不会修改 `LockIndex` 值，但是会删除`Session` key，该操作必须被指定会话执行（管理页面可以强制释放）。

### Sample Payload

The payload is arbitrary, and is loaded directly into Consul as supplied.

### Sample Request

``` 
$ curl  --request PUT  --data @contents  https://consul.rocks/v1/kv/my-key
```

### Sample Response

```json
true
```

## Delete Key

此接口删除单个键或所有共前缀匹配的键。

| Method   | Path                         | Produces                   |
| -------- | ---------------------------- | -------------------------- |
| `DELETE` | `/kv/:key`                   | `application/json`         |

The table below shows this endpoint's support for [blocking queries](/api/index.html#blocking-queries), [consistency modes](/api/index.html#consistency-modes), and [required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required |
| ---------------- | ----------------- | ------------ |
| `NO`             | `none`            | `key:write`  |

### Parameters

- `recurse` `(bool: false)` - key作为匹配前缀，匹配到了都进行删除。不指定该参数则是精确匹配删除

- `cas` `(int: 0)` - 指定`Check-And-Set`操作. 这对于构建复杂的同步阻塞原语是非常有用的。与 `PUT`不同时, `cas`的值必须大于0,0不会删除任何键。如果cas设置的不是0，只有index与 `ModifyIndex` 一致的时候才会删除。


### Sample Request

```
$ curl  --request DELETE https://consul.rocks/v1/kv/my-key
```

### Sample Response

```json
true
```