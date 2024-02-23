# *第十一章*：将现有 PHP 应用迁移到 PHP 8

在整本书中，您已经被警告可能出现代码断裂的情况。不幸的是，目前没有真正好的工具可以扫描您现有的代码并检查潜在的代码断裂。在本章中，我们将带您了解一组类的开发过程，这些类构成了 PHP 8 **向后兼容**（**BC**）断裂扫描器的基础。此外，您还将学习将现有客户 PHP 应用迁移到 PHP 8 的推荐流程。

阅读本章并仔细研究示例后，您将更好地掌握 PHP 8 迁移。了解整体迁移过程后，您将更加自信，并能够以最少的问题执行 PHP 8 迁移。

本章涵盖的主题包括以下内容：

+   了解开发、暂存和生产环境

+   学习如何在迁移之前发现 BC（向后兼容）断裂

+   执行迁移

+   测试和故障排除迁移

# 技术要求

为了检查和运行本章提供的代码示例，最低推荐的硬件配置如下：

+   基于 x86_64 的台式 PC 或笔记本电脑

+   1 **千兆字节**（**GB**）的可用磁盘空间

+   4GB 的 RAM

+   500 **千比特每秒**（**Kbps**）或更快的互联网连接

此外，您还需要安装以下软件：

+   Docker

+   Docker Compose

有关 Docker 和 Docker Compose 安装的更多信息，以及如何构建用于演示本书中解释的代码的 Docker 容器，请参阅*第一章*的*技术要求*部分，介绍新的 PHP 8 面向对象编程特性。在本书中，我们将您为本书恢复示例代码的目录称为`/repo`。

本章的源代码位于[`github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices`](https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices)。我们现在可以开始讨论使用作为整体迁移过程的一部分的环境。

# 了解开发、暂存和生产环境

网站更新的最终目标是以尽可能无缝的方式将更新的应用程序代码从开发环境移动到生产环境。这种应用程序代码的移动被称为**部署**。在这种情况下，移动涉及将应用程序代码和配置文件从一个**环境**复制到另一个环境。

在我们深入讨论将应用程序迁移到 PHP 8 之前，让我们先看看这些环境是什么。了解不同环境可能采用的形式对于您作为开发人员的角色至关重要。有了这种理解，您就能更好地将代码部署到生产环境，减少错误的发生。

## 定义环境

我们使用*环境*一词来描述包括操作系统、Web 服务器、数据库服务器和 PHP 安装在内的软件堆栈的组合。过去，环境等同于*服务器*。然而，在现代，*服务器*这个术语是具有误导性的，因为它暗示着一个金属箱子中的物理计算机，放置在某个看不见的服务器房间的机架上。如今，鉴于云服务提供商和高性能的虚拟化技术（例如 Docker）的丰富，这更有可能不是真实情况。因此，当我们使用*环境*这个术语时，请理解它指的是物理或虚拟服务器。

环境通常分为三个不同的类别：**开发**、**暂存**和**生产**。一些组织还提供单独的**测试**环境。让我们先看看所有环境中的共同点。

### 常见组件

重要的是要注意，所有环境中的内容都受生产环境的驱动。生产环境是应用程序代码的最终目的地。因此，所有其他环境应尽可能与操作系统、数据库、Web 服务器和 PHP 安装匹配。因此，例如，如果生产环境启用了 PHP OPCache 扩展，所有其他环境也必须启用此扩展。

所有环境，包括生产环境，至少需要安装操作系统和 PHP。根据应用程序的需求，安装 Web 服务器和数据库服务器也是非常常见的。Web 和数据库服务器的类型和版本应尽可能与生产环境匹配。

一般来说，开发环境与生产环境越接近，部署后出现错误的几率就越小。

现在我们来看看开发环境需要什么。

### 开发环境

开发环境是您最初开发和测试代码的地方。它具有应用程序维护和开发所需的工具。这包括存储源代码的存储库（例如 Git），以及启动、停止和重置环境所需的各种脚本。

通常，开发环境会有触发自动部署过程的脚本。这些脚本可以取代**提交钩子**，设计用于在提交到源代码存储库时激活。其中一个例子是**Git Hooks**，即可放置在`.git/hooks`目录中的脚本文件。

提示

有关 Git Hooks 的更多信息，请查看此处的文档：[`git-scm.com/book/en/v2/Customizing-Git-Git-Hooks`](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)。

传统的开发环境包括个人计算机、数据库服务器、Web 服务器和 PHP。这种传统范式未能考虑到目标生产环境可能存在的变化。例如，如果您经常与 12 个客户合作，那么这 12 个客户几乎不可能拥有完全相同的操作系统、数据库服务器、Web 服务器和 PHP 版本！*最佳实践*是尽可能模拟生产环境，可以采用虚拟机或 Docker 容器的形式。

因此，代码编辑器或**IDE**（集成开发环境）并不位于开发环境内。相反，您在开发环境之外进行代码创建和编辑。然后，您可以通过直接将文件复制到虚拟开发环境的共享目录，或者通过提交更改到源代码存储库，然后从开发环境虚拟机内拉取更改来本地推送更改。

在开发环境进行单元测试也是合适的。开发单元测试不仅可以更好地保证您的代码在生产环境中运行，而且还是发现应用程序开发早期阶段的错误的好方法。当然，您需要在本地环境中尽可能多地进行调试！在开发中捕获和修复错误通常只需要在生产中发现错误所需的十分之一的时间！

现在让我们来看看暂存环境。

### 暂存环境

大型应用程序开发项目通常会有多个开发人员共同在同一代码库上工作。在这种情况下，使用版本控制存储库至关重要。*暂存*环境是所有开发人员在开发环境测试和调试阶段完成后上传其代码的地方。

暂存环境必须是生产环境的*精确副本*。你可以把暂存环境想象成汽车工厂装配线上的最后一步。这是所有来自一个或多个开发环境的各种部件安装到位的地方。暂存环境是生产应该出现的原型。

重要的是要注意，暂存服务器通常可以直接访问互联网；然而，它通常位于一个需要密码才能访问的安全区域。

最后，让我们来看看生产环境。

### 生产环境

生产环境通常由客户直接维护和托管。这个环境也被称为**现场环境**。打个比方，如果开发环境是练习，暂存环境是彩排，那么生产环境就是现场演出（也许没有歌唱和舞蹈！）。

