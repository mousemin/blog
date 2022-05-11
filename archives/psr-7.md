---
title: PSR-7 HTTP 消息接口规范
date: 2021-09-04T18:39:22+08:00
Description:
Tags: 
 - PHP
Categories:
 - PHP PSR标准规范
DisableComments: false
---

此文档描述了 [RFC 7230](http://tools.ietf.org/html/rfc7230) 和[RFC 7231](http://tools.ietf.org/html/rfc7231) HTTP 消息传递的接口，还有 [RFC 3986](http://tools.ietf.org/html/rfc3986) 里对 HTTP 消息的 URIs 使用。

HTTP 消息是 Web 技术发展的基础。浏览器或 HTTP 客户端如 curl 生成发送 HTTP 请求消息到 Web 服务器，Web 服务器响应 HTTP 请求。服务端的代码接受 HTTP 请求消息后返回 HTTP 响应消息。

通常 HTTP 消息对于终端用户来说是不可见的，但是作为 Web 开发者，我们需要知道 HTTP 机制，如何发起、构建、取用还有操纵 HTTP 消息，知道这些原理，以助我们更好的完成开发任务，无论这个任务是发起一个 HTTP 请求，或者处理传入的请求。 

每一个 HTTP 请求都有专属的格式：
```
POST /path HTTP/1.1
Host: example.com

foo=bar&baz=bat
```
第一行的各个字段意义为： HTTP 请求方法、请求的目标地址（通常是一个绝对路径的 URI 或者路径），HTTP 协议。接下来是 HTTP 头信息，在这个例子中：目的主机。接下来是空行，然后是消息内容。

HTTP 返回消息有类似的结构：
```
HTTP/1.1 200 OK
Content-Type: text/plain

这是返回的消息内容
```
第一行为状态行，包括 HTTP 协议版本，HTTP 状态码，描述文本。和 HTTP 请求类似的，接下来是 HTTP 头信息，在这个例子中：内容类型。接下来是空行，然后是消息内容。
此文档探讨的是 HTTP 请求消息接口，和构建 HTTP 消息需要的元素数据定义。


本文中的 `必须`，`不该`，`需要`，`应`，`不应`，`应该`，`不应该`，`推荐`，`可以` 和 `可选` 等能愿动词按照 [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。

**参考文献**

- [RFC 2119](http://tools.ietf.org/html/rfc2119) 
- [RFC 3986](http://tools.ietf.org/html/rfc3986)
- [RFC 7230](http://tools.ietf.org/html/rfc7230) 
- [RFC 7231](http://tools.ietf.org/html/rfc7231) 

## 详细描述

### 消息

一个 HTTP 消息是指来自于客户端到服务端的请求或者服务端到客户端的响应。以下这两个文档分别为 HTTP 的消息接口做了详细定义 `Psr\Http\Message\RequestInterface` 和 `Psr\Http\Message\ResponseInterface` 。

`Psr\Http\Message\RequestInterface` 和 `Psr\Http\Message\ResponseInterface` 继承于 `Psr\Http\Message\MessageInterface` 。当接口 `Psr\Http\Message\MessageInterface` 可能被直接实现的时候，实现者应该实现 `Psr\Http\Message\RequestInterface` 接口和 `Psr\Http\Message\ResponseInterface` 接口。
从这里开始，当描述这些接口时，命名空间 `Psr\Http\Message` 将会被省略。

### HTTP 请求头信息

#### 大小写不敏感的字段名字

HTTP 消息包含大小写不敏感头信息。使用 `MessageInterface` 接口来设置和获取头信息，大小写不敏感的定义在于，如果你设置了一个 `Foo` 的头信息，`foo` 的值会被重写，你也可以通过 `foo` 来拿到 `FoO` 头对应的值。

```php
$message = $message->withHeader('foo', 'bar');

echo $message->getHeaderLine('foo');
// 输出: bar

echo $message->getHeaderLine('FOO');
// 输出: bar

$message = $message->withHeader('fOO', 'baz');
echo $message->getHeaderLine('foo');
// 输出: baz
```

虽然头信息可以用大小写不敏感的方式取出，但是接口实现类 **必须** 保持自己的大小写规范，特别是用 `getHeaders()` 方法输出的内容。

因为一些非标准的 HTTP 应用程序，可能会依赖于大小写敏感的头信息，所以在此我们把主宰 HTTP 大小写的权利开放出来，以适用不同的场景。

#### 对应多条数组的头信息

为了适用一个 HTTP `键` 可以对应多条数据的情况，我们使用字符串配合数组来实现，你可以从一个 `MessageInterface` 取出数组或字符串，使用 `getHeaderLine($name)` 方法可以获取通过逗号分割的不区分大小写的字符串形式的所有值。也可以通过 `getHeader($name)` 获取数组形式头信息的所有值。

```php
$message = $message
    ->withHeader('foo', 'bar')
    ->withAddedHeader('foo', 'baz');

$header = $message->getHeaderLine('foo');
// $header 包含: 'bar, baz'

$header = $message->getHeader('foo');
// ['bar', 'baz']
```

>注意：并不是所有的头信息都可以适用逗号分割（例如 `Set-Cookie`），当处理这种头信息时候， `MessageInterace` 的继承类 应该 使用 `getHeader($name)` 方法来获取这种多值的情况。

#### 主机信息

在请求中，`Host` 头信息通常和 URI 的 `host` 信息，还有建立起 TCP 连接使用的 Host 信息一致。然而，HTTP 标准规范允许主机 `host` 信息与其他两个不一样。

在构建请求的时候，如果 `host` 头信息未提供的话，实现类库 **必须** 尝试着从 URI 中提取 `host` 信息。
`RequestInterface::withUri()` 会默认的，从传参的 `UriInterface` 实例中提取 `host` ，并替代请求中原有的 `host` 信息。

你可以提供传参第二个参数为 `true` 来保证返回的消息实例中，原有的 `host` 头信息不会被替代掉。

以下表格说明了当 `withUri()` 的第二个参数被设置为 `true` 的时，返回的消息实例中调用 `getHeaderLine('Host')` 方法会返回的内容：



|请求的 Host 信息[1](https://www.php-fig.org/psr/psr-7/#rhh)|请求 URI 中的 Host 信息[2](https://www.php-fig.org/psr/psr-7/#rhc)|传参进去 URI 的 Host[3](https://www.php-fig.org/psr/psr-7/#uhc)|结果|
|:-----:|:-----:|:----:|:-----:|
|||||
||foo.com||foo.com|
||foo.com|bar.com|foo.com|
|foo.com||bar.com|foo.com|
|foo.com|bar.com|baz.com|foo.com|


- 1 当前请求的 `Host` 头信息。
- 2 当前请求 `URI` 中的 `Host` 信息。
- 3 通过 `withUri()` 传参进入的 `URI` 中的 `host` 信息。

### 数据流

HTTP 消息包含开始的一行、头信息、还有消息的内容。HTTP 的消息内容有时候可以很小，有时候确是非常巨大。尝试使用字符串的形式来展示消息内容，会消耗大量的内存，使用数据流的形式来读取消息可以解决此问题。`StreamInterface` 接口用来隐藏具体的数据流读写实现。在一些情况下，消息类型的读取方式为字符串是能容许的，可以使用 `php://memory` 或者 `php://temp`。

`StreamInterface` 暴露出来几个接口，这些接口允许你读取、写入，还有高效的遍历内容。

数据流使用这个三个接口来阐明对他们的操作能力：`isReadable()`、`isWritable()` 和 `isSeekable()`。这些方法可以让数据流的操作者得知数据流能否能提供他们想要的功能。

每一个数据流的实例，都会有多种功能：可以只读、可以只写、可以读和写，可以随机读取，可以按顺序读取等。

`最终，StreamInterface` 定义了一个 `__toString()` 的方法，用来一次性以字符串的形式输出所有消息内容。
与请求和响应的接口不同的是，StreamInterface 并不强调不可修改性。因为在 PHP 的实现内，基本上没有办法保证不可修改性，因为指针的指向，内容的变更等状态，都是不可控的。作为读取者，可以调用只读的方法来返回数据流，以最大程度上保证数据流的不可修改性。使用者要时刻明确的知道数据流的可修改性，建议把数据流附加到消息实例中，来强迫不可修改的特性。

### 请求目标和 URI

根据 [RFC7230](http://tools.ietf.org/html/rfc7230)，请求消息包含`请求目标`做为请求行的第二个段落。请求目标可以是以下形式之一：

- **原始形式** ，由路径和查询字符串（如果存在）组成；这通常被称为相对 URL。通过 TCP 传输的消息通常是原始形式；scheme 和认证数据通常仅通过 CGI 变量存在。
- **绝对形式** ，包括 scheme 、认证数据（``user-info@]host[:port]``，其中括号中的项是可选的），路径（如果存在），查询字符串（如果存在）。这通常被称为绝对 URI，并且是 [RFC 3986](http://tools.ietf.org/html/rfc3986) 中详细说明的唯一指定 URI 的形式。这个形式通常在向 HTTP 代理发出请求时使用。
- **认证形式** ，只包含认证信息。通常仅用于从 HTTP 客户端和代理服务器之间建立连接请求时使用。
- **星号形式** ，仅由字符串 `*` 组成，并与 OPTIONS 方法一起使用，以确定 Web 服务器的性能。

除了这些请求目标之外，通常还有一个不同于请求目标的`有效 URL`。有效 URL 不在 HTTP 消息中传输，但它用于确定发出请求的协议（Http 或 Https）、端口和主机名。

有效 URL 由 `UriInterface` 接口表示。`UriInterface` 是 [RFC 3986](http://tools.ietf.org/html/rfc3986) （主要用例）中指定的 HTTP 和 HTTPS URI 的模型。该接口提供了与各种 URI 部分交互的方法，这将消除重复解析 URI 的需要。还定义了一个 `__toString`() 方法，用于将建模的 URI 转换为其字符串表示形式。

当使用 `getRequestTarget()` 方法检索请求目标时，默认情况下此方法将使用 URI 对象并提取所有必要的组件来构建 原始形式。原始形式 是迄今为止最常见的请求目标。

如果用户希望使用其他三种形式中，或者如果想要显式覆盖请求目标，则可以使用 `withRequestTarget()` 来实现。

调用此方法不会影响 URI，因为 URI 是从 `getUri()` 返回的。

例如，用户可能想要向服务器发起一个星号形式的请求：
```php
$request = $request
    ->withMethod('OPTIONS')
    ->withRequestTarget('*')
    ->withUri(new Uri('https://example.org/'));
```

这个示例最终可能会导致 HTTP 请求类似下例：
```
OPTIONS * HTTP/1.1
```

但是 HTTP 客户端将能够使用有效的 URL （来自 `getUri()` ）来确定协议、主机名和 TCP 端口号。

一个 HTTP 客户端 **必须** 忽略 `Uri::getPath()` 和 `Uri::getQuery()` 的值，而是用 `getRequestTarget()` 返回的值，默认为连接前面两个值。

选择未实现上面四种请求目标形式的客户端，**必须** 依然使用 `getRequestTarget()`。这些客户端 **必须** 拒绝它们不支持的请求目标，并且 **不该** 依赖于 `getUri()` 的值。

`RequestInterface` 提供了检索请求目标或用提供的请求目标创建一个新实例的方法。默认情况下，如果实例中没有专门组合请求目标， `getRequestTarget()` 将会返回组合 URI 的原始形式（如果没有组成 URI 则返回 `/`）。`withRequestTarget($requestTarget)` 使用指定的请求目标创建一个新实例，从而允许开发人员创建表示其他三个请求目标形式（绝对形式、认证形式和星号形式）。使用时，组合的 URI 实例仍然可以使用，特别是在客户端中，它可以用于创建与服务器的连接。

### 服务端请求

`RequestInterface` 提供了 HTTP 请求消息的通常表示形式。但是，由于服务器端环境的性质，服务器端请求需要额外的处理。服务器端处理需要考虑通用网关接口（ CGI ），更具体地说，需要考虑 PHP 通过其服务器 API （ SAPI ）对 CGI 的抽象和扩展。PHP 通过超级全局变量提供了关于输入编组的简化，例如：

- `$_COOKIE` ，反序列化了 HTTP cookie，并提供了简化的访问方式。
- `$_GET` ，反序列化了查询字符串并提供了简化的访问方式。
- `$_POST` ，对通过 urlencode 编码提交的 HTTP POST 信息进化反序列化并提供了简化的访问方式；通常可以认为是解析消息体的结果。
- `$_FILES` ，关于文件上传的元数据反序列化结果。
- `$_SERVER` ，提供了 CGI/SAPI 环境变量的访问，这些变量通常包括请求方法、请求 scheme、请求 URI 和报头。

`ServerRequestInterface` 继承于 `RequestInterface` ，提供围绕这些超全局变量的抽象访问。这种做法有助于减少开发人员对超全局的耦合，鼓励对代码的测试，并提升了测试人员对相应代码的测试能力。

服务器请求提供了一个附加的属性，`attributes`，以便于开发人员可以根据应用程序的特定规则（例如路径匹配、scheme 匹配、主机匹配等）自检、分解和匹配请求。这样，服务器请求还可以在多段请求逻辑中进行消息传递。

### 文件上传

`ServerRequestInterface` 指定了一种在规范化结构中检索上传文件树的方法，每个叶子都是一个 `UploadedFileInterface` 的实例。

超全局变量 `$_FILES` 在处理文件数组式的时候存在一些众所周知的问题。具体而言，页面的表单里有多个 input 框，name 属性是 `files[]`，然后提交文件，PHP 的 `$_FILES` 变量形式如下：
```php
array(
    'files' => array(
        'name' => array(
            0 => 'file0.txt',
            1 => 'file1.html',
        ),
        'type' => array(
            0 => 'text/plain',
            1 => 'text/html',
        ),
        /* 等等其他属性 */
    ),
)
```

而不是预期的：
```php
array(
    'files' => array(
        0 => array(
            'name' => 'file0.txt',
            'type' => 'text/plain',
            /* 等等其他属性 */
        ),
        1 => array(
            'name' => 'file1.html',
            'type' => 'text/html',
            /* 等等其他属性 */
        ),
    ),
)
```
这样造成的结果是开发人员必须知道这种语言实现细节，并为之编写特定的代码。另外，如果发生以下情况， $`_FILES` 会是空数组：

- HTTP 方法不是 POST。
- 单元测试的时候。
- 在非 SAPI 环境下运行的时候，比如 [ReactPHP](http://reactphp.org/)。

在这些情况下，数据需要以不同的方式获取。比如：

- 进程可以解析消息体来发现上传的文件。这种情况下，实现方式可以选择不将上传文件写入文件系统，而是将它们包装在流中以减少内存、I/O 和存储开销。
- 在单元测试的场景下，开发人员需要能够对文件上桩或模仿的方式来验证和检查不同场景的情况。

`getUploadedFiles()` 将为开发者提供规范化的结构。实现方式的返回定义是：

- 聚合上传文件的所有信息，并填充 `Psr\Http\Message\UploadedFileInterface` 实例。
- 重新创建提交的树结构，相应位置的叶结点都是一个适当的 `Psr\Http\Message\UploadedFileInterface` 实例。

引用的树结构 **应该** 模仿提交的文件结构。

在最简单的示例中，这可能是单个被命名的提交表单元素：
```html
<input type="file" name="avatar" />
```
在这种情况下，`$_FILES` 的结构如下：
```php
array(
    'avatar' => array(
        'tmp_name' => 'phpUxcOty',
        'name' => 'my-avatar.png',
        'size' => 90996,
        'type' => 'image/png',
        'error' => 0,
    ),
)
```

`getUploadedFiles()` 返回的规范化形式将是：
```php
array(
    'avatar' => /* UploadedFileInterface 实例 */
)
```

input 名称是一种数组表示形式的情况：
```html
<input type="file" name="my-form[details][avatar]" />
```

`$_FILES` 最终看下来像是这样的：
```php
array(
    'my-form' => array(
        'details' => array(
            'avatar' => array(
                'tmp_name' => 'phpUxcOty',
                'name' => 'my-avatar.png',
                'size' => 90996,
                'type' => 'image/png',
                'error' => 0,
            ),
        ),
    ),
)
```

`getUploadedFiles()` 的返回结果 **应该** 是：
```php
array(
    'my-form' => array(
        'details' => array(
            'avatar' => /* UploadedFileInterface 实例 */
        ),
    ),
)
```

在某些情况下，可以指定文件的 input 为一个数组：
```html
Upload an avatar: <input type="file" name="my-form[details][avatars][]" />
Upload an avatar: <input type="file" name="my-form[details][avatars][]" />
```
（例如，JavaScript 控件可能会产生额外的文件上传输入，以允许一次上传多个文件。）

这种情况下，其实现 必须 按给定的索引聚合所有上传文件的信息。因为这种情况下的 `$_FILES` 偏离了正常结构：
```php
array(
    'my-form' => array(
        'details' => array(
            'avatars' => array(
                'tmp_name' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'name' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'size' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'type' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'error' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
            ),
        ),
    ),
)
```
上面的 `$_FILES` 将对应于 `getUploadedFiles()` 返回的如下结构：
```php
array(
    'my-form' => array(
        'details' => array(
            'avatars' => array(
                0 => /* UploadedFileInterface 实例 */,
                1 => /* UploadedFileInterface 实例 */,
                2 => /* UploadedFileInterface 实例 */,
            ),
        ),
    ),
)
```
开发人员可以用以下形式访问嵌套数组的索引 `1`：
```php
$request->getUploadedFiles()['my-form']['details']['avatars'][1];
```
因为上传的文件数据是派生的（派生于 `$_FILES` 或请求体），所以接口还有一个设置方法 `withUploadedFiles()`，允许修改其内容。

在原始示例的情形下，接口调用者的代码可能如下所示：
```php
$file0 = $request->getUploadedFiles()['files'][0];
$file1 = $request->getUploadedFiles()['files'][1];

printf(
    "Received the files %s and %s",
    $file0->getClientFilename(),
    $file1->getClientFilename()
);

// "Received the files file0.txt and file1.html"
```

这个设计方案还考虑到实现方案可以在非 SAPI 环境中运行。 As such, `UploadedFileInterface` provides methods for ensuring operations will work regardless of environment. 特别是：

- `moveTo($targetPath)` 用来做为一个安全且推荐的代替在临时上传文件上调用 
- `move_uploaded_file()` 的方法。实现将根据环境检查正确的操作。
- `getStream()` 将会返回一个 `StreamInterface` 实例。在非 SAPI 环境中，提出的一种可能性是将单个上传文件解析为 `php://temp` 流而不是直接解析到文件；在这种情况下，不存在上传文件。 因此，无论环境如何，`getStream()` 都可以保证工作。

例如：
```php
// 移动文件至上传目录
$filename = sprintf(
    '%s.%s',
    create_uuid(),
    pathinfo($file0->getClientFilename(), PATHINFO_EXTENSION)
);
$file0->moveTo(DATA_DIR . '/' . $filename);

// 将文件流式传输至 Amazon S3。
// 假设 $s3wrapper 是一个将写入 S3 的 PHP 流，而 Psr7StreamWrapper 是一个将 StreamInterface 作为 PHP StreamWrapper 进行装饰的类。
$stream = new Psr7StreamWrapper($file1->getStream());
stream_copy_to_stream($stream, $s3wrapper);
```

## 扩展包

上面讨论的接口和类库已经整合成为扩展包：`psr/http-message`。
## 接口

### Psr\Http\Message\MessageInterface
```php
<?php
namespace Psr\Http\Message;

/**
 * 
 * HTTP 消息包括客户端向服务器发起的`请求`和服务器端返回给客户端的`响应`。
 * 此接口定义了他们通用的方法。
 * 
 * HTTP 消息是被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套
 * 机制，在内部保持好原有的内容，然后把修改状态后的信息返回。
 *
 * @see http://www.ietf.org/rfc/rfc7230.txt
 * @see http://www.ietf.org/rfc/rfc7231.txt
 */
interface MessageInterface
{
    /**
     * 获取字符串形式的 HTTP 协议版本信息。
     *
     * 字符串 **必须** 包含 HTTP 版本数字（如：`1.1`, `1.0`）。
     *
     * @return string HTTP 协议版本
     */
    public function getProtocolVersion();

    /**
     * 返回指定 HTTP 版本号的消息实例。
     *
     * 传参的版本号只 **必须** 包含 HTTP 版本数字，如："1.1", "1.0"。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个新的带有传参进去的 HTTP 版本的实例
     *
     * @param string $version HTTP 版本信息
     * @return self
     */
    public function withProtocolVersion($version);

    /**
     * 获取所有的报头信息
     *
     * 返回的二维数组中，第一维数组的`键`代表单条报头信息的名字，`值`是
     * 以数组形式返回的，见以下实例：
     *
     *     // 把`值`的数据当成字串打印出来
     *     foreach ($message->getHeaders() as $name => $values) {
     *         echo $name . ': ' . implode(', ', $values);
     *     }
     *
     *     // 迭代的循环二维数组
     *     foreach ($message->getHeaders() as $name => $values) {
     *         foreach ($values as $value) {
     *             header(sprintf('%s: %s', $name, $value), false);
     *         }
     *     }
     *
     * 虽然报头信息是没有大小写之分，但是使用 `getHeaders()` 会返回保留了原本
     * 大小写形式的内容。
     *
     * @return string[][] 返回一个两维数组，第一维数组的`键` **必须** 为单条报头信息的
     *     名称，对应的是由字串组成的数组，请注意，对应的`值` **必须** 是数组形式的。
     */
    public function getHeaders();

    /**
     * 检查是否报头信息中包含有此名称的值，不区分大小写
     *
     * @param string $name 不区分大小写的报头信息名称
     * @return bool 找到返回 true，未找到返回 false
     */
    public function hasHeader($name);

    /**
     * 根据给定的名称，获取一条报头信息，不区分大小写，以数组形式返回
     *
     * 此方法以数组形式返回对应名称的报头信息。
     *
     * 如果没有对应的报头信息，**必须** 返回一个空数组。
     *
     * @param string $name 不区分大小写的报头字段名称。
     * @return string[] 返回报头信息中，对应名称的，由字符串组成的数组值，如果没有对应
     *     的内容，**必须** 返回空数组。
     */
    public function getHeader($name);

    /**
     * 根据给定的名称，获取一条报头信息，不区分大小写，以逗号分隔的形式返回
     * 
     * 此方法返回所有对应的报头信息，并将其使用逗号分隔的方法拼接起来。
     *
     * 注意：不是所有的报头信息都可使用逗号分隔的方法来拼接，对于那些报头信息，请使用
     * `getHeader()` 方法来获取。
     * 
     * 如果没有对应的报头信息，此方法 **必须** 返回一个空字符串。
     *
     * @param string $name 不区分大小写的报头字段名称。
     * @return string 返回报头信息中，对应名称的，由逗号分隔组成的字串，如果没有对应
     *     的内容，**必须** 返回空字符串。
     */
    public function getHeaderLine($name);

    /**
     * 返回替换指定报头信息`键/值`对的消息实例。
     *
     * 虽然报头信息是不区分大小写的，但是此方法必须保留其传参时的大小写状态，并能够在
     * 调用 `getHeaders()` 的时候被取出。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个更新后带有传参进去报头信息的实例
     *
     * @param string $name 不区分大小写的报头字段名称。
     * @param string|string[] $value 报头信息或报头信息数组。
     * @return self
     * @throws \InvalidArgumentException 无效的报头字段或报头信息时抛出
     */
    public function withHeader($name, $value);

    /**
     * 返回一个报头信息增量的 HTTP 消息实例。
     *
     * 原有的报头信息会被保留，新的值会作为增量加上，如果报头信息不存在的话，字段会被加上。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param string $name 不区分大小写的报头字段名称。
     * @param string|string[] $value 报头信息或报头信息数组。
     * @return self
     * @throws \InvalidArgumentException 报头字段名称非法时会被抛出。
     * @throws \InvalidArgumentException 报头头信息的值非法的时候会被抛出。
     */
    public function withAddedHeader($name, $value);

    /**
     * 返回被移除掉指定报头信息的 HTTP 消息实例。
     *
     * 报头信息字段在解析的时候，**必须** 保证是不区分大小写的。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param string $name 不区分大小写的头部字段名称。
     * @return self
     */
    public function withoutHeader($name);

    /**
     * 获取 HTTP 消息的内容。
     *
     * @return StreamInterface 以数据流的形式返回。
     */
    public function getBody();

    /**
     * 返回指定内容的 HTTP 消息实例。
     *
     * 内容 **必须** 是 `StreamInterface` 接口的实例。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param StreamInterface $body 数据流形式的内容。
     * @return self
     * @throws \InvalidArgumentException 当消息内容不正确的时候抛出。
     */
    public function withBody(StreamInterface $body);
}
```

### Psr\Http\Message\RequestInterface#
```php
<?php
namespace Psr\Http\Message;

/**
 * 代表客户端向服务器发起请求的 HTTP 消息对象。
 *
 * 根据 HTTP 规范，此接口包含以下属性：
 *
 * - HTTP 协议版本号
 * - HTTP 请求方法
 * - URI
 * - 报头信息
 * - 消息内容
 *
 * 在构造 HTTP 请求对象的时候，如果没有提供 Host 信息，
 * 实现类库 **必须** 从给出的 URI 中去提取 Host 信息。
 *
 * HTTP 请求是被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的新的 HTTP 请求实例返回。
 */
interface RequestInterface extends MessageInterface
{
    /**
     * 获取消息的请求目标。
     * 
     * 获取消息的请求目标的使用场景，可能是在客户端，也可能是在服务器端，也可能是在指定信息的时候
     * （参阅下方的 `withRequestTarget()`）。
     * 
     * 在大部分情况下，此方法会返回组合 URI 的原始形式，除非被指定过（参阅下方的 `withRequestTarget()`）。
     *
     * 如果没有可用的 URI，并且没有设置过请求目标，此方法 **必须** 返回 `/`。
     *
     * @return string
     */
    public function getRequestTarget();

    /**
     * 返回一个指定目标的请求实例。
     * 
     * 如果请求需要非原始形式的请求目标——例如指定绝对形式、认证形式或星号形式——则此方法
     * 可用于创建指定请求目标的实例。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 请求实例，然后返回
     * 一个新的修改过的 HTTP 请求实例。
     *
     * @see [http://tools.ietf.org/html/rfc7230#section-2.7](http://tools.ietf.org/html/rfc7230#section-2.7) 
     * （关于请求目标的各种允许的格式）
     * 
     * @param mixed $requestTarget
     * @return self
     */
    public function withRequestTarget($requestTarget);

    /**
     * 获取当前请求使用的 HTTP 方法
     *
     * @return string HTTP 方法字符串
     */
    public function getMethod();

    /**
     * 返回更改了请求方法的消息实例。
     *
     * 虽然，在大部分情况下，HTTP 请求方法都是使用大写字母来标示的，但是，实现类库 **不应该**
     * 修改用户传参的大小格式。
     * 
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 请求实例，然后返回
     * 一个新的修改过的 HTTP 请求实例。
     *
     * @param string $method 大小写敏感的方法名
     * @return self
     * @throws \InvalidArgumentException 当非法的 HTTP 方法名传入时会抛出异常。
     */
    public function withMethod($method);

    /**
     * 获取 URI 实例。
     *
     * 此方法 **必须** 返回 `UriInterface` 的 URI 实例。
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @return UriInterface 返回与当前请求相关的 `UriInterface` 类型的 URI 实例。
     */
    public function getUri();

    /**
     * 返回修改了 URI 的消息实例。
     *
     * 当传入的 URI 包含有 HOST 信息时，此方法 **必须** 更新 HOST 信息。如果 URI 
     * 实例没有附带 HOST 信息，任何之前存在的 HOST 信息 **必须** 作为候补，应用
     * 更改到返回的消息实例里。
     * 
     * 你可以通过传入第二个参数来，来干预方法的处理，当 `$preserveHost` 设置为 `true` 
     * 的时候，会保留原来的 HOST 信息。当 `$preserveHost` 设置为 `true` 时，此方法
     * 会如下处理 HOST 信息：
     * 
     * - 如果 HOST 信息不存在或为空，并且新 URI 包含 HOST 信息，则此方法 **必须** 更新返回请求中的 HOST 信息。
     * - 如果 HOST 信息不存在或为空，并且新 URI 不包含 HOST 信息，则此方法 **不该** 更新返回请求中的 HOST 信息。
     * - 如果HOST 信息存在且不为空，则此方法 **不该** 更新返回请求中的 HOST 信息。
     * 
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 请求实例，然后返回
     * 一个新的修改过的 HTTP 请求实例。
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @param UriInterface $uri `UriInterface` 新的 URI 实例
     * @param bool $preserveHost 是否保留原有的 HOST 头信息
     * @return self
     */
    public function withUri(UriInterface $uri, $preserveHost = false);
}
```

### Psr\Http\Message\ServerRequestInterface

```php
<?php
namespace Psr\Http\Message;

/**
 * 表示服务器端接收到的 HTTP 请求。
 *
 * 根据 HTTP 规范，此接口包含以下属性：
 *
 * - HTTP 协议版本号
 * - HTTP 请求方法
 * - URI
 * - 报头信息
 * - 消息内容
 *
 * 此外，它封闭了从 CGI 和/或 PHP 环境变量，包括：
 *
 * - `$_SERVER` 中表示的值。
 * - 提供的任意 Cookie 信息（通常通过 `$_COOKIE` 获取）
 * - 查询字符串参数（通常通过 `$_GET` 获取，或者通过 `parse_str()` 解析）
 * - 如果存在的话，上传文件的信息（通常通过 `$_FILES` 获取）
 * - 反序列化的消息体参数（通常来自于 `$_POST`）
 *
 * `$_SERVER` 的值 **必须** 被视为不可变的，因为代表了请求时应用程序的状态；因此，没有允许修改的方法。
 * 其他值则提供了修改的方法，因为可以从 `$_SERVER` 或请求体中恢复，并且可能在应用程序中被处理
 * （比如可能根据内容类型对消息体参数进行反序列化）。
 *
 * 此外，这个接口要识别请求的扩展信息和匹配其他的参数。
 * （例如，通过 URI 进行路径匹配，解析 Cookie 值，反序列化非表单编码的消息体，报头中的用户名进行匹配认证）
 * 这些参数存储在`attributes`中。
 *
 * HTTP 请求是被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的 HTTP 请求实例返回。
 */
interface ServerRequestInterface extends RequestInterface
{
    /**
     * 返回服务器参数。
     *
     * 返回与请求环境相关的数据，通常从 PHP 的 `$_SERVER` 超全局变量中获取，但不是必然的。
     *
     * @return array
     */
    public function getServerParams();

    /**
     * 获取 Cookie 数据。
     *
     * 获取从客户端发往服务器的 Cookie 数据。
     *
     * 这个数据的结构 **必须** 和超全局变量 `$_COOKIE` 兼容。
     *
     * @return array
     */
    public function getCookieParams();

    /**
     * 返回具体指定 Cookie 的实例。
     *
     * 这个数据不是一定要来源于 `$_COOKIE`，但是 **必须** 与之结构兼容。通常在实例化时注入。
     *
     * 这个方法 **禁止** 更新实例中的 Cookie 报头和服务器参数中的相关值。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     * 
     * @param array $cookies 表示 Cookie 的键值对。
     * @return self
     */
    public function withCookieParams(array $cookies);

    /**
     * 获取查询字符串参数。
     *
     * 如果可以的话，返回反序列化的查询字符串参数。
     *
     * 注意：查询参数可能与 URI 或服务器参数不同步。如果你需要确保只获取原始值，则可能需要调用
     * `getUri()->getQuery()` 或服务器参数中的 `QUERY_STRING` 获取原始的查询字符串并自行解析。
     *
     * @return array
     */
    public function getQueryParams();

    /**
     * 返回具体指定查询字符串参数的实例。
     *
     * 这些值 **应该** 在传入请求的闭包中保持不变。它们 **可能** 在实例化的时候注入，
     * 例如来自 `$_GET` 或者其他一些值（例如 URI）中得到。如果是通过解析 URI 获取，则
     * 数据结构必须与 `parse_str()` 返回的内容兼容，以便处理查询参数、嵌套的代码可以复用。
     *
     * 设置查询字符串参数 **不该** 更改存储的 URI 和服务器参数中的值。
     * 
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param array $query 查询字符串参数数组，通常来源于 `$_GET`。
     * @return self
     */
    public function withQueryParams(array $query);

    /**
     * 获取规范化的上传文件数据。
     *
     * 这个方法会规范化返回的上传文件元数据树结构，每个叶子结点都是 `Psr\Http\Message\UploadedFileInterface` 实例。
     *
     * 这些值 **可能** 在实例化的时候从 `$_FILES` 或消息体中获取，或者通过 `withUploadedFiles()` 获取。
     *
     * @return array `UploadedFileInterface` 的实例数组；如果没有数据则必须返回一个空数组。
     */
    public function getUploadedFiles();

    /**
     * 返回使用指定的上传文件数据的新实例。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param array `UploadedFileInterface` 实例的树结构，类似于 `getUploadedFiles()` 的返回值。
     * @return self
     * @throws \InvalidArgumentException 如果提供无效的结构时抛出。
     */
    public function withUploadedFiles(array $uploadedFiles);

    /**
     * 获取请求消息体中的参数。
     *
     * 如果请求的 Content-Type 是 application/x-www-form-urlencoded 或 multipart/form-data 且请求方法是 POST，
     * 则此方法 **必须** 返回 $_POST 的内容。
     *
     * 如果是其他情况，此方法可能返回反序列化请求正文内容的任何结果；当解析返回返回的结构化内容时，潜在的类型 **必须**
     * 只能是数组或 `object` 类型。`null` 表示没有消息体内容。
     *
     * @return null|array|object 如果存在则返回反序列化消息体参数。一般是一个数组或 `object`。
     */
    public function getParsedBody();

    /**
     * 返回具有指定消息体参数的实例。
     *
     * **可能** 在实例化时注入。
     *
     * 如果请求的 Content-Type 是 application/x-www-form-urlencoded 或 multipart/form-data 且请求方法是 POST，
     * 则方法的参数只能是 $_POST。
     *
     * 数据不一定要来自 $_POST，但是 **必须** 是反序列化请求正文内容的结果。由于需要反序列化/解析返回的结构化数据，
     * 所以这个方法只接受数组、 `object` 类型和 `null`（如果没有可用的数据解析）。
     *
     * 例如，如果确定请求数据是一个 JSON，可以使用此方法创建具有反序列化参数的请求实例。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param null|array|object $data 反序列化的消息体数据，通常是数组或 `object`。
     * @return self
     * @throws \InvalidArgumentException 如果提供的数据类型不支持。
     */
    public function withParsedBody($data);

    /**
     * 获取从请求派生的属性。
     *
     * 请求`attributes`可用于从请求导出的任意参数：比如路径匹配操作的结果；解密 Cookie 的结果；
     * 反序列化非表单编码的消息体的结果；属性将是应用程序与请求特定的，并且可以是可变的。
     *
     * @return mixed[] 从请求派生的属性。
     */
    public function getAttributes();

    /**
     * 获取单个派生的请求属性。
     *
     * 获取 getAttributes() 中声明的某一个属性，如果不存在则返回提供的默认值。
     *
     * 这个方法不需要 hasAttribute 方法，因为允许在找不到指定属性的时候返回默认值。
     *
     * @see getAttributes()
     * @param string $name 属性名称。
     * @param mixed $default 如果属性不存在时返回的默认值。
     * @return mixed
     */
    public function getAttribute($name, $default = null);

    /**
     * 返回具有指定派生属性的实例。
     *
     * 此方法允许设置 getAttributes() 中声明的单个派生的请求属性。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @see getAttributes()
     * @param string $name 属性名。
     * @param mixed $value 属性值。
     * @return self
     */
    public function withAttribute($name, $value);

    /**
     * 返回移除指定属性的实例。
     *
     * 此方法允许移除 getAttributes() 中声明的单个派生的请求属性。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @see getAttributes()
     * @param string $name 属性名。
     * @return self
     */
    public function withoutAttribute($name);
}
```

### Psr\Http\Message\ResponseInterface#
```php
<?php
namespace Psr\Http\Message;

/**
 * 表示服务器返回的响应消息。
 *
 * 根据 HTTP 规范，此接口包含以下各项的属性：
 *
 * - 协议版本
 * - 状态码和原因短语
 * - 报头
 * - 消息体
 * 
 * HTTP 响应是被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的 HTTP 响应实例返回。
 */
interface ResponseInterface extends MessageInterface
{
    /**
     * 获取响应状态码。
     *
     * 状态码是一个三位整数，用于理解请求。
     *
     * @return int 状态码。
     */
    public function getStatusCode();

    /**
     * 返回具有指定状态码和原因短语（可选）的实例。
     *
     * 如果未指定原因短语，实现代码 **可能** 选择 RFC7231 或 IANA 为状态码推荐的原因短语。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @param int $code 三位整数的状态码。
     * @param string $reasonPhrase 为状态码提供的原因短语；如果未提供，实现代码可以使用 HTTP 规范建议的默认代码。
     * @return self
     * @throws \InvalidArgumentException 如果传入无效的状态码，则抛出。
     */
    public function withStatus($code, $reasonPhrase = '');

    /**
     * 获取与响应状态码关联的响应原因短语。
     *
     * 因为原因短语不是响应状态行中的必需元素，所以原因短语 **可能** 是空。
     * 实现代码可以选择返回响应的状态代码的默认 RFC 7231 推荐原因短语（或 IANA HTTP 状态码注册表中列出的原因短语）。
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @return string 原因短语；如果不存在，则 **必须** 返回空字符串。
     */
    public function getReasonPhrase();
}
```

### Psr\Http\Message\StreamInterface#
```php
<?php
namespace Psr\Http\Message;

/**
 * 描述数据流。
 *
 * 通常，实例将包装PHP流; 此接口提供了最常见操作的包装，包括将整个流序列化为字符串。
 */
interface StreamInterface
{
    /**
     * 从头到尾将流中的所有数据读取到字符串。
     *
     * 这个方法 **必须** 在开始读数据前定位到流的开头，并读取出所有的数据。
     *
     * 警告：这可能会尝试将大量数据加载到内存中。
     *
     * 这个方法 **不该** 抛出异常以符合 PHP 的字符串转换操作。
     *
     * @see http://php.net/manual/en/language.oop5.magic.php#object.tostring
     * @return string
     */
    public function __toString();

    /**
     * 关闭流和任何底层资源。
     *
     * @return void
     */
    public function close();

    /**
     * 从流中分离任何底层资源。
     *
     * 分离之后，流处于不可用状态。
     *
     * @return resource|null 如果存在的话，返回底层 PHP 流。
     */
    public function detach();

    /**
     * 如果可知，获取流的数据大小。
     *
     * @return int|null 如果可知，返回以字节为单位的大小，如果未知返回 `null`。
     */
    public function getSize();

    /**
     * 返回当前读/写的指针位置。
     *
     * @return int 指针位置。
     * @throws \RuntimeException 产生错误时抛出。
     */
    public function tell();

    /**
     * 返回是否位于流的末尾。
     *
     * @return bool
     */
    public function eof();

    /**
     * 返回流是否可随机读取。
     *
     * @return bool
     */
    public function isSeekable();

    /**
     * 定位流中的指定位置。
     *
     * @see http://www.php.net/manual/en/function.fseek.php
     * @param int $offset 要定位的流的偏移量。
     * @param int $whence 指定如何根据偏移量计算光标位置。有效值与 PHP 内置函数 `fseek()` 相同。
     *     SEEK_SET：设定位置等于 $offset 字节。默认。
     *     SEEK_CUR：设定位置为当前位置加上 $offset。
     *     SEEK_END：设定位置为文件末尾加上 $offset （要移动到文件尾之前的位置，offset 必须是一个负值）。
     * @throws \RuntimeException 失败时抛出。
     */
    public function seek($offset, $whence = SEEK_SET);

    /**
     * 定位流的起始位置。
     *
     * 如果流不可以随机访问，此方法将引发异常；否则将执行 seek(0)。
     *
     * @see seek()
     * @see http://www.php.net/manual/en/function.fseek.php
     * @throws \RuntimeException 失败时抛出。
     */
    public function rewind();

    /**
     * 返回流是否可写。
     *
     * @return bool
     */
    public function isWritable();

    /**
     * 向流中写数据。
     *
     * @param string $string 要写入流的数据。
     * @return int 返回写入流的字节数。
     * @throws \RuntimeException 失败时抛出。
     */
    public function write($string);

    /**
     * 返回流是否可读。
     *
     * @return bool
     */
    public function isReadable();

    /**
     * 从流中读取数据。
     *
     * @param int $length 从流中读取最多 $length 字节的数据并返回。如果数据不足，则可能返回少于
     *     $length 字节的数据。
     * @return string 返回从流中读取的数据，如果没有可用的数据则返回空字符串。
     * @throws \RuntimeException 失败时抛出。
     */
    public function read($length);

    /**
     * 返回字符串中的剩余内容。
     *
     * @return string
     * @throws \RuntimeException 如果无法读取则抛出异常。
     * @throws \RuntimeException 如果在读取时发生错误则抛出异常。
     */
    public function getContents();

    /**
     * 获取流中的元数据作为关联数组，或者检索指定的键。
     *
     * 返回的键与从 PHP 的 stream_get_meta_data() 函数返回的键相同。
     *
     * @see http://php.net/manual/en/function.stream-get-meta-data.php
     * @param string $key 要检索的特定元数据。
     * @return array|mixed|null 如果没有键，则返回关联数组。如果提供了键并且找到值，
     *     则返回特定键值；如果未找到键，则返回 null。
     */
    public function getMetadata($key = null);
}
```

### Psr\Http\Message\UriInterface#
```php
<?php
namespace Psr\Http\Message;

/**
 * URI 数据对象。
 *
 * 此接口按照 RFC 3986 来构建 HTTP URI，提供了一些通用的操作，你可以自由的对此接口
 * 进行扩展。你可以使用此 URI 接口来做 HTTP 相关的操作，也可以使用此接口做任何 URI 
 * 相关的操作。
 *
 * 此接口的实例化对象被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的实例返回。
 *
 * 通常，HOST 信息也将出现在请求消息中。对于服务器端的请求，通常可以在服务器参数中发现此信息。
 * 
 * @see [URI 通用标准规范](http://tools.ietf.org/html/rfc3986)
 */
interface UriInterface
{
    /**
     * 从 URI 中取出 scheme。
     *
     * 如果不存在 Scheme，此方法 **必须** 返回空字符串。
     *
     * 根据 RFC 3986 规范 3.1 章节，返回的数据 **必须** 是小写字母。
     *
     * 最后部分的`:`字串不属于 Scheme，**不该** 作为返回数据的一部分。
     *
     * @see https://tools.ietf.org/html/rfc3986#section-3.1
     * @return string URI Ccheme 的值。
     */
    public function getScheme();

    /**
     * 返回 URI 认证信息。
     *
     * 如果没有 URI 认证信息的话，**必须** 返回一个空字符串。
     *
     * URI 的认证信息语法是：
     *
     * <pre>
     * [user-info@]host[:port]
     * </pre>
     *
     * 如果端口部分没有设置，或者端口不是标准端口，**不应该** 包含在返回值内。
     *
     * @see https://tools.ietf.org/html/rfc3986#section-3.2
     * @return string URI 认证信息，格式为：`[user-info@]host[:port]`。
     */
    public function getAuthority();

    /**
     * 从 URI 中获取用户信息。
     *
     * 如果不存在用户信息，此方法 **必须** 返回一个空字符串。
     * 
     * 如果 URI 中存在用户，则返回该值；此外，如果密码也存在，它将附加到用户值，用冒号（`:`）分隔。
     *
     * 用户信息后面跟着的 "@" 字符，不是用户信息里面的一部分，**不该** 在返回值里出现。
     *
     * @return string URI 的用户信息，格式："username[:password]" 
     */
    public function getUserInfo();

    /**
     * 从 URI 中获取 HOST 信息。
     *
     * 如果 URI 中没有此值，**必须** 返回空字符串。
     *
     * 根据 RFC 3986 规范 3.2.2 章节，返回的数据 **必须** 是小写字母。
     *
     * @see http://tools.ietf.org/html/rfc3986#section-3.2.2
     * @return string URI 中的 HOST 信息。
     */
    public function getHost();

    /**
     * 从 URI 中获取端口信息。
     *
     * 如果端口信息是与当前 Scheme 的标准端口不匹配的话，就使用整数值的格式返回，如果是一
     * 样的话，**应该** 返回 `null` 值。
     * 
     * 如果不存在端口和 Scheme 信息，**必须** 返回 `null` 值。
     * 
     * 如果不存在端口数据，但是存在 Scheme 的话，**可能** 返回 Scheme 对应的
     * 标准端口，但是 **应该** 返回 `null`。
     * 
     * @return null|int URI 中的端口信息。
     */
    public function getPort();

    /**
     * 从 URI 中获取路径信息。
     *
     * 路径可以是空的，或者是绝对的（以斜线`/`开头），或者相对路径（不以斜线开头）。
     * 实现 **必须** 支持所有三种语法。
     *
     * 根据 RFC 7230 第 2.7.3 节，通常空路径``和绝对路径`/`被认为是相同的。
     * 但是这个方法 **不该** 自动进行这种规范化，因为在具有修剪的基本路径的上下文中，
     * 例如前端控制器中，这种差异将变得显著。用户的任务就是可以将``和`/`都处理好。
     *
     * 返回的值 **必须** 是百分号编码，但 **不该** 对任何字符进行双重编码。
     * 要确定要编码的字符，请参阅 RFC 3986 第 2 节和第 3.3 节。
     *
     * 例如，如果值包含斜线（`/`）而不是路径段之间的分隔符，则该值必须以编码形式（例如`%2F`）
     * 传递给实例。
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.3
     * @return string URI 路径信息。
     */
    public function getPath();

    /**
     * 获取 URI 中的查询字符串。
     *
     * 如果不存在查询字符串，则此方法必须返回空字符串。
     *
     * 前导的`?`字符不是查询字符串的一部分，**不该** 添加在返回值中。
     *
     * 返回的值 **必须** 是百分号编码，但 **不该** 对任何字符进行双重编码。
     * 要确定要编码的字符，请参阅 RFC 3986 第 2 节和第 3.4 节。
     *
     * 例如，如果查询字符串的键值对中的值包含不做为值之间分隔符的（`&`），则该值必须
     * 以编码形式传递（例如`%26`）到实例。
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.4
     * @return string URI 中的查询字符串
     */
    public function getQuery();

    /**
     * 获取 URI 中的片段（Fragment）信息。
     *
     * 如果没有片段信息，此方法 **必须** 返回空字符串。
     *
     * 前导的`#`字符不是片段的一部分，**不该** 添加在返回值中。
     *
     * 返回的值 **必须** 是百分号编码，但 **不该** 对任何字符进行双重编码。
     * 要确定要编码的字符，请参阅 RFC 3986 第 2 节和第 3.5 节。
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.5
     * @return string URI 中的片段信息。
     */
    public function getFragment();

    /**
     * 返回具有指定 Scheme 的实例。
     *
     * 此方法 **必须** 保留当前实例的状态，并返回包含指定 Scheme 的实例。
     *
     * 实现 **必须** 支持大小写不敏感的`http`和`https`的 Scheme，并且在
     * 需要的时候 **可能** 支持其他的 Scheme。
     *
     * 空的 Scheme 相当于删除 Scheme。
     *
     * @param string $scheme 给新实例使用的 Scheme。
     * @return self 具有指定 Scheme 的新实例。
     * @throws \InvalidArgumentException 使用无效的 Scheme 时抛出。
     * @throws \InvalidArgumentException 使用不支持的 Scheme 时抛出。
     */
    public function withScheme($scheme);

    /**
     * 返回具有指定用户信息的实例。
     *
     * 此方法 **必须** 保留当前实例的状态，并返回包含指定用户信息的实例。
     *
     * 密码是可选的，但用户信息 **必须** 包括用户；用户信息的空字符串相当于删除用户信息。
     * 
     * @param string $user 用于认证的用户名。
     * @param null|string $password 密码。
     * @return self 具有指定用户信息的新实例。
     */
    public function withUserInfo($user, $password = null);

    /**
     * 返回具有指定 HOST 信息的实例。
     *
     * 此方法 **必须** 保留当前实例的状态，并返回包含指定 HOST 信息的实例。
     *
     * 空的 HOST 信息等同于删除 HOST 信息。
     *
     * @param string $host 用于新实例的 HOST 信息。
     * @return self 具有指定 HOST 信息的实例。
     * @throws \InvalidArgumentException 使用无效的 HOST 信息时抛出。
     */
    public function withHost($host);

    /**
     * 返回具有指定端口的实例。
     *
     * 此方法 **必须** 保留当前实例的状态，并返回包含指定端口的实例。
     *
     * 实现 **必须** 为已建立的 TCP 和 UDP 端口范围之外的端口引发异常。
     *
     * 为端口提供的空值等同于删除端口信息。
     *
     * @param null|int $port 用于新实例的端口；`null` 值将删除端口信息。
     * @return self 具有指定端口的实例。
     * @throws \InvalidArgumentException 使用无效端口时抛出异常。
     */
    public function withPort($port);

    /**
     * 返回具有指定路径的实例。
     *
     * 此方法 **必须** 保留当前实例的状态，并返回包含指定路径的实例。
     *
     * 路径可以是空的、绝对的（以斜线开头）或者相对路径（不以斜线开头），实现必须支持这三种语法。
     *
     * 如果 HTTP 路径旨在与 HOST 相对而不是路径相对，，那么它必须以斜线开头。
     * 假设 HTTP 路径不以斜线开头，对应该程序或开发人员来说，相对于一些已知的路径。
     *
     * 用户可以提供编码和解码的路径字符，要确保实现了 `getPath()` 中描述的正确编码。
     *
     * @param string $path 用于新实例的路径。
     * @return self 具有指定路径的实例。
     * @throws \InvalidArgumentException 使用无效的路径时抛出。
     */
    public function withPath($path);

    /**
     * 返回具有指定查询字符串的实例。
     *
     * 此方法 **必须** 保留当前实例的状态，并返回包含查询字符串的实例。
     *
     * 用户可以提供编码和解码的查询字符串，要确保实现了 `getQuery()` 中描述的正确编码。
     *
     * 空查询字符串值等同于删除查询字符串。
     *
     * @param string $query 用于新实例的查询字符串。
     * @return self 具有指定查询字符串的实例。
     * @throws \InvalidArgumentException 使用无效的查询字符串时抛出。
     */
    public function withQuery($query);

    /**
     * 返回具有指定 URI 片段（Fragment）的实例。
     *
     * 此方法 **必须** 保留当前实例的状态，并返回包含片段的实例。
     *
     * 用户可以提供编码和解码的片段，要确保实现了 `getFragment()` 中描述的正确编码。
     *
     * 空片段值等同于删除片段。
     *
     * @param string $fragment 用于新实例的片段。
     * @return self 具有指定 URI 片段的实例。
     */
    public function withFragment($fragment);

    /**
     * 返回字符串表示形式的 URI。
     *
     * 根据 RFC 3986 第 4.1 节，结果字符串是完整的 URI 还是相对引用，取决于 URI 有哪些组件。
     * 该方法使用适当的分隔符连接 URI 的各个组件：
     *
     * - 如果存在 Scheme 则 **必须** 以`:`为后缀。
     * - 如果存在认证信息，则必须以`//`作为前缀。
     * - 路径可以在没有分隔符的情况下连接。但是有两种情况需要调整路径以使 URI 引用有效，因为 PHP
     *   不允许在 `__toString()` 中引发异常：
     *     - 如果路径是相对的并且有认证信息，则路径 **必须** 以`/`为前缀。
     *     - 如果路径以多个`/`开头并且没有认证信息，则起始斜线 **必须** 为一个。
     * - 如果存在查询字符串，则 **必须** 以`?`作为前缀。
     * - 如果存在片段（Fragment），则 **必须** 以`#`作为前缀。
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.1
     * @return string
     */
    public function __toString();
}
```

### Psr\Http\Message\UploadedFileInterface#
```php
<?php
namespace Psr\Http\Message;

/**
 * 通过 HTTP 请求上传的一个文件内容。
 *
 * 此接口的实例是被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的实例返回。
 */
interface UploadedFileInterface
{
    /**
     * 获取上传文件的数据流。
     *
     * 此方法必须返回一个 `StreamInterface` 实例，此方法的目的在于允许 PHP 对获取到的数
     * 据流直接操作，如 stream_copy_to_stream() 。
     *
     * 如果在调用此方法之前调用了 `moveTo()` 方法，此方法 **必须** 抛出异常。
     *
     * @return StreamInterface 上传文件的数据流
     * @throws \RuntimeException 没有数据流的情形下。
     * @throws \RuntimeException 无法创建数据流。
     */
    public function getStream();

    /**
     * 把上传的文件移动到新目录。
     *
     * 此方法保证能同时在 `SAPI` 和 `non-SAPI` 环境下使用。实现类库 **必须** 判断
     * 当前处在什么环境下，并且使用合适的方法来处理，如 move_uploaded_file(), rename()
     * 或者数据流操作。
     *
     * $targetPath 可以是相对路径，也可以是绝对路径，使用 rename() 解析起来应该是一样的。
     *
     * 当这一次完成后，原来的文件 **必须** 会被移除。
     * 
     * 如果此方法被调用多次，一次以后的其他调用，都要抛出异常。
     *
     * 如果在 SAPI 环境下的话，$_FILES 内有值，当使用  moveTo(), is_uploaded_file()
     * 和 move_uploaded_file() 方法来移动文件时 **应该** 确保权限和上传状态的准确性。
     * 
     * 如果你希望操作数据流的话，请使用 `getStream()` 方法，因为在 SAPI 场景下，无法
     * 保证书写入数据流目标。
     * 
     * @see http://php.net/is_uploaded_file
     * @see http://php.net/move_uploaded_file
     * @param string $targetPath 目标文件路径。
     * @throws \InvalidArgumentException 参数有问题时抛出异常。
     * @throws \RuntimeException 发生任何错误，都抛出此异常。
     * @throws \RuntimeException 多次运行，也抛出此异常。
     */
    public function moveTo($targetPath);

    /**
     * 获取文件大小。
     *
     * 实现类库 **应该** 优先使用 $_FILES 里的 `size` 数值。
     * 
     * @return int|null 以 bytes 为单位，或者 null 未知的情况下。
     */
    public function getSize();

    /**
     * 获取上传文件时出现的错误。
     *
     * 返回值 **必须** 是 PHP 的 UPLOAD_ERR_XXX 常量。
     *
     * 如果文件上传成功，此方法 **必须** 返回 UPLOAD_ERR_OK。
     *
     * 实现类库 **必须** 返回 $_FILES 数组中的 `error` 值。
     * 
     * @see http://php.net/manual/en/features.file-upload.errors.php
     * @return int PHP 的 UPLOAD_ERR_XXX 常量。
     */
    public function getError();

    /**
     * 获取客户端上传的文件的名称。
     * 
     * 永远不要信任此方法返回的数据，客户端有可能发送了一个恶意的文件名来攻击你的程序。
     * 
     * 实现类库 **应该** 返回存储在 $_FILES 数组中 `name` 的值。
     *
     * @return string|null 用户上传的名字，或者 null 如果没有此值。
     */
    public function getClientFilename();

    /**
     * 客户端提交的文件类型。
     * 
     * 永远不要信任此方法返回的数据，客户端有可能发送了一个恶意的文件类型名称来攻击你的程序。
     *
     * 实现类库 **应该** 返回存储在 $_FILES 数组中 `type` 的值。
     *
     * @return string|null 用户上传的类型，或者 null 如果没有此值。
     */
    public function getClientMediaType();
}
```