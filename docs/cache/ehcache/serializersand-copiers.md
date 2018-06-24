# Overview of Serializers and Copiers

虽然 `Ehcache` 是一个 `Java` 缓存, 但它不总是存储 `Java` 对象.

[on-heap store](http://www.ehcache.org/documentation/3.4/tiering.html#heap-tier) 这层缓存可以缓存对象的引用. 除了堆内缓存这层，其它几层都是通过字节的形式表示的.

`Serializer` 和 `Copier` 是启用这些不同缓存层的抽象.

# Serializers

All stores but the on-heap one need some form of serialization/deserialization of objects to be able to store and retrieve mappings. This is because they cannot internally store plain java objects but only binary representations of them.

`Serializer` is the Ehcache abstraction solving this: every cache that has at least one store that cannot store by reference is going to use a pair of `Serializer` instances, one for the key and another one for the value.

A `Serializer` is scoped at the cache level and all stores of a cache will be using and sharing the same pair of serializers.

## How is a serializer configured?

There are two places where serializers can be configured:

- at the cache level where one can use
  - `CacheConfigurationBuilder.withKeySerializer(Class<? extends Serializer<K>> keySerializerClass)`,
  - `CacheConfigurationBuilder.withKeySerializer(Serializer<K> keySerializer)`,
  - `CacheConfigurationBuilder.withValueSerializer(Class<? extends Serializer<V>> valueSerializerClass)`,
  - and `CacheConfigurationBuilder.withValueSerializer(Serializer<V> valueSerializer)`, which allow by instance or by class configuration.

- at the cache manager level where one can use
  - `CacheManagerBuilder.withSerializer(Class<C> clazz, Class<? extends Serializer<C>> serializer)`

If a serializer is configured directly at the cache level, it will be used, ignoring any cache manager level configuration.

If a serializer is configured at the cache manager level, upon initialization, a cache with no specifically configured serializer will search through its cache manager’s registered list of serializers and try to find one that directly matches the cache’s key or value type. If such search fails, all the registered serializers will be tried in the added order to find one that handles compatible types.

For instance, let’s say you have a `Person` interface and two subclasses: `Employee` and `Customer`. If you configure your cache manager as follows:

``` Java
CacheManagerBuilder.newCacheManagerBuilder().withSerializer(Employee.class,
  EmployeeSerializer.class).withSerializer(Person.class, PersonSerializer.class)
```
then configuring a `Cache<Long, Employee>` would make it use the `EmployeeSerializer` while a `Cache<Long, Customer>` would make it use the `PersonSerializer`.

A `Serializer` configured at the cache level by class will not be shared to other caches when instantiated.

> Given the above, it is recommended to limit `Serializer` registration to concrete classes and not aim for generality.


## Bundled implementations

By default, cache managers are pre-configured with specially optimized `Serializer` that can handle the following types, in the following order:

- java.io.Serializable
- java.lang.Long
- java.lang.Integer
- java.lang.Float
- *java.lang.Double
- java.lang.Character
- java.lang.String
- byte[]

All bundled `Serializer` implementations support both persistent and transient caches.

> A consequence of providing serializers registered by default is that you will not be able to register a generic `Serializer` for `Number` or any other super type and expect it to be picked instead of the default ones for the types listed above.

> However, registering a different `Serializer` for one of the given type means it will be used instead of the default.

## Lifecycle: instances vs. class

When a Serializer is configured by providing an instance, it is up to the provider of that instance to manage its lifecycle. It will need to dispose of any resource the serializer might hold, persisting or reloading the serializer’s state.

When a Serializer is configured by providing a class either at the cache or cache manager level, since Ehcache is responsible for creating the instance, it also is responsible for disposing of it. If the Serializer implements java.io.Closeable then close() will be called when the cache is closed and the Serializer no longer needed.

## Writing your own serializer

Serializer defines a very strict contract. So if you’re planning to write your own implementation you have to keep in mind that the class of the serialized object MUST be retained after deserialization, that is:

object.getClass().equals(mySerializer.read(mySerializer.serialize(object)).getClass())
This is especially important when you are planning to write a serializer for an abstract type, e.g. a serializer of type com.pany.MyInterface should:

deserialize a com.pany.MyClassImplementingMyInterface when the serialized object is of class com.pany.MyClassImplementingMyInterface

return a com.pany.AnotherClassImplementingMyInterface object when the serialized object is of class com.pany.AnotherClassImplementingMyInterface

Implement the following interface, from package `org.ehcache.spi.serialization`:

``` java
/**
 * Defines the contract used to transform type instances to and from a serial form.
 * <p>
 * Implementations must be thread-safe.
 * <p>
 * When used within the default serialization provider, there is an additional requirement.
 * The implementations must define a constructor that takes in a {@code ClassLoader}.
 * The {@code ClassLoader} value may be {@code null}.  If not {@code null}, the class loader
 * instance provided should be used during deserialization to load classes needed by the deserialized objects.
 * <p>
 * The serialized object's class must be preserved; deserialization of the serial form of an object must
 * return an object of the same class. The following contract must always be true:
 * <p>
 * {@code object.getClass().equals( mySerializer.read(mySerializer.serialize(object)).getClass())}
 *
 * @param <T> the type of the instances to serialize
 *
 * @see SerializationProvider
 */
public interface Serializer<T> {

  /**
   * Transforms the given instance into its serial form.
   *
   * @param object the instance to serialize
   *
   * @return the binary representation of the serial form
   *
   * @throws SerializerException if serialization fails
   */
  ByteBuffer serialize(T object) throws SerializerException;

  /**
   * Reconstructs an instance from the given serial form.
   *
   * @param binary the binary representation of the serial form
   *
   * @return the de-serialized instance
   *
   * @throws SerializerException if reading the byte buffer fails
   * @throws ClassNotFoundException if the type to de-serialize to cannot be found
   */
  T read(ByteBuffer binary) throws ClassNotFoundException, SerializerException;

  /**
   * Checks if the given instance and serial form {@link Object#equals(Object) represent} the same instance.
   *
   * @param object the instance to check
   * @param binary the serial form to check
   *
   * @return {@code true} if both parameters represent equal instances, {@code false} otherwise
   *
   * @throws SerializerException if reading the byte buffer fails
   * @throws ClassNotFoundException if the type to de-serialize to cannot be found
   */
  boolean equals(T object, ByteBuffer binary) throws ClassNotFoundException, SerializerException;

}
```

As the Javadoc states, there are some constructor rules, see the section Persistent vs. transient caches for that.

You can optionally implement java.io.Closeable. If you do, Ehcache will call close() when a cache using such a serializer gets disposed of, but only if Ehcache instantiated the serializer itself.

## ClassLoaders

When Ehcache instantiates a serializer itself, it will pass it a ClassLoader via the constructor. Such class loader must be used to access the classes of the serialized types as they might not be available in the current class loader.

## Persistent vs. transient caches

All custom serializers must have a constructor with the following signature:

``` Java
public MySerializer(ClassLoader classLoader) {
}
```

Attempting to configure a serializer that lacks such a constructor on a cache using either of CacheConfigurationBuilder.withKeySerializer(Class<? extends Serializer<K>> keySerializerClass) or CacheConfigurationBuilder.withValueSerializer(Class<? extends Serializer<V>> valueSerializerClass) will cause an exception upon cache initialization.

But if an instance of the serializer is configured using either of CacheConfigurationBuilder.withKeySerializer(Serializer keySerializer) or CacheConfigurationBuilder.withValueSerializer(Serializer valueSerializer) it will work since the instantiation is done by the user code itself.

Registering a serializer that lacks such a constructor at the cache manager level will prevent it from being chosen for caches.

Custom serializer implementations could have some state that is used in the serialization/deserialization process. When configured on a persistent cache, the state of such serializers needs to be persisted across restarts.

To address these requirements you can have a StatefulSerializer implementation. StatefulSerializer is a specialized Serializer with an additional init method with the following signature:

public void init(StateRepository repository) {
}
The StateRepository.getPersistentStateHolder(String, Class<K>, Class<V>) provides a StateHolder (a map like structure) that you can use to store any relevant state. The StateRepository is provided by the authoritative tier of the cache and hence will have the same persistence properties of that tier. For persistent caches it is highly recommended that all state is stored in these holders as the users won’t have to worry about the persistence aspects of this state holder as it is taken care of by Ehcache.

In the case of a disk persistent cache, the contents of the state holder will be persisted locally on to the disk.

For clustered caches, the contents are persisted in the cluster itself so that other clients using the same cache can also access the contents of the state holder.

The constructor with the signature (ClassLoader classLoader, FileBasedPersistenceContext persistenceContext) that existed till v3.1 has been removed since v3.2 in favor of `StatefulSerializer`s.


# Copiers

As the on-heap store is capable of storing plain Java objects as such, it is not necessary to rely on a serialization mechanism to copy keys and values in order to provide by value semantics. Other forms of copy mechanism can be a lot more performant, such as using a copy constructor but it requires custom code to be able to copy user classes.

`Copier` 只用于堆内存这层.

By default, the on-heap mappings are stored by reference. The way to store them by value is to configure copier(s) on the cache for the key, value or both.

Of course, the exact semantic of by value in this context depends heavily on the Copier implementation.

## How is a copier configured?

可以通过以下两种方式进行配置:

- 缓存级（cache level） 配置
  - `CacheConfigurationBuilder.withKeyCopier(Class<? extends Copier<K>> keyCopierClass)`,
  - `CacheConfigurationBuilder.withKeyCopier(Copier<K> keyCopier)`,
  - `CacheConfigurationBuilder.withValueCopier(Class<? extends Copier<V>> valueCopierClass)`,
  - `CacheConfigurationBuilder.withValueCopier(Copier<V> valueCopier)`
- 缓存管理器（cache manager level） 级 配置
  - `CacheManagerBuilder.withCopier(Class<C> clazz, Class<? extends Copier<C>> copier)`

`cache level` 会覆盖 `cache manager level` 的配置.

如果只在`CacheManager` 级别配置了 `Copier`，在初始化缓存的时候会搜索`CacheManager`级别注册的`Copier`，搜索的时候是按照注册准顺序依次搜索的。

例如，加入你有一个 `Person` 接口，两个实现: `Employee` and `Customer`。如果你的 `CacheManager` 配置如下：

``` Java
CacheManagerBuilder.newCacheManagerBuilder().withCopier(Employee.class,
  EmployeeCopier.class).withCopier(Person.class, PersonCopier.class)
```

`Cache<Long, Employee>` 将会使用 `EmployeeCopier` ， `Cache<Long, Customer>` 将会使用 `PersonCopier`.

`Copier` 在 `Cache` 级别的配置跟其他`Cache`是不共享的.

> Given the above, it is recommended to limit Copier registration to concrete classes and not aim for generality.

## Bundled implementations

A `SerializingCopier` class exists in case you want to configure store by value on-heap using the configured (or default) serializer. Note that this implementation performs a serialization / deserialization on each read or write operation.

The `CacheConfigurationBuilder` provides the following methods to make use of this specialized copier:

- `CacheConfigurationBuilder.withKeySerializingCopier()` for the key.
- `CacheConfigurationBuilder.withValueSerializingCopier()` for the value.

## Lifecycle: instances vs class

When a `Copier` is configured by providing an instance, it is up to the provider of that instance to manage its lifecycle. It will need to dispose of any resource it used after it is no longer required.

When a `Copier` is configured by providing a class either at the cache or cache manager level, since Ehcache is responsible for creating the instance, it also is responsible for disposing of it. If the `Copier` implements `java.io.Closeable` then `close()` will be called when the cache is closed and the `Copier` no longer needed.

## Writing your own Copier

Implement the following interface:

``` Java
/**
 * Defines the contract used to copy type instances.
 * <p>
 * The copied object's class must be preserved.  The following must always be true:
 * <p>
 * <code>object.getClass().equals( myCopier.copyForRead(object).getClass() )</code>
 * <code>object.getClass().equals( myCopier.copyForWrite(object).getClass() )</code>
 *
 * @param <T> the type of the instance to copy
 */
public interface Copier<T> {

  /**
   * Creates a copy of the instance passed in.
   * <p>
   * This method is invoked as a value is read from the cache.
   *
   * @param obj the instance to copy
   * @return the copy of the {@code obj} instance
   */
  T copyForRead(T obj);

  /**
   * Creates a copy of the instance passed in.
   * <p>
   * This method is invoked as a value is written to the cache.
   *
   * @param obj the instance to copy
   * @return the copy of the {@code obj} instance
   */
  T copyForWrite(T obj);
}
```

`T copyForRead(T obj)` is invoked when a copy must be made upon a read operation (like a cache get()),

`T copyForWrite(T obj)` is invoked when a copy must be made upon a write operation (like a cache put()).

The separation between copying for read and for write can be useful when you want to store a lighter version of your objects into the cache.

Alternatively, you can extend from org.ehcache.impl.copy.ReadWriteCopier if copying for read and copying for write implementations are identical, in which case you only have to implement:

`public abstract T copy(T obj)`