生产环境可以直接访问互联网，但受到防火墙的保护，通常还受到入侵检测和防御系统的保护（例如，[`snort.org/`](https://snort.org/)）。此外，生产环境可能被隐藏在运行在面向互联网的 Web 服务器上的反向代理配置后面。否则，至少在理论上，生产环境应该是暂存环境的*精确克隆*。

现在你已经对应用程序代码从开发到生产的环境有了一个概念，让我们来看看 PHP 8 迁移的一个关键第一步：发现潜在的 BC 代码中断。

# 学习如何在迁移前发现 BC 中断

理想情况下，你应该带着一个行动计划进入 PHP 8 迁移。这个行动计划的关键部分包括了解当前代码库中存在多少潜在的 BC 中断。在本节中，我们将向您展示如何开发一个自动化查找潜在 BC 中断的 BC 中断嗅探器。

首先，让我们回顾一下到目前为止关于 PHP 8 中可能出现的 BC 问题学到的东西。

## 获取 BC 中断概述

您已经知道，通过阅读本书的前几章，潜在的代码中断源自几个方面。让我们简要总结一下可能导致迁移后代码失败的一般趋势。请注意，我们在本章中不涵盖这些主题，因为这些主题在本书的早期章节中都已经涵盖过了：

+   资源到对象的迁移

+   支持 OS 库的最低版本

+   `Iterator`到`IteratorAggregate`的迁移

+   已删除的函数

+   使用变化

+   魔术方法签名强制执行

许多变化可以通过添加基于`preg_match()`或`strpos()`的简单回调来检测。使用变化更难以检测，因为乍一看，自动断点扫描器无法在不广泛使用`eval()`的情况下检测使用结果。

现在让我们来看看一个中断扫描配置文件可能是什么样子。

## 创建一个 BC 中断扫描配置文件

配置文件允许我们独立于 BC 中断扫描器类开发一组搜索模式。使用这种方法，BC 中断扫描器类定义了用于进行搜索的实际逻辑，而配置文件提供了一系列特定条件以及警告和建议的补救措施。

通过简单查找已在 PHP 8 中删除的函数的存在来检测到许多潜在的代码中断。为此，简单的`strpos()`搜索就足够了。另一方面，更复杂的搜索可能需要我们开发一系列回调。让我们首先看看如何基于简单的`strpos()`搜索开发配置。

### 定义一个简单的 strpos()搜索配置

在简单的`strpos()`搜索的情况下，我们只需要提供一个键/值对数组，其中键是被移除的函数的名称，值是它的建议替代品。BC 破坏扫描器类中的搜索逻辑可以这样做：

```php
$contents = file_get_contents(FILE_TO_SEARCH);
foreach ($config['removed'] as $key => $value)
    if (str_pos($contents, $key) !== FALSE)  echo $value;
```

我们将在下一节中介绍完整的 BC 破坏扫描器类实现。现在，我们只关注配置文件。以下是前几个`strpos()`搜索条目可能出现的方式：

```php
// /repo/ch11/bc_break_scanner.config.php
use Migration\BreakScan;
return [
    // not all keys are shown
    BreakScan::KEY_REMOVED => [
        '__autoload' => 'spl_autoload_register(callable)',
        'each' => 'Use "foreach()" or ArrayIterator',
        'fgetss' => 'strip_tags(fgets($fh))',
        'png2wbmp' => 'imagebmp',
        // not all entries are shown
    ],
];
```

不幸的是，一些 PHP 8 向后不兼容性可能超出了简单的`strpos()`搜索的能力。我们现在将注意力转向检测由 PHP 8 资源到对象迁移引起的潜在破坏。

### 检测与`is_resource()`相关的 BC 破坏

在*第七章*，*在使用 PHP 8 扩展时避免陷阱*，在*PHP 8 扩展资源到对象迁移*部分，您了解到 PHP 中存在一种从资源到对象的普遍趋势。您可能还记得，这种趋势本身并不构成任何 BC 破坏的威胁。然而，如果您的代码在确认连接已建立时使用了`is_resource()`，就有可能发生 BC 破坏。

为了考虑这种 BC 破坏的潜在性，我们的 BC 破坏扫描配置文件需要列出以前产生资源但现在产生对象的任何函数。然后我们需要在 BC 破坏扫描类中添加一个使用此列表的方法（下面讨论）。

这是受影响函数潜在配置键可能出现的方式：

```php
// /repo/ch11/bc_break_scanner.config.php
return [    // not all keys are shown
    BreakScan::KEY_RESOURCE => [
        'curl_init',
        'xml_parser_create',
        // not all entries are shown
    ],
];
```

在破坏扫描类中，我们只需要首先确认是否调用了`is_resource()`，然后检查`BreakScan::KEY_RESOURCE`数组下列出的任何函数是否存在。

我们现在将注意力转向**魔术方法签名**违规。

### 检测魔术方法签名违规

PHP 8 严格执行魔术方法签名。如果您的类使用宽松的定义，即不对方法签名进行数据类型定义，并且对于魔术方法不定义返回值数据类型，那么您就不会受到潜在代码破坏的威胁。另一方面，如果您的魔术方法签名包含数据类型，并且这些数据类型与 PHP 8 中强制执行的严格定义集不匹配，那么您就有可能出现代码破坏！

因此，我们需要创建一组正则表达式，以便检测魔术方法签名违规。此外，我们的配置应包括正确的签名。通过这种方式，如果检测到违规，我们可以在生成的消息中呈现正确的签名，加快更新过程。

这是一个魔术方法签名配置可能出现的方式：

```php
// /repo/ch11/bc_break_scanner.config.php
use Php8\Migration\BreakScan;
return [    
    BreakScan::KEY_MAGIC => [
    '__call' => [ 'signature' => 
        '__call(string $name, array $arguments): mixed',
        'regex' => '/__call\s*\((string\s)?'
            . '\$.+?(array\s)?\$.+?\)(\s*:\s*mixed)?/',
        'types' => ['string', 'array', 'mixed']],
    // other configuration keys not shown
    '__wakeup' => ['signature' => '__wakeup(): void',
        'regex' => '/__wakeup\s*\(\)(\s*:\s*void)?/',
        'types' => ['void']],
    ]
    // other configuration keys not shown
];
```

您可能注意到我们包含了一个额外的选项`types`。这是为了自动生成一个正则表达式。负责此操作的代码没有显示。如果您感兴趣，可以查看`/path/to/repo/ch11/php7_build_magic_signature_regex.php`。

让我们看看在简单的`strpos()`搜索不足以满足的情况下，您可能如何处理复杂的破坏检测。

### 解决复杂的 BC 破坏检测

在简单的`strpos()`搜索不足以证明的情况下，我们可以开发另一组键/值对，其中值是一个回调函数。举个例子，考虑一个可能的 BC 破坏，一个类定义了一个`__destruct()`方法，但也在`__construct()`方法中使用了`die()`或`exit()`。在 PHP 8 中，可能在这些情况下`__destruct()`方法不会被调用。

在这种情况下，简单的`strpos()`搜索是不够的。相反，我们必须开发逻辑来执行以下操作：

+   检查是否定义了`__destruct()`方法。如果是，则无需继续，因为在 PHP 8 中不会出现破坏的危险。

+   检查是否在`__construct()`方法中使用了`die()`或`exit()`。如果是，则发出潜在 BC 破坏的警告。

在我们的 BC 断点扫描配置数组中，回调采用匿名函数的形式。它接受文件内容作为参数。然后我们将回调分配给数组配置键，并包括如果回调返回`TRUE`时要传递的警告消息：

```php
// /repo/ch11/bc_break_scanner.config.php
return [
    // not all keys are shown
   BreakScan::KEY_CALLBACK => [
    'ERR_CONST_EXIT' => [
      'callback' => function ($contents) {
        $ptn = '/__construct.*?\{.*?(die|exit).*?}/im';
        return (preg_match($ptn, $contents)
                && strpos('__destruct', $contents)); },
      'msg' => 'WARNING: __destruct() might not get '
               . 'called if "die()" or "exit()" used '
               . 'in __construct()'],
    ], // etc.
    // not all entries are shown
];
```

在我们的 BC 断点扫描器类（下面讨论）中，调用回调所需的逻辑可能如下所示：

```php
$contents = file_get_contents(FILE_TO_SEARCH);
$className = 'SOME_CLASS';
foreach ($config['callbacks'] as $key => $value)
    if ($value'callback') echo $value['msg'];
```

如果检测到额外的潜在 BC 断点的要求超出了回调的能力，那么我们将在 BC 断点扫描类中定义一个单独的方法。

正如你所看到的，我们可以开发一个支持不仅简单的`strpos()`搜索，还支持使用回调数组进行更复杂搜索的配置数组。

现在你已经对配置数组中会包含什么有了一个概念，是时候定义执行断点扫描的主要类了。

## 开发 BC 断点扫描类

`BreakScan`类是针对单个文件的。在这个类中，我们定义了利用刚刚覆盖的各种断点扫描配置的方法。如果我们需要扫描多个文件，调用程序会生成一个文件列表，并将它们逐个传递给`BreakScan`。

`BreakScan`类可以分为两个主要部分：定义基础设施的方法和定义如何进行给定扫描的方法。后者主要由配置文件的结构来决定。对于每个配置文件部分，我们将需要一个`BreakScan`类方法。

让我们先看看基础方法。

### 定义 BreakScan 类基础方法

在这一部分，我们看一下`BreakScan`类的初始部分。我们还涵盖了执行基础相关活动的方法：

1.  首先，我们设置类基础设施，将其放在`/repo/src/Php8/Migration`目录中：

```php
    // /repo/src/Php8/Migration/BreakScan.php
    declare(strict_types=1);
    namespace Php8\Migration;
    use InvalidArgumentException;
    use UnexpectedValueException;
    class BreakScan {
    ```

1.  接下来，我们定义一组类常量，用于表示任何给定的后扫描失败的性质的消息：

```php
        const ERR_MAGIC_SIGNATURE = 'WARNING: magic method '
            . 'signature for %s does not appear to match '
            . 'required signature';
        const ERR_NAMESPACE = 'WARNING: namespaces can no '
            . 'longer contain spaces in PHP 8.';
        const ERR_REMOVED = 'WARNING: the following function'
            . 'has been removed: %s.  Use this instead: %s';
        // not all constants are shown
    ```

1.  我们还定义了一组表示配置数组键的常量。我们这样做是为了在配置文件和调用程序中保持键定义的一致性（稍后讨论）：

```php
        const KEY_REMOVED         = 'removed';
        const KEY_CALLBACK        = 'callbacks';
        const KEY_MAGIC           = 'magic';
        const KEY_RESOURCE        = 'resource';
    ```

1.  然后我们初始化关键属性，表示配置，要扫描的文件的内容和任何消息：

```php
        public $config = [];
        public $contents = '';
        public $messages = [];
    ```

1.  `__construct()`方法接受我们的断点扫描配置文件作为参数，并循环遍历所有键以确保它们存在：

```php
        public function __construct(array $config) {
            $this->config = $config;
            $required = [self::KEY_CALLBACK,
                self::KEY_REMOVED,
                self::KEY_MAGIC, 
                self::KEY_RESOURCE];
            foreach ($required as $key) {
                if (!isset($this->config[$key])) {
                    $message = sprintf(
                        self::ERR_MISSING_KEY, $key);
                    throw new Exception($message);
                }
            }
        }
    ```

1.  然后我们定义一个方法，读取要扫描的文件的内容。请注意，我们删除回车（`"\r"`)和换行符（`"\n"`)，以便通过正则表达式更容易处理扫描：

