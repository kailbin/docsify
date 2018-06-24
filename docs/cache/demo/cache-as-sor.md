
缓存的使用模式主要分为两大类 `Cache-Aside`和`Cache-As-SoR`(`Read-through`、`Write-through`、`Write-behind`)

- `Cache-Aside` 即业务代码围绕着Cache写，是由业务代码直接维护缓存
- `Cache-As-SoR`即把`Cache`看作为`SoR`，所有操作都是对`Cache`进行，然后`Cache`再委托给`SoR`进行真实的读/写
	- `Read-through`：业务代码首先调用`Cache`，如果`Cache`不命中由`Cache`回源到`SoR`
	- `Write-through`： 业务代码首先调用`Cache`写(新增/修改)数据，然后由`Cache`负责写缓存和写`SoR`，而不是业务代码
	- `Write-behind`：不同于`Write-Through`是同步写`SoR`和`Cache`，`Write-Behind`是异步写


# `Read-through`

## Guava Cache实现

``` Java
LoadingCache<Integer, Long> getCache = CacheBuilder.newBuilder()
    .softValues() // 软引用，内存紧张的时候自动进行回收
    .maximumSize(5000) // 缓存的最大容量，如果超过的话回使用 LRU(最近最少使用) 算法进行淘汰
    .expireAfterWrite(2, TimeUnit.SECONDS) // 指定时间没没有被写入的时候进行回收
    .build(new CacheLoader<Integer, Long>() { // 如果获取不到缓存，自动从 SoR 中进行获取
        @Override
        public Long load(final Integer key) throws Exception {
            return selectDataFromDb(key);
        }
    });
```


## Ehcache 3.x实现

### DefaultCacheLoaderWriter 覆盖掉无需实现的方法
``` Java
public abstract class DefaultCacheLoaderWriter<K, V> implements CacheLoaderWriter<K, V> {

 	@Override
    public V load(K key) throws Exception {
        return null;
    }

    @Override
    public Map<K, V> loadAll(Iterable<? extends K> keys) throws BulkCacheLoadingException, Exception {
        return null;
    }
    
    @Override
    public void write(K key, V value) throws Exception {
    }

    @Override
    public void writeAll(Iterable<? extends Map.Entry<? extends K, ? extends V>> entries) throws BulkCacheWritingException, Exception {
    }

    @Override
    public void delete(K key) throws Exception {
    }

    @Override
    public void deleteAll(Iterable<? extends K> keys) throws BulkCacheWritingException, Exception {
    }
}

```

``` Java
CacheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder().build(true);

ResourcePoolsBuilder resourcePoolsBuilder = ResourcePoolsBuilder.newResourcePoolsBuilder().heap(100, MemoryUnit.MB);

CacheConfigurationBuilder<String, String> configurationBuilder = CacheConfigurationBuilder.newCacheConfigurationBuilder(String.class, String.class, resourcePoolsBuilder)
        .withDispatcherConcurrency(4)
        .withExpiry(Expirations.timeToLiveExpiration(Duration.of(10, TimeUnit.SECONDS)))
        .withLoaderWriter(new DefaultCacheLoaderWriter<String, String>() {
            /**
             * 从缓存中获取数据，如果获取不到的时候调用该方法
             */
            @Override
            public String load(String key) throws Exception {
                return selectDataFromDB(key);
            }

            /**
             * 批量获取的时候 如果获取不到调用该方法
             */
            @Override
            public Map<String, String> loadAll(Iterable<? extends String> iterable) throws BulkCacheLoadingException, Exception {
                String[] keys = Iterables.toArray(iterable, String.class);
                return selectDataFromDBBatch(keys);
            }
        });

Cache<String, String> myCache = cacheManager.createCache("myCache", configurationBuilder);
```

> **Ehcache 3.1没有自己去解决Dog-pile effect。**

# `Write-through`
``` Java
  CacheConfigurationBuilder<String, String> configurationBuilder = CacheConfigurationBuilder.newCacheConfigurationBuilder(String.class, String.class, resourcePoolsBuilder)
                .withDispatcherConcurrency(4)
                .withExpiry(Expirations.timeToLiveExpiration(Duration.of(10, TimeUnit.SECONDS)))
                .withLoaderWriter(new DefaultCacheLoaderWriter<String, String>() {
                    @Override
                    public void write(String key, String value) throws Exception {
                        //write
                    }

                    @Override
                    public void writeAll(Iterable<? extends Map.Entry<? extends String, ? extends String>> entries) throws BulkCacheWritingException, Exception {
                        for (Object entry : entries) {
                            //batch write
                        }
                    }

                    @Override
                    public void delete(String key) throws Exception {
                        //delete
                    }

                    @Override
                    public void deleteAll(Iterable<? extends String> keys) throws BulkCacheWritingException, Exception {
                        for (Object key : keys) {
                            //batch delete
                        }
                    }
                });
```




# `Write-behind`

``` Java
private static <K, V> StripedWriteBehind<K, V> createWriteBehind(CacheLoaderWriter<K, V> loaderWriter) {
    WriteBehindConfiguration build = WriteBehindConfigurationBuilder.newUnBatchedWriteBehindConfiguration()
            .concurrencyLevel(5)
            .queueSize(10)
            .build();

    /*
     * null -> OnDemandExecutionService
     * PooledExecutionServiceConfiguration -> PooledExecutionService
     */
    ExecutionService executionService = new DefaultExecutionServiceFactory().create(null);

    return new StripedWriteBehind<K, V>(executionService, "Striped-Write-Behind", build, loaderWriter);
}
```

``` Java
CacheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder().build(true);

ResourcePoolsBuilder resourcePoolsBuilder = ResourcePoolsBuilder.newResourcePoolsBuilder().heap(100, MemoryUnit.MB);


DefaultCacheLoaderWriter<String, String> loaderWriter = new DefaultCacheLoaderWriter<String, String>() {
    @Override
    public void write(String key, String value) throws Exception {
        //write
        Thread.sleep(1000);
        System.out.println(key);
    }

    @Override
    public void delete(String key) throws Exception {
        //delete
        Thread.sleep(1000);
        System.out.println(key);
    }

};


CacheConfigurationBuilder<String, String> configurationBuilder = CacheConfigurationBuilder.newCacheConfigurationBuilder(String.class, String.class, resourcePoolsBuilder)
        .withDispatcherConcurrency(4)
        .withExpiry(Expirations.timeToLiveExpiration(Duration.of(10, TimeUnit.SECONDS)))
        /*
         * 异步写入
         */
        .withLoaderWriter(createWriteBehind(loaderWriter)) 
        ;
```


# Read More

[ 高并发服务设计——缓存](http://blog.csdn.net/foreverling/article/details/78012205)
[Guava Cache](GuavaCache.md)