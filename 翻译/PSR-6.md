# [PSR-6: Caching Interface](https://www.php-fig.org/psr/psr-6/)

缓存是提高任何项目性能的常见方式，因此，使用缓存库是许多框架和库最常见的功能之一。
这导致了许多库推出自己的缓存库，具有各种级别的功能。
这些差异导致开发者不得不学习多个可能提供也可能没有提供他们所需功能的系统。
此外，缓存库的开发者也需要在只支持有限数量的框架或创建大量的适配器类之间进行抉择。

缓存系统的通用接口将解决这些问题。
库和框架的开发者可以依赖于他们期望的缓存系统，而缓存系统的开发者只需要实现一组接口而不是各种各样的适配器。

## Goal - 目的

此PSR的目标是允许开发者创建可以集成到现有框架和系统中的缓存感知库，而无需定制开发。

## Definition - 定义

- Calling Library （调用库）- 实际需要缓存服务的库或代码。
该库将利用实现此标准接口的缓存服务，但不会知道这些缓存服务的实现细节。

- Implementing Library （实现库）- 该库负责实现此标准，以便为任何调用库提供缓存服务。
该类库MUST（必须）提供实现了``Cache\CacheItemPoolInterface``和``Cache\CacheItemInterface``的类库。实现库MUST（必须）支持至少TTL功能，如下所述，具有全秒级粒度。

- TTL （生存时间） - The Time To Live（TTL）是从数据被存储到被认为过期之间的时长。TTL通常被定义为以秒为单位的时间的整数或``DateInterval``对象。

- Expiration （过期）-  数据设置为过期的实际时间。这通常通过将TTL添加到数据被存储的时间来计算，但也可以使用DateTime对象限式设置。
一个带有300TTL的数据在1:30:00被存储，那么他的过期时间为1:35:00。
实现的库MAY（可以）在其请求的过期时间之前将数据过期，但是一旦达到过期时间，MUST（必须）将数据视为已过期。
如果调用库要求存储数据但不指定过期时间，或指定一个为null的过期时间或TTL，则实现库MAY（可以）使用配置的默认的持续时间。
如果没有设置默认持续时间，实现库MUST（必须）将其解释为“永久”缓存数据的请求，或是底层实现支持的最大时间。

- Key （键）- 至少包含一个字符的字符串，用于唯一标识缓存的数据。
实现库MUST（必须）支持由``A-Z``、``a-z``、``0-9``、``_``和``.``按任意顺序组成的、以UTF-8编码的、最多64个字符的key。
实现库MAY（可能）支持其它的字符、编码方式或更长的长度，但是必须支持上述的最低要求。
实现库负责自己的key转义，但必须能够返回原始未修改的key字符串。下述字符为之后的扩展做保留，并且实现库MUST NOT（禁止）支持：``{}()/\@:``

- Hit （命中）- 缓存命中发生在调用库通过key请求数据，并且有找到key匹配的数据，并且数据没有过期，并且数据没有因为其它的原因失效。
调用库SHOULD（应该）确保在所有get()调用上验证isHit()。

- Miss （未命中）- 缓存未命中与缓存命中相反。
缓存未命中发生在调用库通过key请求数据，并且没有找到匹配的数据，或数据找到但已过期，或数据因为其它原因失效。
过期的数据MUST（必须）始终被认为是缓存未命中。

- Deferred （延迟）- 延迟缓存保存表示池可能不会立即保存数据。池对象MAY（可以）延迟持久化缓存数据，以便利用某些存储引擎的批量设置操作。
池MUST（必须）确保最终持久化任何延迟缓存数据并且数据不会丢失，并且MAY（可以）在调用库请求将他们持久化之前持久化它们。
当调用库调用commit()方法时，所有未持久化的延迟缓存MUST（必须）被持久化。
实现库可以使用适当的逻辑来确定何时持久化延迟缓存，例如对象的析构函数，持久保存方法save()，超时或最大数量检测或任何其它适当的逻辑。
对延迟持久化的缓存数据的请求必须返回延迟但尚未持久化的数据。

## Data - 数据

实现库MUST（必须）支持所有可序列化的php数据类型，包括：

