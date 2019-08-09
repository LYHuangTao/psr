# [PSR-11: Container Interface](https://www.php-fig.org/psr/psr-11/)

本文档描述了依赖注入容器的通用接口

``ContainerInterface``设置的目的是标准化框架和库如何使用容器来获取对象和参数（本文档的其余部分中称为entries）。

``implementor``在本文档中指在依赖注入库或框架实现``ContainerInterface``的人（实现者）。
依赖注入容器（DIC）的用户称为``user``（用户）。

## 1.Specification - 规格

### 1.1 Basics - 基础

#### 1.1.1 Entry identifiers - 条目标识符

条目标识符是至少一个字符的任何php合法字符串，用于唯一标识容器中的项目。
条目标识符是不透明的字符串，所以调用者SHOULD NOT（不应该）假设字符串的结构带有任何语义含义。

#### 1.1.2 Reading from a container - 从容器中读取

- ``Psr\Container\ContainerInterface``暴漏两个方法：``get``和``set``。

- ``get``有一个必须参数：条目标识符，MUST（必须）是一个字符串。
``get``可以返回任何内容（mixed value），如果容器不知道标识符，则抛出``NotFoundExceptionInterface``。
使用相同标识符的两次连续调用SHOULD（应该）返回相同的值。
然而，根据实现者的设计或用户的配置，可能会返回不同的值，因此用户SHOULD NOT（不应该）依赖于从两次连续调用中获取相同的值。
- ``has``有一个必须的参数：条目标识符，MUST（必须）是一个字符串。``has``MUST（必须）返回``true``如果一个容器知道这个条目标识符否则返回``false``。如果``has($id)``返回false，``get($id)``MUST（必须）抛出``NotFoundExceptionInterface``。

### 1.2 Exceptions - 异常

从容器直接抛出的异常SHOULD（应该）实现``Psr\Container\ContainerExceptionInterface``。

使用不存在的id调用``get``方法MUST（必须）抛出``Psr\Container\NotFoundExceptionInterface``。

### 1.3 Recommended usage - 推荐用法

用户SHOULD NOT（不应该）将容器传递给对象，以至于对象可以自行取出所需的依赖。
这意味着容器被用作Service Locator（服务定位器），这是一种通常不鼓励的模式。

更多详情可以参照[META](https://www.php-fig.org/psr/psr-11/meta/)文档的第四章节。

## 2. Package - 包

上述接口（interface）和类（class）以及相关的异常（Exception）作为[psr/container](https://packagist.org/packages/psr/container)包的一部分提供。

提供PSR容器实现的包应该声明他们提供``psr/container-implementation 1.0.0``。

需要其实现的项目应该require``psr/container-implementation 1.0.0``。

## 3. Interfaces - 接口

### 3.1 ``Psr\Container\ContainerInterface``

```php
<?php
namespace Psr\Container;

/**
 * Describes the interface of a container that exposes methods to read its entries.
 */
interface ContainerInterface
{
    /**
     * Finds an entry of the container by its identifier and returns it.
     *
     * @param string $id Identifier of the entry to look for.
     *
     * @throws NotFoundExceptionInterface  No entry was found for **this** identifier.
     * @throws ContainerExceptionInterface Error while retrieving the entry.
     *
     * @return mixed Entry.
     */
    public function get($id);

    /**
     * Returns true if the container can return an entry for the given identifier.
     * Returns false otherwise.
     *
     * `has($id)` returning true does not mean that `get($id)` will not throw an exception.
     * It does however mean that `get($id)` will not throw a `NotFoundExceptionInterface`.
     *
     * @param string $id Identifier of the entry to look for.
     *
     * @return bool
     */
    public function has($id);
}
```

### 3.2 ``Psr\Container\ContainerExceptionInterface``

```php
<?php
namespace Psr\Container;

/**
 * Base interface representing a generic exception in a container.
 */
interface ContainerExceptionInterface
{
}
```

### 3.3 ``Psr\Container\NotFoundExceptionInterface``

```php
<?php
namespace Psr\Container;

/**
 * No entry was found in the container.
 */
interface NotFoundExceptionInterface extends ContainerExceptionInterface
{
}
```