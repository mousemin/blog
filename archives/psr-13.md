---
title: PSR-13 超媒体链接
date: 2021-10-26T10:29:30+08:00
Description:
Tags: 
 - PHP
Categories:
 - PHP PSR标准规范
DisableComments: false
---

在 `HTML` 和各种 `API` 格式的上下文中，超媒体链接已经变成 `Web` 越来越重要的一部分。然而遗憾的是，没有一种通用单一的超媒体格式，也没有一种通用的方式来表示链接间的格式。
该规范旨在为 `PHP` 开发人员提供一种简单的、通用的方式来表示一个独立于所使用的序列化格式的超媒体链接。 这反过来又允许系统将超媒体链接的响应序列化为一种或多种有线格式，而不依赖于决定这些链接应该是什么的过程。

本文中的 `必须`，`不得`，`需要`，`应`，`不应`，`应该`，`不应该`，`推荐`，`可以` 和 `可选` 等能愿动词按照 [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。

### 参考文献

- [RFC 2119](https://tools.ietf.org/html/rfc2119)
- [RFC 4287](https://tools.ietf.org/html/rfc4287)
- [RFC 5988](https://tools.ietf.org/html/rfc5988)
- [RFC 6570](https://tools.ietf.org/html/rfc6570)
- [IANA Link Relations Registry](http://www.iana.org/assignments/link-relations/link-relations.xhtml)
- [Microformats Relations List](http://microformats.org/wiki/existing-rel-values#HTML5_link_type_extensions)

## 规范

### 基本链接

一个超媒体链接至少是由这些组成的：

- 一个 `URI` 表示目标资源被引用。
- 一个关系定义如何把目标资源与源联系起来。


链接的各种其他属性可能存在，具体取决于所使用的格式。作为额外的属性将不能标准化和通用，故本规范不寻求规范他们。

就本规范而言，下列定义是适用的：

- `Implementing Object` - 通过这个规范一个对象实现一个接口的定义。
- `Serializer` - 一个库或者其它系统需要一个或多个 Link 对象，并用它定义的一些格式产生一个序列化的表示。


### 属性

所有的链接 **可以** 包含零个或者多个 `URI` 和关系之外的附加属性。没有正式的在这里允许的能注册的值和值的有效性取决于上下文，并且通常取决于一个特定的序列化格式。一般情况下支持的值包括 `hreflang`、`title` 和 `type`。

如果序列化格式需要，序列化 **可以** 忽略链接对象上的属性。不管怎样，序列化 **应该** 对所有提供的属性尽可能地进行编码，以便允许用户扩展，除非有通过序列化格式的定义去阻止的情况。

一些属性（一般为 `hreflang`）在他们的上下文中 **可以** 多次出现。因此，一个属性值 **可以** 是一个数组形式的值而不是一个简单的值。序列化 **可以** 对任何适合于序列化格式的格式对该数组进行编码（比如：一个空格分隔的列表，逗号分隔的列表等等）。如果在一个特定的上下文中，指定的一个属性不允许有多个值，序列化 **必须** 使用第一个提供的值而忽略所有后续的值。

如果一个属性的值为布尔值 `true`，则序列化 **可以** 使用序列化格式支持的和合适的缩写形式。例如：当属性的存在有布尔意义时，`HTML` 允许属性没有值。当且仅当该属性为布尔值 `true` 时，这个规则才适用，而不适用于 `PHP` 中的其他任何 `truthy` 值，例如整数 `1`。

如果一个属性的值为布尔值 `false`，序列化 应该 完全省略属性，除非这样做会改变结果的语义含义。当且仅当该属性为布尔值 `false` 时，这个规则才适用，而不适用于 `PHP` 中的其他任何 `falsey` 值，例如整数 `0`。

### 关系

链接关系定义为字符串，在公开定义关系的情况下为一个简单关键字，或者在私有关系的情况下为一个绝对 URI。
在使用一个简单的关键字的情况下，它 **应该** 从 `IANA` 注册表中的一个匹配：

http://www.iana.org/assignments/link-relations/link-relations.xhtml

**可以** 选择使用 `microformats.org` 注册表，但这可能不适用于任何情况：

http://microformats.org/wiki/existing-rel-values


一个未在上述其中一个注册表或者一个类似的公共注册表中定义的关系被视为 `private`，也就是说，明确到一个特定的应用和使用场景。例如：关系必须使用一个绝对 `URI` 的情形。

### 链接模板
[RFC 6570](https://tools.ietf.org/html/rfc6570) 为 `URI` 模板定义了一种格式，也就是说，`URI` 这种模式期望通过客户端工具提供的值去填充。有些超媒体格式支持模板链接而有些则不支持，并且可能有一种特殊的方式来表示链接是一个模板。一个不支持 `URI` 模板格式化程序 **必须** 忽略它遇到的任何模板的链接。

### 可演进的提供者
在某些情况下，一个链接提供者可能需要添加其他链接的能力。在其他情况下，链接提供者必须是只读的，其中链接在运行时从其他某个数据源衍生。出于这个原因，可修改的提供者可以可选地实现辅助的接口。

另外，一些链接提供者对象，如 [PSR-7](/archives/psr-7/) 响应对象，被设计为不可变的。这意味着在就地添加链接的方法将是不兼容的。因此，`EvolvableLinkProviderInterface` 类的单一方法需要返回一个新的对象，与原始对象相同，但要包含一个额外的链接对象。

### 可演进的链接对象
链接对象在大部分情况下是值对象。因此，允许它们去演进，与 [PSR-7](/archives/psr-7/) 值对象一样是个有用的选项。为了这个缘故，一个额外的 `EvolvableLinkInterface` 类被包含进来，它提供了只需一次更改而生成新对象实例的方法。相同的模式被使用在 [PSR-7](/archives/psr-7/) 中，归功于 `PHP` 的 `copy-on-write` 机制，使 `CPU` 和内存依然高效。

然而，模板没有可演进的方法，由于一个链接的模板值是基于专门的 `href` 值。它被 **禁止** 独立地设置，但衍生自 `href` 值是否为 [RFC 6570](https://tools.ietf.org/html/rfc6570) 的链接模板。

## 包
描述的接口和类作为 [psr/link](https://packagist.org/packages/psr/link) 包的一部分被提供。

## 接口

### `Psr\Link\LinkInterface`

```php
<?php

namespace Psr\Link;

/**
 * 一个可读的链接对象。
 */
interface LinkInterface
{
    /**
     * 返回链接的目标。
     *
     * 目标链接必须是以下中的一个：
     * 
     * - 一个绝对的 URI，由 RFC 5988 定义的。
     * - 一个相对 URI，由 RFC 5988 定义的。相对链接的基础
     *   被假定为基于客户端的上下文而已知。
     * - 一个由 RFC 6570 定义的 URI 模板。
     *
     * 如果返回一个 URI 模板，isTemplated 必须返回 True。
     *
     * @return string
     */
    public function getHref();

    /**
     * 返回的是否为一个模板链接。
     *
     * @return bool True 表示链接对象是模板, False 相反。
     */
    public function isTemplated();

    /**
     * 返回链接的关系类型。
     *
     * 此方法返回一个链接的 0 个或更多关系类型，返回值为
     * 字符串数组。
     *
     * @return string[]
     */
    public function getRels();

    /**
     * 返回描述目标 URI 的一个属性列表。
     * 
     * @return array
     *  属性的一个键值对列表，其中键是一个字符串，值要么是一个 PHP 原生提供的，要么是 PHP 字符串数组。
     *  如果没有值，必须返回一个空的数组。
     */
    public function getAttributes();
}
```

### `Psr\Link\EvolvableLinkInterface`

```php
<?php

namespace Psr\Link;

/**
 * 一个可演进的值对象.
 */
interface EvolvableLinkInterface extends LinkInterface
{
    /**
     * 返回一个指定的 href 实例。
     * 
     * @param string $href
     *  这个 href 值必须包括以下其中一项：
     *   - 一个由 RFC 5988 定义的绝对 URI。
     *   - 一个由 RFC 5988 定义的相对 URI。相对链接的基准假设是由已知客户端基于上下文的。
     *   - 一个由 RFC 6570 定义的 URI 模板。
     *   - 一个实现 __toString() 方法的对象，它产生上述某个值。
     *   
     *   一个实现库应当立即将传递的对象评估为字符串，而不是等待它稍后返回。
     *   
     * @return static
     */
    public function withHref($href);

    /**
     * 返回一个包含指定关系的实例。
     *
     * 如果指定的 rel 已经存在，这个方法必须正常返回而没有错误，但不会再次添加 rel。
     *
     * @param string $rel 要添加的关系值。
     * 
     * @return static
     */
    public function withRel($rel);

    /**
     * 返回一个排除指定关系的实例。
     *
     * 如果指定的 rel 已经不存在，这个方法必须正常返回而没有错误。
     *
     * @param string $rel 要排除的关系值。
     * 
     * @return static
     */
    public function withoutRel($rel);

    /**
     * 返回一个添加了指定属性的实例。
     * 
     * 如果指定的属性已经存在，那么属性的值将被新值覆盖。
     * 
     * @param string $attribute 包含的属性键名。
     * @param string $value 属性待设置的值。
     * 
     * @return static
     */
    public function withAttribute($attribute, $value);

    /**
     * 返回一个排除了指定属性的实例。
     * 
     * 如果指定的属性不存在，这个方法必须正常返回而没有错误。
     * 
     * @param string $attribute 移除的属性键名。
     * 
     * @return static
     */
    public function withoutAttribute($attribute);
}
```

### `Psr\Link\LinkProviderInterface`

```php
<?php

namespace Psr\Link;

/**
 * 一个链接提供者对象.
 */
interface LinkProviderInterface
{
    /**
     * 返回一个可迭代的 LinkInterface 对象。
     * 
     * 迭代可能是一个数组或者任何实现 PHP \Traversable 接口的对象。
     * 如果没有可用的链接，一个空的数组或者实现 \Traversable 接口的
     * 对象必须被返回。
     * 
     * @return LinkInterface[]|\Traversable
     */
    public function getLinks();

    /**
     * 返回一个指定关系的可迭代 LinkInterface 对象。
     * 
     * 迭代可能是一个数组或者任何实现 PHP \Traversable 接口的对象。
     * 如果没有与该关系的链接是可用的，一个空的数组或者实现 \Traversable 
     * 接口的对象必须被返回。
     *
     * @return LinkInterface[]|\Traversable
     */
    public function getLinksByRel($rel);
}
```

### `Psr\Link\EvolvableLinkProviderInterface`

```php
<?php

namespace Psr\Link;

/**
 * 一个可演进的链接提供者值对象.
 */
interface EvolvableLinkProviderInterface extends LinkProviderInterface
{
    /**
     * 返回一个包含指定链接的实例。
     * 
     * 如果指定的链接已经存在，这个方法必须正常返回而没有错误。
     * 如果 $link 全等于（===）集合中已有的 link 对象，则链接存在。
     *
     * @param LinkInterface $link 应该包含在此集合中的链接对象。
     * 
     * @return static
     */
    public function withLink(LinkInterface $link);

    /**
     * 返回一个移除指定链接的实例。
     * 
     * 如果指定的链接不存在，这个方法必须正常返回而没有错误。
     * 如果 $link 全等于（===）集合中已有的 link 对象，则链接存在。
     *
     * @param LinkInterface $link 移除的链接。
     * 
     * @return static
     */
    public function withoutLink(LinkInterface $link);
}
```