- String - 任何php兼容编码中任意长度的字串。

- Integers - php支持的任何大小的所有整型，最多64位（bit）有符号。

- Floats - 所有有符号的浮点值。

- Boolean - True 和 False。

- Null - 真实的null值。

- Arrays - 索引数组，关联数组和任意深度的多维数组。

- Object - 任何支持无损序列和反序列化化的对象，比如 ``$o == unserialize(serialize($o))``。对象MAY（可以）利用PHP的``Serializable``接口，__sleep()或__wakeup()魔术方法，或适当的类似的语言功能。

所有传入实现库的数据MUST（必须）完全按原样返回，包括数据类型。比如，传入(int)5但返回(string)5就是错误的。
实现库MAY（可以）在内部使用PHP的serialize()/unserialize()方法，但不是必须这样做。
与它们的兼容性仅用作可接受对象值的基准。

如果由于任何原因不能完全按原样返回保存的数据，实现库MUST（必须）以缓存未命中而不是损坏的数据进行响应。

## Key Concepts - 关键概念

### Pool - 池

池代表在缓存系统中的数据集合。
池是它所包含的所有数据的逻辑存储库。
所有可缓存数据作为item对象从池中检索，并且通过池与整个“缓存世界”的所有交互。

### Items - 项

一个item代表池的一个key/value对。key是用来唯一标识一个item的并且MUST（必须）一直保持不变。value MAY（可以）随时改变。

## Interface - 接口

### CacheItemInterface

```CacheItemInterface```定义缓存系统内的Item。
每个Item对象必须与一个特定的key关联，该key可以根据实现系统进行设置，并且通常由``Cache\CacheItemPoolInterface``对象I递。
``Cache\CacheItemInterface``对象封装了缓存项的存储和检索。
每个``Cache\CacheItemInterface``都由``Cache\CacheItemPoolInterface``对象生成，它负责任何所需的设置以及将对象与唯一的key进行关联。
``Cache\CacheItemInterface``对象MUST（必须）可以存储和检索本文档Data章节中定义的任何类型的php值。
调用库MUST NOT（禁止）自行实例化Item对象。它们只能通过getItem()方法从Pool对象请求。
调用库SHOULD NOT（不应该）假设由一个实现库创建的item会被另一个实现库的Pool对象兼容。

```php
<?php

namespace Psr\Cache;

/**
 * CacheItemInterface defines an interface for interacting with objects inside a cache.
 */
interface CacheItemInterface
{
    /**
     * Returns the key for the current cache item.
     *
     * The key is loaded by the Implementing Library, but should be available to
     * the higher level callers when needed.
     *
     * @return string
     *   The key string for this cache item.
     */
    public function getKey();

    /**
     * Retrieves the value of the item from the cache associated with this object's key.
     *
     * The value returned must be identical to the value originally stored by set().
     *
     * If isHit() returns false, this method MUST return null. Note that null
     * is a legitimate cached value, so the isHit() method SHOULD be used to
     * differentiate between "null value was found" and "no value was found."
     *
     * @return mixed
     *   The value corresponding to this cache item's key, or null if not found.
     */
    public function get();

    /**
     * Confirms if the cache item lookup resulted in a cache hit.
     *
     * Note: This method MUST NOT have a race condition between calling isHit()
     * and calling get().
     *
     * @return bool
     *   True if the request resulted in a cache hit. False otherwise.
     */
    public function isHit();

    /**
     * Sets the value represented by this cache item.
     *
     * The $value argument may be any item that can be serialized by PHP,
     * although the method of serialization is left up to the Implementing
     * Library.
     *
     * @param mixed $value
     *   The serializable value to be stored.
     *
     * @return static
     *   The invoked object.
     */
    public function set($value);

    /**
     * Sets the expiration time for this cache item.
     *
     * @param \DateTimeInterface|null $expiration
     *   The point in time after which the item MUST be considered expired.
     *   If null is passed explicitly, a default value MAY be used. If none is set,
     *   the value should be stored permanently or for as long as the
     *   implementation allows.
     *
     * @return static
     *   The called object.
     */
    public function expiresAt($expiration);

    /**
     * Sets the expiration time for this cache item.
     *
     * @param int|\DateInterval|null $time
     *   The period of time from the present after which the item MUST be considered
     *   expired. An integer parameter is understood to be the time in seconds until
     *   expiration. If null is passed explicitly, a default value MAY be used.
     *   If none is set, the value should be stored permanently or for as long as the
     *   implementation allows.
     *
     * @return static
     *   The called object.
     */
    public function expiresAfter($time);

}
```

