


使用Ehcache，可以通过**编程**或**XML**配置`CacheManager`和`Cache`。

# 通过编程配置（Programmatic configuration）

``` java
CacheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder() // ①
    .withCache("preConfigured",
        CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class, ResourcePoolsBuilder.heap(10)))  // ②
    .build();  // ③
cacheManager.init(); // ④

Cache<Long, String> preConfigured = cacheManager.getCache("preConfigured", Long.class, String.class);  // ⑤

Cache<Long, String> myCache = cacheManager.createCache("myCache", // ⑥
    CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class, ResourcePoolsBuilder.heap(10)));

myCache.put(1L, "da one!"); // ⑦
String value = myCache.get(1L); // ⑧

cacheManager.removeCache("preConfigured"); // ⑨

cacheManager.close(); // ⑩
```

1. 这个静态方法 `org.ehcache.config.builders.CacheManagerBuilder.newCacheManagerBuilder` 返回一个`org.ehcache.config.builders.CacheManagerBuilder` 的实例.

2. 使用构建器构建一个 `Cache` 别名是 "preConfigured". 调用 `cacheManager.build()` 的时候，这个缓存会被创建. 第一个 `String` 参数是cache的别名, 可以通过别名从 `CacheManager`中获取缓存. 第二个参数 `org.ehcache.config.CacheConfiguration`用来配置 `Cache`. 我们使用 `org.ehcache.config.builders.CacheConfigurationBuilder`的静态方法`newCacheConfigurationBuilder()`创建一个默认的配置.

3. 最后, 调用 `build()` 实例化`CacheManager`, 这里并没有初始化(`build(true)`).

4. 在使用`CacheManager`之前需要先初始化它，可以使用一下两种方法进行初始化:  
    - 使用 `CacheManager`的实例调用 init方法（`cacheManager.init()`）  
    - 在构建时候，`CacheManagerBuilder.build(boolean init)`方法参数设置为true  

5. 通过**缓存别名**、**Key的类型**、**Value的类型**从 `CacheManager`获取缓存实例. **缓存别名**、**Key的类型**、**Value的类型**必须与定义的时候完全匹配，否则无法获取到`Cache`实例.

6. `CacheManager`也可以用来创建新的缓存实例，就像第二步中那样。

7. 使用新创建的Cache实例存储数据，键值的类型必须与定义的时候保持一致。键是唯一的。

8. 调用`cache.get(key)`方法从缓存中获取数据. 如果没有键关联的值，会返回 `null`.

9. 通过调用`CacheManager.removeCache(String)`删除一个缓存. `CacheManager`不仅会删除缓存的引用，还会释放本地保存的缓存资源，此时对该缓存的引用变得不可用.

10. 调用 `CacheManager.close()` 释放所有缓存的资源（内存、线程...）.

下面是一个简单的版本，有三件比较重要的事：
``` java
try(CacheManager cacheManager = newCacheManagerBuilder()  // 1
  .withCache("preConfigured", newCacheConfigurationBuilder(Long.class, String.class, heap(10))) // 2
  .build(true)) {  // 3
  // Same code as before [...]
}
```
1.  `CacheManager` 实现了 `Closeable` 接口，所以可以被 `try-with-resources`语法自动释放掉. 
2. 通过静态导入，直接使用静态方法
3. 构建的时候自动初始化.


## 核心概念
1. CacheManager(CacheManagerBuilder)
    - 默认对象大小(withDefaultSizeOfMaxObjectSize)
    - 默认对象深度(withDefaultSizeOfMaxObjectGraph)
    - ...
2. ResourcePools(ResourcePoolsBuilder)
    - 堆内存
    - 堆外内存
    - 磁盘
    - ....
3. CacheConfiguration(CacheConfigurationBuilder)
    - 过期策略
    - 序列化
    - ...
4. Cache(通过 CacheManager 创建) 



# 通过XML配置(XML configuration)

``` xml
<config
    xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance'
    xmlns='http://www.ehcache.org/v3'
    xsi:schemaLocation="http://www.ehcache.org/v3 http://www.ehcache.org/schema/ehcache-core.xsd">

  <cache alias="foo"> <!-- 1 -->
    <key-type>java.lang.String</key-type> <!--2 -->
    <value-type>java.lang.String</value-type> <!-- 2 -->
    <resources>
      <heap unit="entries">2000</heap> <!-- 3 -->
      <offheap unit="MB">500</offheap> <!-- 4 -->
    </resources>
  </cache>

  <cache-template name="myDefaults"> <!-- 5 -->
    <key-type>java.lang.Long</key-type>
    <value-type>java.lang.String</value-type>
    <heap unit="entries">200</heap>
  </cache-template>

  <cache alias="bar" uses-template="myDefaults"> <!-- 6 -->
    <key-type>java.lang.Number</key-type>
  </cache>

  <cache alias="simpleCache" uses-template="myDefaults" /> <!-- 7 -->

</config>
```

1. 声明 `Cache` 的别名为 `foo`.
2. 声明键值对类型; 如果没有声明，默认是 `java.lang.Object`.
3. 声明`foo` 的堆内存可以保存 2,000 个对象
4. 在驱逐之前堆外内存可以保存 500 MB 的数据
5. `<cache-template>` 元素创建一个抽象的配置，`<cache>` 配置可以集成它
6. `bar` 通过 `<cache-template>` 的标签名 `myDefaults` 重写了 Key 的类型.
7. `simpleCache` 的配置和 `myDefaults` 的配置完全一样.

