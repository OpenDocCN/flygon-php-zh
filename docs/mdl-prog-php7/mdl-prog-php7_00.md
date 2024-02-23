# 前言

构建模块化应用程序是一项具有挑战性的任务。它涉及广泛的知识领域，从设计模式和原则到所选择技术栈的方方面面。PHP 生态系统有相当多的工具、库、框架和平台，可以帮助我们实现模块化应用程序开发的目标。

PHP 7 带来了许多改进，可以进一步帮助实现这一目标。我们将从这些改进中开始我们的旅程。在本书结束时，我们的最终交付物将是一个由 Symfony 框架构建的模块化网络商店应用程序。

# 本书内容

第一章，生态系统概述，对 PHP 生态系统的当前状态进行了简要介绍。它探讨了 PHP 7 的最新功能，其中一些功能为模块化开发中的新概念打开了大门。此外，本章还概述了流行的 PHP 框架。

第二章，GoF 设计模式，描述了软件设计中常见问题的重复解决方案。为以下每种模式提供了实际的 PHP 示例：创建模式类型、结构模式和行为模式。

第三章，SOLID 设计原则，深入探讨了面向对象编程和设计的五个基本原则，这些原则使用 SOLID（单一责任、开闭原则、里氏替换、接口隔离和依赖反转）的首字母缩写。它提供了实际示例，并解释了这些原则在模块化开发中的重要性。

第四章，模块化网络商店应用的需求规范，指导读者定义整体应用程序需求的过程。它从定义实际应用程序功能需求开始，并逐步进行技术栈选择。

第五章，Symfony 概述，对 Symfony 作为框架、一组工具和开发方法论进行了高层次的概述。它侧重于我们构建模块化应用程序所需的构建模块。

第六章，构建核心模块，指导您通过基于 Symfony 捆绑包设置核心模块。然后，核心模块用于为其他模块设置结构和依赖关系。

第七章，构建目录模块，指导我们通过构建一个与网络商店仅目录功能集相匹配的自给模块。它向我们展示了如何设置与模块功能相关的实体，以及如何使用现有框架管理这些实体及其交互。

第八章，构建客户模块，指导我们通过构建一个与网络商店客户相关的功能集相匹配的自给模块。它向我们展示了如何设置与模块功能相关的实体，以及如何使用现有框架管理这些实体及其交互。它进一步向我们展示了如何创建注册和登录系统。

第九章，构建支付模块，指导我们通过构建一个与网络商店支付相关的功能集相匹配的自给模块。它向我们展示了如何与第三方支付提供商集成。它进一步向我们展示了如何将支付提供商作为服务提供给其他模块使用。

第十章，“构建发货模块”，指导我们构建一个自给自足的模块，与网店的发货相关功能相匹配。它向我们展示了如何定义几个扁平方法，根据不同的购物车产品属性产生不同的发货定价。它进一步向我们展示了如何将发货方法公开为其他模块使用的服务。

第十一章，“构建销售模块”，指导我们构建一个自给自足的模块，与网店仅销售相关的功能集相匹配。它向我们展示了如何设置与模块功能相关的购物车、购物车项目、订单和订单项目实体，以及如何使用现有框架管理这些实体及其交互。

第十二章，“集成和分发模块”，将前几章中构建的所有模块集成到一个单一的功能应用程序中。接下来，它指导我们通过现代 PHP 模块分发技术。这些技术包括 Git 和 Composer，间接包括 GitHub 和 Packagist。

# 您需要什么

为了成功运行本书提供的所有示例，您需要自己的 Web 服务器或第三方 Web 托管解决方案。高级技术栈包括 PHP 7.0 或更高版本，Apache/Nginx 和 MySQL。

Symfony 框架本身带有详细的系统要求列表，可以在[`symfony.com/doc/current/reference/requirements.html`](http://symfony.com/doc/current/reference/requirements.html)找到。

本书假设读者熟悉设置完整的开发环境。

# 本书的受众

本书主要面向中级 PHP 开发人员，他们对模块化编程几乎没有了解，希望了解设计模式和原则，以更好地利用现有框架进行模块化应用程序开发。

本书开发的模块化网店应用程序使用 Symfony 框架。但是，不需要假设或要求对 Symfony 框架有任何先前的了解。

# 约定

在本书中，您会发现一些区分不同类型信息的文本样式。以下是一些这些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“我们可以通过使用`include`指令包含其他上下文。”

代码块设置如下：

```php
function hint (int $A, float $B, string $C, bool $D)
{
    var_dump($A, $B, $C, $D);
}
```

任何命令行输入或输出都以以下形式编写：

```php
**sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony**
**sudo chmod a+x /usr/local/bin/symfony**

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会出现在文本中，如：“单击**下一步**按钮将您移至下一个屏幕。”

### 注意

警告或重要说明会出现在这样的框中。

### 提示

技巧和窍门会以这种方式出现。