### CacheItemPoolInterface

``Cache\CacheItemPoolInterface ``的主要目的是从调用库接收一个key并返回相关的``Cache\CacheItemInterface``对象。
它同时也是和整个缓存集合进行交互的关键点。
Pool（池）的所有配置和初始化都由实现库决定。

```php
<?php

namespace Psr\Cache;

/**
 * CacheItemPoolInterface generates CacheItemInterface objects.
 */
interface CacheItemPoolInterface
{
    /**
     * Returns a Cache Item representing the specified key.
     *
     * This method must always return a CacheItemInterface object, even in case of
     * a cache miss. It MUST NOT return null.
     *
     * @param string $key
     *   The key for which to return the corresponding Cache Item.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return CacheItemInterface
     *   The corresponding Cache Item.
     */
    public function getItem($key);

    /**
     * Returns a traversable set of cache items.
     *
     * @param string[] $keys
     *   An indexed array of keys of items to retrieve.
     *
     * @throws InvalidArgumentException
     *   If any of the keys in $keys are not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return array|\Traversable
     *   A traversable collection of Cache Items keyed by the cache keys of
     *   each item. A Cache item will be returned for each key, even if that
     *   key is not found. However, if no keys are specified then an empty
     *   traversable MUST be returned instead.
     */
    public function getItems(array $keys = array());

    /**
     * Confirms if the cache contains specified cache item.
     *
     * Note: This method MAY avoid retrieving the cached value for performance reasons.
     * This could result in a race condition with CacheItemInterface::get(). To avoid
     * such situation use CacheItemInterface::isHit() instead.
     *
     * @param string $key
     *   The key for which to check existence.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if item exists in the cache, false otherwise.
     */
    public function hasItem($key);

    /**
     * Deletes all items in the pool.
     *
     * @return bool
     *   True if the pool was successfully cleared. False if there was an error.
     */
    public function clear();

    /**
     * Removes the item from the pool.
     *
     * @param string $key
     *   The key to delete.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if the item was successfully removed. False if there was an error.
     */
    public function deleteItem($key);

    /**
     * Removes multiple items from the pool.
     *
     * @param string[] $keys
     *   An array of keys that should be removed from the pool.

     * @throws InvalidArgumentException
     *   If any of the keys in $keys are not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if the items were successfully removed. False if there was an error.
     */
    public function deleteItems(array $keys);

    /**
     * Persists a cache item immediately.
     *
     * @param CacheItemInterface $item
     *   The cache item to save.
     *
     * @return bool
     *   True if the item was successfully persisted. False if there was an error.
     */
    public function save(CacheItemInterface $item);

    /**
     * Sets a cache item to be persisted later.
     *
     * @param CacheItemInterface $item
     *   The cache item to save.
     *
     * @return bool
     *   False if the item could not be queued or if a commit was attempted and failed. True otherwise.
     */
    public function saveDeferred(CacheItemInterface $item);

    /**
     * Persists any deferred cache items.
     *
     * @return bool
     *   True if all not-yet-saved items were successfully saved or there were none. False otherwise.
     */
    public function commit();
}
```

### CacheException

此异常接口旨在用于发生严重（critical）错误时，包括但不限于缓存设置，如连接到缓存服务器或提供的无效凭据。

实现库抛出的异常必须实现该接口。

```php
<?php

namespace Psr\Cache;

/**
 * Exception interface for all exceptions thrown by an Implementing Library.
 */
interface CacheException
{
}
```

### InvalidArgumentException

```php
<?php

namespace Psr\Cache;

/**
 * Exception interface for invalid cache arguments.
 *
 * Any time an invalid argument is passed into a method it must throw an
 * exception class which implements Psr\Cache\InvalidArgumentException.
 */
interface InvalidArgumentException extends CacheException
{
}
```