```php
        public function getFileContents(string $fn) {
            if (!file_exists($fn)) {
                self::$className = '';
                $this->contents  = '';
                throw new  Exception(
                    sprintf(self::ERR_FILE_NOT_FOUND, $fn));
            }
            $this->contents = file_get_contents($fn);
            $this->contents = str_replace(["\r","\n"],
                ['', ' '], $this->contents);
            return $this->contents;
        }
    ```

1.  一些回调需要一种方法来提取类名或命名空间。为此，我们定义了静态的`getKeyValue()`方法：

```php
        public static function getKeyValue(
            string $contents, string $key, string $end) {
            $pos = strpos($contents, $key);
            $end = strpos($contents, $end, 
                $pos + strlen($key) + 1);
            return trim(substr($contents, 
                $pos + strlen($key), 
                $end - $pos - strlen($key)));
        }
    ```

这个方法寻找关键字（例如，`class`）。然后找到关键字后面的内容，直到分隔符（例如，`';'）。所以，如果你想要获取类名，你可以执行以下操作：`$name = BreakScan::geyKeyValue($contents,'class',';')`。

1.  我们还需要一种方法来检索和重置`$this->messages`。以下是这两种方法：

```php
        public function clearMessages() : void {
            $this->messages = [];
        }
        public function getMessages(bool $clear = FALSE) {
            $messages = $this->messages;
            if ($clear) $this->clearMessages();
            return $messages;
        }
    ```

1.  然后我们定义一个运行所有扫描的方法（在下一节中涵盖）。这个方法还会收集检测到的潜在 BC 断点的数量并报告总数：

```php
        public function runAllScans() : int {
            $found = 0;
            $found += $this->scanRemovedFunctions();
            $found += $this->scanIsResource();
            $found += $this->scanMagicSignatures();
            $found += $this->scanFromCallbacks();
            return $found;
        }
    ```

现在你已经对基本的`BreakScan`类基础设施可能是什么有了一个概念，让我们来看看单独的扫描方法。

### 检查单独的扫描方法

四个单独的扫描方法直接对应于断点扫描配置文件中的顶级键。每个方法都应该累积关于潜在 BC 断点的消息在`$this->messages`中。此外，每个方法都应该返回一个表示检测到的潜在 BC 断点总数的整数。

现在让我们按顺序检查这些方法：

1.  我们首先检查的方法是`scanRemovedFunctions()`。在这个方法中，我们搜索函数名称，后面直接跟着开括号`'('`，或者是空格和开括号`' ('`。如果找到函数，我们递增`$found`，并将适当的警告和建议的替换添加到`$this-> messages`中。如果没有发现潜在的破坏，我们添加一个成功消息并返回`0`：

```php
    public function scanRemovedFunctions() : int {
        $found = 0;
        $config = $this->config[self::KEY_REMOVED];
        foreach ($config as $func => $replace) {
            $search1 = ' ' . $func . '(';
            $search2 = ' ' . $func . ' (';
            if (
                strpos($this->contents, $search1) !== FALSE
                || 
                strpos($this->contents, $search2) !== FALSE)
            {
                $this->messages[] = sprintf(
                    self::ERR_REMOVED, $func, $replace);
                $found++;
            }
        }
        if ($found === 0)
            $this->messages[] = sprintf(
                self::OK_PASSED, __FUNCTION__);
        return $found;
    }
    ```

这种方法的主要问题是，如果函数没有在空格之前，则不会检测到其使用。但是，如果我们在搜索中不包括前导空格，我们可能会得到错误的结果。例如，没有前导空格，每个`foreach()`的实例在寻找`each()`时都会触发破坏扫描器的警告！

1.  接下来，我们看一下扫描`is_resource()`使用的方法。如果找到引用，此方法将遍历不再生成资源的函数列表。如果同时找到`is_resource()`和其中一个这些方法，将标记潜在的 BC 破坏：

```php
    public function scanIsResource() : int {
        $found = 0;
        $search = 'is_resource';
        if (strpos($this->contents, $search) === FALSE)
            return 0;
        $config = $this->config[self::KEY_RESOURCE];
        foreach ($config as $func) {
            if ((strpos($this->contents, $func) !== FALSE)){
                $this->messages[] =
                    sprintf(self::ERR_IS_RESOURCE, $func);
                $found++;
            }
        }
        if ($found === 0)
            $this->messages[] = 
                sprintf(self::OK_PASSED, __FUNCTION__);
        return $found;
    }
    ```

1.  然后我们看一下需要通过我们的回调列表的内容。您还记得，我们需要在简单的`strpos()`无法满足的情况下使用回调。因此，我们首先收集所有回调子键并依次循环遍历每个子键。如果没有底层键*callback*，我们会抛出一个`Exception`。否则，我们运行回调，提供`$this->contents`作为参数。如果发现任何潜在的 BC 破坏，我们添加适当的错误消息，并递增`$found`：

```php
    public function scanFromCallbacks() {
        $found = 0;
        $list = array_keys($this-config[self::KEY_CALLBACK]);
        foreach ($list as $key) {
            $config = $this->config[self::KEY_CALLBACK][$key] 
                ?? NULL;
            if (empty($config['callback']) 
                || !is_callable($config['callback'])) {
                $message = sprintf(self::ERR_INVALID_KEY,
                    self::KEY_CALLBACK . ' => ' 
                    . $key . ' => callback');
                throw new Exception($message);
            }
            if ($config'callback') {
                $this->messages[] = $config['msg'];
                $found++;
            }
        }
        return $found;
    }
    ```

1.  最后，我们转向迄今为止最复杂的方法，该方法扫描无效的魔术方法签名。主要问题是方法签名差异很大，因此我们需要构建单独的正则表达式来正确测试有效性。正则表达式存储在 BC 破坏配置文件中。如果检测到魔术方法，我们检索其正确的签名并将其添加到`$this->messages`中。

1.  首先，我们检查是否有任何魔术方法，通过查找与`function __`匹配的内容：

```php
    public function scanMagicSignatures() : int {
        $found   = 0;
        $matches = [];
        $result  = preg_match_all(
            '/function __(.+?)\b/', 
            $this->contents, $matches);
    ```

1.  如果匹配数组不为空，我们循环遍历匹配集并将魔术方法名称分配给`$key`：

```php
       if (!empty($matches[1])) {
            $config = $this->config[self::KEY_MAGIC] ?? NULL;
            foreach ($matches[1] as $name) {
                $key = '__' . $name;
    ```

1.  如果未设置与假定魔术方法匹配的配置键，我们假设它既不是魔术方法，也不在配置文件中，因此无需担心。否则，如果存在键，我们提取表示分配给`$sub`的方法调用的子字符串：

```php
                if (empty($config[$key])) continue;
                if ($pos = strpos($this->contents, $key)) {
                    $end = strpos($this->contents, 
                        '{', $pos);
                $sub = (empty($sub) || !is_string($sub))
                     ? '' : trim($sub);
    ```

1.  然后，我们从配置中提取正则表达式并将其与子字符串匹配。该模式表示该特定魔术方法的正确签名。如果`preg_match()`返回`FALSE`，我们知道实际签名不正确，并将其标记为潜在的 BC 破坏。我们检索并存储警告消息并递增`$found`：

```php
                $ptn = $config[$key]['regex'] ?? '/.*/';
                if (!preg_match($ptn, $sub)) {
                    $this->messages[] = sprintf(
                      self::ERR_MAGIC_SIGNATURE, $key);
                    $this->messages[] = 
                      $config[$key]['signature'] 
                      ?? 'Check signature'
                    $found++;
        }}}}
        if ($found === 0)
            $this->messages[] = sprintf(
                self::OK_PASSED, __FUNCTION__);
         return $found;
    }
    ```

这结束了我们对`BreakScan`类的审查。现在我们将注意力转向定义调用程序，该程序需要运行`BreakScan`类中编程的扫描。

### 构建一个调用程序的 BreakScan 类

调用`BreakScan`类的程序的主要工作是接受一个路径参数，并递归构建该路径中的 PHP 文件列表。然后，我们循环遍历列表，依次提取每个文件的内容，并运行 BC 破坏扫描。最后，我们提供一个报告，可以是简洁的或详细的，取决于所选的详细级别。

请记住，`BreakScan`类和我们即将讨论的调用程序都是设计用于在 PHP 7 下运行。我们不使用 PHP 8 的原因是因为我们假设开发人员希望在进行 PHP 8 更新之前运行 BC 破坏扫描器：

1.  我们首先通过配置自动加载程序并从命令行（`$argv`）或 URL（`$_GET`）获取路径和详细级别。此外，我们提供了一个选项，将结果写入 CSV 文件，并接受此类文件的名称作为参数。您可能注意到我们还进行了一定程度的输入消毒，尽管理论上 BC 破坏扫描器只会在开发服务器上直接由开发人员使用：

```php
    // /repo/ch11/php7_bc_break_scanner.php
    define('DEMO_PATH', __DIR__);
    require __DIR__ . '/../src/Server/Autoload/Loader.php';
    $loader = new \Server\Autoload\Loader();
    use Php8\Migration\BreakScan;
    // some code not shown
    $path = $_GET['path'] ?? $argv[1] ?? NULL;
    $show = $_GET['show'] ?? $argv[2] ?? 0;
    $show = (int) $show;
    $csv  = $_GET['csv']  ?? $argv[3] ?? '';
    $csv  = basename($csv);
    ```

1.  接下来我们确认路径。如果找不到，我们将退出并显示使用信息（`$usage`未显示）：

```php
    if (empty($path)) {
        if (!empty($_SERVER['REQUEST_URI']))
            echo '<pre>' . $usage . '</pre>';
        else
            echo $usage;
        exit;
    }
    ```

1.  然后我们抓取 BC 破坏配置文件并创建`BreakScan`实例：

```php
    $config  = include __DIR__ 
    . '/php8_bc_break_scanner_config.php';
    $scanner = new BreakScan($config);
    ```

1.  为了构建文件列表，我们使用`RecursiveDirectoryIterator`，包装在`RecursiveIteratorIterator`中，从给定路径开始。然后，这个列表通过`FilterIterator`进行过滤，限制扫描仅限于 PHP 文件：

```php
    $iter = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($path));
    $filter = new class ($iter) extends FilterIterator {
        public function accept() {
            $obj = $this->current();
            return ($obj->getExtension() === 'php');
        }
    };
    ```

1.  如果开发人员选择 CSV 选项，将创建一个`SplFileObject`实例。与此同时，我们还输出了一个标题数组。此外，我们定义了一个写入 CSV 文件的匿名函数：

```php
    if ($csv) {
        $csv_file = new SplFileObject($csv, 'w');
        $csv_file->fputcsv(
            ['Directory','File','OK','Messages']);
    }
    $write = function ($dir, $fn, $found, $messages) 
        use ($csv_file) {
        $ok = ($found === 0) ? 1 : 0;
        $csv_file->fputcsv([$dir, $fn, $ok, $messages]);
        return TRUE;
    };
    ```

1.  我们通过循环遍历`FilterIterator`实例呈现的文件列表来启动扫描。由于我们是逐个文件扫描，所以在每次通过时`$found`都被清零。但是，我们确实保持`$total`，以便在最后给出潜在 BC 破坏的总数。您可能还注意到我们区分文件和目录。如果目录发生变化，其名称将显示为标题：

```php
    $dir   = '';
    $total = 0;
    foreach ($filter as $name => $obj) {
        $found = 0;
        $scanner->clearMessages();
        if (dirname($name) !== $dir) {
            $dir = dirname($name);
            echo "Processing Directory: $name\n";
        }
    ```

1.  我们使用`SplFileObject::isDir()`来确定文件列表中的项目是否是目录。如果是，我们将继续处理列表中的下一个项目。然后我们将文件内容推送到`$scanner`并运行所有扫描。然后以字符串形式检索消息：

```php
        if ($obj->isDir()) continue;
        $fn = basename($name);
        $scanner->getFileContents($name);
        $found    = $scanner->runAllScans();
        $messages = implode("\n", $scanner->getMessages());
    ```

1.  我们使用`switch()`块根据`$show`表示的显示级别采取行动。级别`0`仅显示发现潜在 BC 破坏的文件。级别`1`显示此外还有消息。级别`2`显示所有可能的输出，包括成功消息：

```php
        switch ($show) {
            case 2 :
                echo "Processing: $fn\n";
                echo "$messages\n";
                if ($csv) 
                    $write($dir, $fn, $found, $messages);
                break;
            case 1 :
                if (!$found) break;
                echo "Processing: $fn\n";
                echo BreakScan::WARN_BC_BREAKS . "\n";
                printf(BreakScan::TOTAL_BREAKS, $found);
                echo "$messages\n";
                if ($csv) 
                    $write($dir, $fn, $found, $messages);
                break;
            case 0 :
            default :
                if (!$found) break;
                echo "Processing: $fn\n";
                echo BreakScan::WARN_BC_BREAKS . "\n";
                if ($csv) 
                    $write($dir, $fn, $found, $messages);
        }
    ```

1.  最后，我们累积总数并显示最终结果：

```php
        $total += $found;
    }
    echo "\n" . str_repeat('-', 40) . "\n";
    echo "\nTotal number of possible BC breaks: $total\n";
    ```

现在您已经了解了调用可能的外观，让我们来看一下测试扫描的结果。

### 扫描应用程序文件

为了演示目的，在与本书相关的源代码中，我们包含了一个较旧版本的**phpLdapAdmin**。您可以在`/path/to/repo/sample_data/phpldapadmin-1.2.3`找到源代码。对于此演示，我们打开了 PHP 7 容器的 shell，并运行了以下命令：

```php
root@php8_tips_php7 [ /repo ]# 
php ch11/php7_bc_break_scanner.php \
    sample_data/phpldapadmin-1.2.3/ 1 |less
