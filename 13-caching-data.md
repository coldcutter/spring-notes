# 13 Caching data

## 13.1 Enabling cache support

Spring’s cache abstraction comes in two forms:

* Annotation-driven caching
* XML-declared caching

![](/assets/QQ20161021-1.png)

Under the covers, @EnableCaching and &lt;cache:annotation-driven&gt; work the same way. They create an aspect with pointcuts that trigger off of Spring’s caching annotations.

### 13.1.1 Configuring a cache manager

Out of the box, Spring 3.1 comes with five cache-manager implementations:

* SimpleCacheManager
* NoOpCacheManager
* ConcurrentMapCacheManager  
* CompositeCacheManager
* EhCacheCacheManager 

Spring 3.2 introduced another cache manager for working with JCache \(JSR-107\) based cache providers. Outside of the core Spring Framework, Spring Data offers two more cache managers:

* RedisCacheManager \(from Spring Data Redis\)
* GemfireCacheManager \(from Spring Data GemFire\)

**CACHING WITH EHCACHE**

![](/assets/QQ20161021-2.png)

For example, the following Ehcache configuration declares a cache named spittleCache with 50 MB of maximum heap storage and a time-to-live of 100 seconds.

```
<ehcache>
  <cache name="spittleCache"
         maxBytesLocalHeap="50m"
         timeToLiveSeconds="100">
  </cache>
</ehcache>
```

**USING REDIS FOR CACHING**

To use RedisCacheManager, you’ll need a RedisTemplate bean and a bean that’s an implementation of RedisConnectionFactory \(such as JedisConnectionFactory\).

![](/assets/QQ20161021-3.png)

**WORKING WITH MULTIPLE CACHE MANAGERS**

CompositeCacheManager is configured with one or more cache managers and iterates over them all as it tries to find a previously cached value.

![](/assets/QQ20161021-4.png)

## 13.2 Annotating methods for caching

![](/assets/QQ20161021-5.png)

![](/assets/QQ20161021-6.png)

### 13.2.1 Populating the cache

![](/assets/QQ20161021-7.png)

**可以把注解放在接口定义中，这样所有实现类都可以继承该缓存规则。**

```
@CachePut("spittleCache")
Spittle save(Spittle spittle);
```

There’s only one problem: the cache key. As I mentioned earlier, the default cache key is based on the parameters to the method. Because the only parameter to save\(\) is a Spittle, it’s used as the cache key. Doesn’t it seem odd to place a Spittle in a cache where the key is the same Spittle?

**CUSTOMIZING THE CACHE KEY**

![](/assets/QQ20161021-8.png)

```
@CachePut(value="spittleCache", key="#result.id")
Spittle save(Spittle spittle);
```

**CONDITIONAL CACHING**

如果unless属性表达式为true，则方法返回的值不会被缓存，如果condition属性表达式为false，则该方法的缓存被完全禁用。

区别：unless只能阻止返回值被缓存，但是缓存还是会被检索，如果找到，则返回。condition则完全禁用缓存，既不检索缓存，也不更新缓存。

As an example \(albeit a contrived one\), suppose you don’t want to cache any Spittle objects whose message property contains the text “NoCache”.

```
@Cacheable(value="spittleCache", unless="#result.message.contains('NoCache')")
Spittle findOne(long id);
```

For instance, suppose you don’t want caching to be applied to any Spittle whose ID is less than 10.

```
@Cacheable(value="spittleCache", unless="#result.message.contains('NoCache')", condition="#id >= 10")
Spittle findOne(long id);
```

### 13.2.2 Removing cache entries

Unlike @Cacheable and @CachePut, @CacheEvict can be used on void methods. @Cacheable and @CachePut require a non-void return value, which is the item to place in the cache.

![](/assets/QQ20161021-1@2x.png)

![](/assets/QQ20161021-2@2x.png)

## 13.3 Declaring caching in XML 

略。

