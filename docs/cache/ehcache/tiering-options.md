

# Moving out of heap

在堆外还有其他缓存层的时候，就有一些其它事情需要考虑了：

- 向缓存中增加的键值对映射必须可以被序列化.
- 读取缓存的时候，需要反序列化键值对.

根据以上两点，你就需要考虑，序列化和反序列化部分的操作是否会过多的影响到缓存的读写性能(see the section (Serializers)[http://www.ehcache.org/documentation/3.4/serializers-copiers.html#serializers]). 

# Single tier setups

所有的缓存层可以单独使用。例如你可以 只用堆外缓存 或者 只用集群缓存

下面是可用的配置:

- heap
- offheap
- disk
- clustered

只使用其中一层，只需要在配置中定义单个资源即可:

``` Java
CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class, // 1
  ResourcePoolsBuilder.newResourcePoolsBuilder().offheap(2, MemoryUnit.GB)).build(); // 2
``` 
1. 使用配置 构建器 构建 键值对类型.
2. 层指定你想用的资源曾. 这里只使用了 `off-heap`.

## Heap Tier

每个刚放进去的缓存由于没有序列化，肯定是最快的。
The starting point of every cache and also the faster since no serialization is necessary. You can optionally use copiers (see the section (Serializers and Copiers)[http://www.ehcache.org/documentation/3.4/serializers-copiers.html#copiers]) to pass keys and values by-value, the default being by-reference.

堆内存这一层可以设置存储 对象的个数 或者 占用的内存空间.

``` java
ResourcePoolsBuilder.newResourcePoolsBuilder().heap(10, EntryUnit.ENTRIES); // 1
// or
ResourcePoolsBuilder.newResourcePoolsBuilder().heap(10); // 2
// or
ResourcePoolsBuilder.newResourcePoolsBuilder().heap(10, MemoryUnit.MB); // 3
```

1. 仅允许存储10个对象，如果超过10个会进行驱逐.
2. 跟1一样，1的缩写形式.
3. 只允许10M的内存占用，超出进行驱逐.

### Byte-sized heap

当堆的大小限制是 字节 而不是对象个数的时候，计算起来会更复杂一些.

计算字节大小对运行时性能的影响，取决于环迅数据的大小和引用图的复杂性。

``` java
CacheConfiguration<Long, String> usesConfiguredInCacheConfig = CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
  ResourcePoolsBuilder.newResourcePoolsBuilder()
    .heap(10, MemoryUnit.KB) // 1
    .offheap(10, MemoryUnit.MB)) // 2
  .withSizeOfMaxObjectGraph(1000)
  .withSizeOfMaxObjectSize(1000, MemoryUnit.B) // 3
  .build();

CacheConfiguration<Long, String> usesDefaultSizeOfEngineConfig = CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
  ResourcePoolsBuilder.newResourcePoolsBuilder()
    .heap(10, MemoryUnit.KB))
  .build();

CacheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder()
  .withDefaultSizeOfMaxObjectSize(500, MemoryUnit.B)
  .withDefaultSizeOfMaxObjectGraph(2000) // 4 
  .withCache("usesConfiguredInCache", usesConfiguredInCacheConfig)
  .withCache("usesDefaultSizeOfEngine", usesDefaultSizeOfEngineConfig)
  .build(true);
```

2.设置仅由堆层使用。 所以堆外根本不会使用它。
3.大小还可以通过另外两个配置设置来进一步限制：第一个指定了在对象图表中行走时要穿过的对象的最大数量（默认值：1000），第二个定义单个对象的最大大小（默认值： Long.MAX_VALUE，所以几乎是无限的）。 如果大小超出这两个限制中的任何一个，则该条目将不会被存储在缓存中。
4.除非明确定义，否则可以在CacheManager级别提供默认配置以供高速缓存使用。

1. 限制堆内存的键值对使用量，内存的使用和对象的大小是有志鉴关系的
2. The settings are only used by the heap tier. So off-heap won’t use it at all.
3. 

3. The sizing can also be further restrained by 2 additional configuration settings: The first one specifies the maximum number of objects to traverse while walking the object graph (default: 1000), the second defines the maximum size of a single object (default: Long.MAX_VALUE, so almost infinite). If the sizing goes above any of these two limits, the entry won’t be stored in cache.
4. A default configuration can be provided at CacheManager level to be used by the caches unless defined explicitly.

## Off-heap Tier

If you wish to use off-heap, you’ll have to define a resource pool, giving the memory size you want to allocate.

``` java
ResourcePoolsBuilder.newResourcePoolsBuilder().offheap(10, MemoryUnit.MB); // 1
```
1. Only 10 MB allowed off-heap. Eviction will occur when full.

The example above allocates a very small amount of off-heap. You will normally use a much bigger space.

Remember that data stored off-heap will have to be serialized and deserialized - and is thus slower than heap.

You should thus favor off-heap for large amounts of data where on-heap would have too severe an impact on garbage collection.

Do not forget to define in the java options the -XX:MaxDirectMemorySize option, according to the off-heap size you intend to use.

## Disk Tier

For the Disk tier, the data is stored on disk. The faster and more dedicated the disk is, the faster accessing the data will be.

``` java
PersistentCacheManager persistentCacheManager = CacheManagerBuilder.newCacheManagerBuilder() // 1
  .with(CacheManagerBuilder.persistence(new File(getStoragePath(), "myData"))) // 2
  .withCache("persistent-cache", CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
    ResourcePoolsBuilder.newResourcePoolsBuilder().disk(10, MemoryUnit.MB, true)) // 3
  )
  .build(true);

persistentCacheManager.close();
```

1. To obtain a PersistentCacheManager which is a normal CacheManager but with the ability to destroy caches.
2. Provide a location where data should be stored.
3. Defines a resource pool for the disk that will be used by the cache. The third parameter is a boolean value which is used to set whether the disk pool is persistent. When set to true, the pool is persistent. When the version with 2 parameters disk(long, MemoryUnit) is used, the pool is not persistent.
The example above allocates a very small amount of disk storage. You will normally use a much bigger storage.

Persistence means the cache will survive a JVM restart. Everything that was in the cache will still be there after restarting the JVM and creating a CacheManager disk persistence at the same location.

> A disk tier can’t be shared between cache managers. A persistence directory is dedicated to one cache manager at the time.
Remember that data stored on disk will have to be serialized / deserialized and written to / read from disk - and is thus slower than heap and offheap. So disk storage is interesting if:

- You have a large amount of data that can’t fit off-heap
- Your disk is much faster than the storage it is caching
- You are interested in persistence

> Ehcache 3 only offers persistence in the case of clean shutdowns (close() was called). If the JVM crashes there is no data integrity guarantee. At restart, Ehcache will detect that the CacheManager wasn’t cleanly closed and will wipe the disk storage before using it.

## Segments

Disk storage is separated into segments which provide concurrency access but also hold open file pointers. The default is 16. In some cases, you might want to reduce the concurrency and save resources by reducing the number of segments.

``` java
String storagePath = getStoragePath();
PersistentCacheManager persistentCacheManager = CacheManagerBuilder.newCacheManagerBuilder()
  .with(CacheManagerBuilder.persistence(new File(storagePath, "myData")))
  .withCache("less-segments",
    CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
      ResourcePoolsBuilder.newResourcePoolsBuilder().disk(10, MemoryUnit.MB))
    .add(new OffHeapDiskStoreConfiguration(2))  // 1
  )
  .build(true);

persistentCacheManager.close();
```
1. Define an OffHeapDiskStoreConfiguration instance specifying the required number of segments.

## Clustered

A clustered tier means the client is connecting to the Terracotta Server Array where the cached data is stored. It is also as way to have a shared cache between JVMs.

See the section Clustered Caches for details of using the cluster tier.

# Multiple tier setup

If you want to use more than one tier, you have to observe some constraints:

1. There must always be a heap tier in a multi tier setup.
2. You cannot combine disk tiers and clustered tiers.
3. Tiers should be sized in a pyramidal fashion, i.e. tiers higher up the pyramid are configured to use less memory than tiers lower down.

For 1, this is a limitation of the current implementation.

For 2, this restriction is necessary, because having two tiers with content that can outlive the life of a single JVM can lead to consistency questions on restart.

For 3, the idea is that tiers are related to each others. The fastest tier (the heap tier) is on top, while the slower tiers are below. In general, heap is more constrained than the total memory of the machine, and offheap memory is more constrained than disk or the memory available on the cluster. This leads to the typical pyramid shape for a multi-tiered setup.

// TODO 图片
Figure 1. Tiers hierarchy
Ehcache requires the size of the heap tier to be smaller than the size of the offheap tier, and the size of the offheap tier to be smaller than the size of the disk tier. While Ehcache cannot verify at configuration time that a count-based sizing for heap will be smaller than a byte-based sizing for another tier, you should make sure that is the case during testing.

Taking the above into account, the following possibilities are valid configurations:

- heap + offheap
- heap + offheap + disk
- heap + offheap + clustered
- heap + disk
- heap + clustered

Here is an example using heap, offheap and clustered.

``` java
PersistentCacheManager persistentCacheManager = CacheManagerBuilder.newCacheManagerBuilder()
  .with(cluster(CLUSTER_URI).autoCreate()) // 1
  .withCache("threeTierCache",
    CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
      ResourcePoolsBuilder.newResourcePoolsBuilder()
        .heap(10, EntryUnit.ENTRIES) // 2
        .offheap(1, MemoryUnit.MB) // 3
        .with(ClusteredResourcePoolBuilder.clusteredDedicated("primary-server-resource", 2, MemoryUnit.MB)) // 4
    )
  ).build(true);
```

1. Clustered specific information telling how to connect to the Terracotta cluster
2. Define the Heap tier which is the smallest but fastest caching tier.
3. Define the Offheap tier. Next in line as caching tier.
4. Define the Clustered tier. The authoritative tier for this cache.

# Resource pools
Tiers are configured using resource pools. Most of the time using a `ResourcePoolsBuilder`. Let’s revisit an example used earlier:

``` java
PersistentCacheManager persistentCacheManager = CacheManagerBuilder.newCacheManagerBuilder()
  .with(CacheManagerBuilder.persistence(new File(getStoragePath(), "myData")))
  .withCache("threeTieredCache",
    CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
      ResourcePoolsBuilder.newResourcePoolsBuilder()
        .heap(10, EntryUnit.ENTRIES)
        .offheap(1, MemoryUnit.MB)
        .disk(20, MemoryUnit.MB, true)
    )
  ).build(true);
```

This is a cache using 3 tiers (heap, offheap, disk). They are created and chained using the `ResourcePoolsBuilder`. The declaration order doesn’t matter (e.g. offheap can be declared before heap) because each tier has a height. The higher the height of a tier is, the closer the tier will be to the client.

It is really important to understand that a resource pool is only specifying a configuration. It is not an actual pool that can be shared between caches. Consider for instance this code:

``` java
ResourcePools pool = ResourcePoolsBuilder.newResourcePoolsBuilder().heap(10).build();

CacheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder()
  .withCache("test-cache1", CacheConfigurationBuilder.newCacheConfigurationBuilder(Integer.class, String.class, pool))
  .withCache("test-cache2", CacheConfigurationBuilder.newCacheConfigurationBuilder(Integer.class, String.class, pool))
  .build(true);
```

You will end up with two caches that can contain 10 entries each. Not a shared pool of 10 entries. Pools are never shared between caches. The exception being clustered caches, that can be shared or dedicated.

## Update ResourcePools

Limited size adjustment can be performed on a live cache.

> `updateResourcePools()` only allows you to change the heap tier sizing, not the pool type. Thus you can’t change the sizing of off-heap or disk tiers.

``` java
ResourcePools pools = ResourcePoolsBuilder.newResourcePoolsBuilder().heap(20L, EntryUnit.ENTRIES).build(); // 1
cache.getRuntimeConfiguration().updateResourcePools(pools); // 2
assertThat(cache.getRuntimeConfiguration().getResourcePools()
  .getPoolForResource(ResourceType.Core.HEAP).getSize(), is(20L));
```

1. You will need to create a new ResourcePools object with resources of the required size, using ResourcePoolsBuilder. This object can then be passed to the said method so as to trigger the update.
2. To update the capacity of ResourcePools, the updateResourcePools(ResourcePools) method in RuntimeConfiguration can be of help. The ResourcePools object created earlier can then be passed to this method so as to trigger the update.

# Destroy persistent tiers
The disk tier and clustered tier are the two persistent tiers. It means that when the JVM is stopped, all the created caches and their data are still existing on disk or on the cluster.

Once in a while, you might want to fully remove them. Their definition as PersistentCacheManager gives access to the following methods:

`destroy()`
This method destroys everything related to the cache manager (including caches, of course). The cache manager must be closed or uninitialized to call this method. Also, for a clustered tier, no other cache manager should currently be connected to the same cache manager server entity.

`destroyCache(String cacheName)`
This method destroys a given cache. The cache shouldn’t be in use by another cache manager.

# Sequence Flow for Cache Operations with Multiple Tiers
In order to understand what happens for different cache operations when using multiple tiers, here are examples of Put and Get operations. The sequence diagrams are oversimplified but still show the main points.

Put
Figure 2. Multiple tiers using Put
Get
Figure 3. Multiple tiers using Get

You should then notice the following:

- When putting a value into the cache, it goes straight to the authoritative tier, which is the lowest tier.
- A following get will push the value upwards in the caching tiers.
- Of course, as soon as a value is put in the authoritative tier, all higher-level caching tiers are invalidated.
- A full cache miss (the value isn’t on any tier) will always go all the way down to the authoritative tier.

> The slower your authoritative tier, the slower your put operations will be. For a normal cache usage, it usually doesn’t matter since`get` operations are much more frequent than`put` opreations. The opposite would mean you probably shouldn’t be using a cache in the first place.