```

这是运行此命令的部分结果：

```php
Processing: functions.php
WARNING: the code in this file might not be 
compatible with PHP 8
Total potential BC breaks: 4
WARNING: the following function has been removed: function __autoload.  
Use this instead: spl_autoload_register(callable)
WARNING: the following function has been removed: create_function.  Use this instead: Use either "function () {}" or "fn () => <expression>"
WARNING: the following function has been removed: each.  Use this instead: Use "foreach()" or ArrayIterator
PASSED this scan: scanIsResource
PASSED this scan: scanMagicSignatures
WARNING: using the "@" operator to suppress warnings 
no longer works in PHP 8.
```

从输出中可以看出，尽管`functions.php`通过了`scanMagicSignatures`和`scanIsResource`扫描，但这个代码文件使用了在 PHP 8 中已删除的三个函数：`__autoload()`，`create_function()`和`each()`。您还会注意到这个文件使用`@`符号来抑制错误，在 PHP 8 中不再有效。

如果您指定了 CSV 文件选项，您可以在任何电子表格程序中打开它。以下是在 Libre Office Calc 中的显示方式：

![图 11.1-在 Libre Office Calc 中打开的 CSV 文件](img/Figure_11.1_B16562.jpg)

图 11.1-在 Libre Office Calc 中打开的 CSV 文件

现在您已经了解了如何创建自动化程序来检测潜在的 BC 破坏。请记住，代码远非完美，并不能涵盖每一个可能的代码破坏。为此，您必须在仔细审阅本书材料后依靠自己的判断。

现在是时候将我们的注意力转向实际迁移本身了。

# 执行迁移

执行从当前版本到 PHP 8 版本的实际迁移，就像部署新功能集到现有应用程序的过程一样。如果可能的话，您可以考虑并行运行两个网站，直到您确信新版本按预期工作为止。许多组织为此目的并行运行暂存环境和生产环境。

在本节中，我们提供了一个**十二步指南**来执行成功的迁移。虽然我们专注于迁移到 PHP 8，但这十二个步骤可以适用于您可能希望执行的任何 PHP 更新。仔细理解并遵循这些步骤对于您的生产网站的成功至关重要。在这十二个步骤中，有很多地方可以在遇到问题时恢复到早期版本。

在我们从旧版本的 PHP 迁移到 PHP 8 的十二步迁移过程中，这是一个概述：

1.  仔细阅读 PHP 文档附录中的适当迁移指南。在我们的情况下，我们选择*Migrating from PHP 7.4x to PHP 8.0x*。([`www.php.net/manual/en/appendices.php`](https://www.php.net/manual/en/appendices.php))。

1.  确保您当前的代码在当前版本的 PHP 上运行正常。

1.  备份数据库（如果有），所有源代码和任何相关的文件和资产（例如，CSS，JavaScript 或图形图像）。

1.  在您的版本控制软件中为即将更新的应用程序代码创建一个新分支。

1.  扫描 BC 中断（可能使用前一节中讨论的`BreakScan`类）。

1.  更新任何不兼容的代码。

1.  根据需要重复*步骤 5*和*6*。

1.  将您的源代码上传到存储库。

1.  在尽可能模拟生产服务器的虚拟环境中测试源代码。

1.  如果虚拟化模拟不成功，请返回到*步骤 5*。

1.  将暂存服务器（或等效的虚拟环境）更新到 PHP 8，确保可以切换回旧版本。

1.  运行您能想象到的每一个测试。如果不成功，请切换回主分支并返回到*步骤 5*。如果成功，克隆暂存环境到生产环境。

现在让我们依次看看每一步。

## 第 1 步 - 查看迁移指南

随着每个 PHP 的主要发布，PHP 核心团队都会发布一个**迁移指南**。我们在本书中主要关注的指南是*Migrating from PHP 7.4.x to PHP 8.0.x*，位于[`www.php.net/manual/en/migration80.php`](https://www.php.net/manual/en/migration80.php)。这个迁移指南分为四个部分：

+   新功能

+   向后不兼容的更改

+   弃用功能

+   其他更改

如果您正在从 PHP 7.4 以外的版本迁移到 PHP 8.0，您还应该查看您当前 PHP 版本的所有过去迁移指南，直到 PHP 8。我们现在将看看迁移过程中的其他推荐步骤。

## 第 2 步 - 确保当前代码正常工作

在开始对当前代码进行更改以确保其在 PHP 8 中正常工作之前，确保它绝对正常工作是非常关键的。如果现在代码不起作用，那么一旦迁移到 PHP 8，它肯定也不会起作用！运行任何单元测试以及任何**黑盒测试**，以确保代码在当前版本的 PHP 中正常运行。

如果在迁移之前对当前代码进行了任何更改，请确保这些更改反映在您版本控制软件的主分支（通常称为**主分支**）中。

## 第 3 步 - 备份所有内容

下一步是备份所有内容。这包括数据库、源代码、JavaScript、CSS、图像等。还请不要忘记备份重要的配置文件，如`php.ini`文件、web 服务器配置和与 PHP 和 web 通信相关的任何其他配置文件。

## 第 4 步 - 创建版本控制分支

在这一步中，您应该在您的版本控制系统中创建一个新的分支并检出该分支。在主分支中，您应该只有当前有效的代码。

这是使用 Git 进行此类命令的方式：

```php
$ git branch php8_migration
$ git checkout php8_migration
Switched to branch 'php8_migration'
```

所示的第一条命令创建了一个名为`php8_migration`的分支。第二条命令使`git`切换到新分支。在这个过程中，所有现有的代码都被移植到了新分支。主分支现在是安全的，并且在新分支中进行任何更改都得到了保留。

有关使用 Git 进行版本控制的更多信息，请查看这里：[`git-scm.com/`](https://git-scm.com/)。

## 第 5 步 - 扫描 BC 破坏

现在是时候充分利用`BreakScan`类了。运行调用程序，并作为参数提供项目的起始目录路径以及详细级别（`0`，`1`或`2`）。您还可以指定一个 CSV 文件作为第三个选项，就像*图 11.1*中早些时候所示的那样。

## 第 6 步 - 修复不兼容性

在这一步中，知道破坏的位置，您可以继续修复不兼容性。您应该能够以这样的方式进行修复，使得代码在当前版本的 PHP 中继续运行，同时也可以在 PHP 8 中运行。正如我们在整本书中一直指出的那样，BC 破坏在很大程度上源自糟糕的编码实践。通过修复不兼容性，您同时改进了您的代码。

## 第 7 步 - 根据需要重复步骤 5 和 6

有一句名言在许多好莱坞电影中反复出现，医生对焦虑的病人说，“服用两片阿司匹林，明天早上给我打电话”。同样的建议也适用于解决 BC 破坏的过程。您必须要有耐心，继续修复和扫描，修复和扫描。一直这样做，直到扫描不再显示潜在的 BC 破坏为止。

## 第 8 步 - 将更改提交到存储库

一旦您相对确信没有进一步的 BC 破坏，就是时候将更改提交到您在版本控制软件中创建的新 PHP 8 迁移分支。现在可以推送更改。然后，您可以在生产服务器上解决 PHP 更新后，从该分支检索更新的代码。

请记住这一重要点：您当前的工作代码安全地存储在主分支中。您只是在这个阶段保存到 PHP 8 迁移分支，所以您随时可以切换回去。

## 第 9 步 - 在模拟虚拟环境中进行测试

将这一步看作是真正事情的彩排。在这一步中，您创建一个虚拟环境（例如，使用 Docker 容器），最接近模拟生产服务器。在这个虚拟环境中，然后安装 PHP 8。一旦创建了虚拟环境，您可以打开一个命令行进入其中，并从 PHP 8 迁移分支下载您的源代码。

然后，您可以运行单元测试和任何其他您认为必要的测试，以测试更新后的代码。希望在这一步中能够捕获任何额外的错误。

## 第 10 步 - 如果测试不成功，则返回第 5 步

如果在虚拟环境中进行的单元测试、黑盒测试或其他测试显示您的应用程序代码失败，您必须返回到*第 5 步*。在面对明显的失败时继续前往实际生产站点将是极不明智的！

## 第 11 步 - 在暂存环境中安装 PHP 8

下一步是在暂存环境中安装 PHP 8。您可能还记得我们在本章第一部分讨论中提到的，传统流程是从开发环境到暂存环境，然后再到生产环境。一旦在暂存环境上完成了所有测试，您就可以将暂存克隆到生产环境。

PHP 的安装在主[php.net](http://php.net)网站上有详细的文档，因此这里不需要进一步的细节。相反，在本节中，我们为您提供了 PHP 安装的简要概述，重点是能够在 PHP 8 和当前 PHP 版本之间切换的能力。

提示

有关在各种环境中安装 PHP 的信息，请参阅此文档页面：[`www.php.net/manual/en/install.php`](https://www.php.net/manual/en/install.php)。

为了举例说明，我们选择讨论两个主要 Linux 分支上的 PHP 8 安装：Debian/Ubuntu 和 Red Hat/CentOS/Fedora。让我们从 Debian/Ubuntu Linux 开始。

### 在 Debian/Ubuntu Linux 上安装 PHP 8

安装 PHP 8 的最佳方法是使用现有的一组预编译的二进制文件。较新的 PHP 版本往往比发布日期晚得多，并且 PHP 8 也不例外。在这种情况下，建议您使用（**Personal Package Archive(PPA**）。托管在[`launchpad.net/~ondrej`](https://launchpad.net/~ondrej)的 PPA 是最全面和广泛使用的。

如果您想在自己的计算机上模拟以下步骤，请使用以下命令运行一个预先安装了 PHP 7.4 的 Ubuntu Docker 镜像：

```php
docker run -it \
  unlikelysource/ubuntu_focal_with_php_7_4:latest /bin/bash