参考 [XML documentation](http://www.ehcache.org/documentation/3.4/xml.html) 获取更多XML格式的配置.

通过 `XmlConfiguration` J解析XMl配置:
``` java
URL myUrl = getClass().getResource("/my-config.xml"); 
Configuration xmlConfig = new XmlConfiguration(myUrl); 
CacheManager myCacheManager = CacheManagerBuilder.newCacheManager(xmlConfig); 
```
1. 获取本地XMl配置的 `URL`
2. 实例化 `XmlConfiguration`
3. 使用静态方法 `org.ehcache.config.builders.CacheManagerBuilder.newCacheManager(org.ehcache.config.Configuration)` 实例化 `CacheManager`


# 集群缓存管理器(Creating a cache manager with clustering support)

To enable Clustering with Terracotta, firstly you will have to [start the Terracotta server](http://www.ehcache.org/documentation/3.4/clustered-cache.html#starting-server) configured with clustered storage. In addition, for creating the cache manager with clustering support, you will need to provide the clustering service configuration:

``` java
CacheManagerBuilder<PersistentCacheManager> clusteredCacheManagerBuilder =
    CacheManagerBuilder.newCacheManagerBuilder() // 1
        .with(ClusteringServiceConfigurationBuilder.cluster(URI.create("terracotta://localhost/my-application")) // 2
            .autoCreate()); // 3
PersistentCacheManager cacheManager = clusteredCacheManagerBuilder.build(true); // 4

cacheManager.close(); // 5
```

1. Returns the `org.ehcache.config.builders.CacheManagerBuilder` instance;
2. Use the `ClusteringServiceConfigurationBuilder`'s static method `.cluster(URI)` for connecting the cache manager to the clustering storage at the URI specified that returns the clustering service configuration builder instance. The sample URI provided in the example points to the clustered storage with clustered storage identifier my-application on the Terracotta server (assuming the server is running on localhost and port **9410**); the query-param `auto-create` creates the clustered storage in the server if it doesn’t already exist.
3. Returns a fully initialized cache manager that can be used to create clustered caches.
4. Auto-create the clustered storage if it doesn’t already exist.
5. Close the cache manager.

> See [the clustered cache documentation](http://www.ehcache.org/documentation/3.4/clustered-cache.html) for more information on this feature.


# 存储层级(Storage Tiers)

Ehcache 3, as in previous versions, offers a tiering model to allow storing increasing amounts of data on slower tiers (which are generally more abundant).

The idea is that resources related to faster storage are more rare, but are located where the 'hottest' data is preferred to be. Thus less-hot (less frequently used) data is moved to the more abundant but slower tiers. Hotter data is moved onto the faster tiers.

## Three tiers

A classical example would be using 3 tiers with a persistent disk storage.

``` java
PersistentCacheManager persistentCacheManager = CacheManagerBuilder.newCacheManagerBuilder()
    .with(CacheManagerBuilder.persistence(new File(getStoragePath(), "myData"))) // 1
    .withCache("threeTieredCache",
        CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
            ResourcePoolsBuilder.newResourcePoolsBuilder()
                .heap(10, EntryUnit.ENTRIES) // 2
                .offheap(1, MemoryUnit.MB) // 3
                .disk(20, MemoryUnit.MB, true) // 4
            )
    ).build(true);

Cache<Long, String> threeTieredCache = persistentCacheManager.getCache("threeTieredCache", Long.class, String.class);
threeTieredCache.put(1L, "stillAvailableAfterRestart"); // 5

persistentCacheManager.close();
```

1. If you wish to use disk storage (like for persistent `Cache` instances), you’ll have to provide a location where data should be stored on disk to the `CacheManagerBuilder.persistence()` static method.
2. You define a resource pool for the heap. This will be your faster but smaller pool.
3. You define a resource pool for the off-heap. Still pretty fast and a bit bigger.
4. You define a persistent resource pool for the disk. It is persistent because said it should be (last parameter is `true`).
5. All values stored in the cache will be available after a JVM restart (assuming the `CacheManager` has been closed cleanly by calling `close()`).
For details, read the [tiering documentation](http://www.ehcache.org/documentation/3.4/tiering.html).



# 数据新鲜度（Data freshness）

In Ehcache, data freshness is controlled through `Expiry`. The following illustrates how to configure a time-to-live expiry.

``` java
CacheConfiguration<Long, String> cacheConfiguration = CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
        ResourcePoolsBuilder.heap(100)) // 1
    .withExpiry(Expirations.timeToLiveExpiration(Duration.of(20, TimeUnit.SECONDS)))  // 2
    .build();
```
1. Expiry is configured at the cache level, so start by defining a cache configuration,
2. then add to it an `Expiry`, here using the predefined time-to-live one, configured with the required Duration.

See the section on [expiry](http://www.ehcache.org/documentation/3.4/expiry.html) for more information about the options available.


# Read More
Basic topics:[Getting Started](http://www.ehcache.org/documentation/3.4/getting-started.html)