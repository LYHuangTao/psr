
- 消息MAY（可能）包含占位符，实现者MAY（可以）使用上下文数组中的值替换它。

    占位符名MUST（必须）对应上下文数组中的key。

    占位符名MUST（必须）用一个开始花括号 **{** 和一个闭合花括号 **}** 分隔。分割符和占位符名之间MUST NOT（禁止)有空白符出现。

    占位符名SHOULD（应该）只由字符 ``A-Z``，``a-z``，``0-9``，下划线``_``和点``.``组成。其它字符的使用保留用于将来修改占位符规范。

    实现者MAY（可以）使用占位符来实现各种转义策略并转换日志以供显示。用户SHOULD NOT（不应该）预转义占位符值，因为他们无法知道数据将在哪个上下中显示。

    下面是占位符插值的示例实现，仅供参考。


```
<?php

/**
 * Interpolates context values into the message placeholders.
 */
function interpolate($message, array $context = array())
{
    // build a replacement array with braces around the context keys
    $replace = array();
    foreach ($context as $key => $val) {
        // check that the value can be casted to string
        if (!is_array($val) && (!is_object($val) || method_exists($val, '__toString'))) {
            $replace['{' . $key . '}'] = $val;
        }
    }

    // interpolate replacement values into the message and return
    return strtr($message, $replace);
}

// a message with brace-delimited placeholder names
$message = "User {username} created";

// a context array of placeholder names => replacement values
$context = array('username' => 'bolivar');

// echoes "User bolivar created"
echo interpolate($message, $context);
```

### 1.3 Context - 上下文

- 每个方法都会接收一个数组作为上下文数据。这意味着保存任何不适合字符串的无关信息。这个数组可以包含任何值。实现者MUST（必须）确保他们尽可能正确地对待上下文数据。一个给定的上下文中的值MUST NOT（禁止）抛出异常也不能造成任何php error、warning或notice。

- 如果一个``Exception``对象被传入上下文数据中，它MUST（必须）在``exception``key中。记录异常是一种常见的模式，这允许实现者在日志后端支持时从异常中体恤堆栈跟踪。实现者MUST（必须）在使用它之前验证``exception`` key真的是一个``Exception``,因为MAY（可能）包含任何值。

### 1.4 Helper classes and interfaces - 助手类和接口

- ``Psr\Log\AbstractLogger``类使你实现``LoggerInterface``非常简单，extent（继承）它并实现实体`log`方法。其它八种方法是将消息和上下文转发给它。

- 相似的，使用``Psr\Log\LoggerTrait``只需要你实现实体``log``方法。注意因为traits不能implement（实现）接口，这种情况下你仍需要实现``LoggerInterface``。

- ``Psr\Log\NullLogger``随接口一起提供。如果没有为它们提供记录器，接口的用户MAY（可以）使用它来提后备“黑洞”实现。然而，如果上下文数据创建是昂贵的话，条件记录可能是更好的途径。

- ``Psr\Log\LoggerAwareInterface``只包含一个``setLogger(LoggerInterface $logger)``方法并且框架可以使用记录器连接任意示例。

- ``Psr\Log\LoggerAwareTrait`` trait可以用于在任意类中轻松实现相等的接口。它为你提供了``$this->logger``的访问。

- ``Psr\Log\LogLevel``类包含了八个日志等级的常量。

## 2.Package - 包

上面提到的interface和class以及相关的异常类和验证实现的测试套件是psr/log包的一部分提供的。

## 3.``Psr\Log\LoggerInterface``

```php
<?php

namespace Psr\Log;

/**
 * Describes a logger instance.
 *
 * The message MUST be a string or object implementing __toString().
 *
 * The message MAY contain placeholders in the form: {foo} where foo
 * will be replaced by the context data in key "foo".
 *
 * The context array can contain arbitrary data, the only assumption that
 * can be made by implementors is that if an Exception instance is given
 * to produce a stack trace, it MUST be in a key named "exception".
 *
 * See https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md
 * for the full interface specification.
 */
interface LoggerInterface
{
    /**
     * System is unusable.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function emergency($message, array $context = array());

    /**
     * Action must be taken immediately.
     *
     * Example: Entire website down, database unavailable, etc. This should
     * trigger the SMS alerts and wake you up.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function alert($message, array $context = array());

    /**
     * Critical conditions.
     *
     * Example: Application component unavailable, unexpected exception.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function critical($message, array $context = array());

    /**
     * Runtime errors that do not require immediate action but should typically
     * be logged and monitored.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function error($message, array $context = array());

    /**
     * Exceptional occurrences that are not errors.
     *
     * Example: Use of deprecated APIs, poor use of an API, undesirable things
     * that are not necessarily wrong.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function warning($message, array $context = array());

    /**
     * Normal but significant events.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function notice($message, array $context = array());

    /**
     * Interesting events.
     *
     * Example: User logs in, SQL logs.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function info($message, array $context = array());

    /**
     * Detailed debug information.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function debug($message, array $context = array());

    /**
     * Logs with an arbitrary level.
     *
     * @param mixed $level
     * @param string $message
     * @param array $context
     * @return void
     */
    public function log($level, $message, array $context = array());
}

```

## 4.``Psr\Log\LoggerAwareInterface``

```php
<?php

namespace Psr\Log;

/**
 * Describes a logger-aware instance.
 */
interface LoggerAwareInterface
{
    /**
     * Sets a logger instance on the object.
     *
     * @param LoggerInterface $logger
     * @return void
     */
    public function setLogger(LoggerInterface $logger);
}
```

## 5.``Psr\Log\LogLevel``

```php
<?php

namespace Psr\Log;

/**
 * Describes log levels.
 */
class LogLevel
{
    const EMERGENCY = 'emergency';
    const ALERT     = 'alert';
    const CRITICAL  = 'critical';
    const ERROR     = 'error';
    const WARNING   = 'warning';
    const NOTICE    = 'notice';
    const INFO      = 'info';
    const DEBUG     = 'debug';
}
```