```

为了在 Debian 或 Ubuntu Linux 上安装 PHP 8，打开一个命令行到生产服务器（或演示容器）上，并以*root*用户的身份进行如下操作。或者，如果没有*root*用户访问权限，可以在每个显示的命令前加上`sudo`。

从命令行安装 PHP 8，请按照以下步骤进行：

1.  使用**apt**实用程序更新和升级当前的软件包集。可以使用任何软件包管理器；但是，我们展示了使用`apt`来保持这里涵盖的安装步骤之间的一致性：

```php
    apt update
    apt upgrade
    ```

1.  将`Ondrej PPA`存储库添加到您的`apt`源中：

```php
    add-apt-repository ppa:ondrej/php
    ```

1.  安装 PHP 8。这只安装了 PHP 8 核心和基本扩展：

```php
    apt install php8.0
    ```

1.  使用以下命令扫描存储库以获取额外的扩展，并使用`apt`根据需要安装它们：

```php
    apt search php8.0-*
    ```

1.  进行 PHP 版本检查，以确保您现在正在运行 PHP 8：

```php
    php --version
    ```

以下是版本检查输出：

```php
root@ec873e16ee93:/# php --version
PHP 8.0.7 (cli) (built: Jun  4 2021 21:26:10) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.0.7, Copyright (c) Zend Technologies
with Zend OPcache v8.0.7, Copyright (c), by Zend Technologies
```

现在您已经对 PHP 8 安装可能进行的基本步骤有了基本的了解，让我们看看如何在当前版本和 PHP 8 之间切换。为了举例说明，我们假设在安装 PHP 8 之前，PHP 7.4 是当前的 PHP 版本。

### 在 Debian 和 Ubuntu Linux 之间切换 PHP 版本

如果您检查 PHP 的位置，您会注意到在 PHP 8 安装后，较早的版本 PHP 7.4 仍然存在。您可以使用`whereis php`来实现这一目的。我们模拟的 Ubuntu Docker 容器上的输出如下：

```php
root@ec873e16ee93:/# whereis php
php: /usr/bin/php /usr/bin/php8.0 /usr/bin/php7.4 /usr/lib/php /etc/php /usr/share/php7.4-opcache /usr/share/php8.0-opcache /usr/share/php8.0-readline /usr/share/php7.4-readline /usr/share/php7.4-json /usr/share/php8.0-common /usr/share/php7.4-common
```

如您所见，我们现在安装了 7.4 和 8.0 版本的 PHP。要在两者之间切换，请使用此命令：

```php
update-alternatives --config php
```

然后会出现一个选项屏幕，让您选择哪个 PHP 版本应该处于活动状态。以下是 Ubuntu Docker 镜像上输出屏幕的样子：

```php
root@ec873e16ee93:/# update-alternatives --config php 
There are 2 choices for the alternative php 
(providing /usr/bin/php).
  Selection    Path             Priority   Status
