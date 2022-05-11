---
title: PSR-3 日志接口规范
date: 2021-07-26T11:51:44+08:00
Description:
Tags: 
 - PHP
Categories:
 - PHP PSR标准规范
DisableComments: false
---

本文制定了日志类库的通用接口规范。

本规范的主要目的，是为了让日志类库以简单通用的方式，通过接收一个 `Psr\Log\LoggerInterface` 对象，来记录日志信息。 框架以及 `CMS` 内容管理系统如有需要，可以 对此接口进行扩展，但需遵循本规范，

这才能保证在使用第三方的类库文件时，日志接口仍能正常对接。
本文中的 `必须`，`不得`，`需要`，`应`，`不应`，`应该`，`不应该`，`推荐`，`可以` 和 `可选` 等能愿动词按照 [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。

本文档中的 `implementor` 一词应理解为在日志的库中实现  `LoggerInterface`  的人。记录器的调用者称为 `user`。

## 规范

### 基本规范

- `LoggerInterface` 接口对外定义了八个方法，分别用来记录 [RFC 5424](http://tools.ietf.org/html/rfc5424) 中定义的八个等级的日志：`debug`、 `info`、 `notice`、 `warning`、 `error`、 `critical`、 `alert` 以及 `emergency`。

- 第九个方法 —— `log`，其第一个参数为记录的等级。可使用一个预先定义的等级常量作为参数来调用此方法，**必须** 与直接调用以上八个方法具有相同的效果。如果传入的等级常量参数没有预先定义，则 **必须** 抛出 `Psr\Log\InvalidArgumentException` 类型的异常。在不确定的情况下，使用者 **不应该** 使用未支持的等级常量来调用此方法。

### 消息
以上每个方法都接受一个字符串类型或者是有 `__toString()` 方法的对象作为记录信息参数，这样，实现者就能把它当成字符串来处理，否则实现者 **必须** 自己把它转换成字符串。

记录信息参数 **可以** 携带占位符，实现者 **可以** 根据上下文将其它替换成相应的值。

其中占位符 **必须** 与上下文数组中的键名保持一致。

占位符的名称 **必须** 由一个左花括号 `{` 以及一个右括号 `}` 包含。但花括号与名称之间 **不得** 有空格符。

占位符的名称 应该 只由 `A-Z`、`a-z`、`0-9`、下划线 `_`、以及英文的句号 `.` 组成，其它字符作为将来占位符规范的保留。

实现者 **可以** 通过对占位符采用不同的转义和转换策略，来生成最终的日志。
而使用者在不知道上下文的前提下，**不应该** 提前转义占位符。

以下是一个占位符使用的例子：
```php
<?php

/**
 * 用上下文信息替换记录信息中的占位符
 */
function interpolate($message, array $context = array())
{
    // 构建一个花括号包含的键名的替换数组
    $replace = array();
    foreach ($context as $key => $val) {
        // 检查该值是否可以转换为字符串
        if (!is_array($val) && (!is_object($val) || method_exists($val, '__toString'))) {
            $replace['{' . $key . '}'] = $val;
        }
    }

    // 替换记录信息中的占位符，最后返回修改后的记录信息。
    return strtr($message, $replace);
}

// 含有带花括号占位符的记录信息。
$message = "User {username} created";

// 带有替换信息的上下文数组，键名为占位符名称，键值为替换值。
$context = array('username' => 'bolivar');

// 输出 "User bolivar created"
echo interpolate($message, $context);
```
###  上下文

- 每个记录函数都接受一个上下文数组参数，用来装载字符串类型无法表示的信息。它 **可以** 装载任何信息，所以实现者 **必须** 确保能正确处理其装载的信息，对于其装载的数据， **不得** 抛出异常，或产生 PHP 出错、警告或提醒信息（`error`、`warning`、`notice`）。

- 如需通过上下文参数传入了一个 `Exception` 对象，**必须** 以 `exception` 作为键名。
记录异常信息是很普遍的，所以如果它能够在记录类库的底层实现，就能够让实现者从异常信息中抽丝剥茧。
当然，实现者在使用它时，**必须** 确保键名为 `exception` 的键值是否真的是一个 `Exception`，毕竟它 **可以** 装载任何信息。

### 助手类和接口

- `Psr\Log\AbstractLogger` 类使得只需继承它和实现其中的 `log` 方法，就能够很轻易地实现 `LoggerInterface `接口，而另外八个方法就能够把记录信息和上下文信息传给它。

- 同样地，使用 `Psr\Log\LoggerTrait` 也只需实现其中的 `log` 方法。不过，需要特别注意的是，在 `traits` 可复用代码块还不能实现接口前，还需要 `implement LoggerInterface`。

- 在没有可用的日志记录器时，`Psr\Log\NullLogger` 接口 **可以** 为使用者提供一个备用的日志 `黑洞`。不过，当上下文的构建非常消耗资源时，带条件检查的日志记录或许是更好的办法。

- `Psr\Log\LoggerAwareInterface` 接口仅包括一个 `setLogger(LoggerInterface $logger)` 方法，框架可以使用它实现自动连接任意的日志记录实例。

- `Psr\Log\LoggerAwareTrait` `trait` 可复用代码块可以在任何的类里面使用，只需通过它提供的 `$this->logger`，就可以轻松地实现等同的接口。

- `Psr\Log\LogLevel` 类装载了八个记录等级常量。

##  包
接口和类的描述、相关的异常类以及用于验证你所写代码的测试套件都将作为 [psr/log](https://packagist.org/packages/psr/log) 包的一部分提供。

## Psr\Log\LoggerInterface

```php
<?php

namespace Psr\Log;

/**
 * 描述一个日志记录器实例
 *
 * 该消息必须实现一个__toString()的字符串或者对象.
 *
 * 该消息可能包含以下形式的占位符: {foo}  
 * foo 将会被关键词 "foo"中的上下文数据替换.
 *
 * 上下文数组可以包含任意数据, 我们只能假设代码实现者
 * 如果给出一个生成堆栈跟踪的异常实例, 那么它的键名
 * 必须为 "exception"。
 *
 * 请前往 https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md
 * 查看完整的接口规范.
 */
interface LoggerInterface
{
    /**
     * 系统无法使用。
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function emergency($message, array $context = array());

    /**
     * 必须立即采取行动。
     *
     * 例如: 整个网站宕机了，数据库挂了，等等。 这应该
     * 发送短信通知警告你.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function alert($message, array $context = array());

    /**
     * 临界条件。
     *
     * 例如: 应用组件不可用，意外的异常。
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function critical($message, array $context = array());

    /**
     * 运行时错误不需要马上处理，
     * 但通常应该被记录和监控。
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function error($message, array $context = array());

    /**
     * 例外事件不是错误。
     *
     * 例如: 使用过时的API，API使用不当，不合理的东西不一定是错误。
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function warning($message, array $context = array());

    /**
     * 正常但重要的事件.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function notice($message, array $context = array());

    /**
     * 有趣的事件.
     *
     * 例如: 用户登录，SQL日志。
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function info($message, array $context = array());

    /**
     * 详细的调试信息。
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function debug($message, array $context = array());

    /**
     * 可任意级别记录日志。
     *
     * @param mixed $level
     * @param string $message
     * @param array $context
     * @return void
     */
    public function log($level, $message, array $context = array());
}
```

## `Psr\Log\LoggerAwareInterface`

```php
<?php

namespace Psr\Log;

/**
 * logger-aware 定义实例
 */
interface LoggerAwareInterface
{
    /**
     * 设置一个日志记录实例
     *
     * @param LoggerInterface $logger
     * @return void
     */
    public function setLogger(LoggerInterface $logger);
}
5. Psr\Log\LogLevel
<?php

namespace Psr\Log;

/**
 * 日志等级常量定义
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
