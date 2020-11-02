---
title: Spring缓存
---
## 概述
Spring定义了org.springframework.cache.CacheManager和org.springframework.cache.Cache接口来统一不同的缓存技术。其中，CacheManager是Spring提供的各种缓存技术抽象接口，Cache接口包含缓存的各种操作。
### CacheManager接口
Spring提供了不同的CacheManager实现来使用不同的技术：SimpleCacheManager使用简单的Collection来存储缓存，用于测试；ConcurrentMapCacheManager使用ConcurrentMap来存储缓存；RedisCacheManager使用redis作为缓存技术。

Spring需要使用`@EnableCaching`来开启对缓存的使用，并通过对应的对象去创建Bean。
``` java
    @Bean
    public ConcurrentMapCacheManager cacheManager() {
        return new ConcurrentMapCacheManager();
    }
```

而Spring boot中已经自动配置了CacheManger，默认用SimpleCacheConfiguration配置，使用的是ConcurrentMapCacheManager，可以用spring.cache为前缀的属性来改变缓存默认配置。也需要在配置类中使用`@EnableCaching`开启对缓存的支持。

### 声明式缓存注解
注解|解释
--|--
`@Cacheable`|在方法执行前，Spring先检查缓存中是否有数据，如果有则返回缓存中的数据，否则调用方法并将方法返回值放入缓存。
`@CachePut`|无论怎样，都把方法返回值放入缓存，可用于save方法。
`@CacheEvict`|将一条或者多条数据从缓存中删除，可用于delete方法。
`@Cacheing`|可以通过`@Cacheing`注解组合多个注解策略在一个方法上。