------------------------------------------------------------
* 0            /usr/bin/php8.0   80        auto mode
  1            /usr/bin/php7.4   74        manual mode
  2            /usr/bin/php8.0   80        manual mode
Press <enter> to keep the current choice[*], or type selection number:
```

切换后，您可以再次执行`php --version`来确认另一个 PHP 版本是否处于活动状态。

现在让我们把注意力转向 Red Hat Linux 及其衍生产品上的 PHP 8 安装。

### 在 Red Hat、CentOS 或 Fedora Linux 上安装 PHP 8

Red Hat、CentOS 或 Fedora Linux 上的 PHP 安装遵循一系列与 Debian/Ubuntu 安装过程相似的命令。主要区别在于，您很可能会使用`dnf`和`yum`的组合来安装预编译的 PHP 二进制文件。

如果您想跟随本节中我们概述的安装步骤，可以使用一个已经安装了 PHP 7.4 的 Fedora Docker 容器进行模拟。以下是运行模拟的命令：

```php
docker run -it unlikelysource/fedora_34_with_php_7_4 /bin/bash
```

与前一节描述的 PPA 环境非常相似，在 Red Hat 世界中，**Remi's RPM Repository**项目（[`rpms.remirepo.net/`](http://rpms.remirepo.net/)）以**Red Hat Package Management**（**RPM**）格式提供预编译的二进制文件。

要在 Red Hat、CentOS 或 Fedora 上安装 PHP 8，请打开一个命令行到生产服务器（或演示环境）上，并以*root*用户的身份进行如下操作：

1.  首先，确认您正在使用的操作系统版本和发行版是一个好主意。为此，使用`uname`命令，以及一个简单的`cat`命令来查看发行版（存储在`/etc`目录中的文本文件）：

```php
    [root@9d4e8c93d7b6 /]# uname -a
    Linux 9d4e8c93d7b6 5.8.0-55-generic #62~20.04.1-Ubuntu
    SMP Wed Jun 2 08:55:04 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
    [root@9d4e8c93d7b6 /]# cat /etc/fedora-release 
    Fedora release 34 (Thirty Four)
    ```

1.  在开始之前，请确保更新`dnf`并安装配置管理器：

```php
    dnf upgrade  
    dnf install 'dnf-command(config-manager)'
    ```

1.  然后，您可以将 Remi 的存储库添加到您的软件包源中，使用您喜欢的版本号替换`NN`：

```php
    dnf install \
      https://rpms.remirepo.net/fedora/remi-release-NN.rpm
    ```

1.  此时，您可以使用`dnf module list`确认已安装的 PHP 版本。我们还使用`grep`来限制显示的模块列表仅为 PHP。`[e]`表示*已启用*：

```php
    [root@56b9fbf499d6 /]# dnf module list |grep php
    php                    remi-7.4 [e]     common [d] [i],
    devel, minimal    PHP scripting language                        php                    remi-8.0         common [d], devel, minimal        PHP scripting language
    ```

1.  然后我们检查当前的 PHP 版本：

```php
    [root@d044cbe477c8 /]# php --version
    PHP 7.4.20 (cli) (built: Jun  1 2021 15:41:56) (NTS)
    Copyright (c) The PHP Group
    Zend Engine v3.4.0, Copyright (c) Zend Technologies
    ```

1.  接下来，我们重置 PHP 模块，并安装 PHP 8：

```php
    dnf -y module reset php
    dnf -y module install php:remi-8.0
    ```

1.  另一个快速的 PHP 版本检查显示我们现在使用的是 PHP 8 而不是 PHP 7：

```php
    [root@56b9fbf499d6 /]# php -v
    PHP 8.0.7 (cli) (built: Jun  1 2021 18:43:05) 
    ( NTS gcc x86_64 ) Copyright (c) The PHP Group
    Zend Engine v4.0.7, Copyright (c) Zend Technologies
    ```

1.  要切换回较早版本的 PHP，请按照以下步骤进行，其中`X.Y`是您打算使用的版本：

```php
    dnf -y module reset php
    dnf -y module install php:remi-X.Y
    ```

这完成了 Red Hat、CentOS 或 Fedora 的 PHP 安装说明。在本演示中，我们只向您展示了 PHP 命令行安装。如果您计划与 Web 服务器一起使用 PHP，还需要安装适当的 PHP Web 服务器包和/或安装 PHP-FPM（FastCGI 处理模块）包。

现在让我们来看看最后一步。

## 第 12 步 – 测试并将暂存环境克隆到生产环境

在最后一步中，您将从 PHP 8 迁移分支下载源代码到暂存环境，并运行各种测试以确保一切正常。一旦您确保成功，然后将暂存环境克隆到生产环境。

如果您使用虚拟化，克隆过程可能只涉及创建一个相同的 Docker 容器或虚拟磁盘文件。否则，如果涉及实际硬件，您可能最终会克隆硬盘，或者根据您的设置选择适当的方法。

这完成了我们关于如何执行迁移的讨论。现在让我们来看看测试和故障排除。

# 测试和故障排除迁移

在理想的情况下，迁移故障排除将在上线服务器或模拟的虚拟环境上进行，远在实际上线之前。然而，正如经验丰富的开发人员所知，我们需要抱最好的希望，但做最坏的准备！在本节中，我们将涵盖一些可能被轻易忽视的测试和故障排除的其他方面。

在本节中，如果您正在遵循 Debian/Ubuntu 或 Red Hat/CentOS/Fedora 安装过程，可以退出临时 shell。返回用于本课程的 Docker 容器，并打开 PHP 8 容器的命令 shell。如果您不确定如何操作，请参阅*第一章*的*技术要求*部分，了解更多信息。

## 测试和故障排除工具

这里有太多优秀的测试和故障排除工具可用，无法在此处一一列举，因此我们将重点放在一些开源工具上，以帮助测试和故障排除。

### 使用 Xdebug

Xdebug 是一个工具，提供诊断、分析、跟踪和逐步调试等功能。它是一个 PHP 扩展，因此能够在您遇到无法轻松解决的问题时提供详细信息。主要网站是[`xdebug.org/`](https://xdebug.org/)。

要启用 Xdebug 扩展，您可以像安装任何其他 PHP 扩展一样安装它：使用`pecl`命令，或者从[`pecl.php.net/package/xdebug`](https://pecl.php.net/package/xdebug)下载并编译源代码。

此外，至少应设置以下`/etc/php.ini`设置：

```php
zend_extension=xdebug.so
xdebug.log=/repo/xdebug.log
xdebug.log_level=7
xdebug.mode=develop,profile
```

*图 11.2*显示了从`/repo/ch11/php8_xdebug.php`调用的`xdebug_info()`命令的输出：

![图 11.2 – xdebug_info()输出](img/Figure_11.2_B16562.jpg)

图 11.2 – xdebug_info()输出

现在让我们来看看另一个从外部视角检查您的应用程序的工具。

### 使用 Apache JMeter

用于测试 Web 应用程序的一个非常有用的开源工具是**Apache JMeter**([`jmeter.apache.org/`](https://jmeter.apache.org/))。它允许您开发一系列测试计划，模拟来自浏览器的请求。您可以模拟数百个用户请求，每个请求都有自己的 cookie 和会话。尽管主要设计用于 HTTP 或 HTTPS，但它还能够处理其他十几种协议。除了出色的图形用户界面外，它还有一个命令行模式，可以将 JMeter 纳入自动部署过程中。

安装非常简单，只需从[`jmeter.apache.org/download_jmeter.cgi`](https://jmeter.apache.org/download_jmeter.cgi)下载一个文件。在运行 JMeter 之前，您必须安装**Java 虚拟机**（**JVM**）。测试计划的执行超出了本书的范围，但文档非常详尽。另外，请记住，JMeter 设计为在客户端上运行，而不是在服务器上运行。因此，如果您希望在本书的 Docker 容器中测试网站，您需要在本地计算机上安装 Apache JMeter，然后构建一个指向 Docker 容器的测试计划。通常，PHP 8 容器的 IP 地址是`172.16.0.88`。

*图 11.3*显示了在本地计算机上运行的 Apache JMeter 的开屏幕：

![图 11.3 – Apache JMeter](img/Figure_11.3_B16562.jpg)

图 11.3 – Apache JMeter

从这个屏幕上，您可以开发一个或多个测试计划，指示要访问的 URL，模拟`GET`和`POST`请求，设置用户数量等。

提示

如果您在尝试运行`jmeter`时遇到此错误：“无法加载库：/usr/lib/jvm/java-11-openjdk-amd64/lib/ libawt_xawt.so”，请尝试安装*OpenJDK 8*。然后，您可以使用前面部分提到的技术来在不同版本的 Java 之间切换。

现在让我们看看在 PHP 8 升级后可能出现的 Composer 问题。

## 处理 Composer 的问题

在迁移到 PHP 8 后，开发人员可能面临的一个常见问题是与第三方软件有关。在本节中，我们讨论了使用流行的*Composer*包管理器为 PHP 可能遇到的潜在问题。

您可能会遇到的第一个问题与 Composer 本身的版本有关。在 2020 年，Composer 2 版本发布了。然而，并非所有驻留在主要打包网站([`packagist.org/`](https://packagist.org/))上的 30 万多个软件包都已更新到版本 2。因此，为了安装特定软件包，您可能需要在 Composer 2 和 Composer 1 之间切换。每个版本的最新发布都在这里：

+   版本 1：[`getcomposer.org/download/latest-1.x/composer.phar`](https://getcomposer.org/download/latest-1.x/composer.phar)

+   版本 2：[`getcomposer.org/download/latest-2.x/composer.phar`](https://getcomposer.org/download/latest-2.x/composer.phar)

另一个更严重的问题与您可能使用的各种 Composer 软件包的平台要求有关。每个软件包都有自己的`composer.json`文件，具有自己的要求。在许多情况下，软件包提供者可能会添加 PHP 版本要求。

问题在于，虽然大多数 Composer 软件包现在在 PHP 7 上运行，但要求是以一种排除 PHP 8 的方式指定的。在 PHP 8 更新后，当您使用 Composer 更新第三方软件包时，会出现错误并且更新失败。具有讽刺意味的是，大多数 PHP 7 软件包也可以在 PHP 8 上运行！

例如，我们安装了一个名为`laminas-api-tools`的 Composer 项目。在撰写本文时，尽管软件包本身已准备好用于 PHP 8，但其许多依赖软件包尚未准备好。在运行安装 API 工具的命令时，会遇到以下错误：

```php
root@php8_tips_php8 [ /srv ]# 
composer create-project laminas-api-tools/api-tools-skeleton
Creating a "laminas-api-tools/api-tools-skeleton" project at "./api-tools-skeleton"
Installing laminas-api-tools/api-tools-skeleton (1.3.1p1)
  - Downloading laminas-api-tools/api-tools-skeleton (1.3.1p1)
  - Installing laminas-api-tools/api-tools-skeleton (1.3.1p1):
