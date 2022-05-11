---
title: PSR-12 编码规范扩充
date: 2021-10-10T15:21:35+08:00
Description:
Tags: 
 - PHP
Categories:
 - PHP PSR标准规范
DisableComments: false
---

本文中的 `必须`，`不得`，`需要`，`应`，`不应`，`应该`，`不应该`，`推荐`，`可以` 和 `可选` 等能愿动词按照 [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。

## 概览

此规范起到继承，扩展和替换 [PSR-2] 的作用， 同时编码风格遵守 [PSR-1] 这个基础编码标准。

与 [PSR-2] 一样， 此规范的目的是减少不同人在阅读代码时认知冲突。 它通过列举一套如何格式化 `PHP` 代码的公共的规则和期望来实现这个目标。 `PSR` 力图提供一套方法，编码风格工具可以利用，项目可以遵守，开发人员可以方便的在不同的项目中使用。当各个的开发人员在进行多项目合作的时候，它可以帮助在这些项目中提供一套通用的指导。所以，本指南的价值不是规则本身，而是这些规则的共享。
[PSR-2] 在 2012 年被接受，随后 `PHP` 经历了很多变化，影响了编码风格。同时 [PSR-2] 是 `PHP` 编码时候的基础功能，被广泛的采用。因此，`PSR` 力图通过一种更加现代的方式说明 [PSR-2] 的内容和新功能，并对 [PSR-2] 进行更正。

[PSR-1]: /archives/psr-1/
[PSR-2]: /archives/psr-2/

### 以前的语言版本

在整个文档中，任何说明都可以被忽略，如果它们不存在于你项目所支持的 `PHP` 版本中。

### 例如

此示例包含以下一些规则作为快速概述：
```php
<?php

declare(strict_types=1);

namespace Vendor\Package;

use Vendor\Package\{ClassA as A, ClassB, ClassC as C};
use Vendor\Package\SomeNamespace\ClassD as D;

use function Vendor\Package\{functionA, functionB, functionC};

use const Vendor\Package\{ConstantA, ConstantB, ConstantC};

class Foo extends Bar implements FooInterface
{
    public function sampleFunction(int $a, int $b = null): array
    {
        if ($a === $b) {
            bar();
        } elseif ($a > $b) {
            $foo->bar($arg1);
        } else {
            BazClass::bar($arg2, $arg3);
        }
    }

    final public static function bar()
    {
        // 方法内容
    }
}
```
## 总则

### 基本编码标准

代码必须遵循 [PSR-1] 中列出的所有规则。
[PSR-1] 中的术语 `StudlyCaps` 必须解释为 `PascalCase` （帕斯卡命名法：大驼峰式命名法），其中每个单词的第一个字母大写，包括第一个字母。

[PSR-1]: /archives/psr-1/


### 文件

- 所有 `PHP` 文件只能使用 `Unix LF (换行符)` 结尾。
- 所有的 `PHP` 文件都必须以非空行结尾，以一个 `LF` 结尾。
- 在仅包含 `PHP` 代码的文件中，必须省略结尾的 `?>` 标记。

### 代码行

- 行长度不得有硬限制。
- 行长度的软限制必须为 `120` 个字符。
- 行的长度不应超过 `80` 个字符；超过该长度的行应拆分为多个后续行，每个行的长度不应超过 `80` 个字符。
- 行尾不能有尾随空格。
- 可以添加空行以提高可读性并指示相关的代码块，除非明确禁止。
- 每行不能有多个语句。

### 缩进

代码必须为每个缩进级别使用 `4` 个空格的缩进，并且不能使用缩进标签。

### 关键词和类型

- `PHP` 的所有关键字和类型 [1](https://www.php.net/manual/en/reserved.keywords.php)[2](https://www.php.net/manual/en/reserved.other-reserved-words.php) 都必须使用小写。
- `PHP` 未来版本中新加的所有关键字和类型也都必须使用小写。
类型关键字必须使用缩写。使用 `bool` 而不是 `boolean`，使用 `int` 而不是 `integer` 等等。

## 声明、命名空间以及导入

一个 `PHP` 文件的头部可能会包含多个块。如果包含多个块，则每个块都必须用空白行和其他块分隔，并且块内不能包含空白行。所有的块都必须按照下面的顺序排列，如果不存在该块则忽略。

- 开始标签： <?php。
- 文件级文档块。
- 一个或多个声明语句。
- 命名空间声明语句。
- 一个或多个基于类的 use 声明语句。
- 一个或多个基于方法的 use 声明语句。
- 一个或多个基于常量的 use 声明语句。
- 其余代码。

当文件包含 `HTML` 和 `PHP` 的混合代码时，可以使用上面列出的任何部分。如果是这种情况的话，即时代码的其他部分包含有 `PHP` 结束符，然后再包含 `HTML` 和 `PHP` 代码，声明、命名空间和导入语句块也必须放在文件的顶部。

什么时候开始 `<?php` 标签位于文件的第一行，它必须位于自己的行，没有其他语句，除非它是一个包含 `PHP` 之外的标记的文件打开和关闭标记。

`import` 语句不能以前导反斜杠开头，因为它们必须始终完全合格。
以下示例演示了所有块的完整列表：
```php
<?php

/**
 * This file contains an example of coding styles.
 */

declare(strict_types=1);

namespace Vendor\Package;

use Vendor\Package\{ClassA as A, ClassB, ClassC as C};
use Vendor\Package\SomeNamespace\ClassD as D;
use Vendor\Package\AnotherNamespace\ClassE as E;

use function Vendor\Package\{functionA, functionB, functionC};
use function Another\Vendor\functionD;

use const Vendor\Package\{CONSTANT_A, CONSTANT_B, CONSTANT_C};
use const Another\Vendor\CONSTANT_D;

/**
 * FooBar is an example class.
 */
class FooBar
{
    // ... additional PHP code ...
}
```

**不得**使用深度超过两层的复合名称空间，因此以下展示了允许的最大复合深度。
```php
<?php

use Vendor\Package\SomeNamespace\{
    SubnamespaceOne\ClassA,
    SubnamespaceOne\ClassB,
    SubnamespaceTwo\ClassY,
    ClassZ,
};
```
并且不允许以下内容:
```php
<?php

use Vendor\Package\SomeNamespace\{
    SubnamespaceOne\AnotherNamespace\ClassA,
    SubnamespaceOne\ClassB,
    ClassZ,
};
```

当希望在 `PHP` 外部包含标记的文件中声明严格类型时打开和关闭标签，声明必须写在文件的第一行并且包含在一个开始的 `PHP` 标签，以及严格的类型声明和结束标签。
例如:
```php
<?php declare(strict_types=1) ?>
<html>
<body>
    <?php
        // ... additional PHP code  ...
    ?>
</body>
</html>
```

声明语句不能包含空格，并且必须完全是 `declare(strict_types=1)` (带有可选的分号终止符)。
允许使用块声明语句，并且必须按照以下的格式设置。注意的位置括号和间距：
```php
declare(ticks=1) {
    // 一些代码
}
```

## 类，属性，和方法

这里的 `类` 指的是所有`类`，`接口`，以及 `trait` 。

任何注释和语句 **不得** 跟在其右花括号后的同一行。

当实例化一个类时，后面的圆括号 必须 写出来，即使没有参数传进其构造函数。
```php
new Foo();
```
### 继承和实现

- 关键字 `extends` 和 `implments` **必须** 在类名的同一行声明。
- 类的左花括号 **必须** 另起一行；右花括号 **必须** 跟在类主体的下一行。
- 类的左花括号 **必须** 独自成行，且 **不得** 在其上一行或下一行存在空行。
- 右花括号 **必须** 独自成行，且 不得 在其上一行存在空行。

```php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements \ArrayAccess, \Countable
{
    // 常量，属性，方法
}
```
如果有接口， `implements` 接口和 `extends`父类  **可以** 分为多行，前者每行需缩进一次。当这么做时，第一个接口 **必须** 写在下一行，且每行 **必须** 只能写一个接口。
```php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements
    \ArrayAccess,
    \Countable,
    \Serializable
{
    // 常量，属性，方法
}
```
### 使用 trait

在类里面用于实现 `trait` 的关键字 `use` **必须** 在左花括号的下一行声明。
```php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;

class ClassName
{
    use FirstTrait;
}
```
每个导入类的 **trait** **必须** 每行一个包含声明，且每个包含声明 **必须** 有其 `use` 导入语句。

```php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;
use Vendor\Package\SecondTrait;
use Vendor\Package\ThirdTrait;

class ClassName
{
    use FirstTrait;
    use SecondTrait;
    use ThirdTrait;
}
```
在类文件中，如果在使用 `use Trait` 之后没有其他内容了 ，类名右大括号必须另起一行。

```php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;

class ClassName
{
    use FirstTrait;
}
```
如有其他内容，两者之间需空一行。

```php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;

class ClassName
{
    use FirstTrait;

    private $property;
}
```
当使用 `insteadof` 和 `as` 运算符时，它们必须如图所示使用，注意缩进、间距和另起一行。
```php
<?php

class Talker
{
    use A, B, C {
        B::smallTalk insteadof A;
        A::bigTalk insteadof C;
        C::mediumTalk as FooBar;
    }
}
```
### 属性和常量

- 所有属性 **必须** 声明可见性。
- 如果你的项目 `PHP` 最小版本支持常量可见性（ PHP 7.1 或以上），所有常量 **必须** 声明可见性。
- 关键字 `var` **不得** 用于声明属性。
- 每条声明语句 **不得** 声明多于一个属性。
- 属性名 **不得** 用单个下划线开头表明其受保护的或私有的可见性。也就是说，一个下划线开头显然是没有意义的。
- 类型声明和属性名之间 **必须** 有一个空格。

一个属性声明看上去如下所示：
```php
<?php

namespace Vendor\Package;

class ClassName
{
    public $foo = null;
    public static int $bar = 0;
}
```
### 方法和函数

- 所有的方法 **必须** 事先声明类型。
- 方法命名 **不得**  用单个下划线来区分是 `protected` 或 `private` 类型。也就是说，不要用一个没有意义的下划线开头。
- 方法和函数名称中，方法命名后面 **不得** 使用空格。方法开始的花括号 **必须** 写在方法声明后自成一行， 结束花括号也 **必须** 写在方法后面自成一行。开始左括号后和结束右括号前，都 **不得** 有空格符。

一个方法的声明应该如下所示。注意括号，逗号，空格和花括号的位置：
```php
<?php

namespace Vendor\Package;

class ClassName
{
    public function fooBarBaz($arg1, &$arg2, $arg3 = [])
    {
        // 方法主体
    }
}
```

一个函数的声明应该如下所示。注意括号，逗号，空格和花括号的位置：

```php
<?php

function fooBarBaz($arg1, &$arg2, $arg3 = [])
{
    // 函数主体
}
```
#### 方法和函数参数

在参数列表中， **不得** 在每个逗号前存在空格，且 **必须** 在每个逗号后有一个空格。
方法和函数中带有默认值的参数 **必须** 放在参数列表的最后。

```php
<?php

namespace Vendor\Package;

class ClassName
{
    public function foo(int $arg1, &$arg2, $arg3 = [])
    {
        // 方法主体
    }
}
```
参数列表 **可以** 分为多行，每行参数缩进一次。当这么做时，第一个参数 **必须** 放在下一行，且每行 **必须** 只能有一个参数。

当参数列表分成多行时，右圆括号和左花括号 **必须** 放在同一行且单独成行，两者之间存在一个空格。
```php
<?php

namespace Vendor\Package;

class ClassName
{
    public function aVeryLongMethodName(
        ClassTypeHint $arg1,
        &$arg2,
        array $arg3 = []
    ) {
        // 方法主体
    }
}
```
当你定义一个返回值类型声明时，冒号后面的类型声明 **必须** 用空格符隔开。冒号和声明 **必须** 在同一行，且跟参数列表后的结束括号之间没有空格。
```php
<?php

declare(strict_types=1);

namespace Vendor\Package;

class ReturnTypeVariations
{
    public function functionName(int $arg1, $arg2): string
    {
        return 'foo';
    }

    public function anotherFunction(
        string $foo,
        string $bar,
        int $baz
    ): string {
        return 'foo';
    }
}
```

在可空类型声明中，问号和类型声明之间不能有空格。
```php
<?php

declare(strict_types=1);

namespace Vendor\Package;

class ReturnTypeVariations
{
    public function functionName(?string $arg1, ?int &$arg2): ?string
    {
        return 'foo';
    }
}
```
当在参数之前使用引用运算符 `&` 时，引用运算符之后不能有空格，例如上面的示例。

可变参数声明的三个点和参数名称之间不能有空格：
```php
public function process(string $algorithm, ...$parts)
{
    // 函数体
}
```

当同时使用引用运算符和可变参数运算符时，它们之间不能有任何空格：

```php
public function process(string $algorithm, &...$parts)
{
    // 函数体
}
```

### `abstract`, `final`, `static`

如果是  `abstract`, `final` ，那么申明的时候必须是可见性声明。
如果是  `static` ，声明必须位于可见性声明之后。
```php
<?php

namespace Vendor\Package;

abstract class ClassName
{
    protected static $foo;

    abstract protected function zim();

    final public static function bar()
    {
        // 请求体
    }
}
```
### 方法和函数的调用

当我们在进行方法或者函数调用的时候，方法名或函数名与左括号之间不能出现空格，在右括号之后也不能出现空格，并且在右括号之前也不能有空格。在参数列表中，每个逗号前面不能有空格，每个逗号后面必须有一个空格。
```php
<?php

bar();
$foo->bar($arg1);
Foo::bar($arg2, $arg3);
```

参数列表可以分为多行，每行后面缩进一次。这样做时，列表中的第一项必须位于下一行，并且每一行必须只有一个参数。跨多个行拆分单个参数 (就像匿名函数或者数组那样) 并不构成拆分参数列表本身。

```php
<?php

$foo->bar(
    $longArgument,
    $longerArgument,
    $muchLongerArgument
);
```

```php
<?php

somefunction($foo, $bar, [
  // ...
], $baz);

$app->get('/hello/{name}', function ($name) use ($app) {
    return 'Hello ' . $app->escape($name);
});
```
## 流程控制

如下是主要的流程控制风格规则：

- 流程控制关键词之后 **必须** 要有一个空格
- 左括号后面 **不能** 有空格
- 右括号前面 **不能** 有空格
- 右括号与左大括号之间 **必须** 要有一个空格
- 流程主体 **必须** 要缩进一次
- 流程主体 **必须** 在左大括号之后另起一行
- 右大括号 **必须** 在流程主体之后另起一行

每个流程控制主体 **必须** 以封闭的括号结束。这将标准化流程结构，同时减少由于流程中添加新的内容而引入错误的可能性。

### if, elseif, else

`if` 结构如下。注意括号，空格，和大括号的位置；`else` 和 `elseif` 都在同一行，和右大括号一样在主体的前面。
```php
<?php

if ($expr1) {
    // if body
} elseif ($expr2) {
    // elseif body
} else {
    // else body;
}
```
**应该** 使用关键词 `elseif` 替换 `else if`，这样所有的控制关键词看起来都像单个词。

括号中的表达式 **可能** 会被分开为多行，每一行至少缩进一次。如果这样做，第一个条件 **必须** 在新的一行。右括号和左大括号 **必须** 在同一行，而且中间有一个空格。条件中间的布尔控制符 **必须** 在每一行的开头或者结尾，而不是混在一起。

```php
<?php

if (
    $expr1
    && $expr2
) {
    // if body
} elseif (
    $expr3
    && $expr4
) {
    // elseif body
}
```
### switch, case

`switch` 结构如下。注意括号，空格和大括号的位置。**case** **必须** 缩进一次从 `switch` 开始， `break` 关键词 (或者其他终止关键词) **必须** 缩进和 `case` 主体保持一致。必须 要有一个像 `// no break` 这样的注释在不为空且不需要中断的 **case** 主体之中。

```php
<?php

switch ($expr) {
    case 0:
        echo 'First case, with a break';
        break;
    case 1:
        echo 'Second case, which falls through';
        // no break
    case 2:
    case 3:
    case 4:
        echo 'Third case, return instead of break';
        return;
    default:
        echo 'Default case';
        break;
}
```
括号中的表达式 **可能** 会被分开多行，每一行至少要缩进一次。如果这样做，第一个条件 **必须** 在新的一行。右括号和左大括号 **必须** 在同一行，而且中间有一个空格。条件中间的布尔控制符 **必须** 在一行的开头或者结尾，而不是混在一起。

```php
<?php

switch (
    $expr1
    && $expr2
) {
    // structure body
}
```
### while, do while

`while` 结构如下。注意括号，空格和大括号的位置。

```php
<?php

while ($expr) {
    // structure body
}
```
括号中的表达式 **可能** 会被分开多行，每一行至少要缩进一次。如果这样做，第一个条件 **必须** 在新的一行。右括号和左大括号 **必须** 在同一行，而且中间有一个空格。条件中间的布尔控制符 **必须** 在每一行的开头或者结尾，而不是混在一起。
```php
<?php

while (
    $expr1
    && $expr2
) {
    // structure body
}
```

同样的， `do while` 申明如下。注意括号，空格和大括号的位置。
```php
<?php

do {
    // structure body;
} while ($expr);
```
括号中的表达式 **可能** 会被分开多行，每一行至少要缩进一次。如果这样做，第一个条件 **必须** 在新的一行。条件中间的布尔控制符 **必须** 在每一行的开头或者结尾，而不是混在一起。

```php
<?php

do {
    // structure body;
} while (
    $expr1
    && $expr2
);
```
### for

`for` 申明如下。注意括号，空格和大括号的位置。
```php
<?php

for ($i = 0; $i < 10; $i++) {
    // for body
}
```
括号中的表达式 **可能** 会被分开多行，每一行至少要缩进一次。如果这样做，第一个条件 **必须** 在新的一行。右括号和左大括号 **必须** 在同一行，而且中间有一个空格。

```php
<?php

for (
    $i = 0;
    $i < 10;
    $i++
) {
    // for body
}
```
### foreach

`foreach` 语句的写法如下所示。请注意它的圆括号、空格和花括号。

```php
<?php

foreach ($iterable as $key => $value) {
    // 迭代主体
}
```
### try, catch, finally

一个 `try-catch-finally` 模块包含下面这些内容。请注意它的圆括号、空格和花括号。
```php
<?php

try {
    // try 主体
} catch (FirstThrowableType $e) {
    // 捕获异常主体
} catch (OtherThrowableType | AnotherThrowableType $e) {
    // 捕获异常主体
} finally {
    // finally 主体
}
```
## 运算符

- 运算符的样式规则按元数分组（其接受的操作数个数）。
- 当运算符周围允许出现空格时， **可以** 出于可读性目的打多个空格。
- 所有这里没描述的运算符暂不作限定。

### 一元运算符

`递增` / `递减`运算符和操作数之间 **不得** 有任何空格。
```php
$i++;
++$j;
```
类型转换运算符的圆括号内部 **不得** 有任何空格：

```php
$intValue = (int) $input;
```
### 二元运算符

所有二进制 [算术](http://php.net/manual/en/language.operators.arithmetic.php)，[比较](http://php.net/manual/en/language.operators.comparison.php)，[赋值](http://php.net/manual/en/language.operators.assignment.php)，[按位](http://php.net/manual/en/language.operators.bitwise.php)，[逻辑](http://php.net/manual/en/language.operators.logical.php)、[字符串](http://php.net/manual/en/language.operators.string.php)和[类型](http://php.net/manual/en/language.operators.type.php)运算符必须在前后跟至少一个空格：
```php
if ($a === $b) {
    $foo = $bar ?? $a ?? $b;
} elseif ($a > $b) {
    $foo = $a + $b * $c;
}
```

### 三元运算符


条件运算符，也称为三元运算符，必须在 `?` 和 `:` 这两个字符之间：
```php
$variable = $foo ? 'foo' : 'bar';
```
如果省略条件运算符的中间操作数，运算符必须遵循与其他二进制比较运算符相同的样式规则：
```php
$variable = $foo ?: 'bar';
```
## 闭包（Closures）

- 闭包声明时 **必须** 在 `function` 关键字后留有 `1` 个空格，并且在 `use` 关键字前后各留有 `1` 个空格。
- 左花括号 **必须** 跟随前文写在同一行，右花括号必须在函数体后换行放置。
- **不能**在参数和变量的左括号后和右括号前放置空格。
- **不能**在参数和变量的逗号前放置空格，但必须在逗号后放置 1 个空格。
- 闭包参数如果有默认值，该参数必须放在参数列表末尾。
- 如果声明了返回类型，它必须遵循普通函数和方法相同的规则；如果使用 `use` 关键字，冒号必须在 `use` 右括号后且冒号前后不能有空格。

闭包的声明方式如下，留意括号，逗号，空格和花括号：
```php
<?php

$closureWithArgs = function ($arg1, $arg2) {
    // 函数体
};

$closureWithArgsAndVars = function ($arg1, $arg2) use ($var1, $var2) {
    // 函数体
};

$closureWithArgsVarsAndReturn = function ($arg1, $arg2) use ($var1, $var2): bool {
    // 函数体
};
```

参数和变量可以分多行放置，每个后续行缩进一次。执行此操作时，列表中的第一项 **必须** 放在下一行，并且每行只能有一个参数或变量。

结束多行列表（或者参数，变量）的时候，右括号和左大括号 **必须** 要放在一行，而且中间有一个空格。

下面是有和没有多行参数列表与变量列表的闭包示例。
```php
<?php

$longArgs_noVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) {
   // body
};

$noArgs_longVars = function () use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
   // body
};

$longArgs_longVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
   // body
};

$longArgs_shortVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) use ($var1) {
   // body
};

$shortArgs_longVars = function ($arg) use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
   // body
};
```

注意格式化规则也适用一个闭包在一个方法或者操作中作为参数被直接引用。
```php
<?php

$foo->bar(
    $arg1,
    function ($arg2) use ($var1) {
        // body
    },
    $arg3
);
```

## 匿名类

匿名类 **必须** 遵循上面章节中和闭包一样的方针和准则。

```php
<?php

$instance = new class {};
```
只要 `implements` 接口列表不换行，左花括号 **可以** 和关键字 **class** 在同一行。如果接口列表换行，花括号 **必须** 放在最后一个接口的下一行。

```php
<?php

// 花括号在同一行
$instance = new class extends \Foo implements \HandleableInterface {
    // 类内容
};

// 花括号在下一行
$instance = new class extends \Foo implements
    \ArrayAccess,
    \Countable,
    \Serializable
{
    // 类内容
};
```