Extracting archiveCreated project in /srv/api-tools-skeleton
Loading composer repositories with package information
Updating dependencies
Your requirements could not be resolved to an installable set of packages.
  Problem 1
    - Root composer.json requires laminas/laminas-developer-tools dev-master, found laminas/laminas-developer-tools[dev-release-1.3, 0.0.1, 0.0.2, 1.0.0alpha1, ..., 1.3.x-dev, 2.0.0, ..., 2.2.x-dev] but it does not match the constraint.
  Problem 2
    - zendframework/zendframework 2.5.3 requires php ⁵.5     || ⁷.0 -> your php version (8.1.0-dev) does not satisfy     that requirement.
```

刚刚显示的输出的最后部分突出显示的核心问题是，其中一个依赖包需要 PHP `⁷.0`。在 `composer.json` 文件中，这表示从 PHP 7.0 到 PHP 8.0 的一系列版本。在这个特定的例子中，使用的 Docker 容器运行的是 PHP 8.1，所以我们有问题。

幸运的是，在这种情况下，我们有信心，如果这个包在 PHP 8.0 中运行，它也应该在 PHP 8.1 中运行。因此，我们只需要添加 `--ignore-platform-reqs` 标志。当我们重新尝试安装时，如下输出所示，安装成功了：

```php
root@php8_tips_php8 [ /srv ]# 
composer create-project --ignore-platform-reqs \
    laminas-api-tools/api-tools-skeleton
Creating a "laminas-api-tools/api-tools-skeleton" project at "./api-tools-skeleton"
Installing laminas-api-tools/api-tools-skeleton (1.6.0)
  - Downloading laminas-api-tools/api-tools-skeleton (1.6.0)
  - Installing laminas-api-tools/api-tools-skeleton (1.6.0):
Extracting archive
Created project in /srv/api-tools-skeleton
Installing dependencies from lock file (including require-dev)
Verifying lock file contents can be installed on current
platform.
Package operations: 109 installs, 0 updates, 0 removals
- Downloading laminas/laminas-zendframework-bridge (1.3.0)
- Downloading laminas-api-tools/api-tools-asset-manager
(1.4.0)
- Downloading squizlabs/php_codesniffer (3.6.0)
- Downloading dealerdirect/phpcodesniffer-composer-installer
(v0.7.1)
- Downloading laminas/laminas-component-installer (2.5.0)
... not all output is shown
```

刚刚显示的输出中，没有出现平台要求错误，我们可以继续使用应用程序。

现在让我们把注意力转向单元测试。

## 使用单元测试

使用 PHPUnit 进行单元测试是确保应用程序在添加新功能或进行 PHP 更新后能够运行的关键因素。大多数开发人员至少创建一组单元测试，以至少执行最低要求，以证明应用程序的预期性能。测试是一个类中的方法，该类扩展了 `PHPUnit\Framework\TestCase`。测试的核心是所谓的“断言”。

提示

本书不涵盖如何创建和运行测试的说明。但是，您可以在主要 PHPUnit 网站的出色文档中找到大量示例：[`phpunit.de/`](https://phpunit.de/)。

在进行 PHP 迁移后，您可能会遇到的问题是 PHPUnit（[`phpunit.de/`](https://phpunit.de/)）本身可能会失败！原因是因为 PHPUnit 每年都会发布一个新版本，对应于当年的 PHP 版本。较旧的 PHPUnit 版本是基于官方支持的 PHP 版本。因此，您的应用程序当前安装的 PHPUnit 版本可能是不支持 PHP 8 的较旧版本。最简单的解决方案是使用 Composer 进行更新。

为了说明可能的问题，让我们假设应用程序的测试目录当前包括 PHP unit 5。如果我们在运行 PHP 7.1 的 Docker 容器中运行测试，一切都按预期工作。以下是输出：

```php
root@php8_tips_php7 [ /repo/test/phpunit5 ]# php --version
PHP 7.1.33 (cli) (built: May 16 2020 12:47:37) (NTS)
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
    with Xdebug v2.9.1, Copyright (c) 2002-2020, by Derick
Rethans
root@php8_tips_php7 [ /repo/test/phpunit5 ]#
vendor/bin/phpunit 
PHPUnit 5.7.27 by Sebastian Bergmann and contributors.
........                                                          8 / 8 (100%)
Time: 27 ms, Memory: 4.00MB
OK (8 tests, 8 assertions)
```

然而，如果我们在运行 PHP 8 的 Docker 容器中运行相同的版本，结果会大不相同：

```php
root@php8_tips_php8 [ /repo/test/phpunit5 ]# php --version
PHP 8.1.0-dev (cli) (built: Dec 24 2020 00:13:50) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.0-dev, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.0-dev, Copyright (c), 
    by Zend Technologies
root@php8_tips_php8 [ /repo/test/phpunit5 ]#
vendor/bin/phpunit 
PHP Warning:  Private methods cannot be final as they are never overridden by other classes in /repo/test/phpunit5/vendor/ phpunit/phpunit/src/Util/Configuration.php on line 162
PHPUnit 5.7.27 by Sebastian Bergmann and contributors.
........                                                            8 / 8 (100%)
Time: 33 ms, Memory: 2.00MB
OK (8 tests, 8 assertions)
```

从输出中可以看出，PHPUnit 本身报告了一个错误。当然，简单的解决方案是，在 PHP 8 升级后，您还需要重新运行 Composer，并更新您的应用程序及其使用的所有第三方包。

这就结束了我们对测试和故障排除的讨论。您现在知道可以使用哪些额外工具来帮助您进行测试和故障排除。请注意，这绝不是所有测试和故障排除工具的全面列表。还有许多其他工具，有些是免费开源的，有些提供免费试用期，还有一些只能通过购买获得。

# 总结

在本章中，您了解到术语“环境”是指“服务器”，因为如今许多网站使用虚拟化服务。然后，您了解到在部署阶段使用了三种不同的环境：开发、暂存和生产。

介绍了一种自动化工具，能够扫描您的应用程序代码，以寻找潜在的代码错误。正如您在该部分学到的那样，一个扫描应用程序可能包括一个配置文件，用于处理已删除的功能、方法签名的更改、不再生成资源的函数，以及用于复杂用法检测的一组回调，一个扫描类，以及一个收集文件名的调用程序。

接下来，您将看到一个典型的十二步 PHP 8 迁移过程，确保在最终准备升级生产环境时成功的机会更大。每个步骤都旨在发现潜在的代码错误，并在出现问题时有备用程序。您还学会了如何在两个常见平台上安装 PHP 8，以及如何轻松地恢复到旧版本。最后，您了解了一些可以帮助测试和故障排除的免费开源工具。

总的来说，仔细阅读本章并学习示例后，您现在不仅可以使用现有的测试和故障排除工具，还可以想到如何开发自己的扫描工具，大大降低 PHP 8 迁移后潜在代码错误的风险。您现在也对 PHP 8 迁移涉及的内容有了很好的了解，并且可以进行更顺畅的过渡，而不必担心失败。您新的预期和解决迁移问题的能力将减轻您可能会遇到的任何焦虑。您还可以期待拥有快乐和满意的客户。

下一章将介绍 PHP 编程中的新潮流和令人兴奋的趋势，可以进一步提高性能。
