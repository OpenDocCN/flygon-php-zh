# 前言

本书是使用 Yii Web 应用程序开发框架开发真实应用程序的逐步教程。本书试图模拟一个被要求构建在线应用程序的软件开发团队的环境，涵盖软件开发生命周期的每个方面，从构思到生产部署，构建项目任务管理应用程序。

在对 Yii 框架进行简要的一般介绍，并通过标志性的“Hello World”示例之后，剩下的章节以与真实项目中软件开发迭代相同的方式进行了分解。我们首先创建一个具有有效，经过测试的与数据库连接的工作应用程序。

然后我们开始定义我们的主要数据库实体和领域对象模型，并熟悉 Yii 的**对象关系映射**（**ORM**）层*Active Record*。我们学习如何依靠 Yii 的代码生成工具自动构建针对我们新创建的模型的创建/读取/更新/删除（CRUD）功能。我们还关注 Yii 的表单验证和提交模型的工作原理。到第五章 *管理问题* 结束时，您将拥有一个可以管理项目和项目中的问题（任务）的工作应用程序。

然后我们转向用户管理的话题。我们了解 Yii 内置的身份验证模型，以帮助应用程序的登录和注销功能。我们深入研究授权模型，首先利用 Yii 的简单访问控制模型，然后实施 Yii 提供的更复杂的**基于角色的访问控制**（**RBAC**）框架。

到第七章 *用户访问控制* 结束时，任务管理应用程序的所有基础都已就位。接下来的几章将开始专注于额外的用户功能，用户体验和设计。我们添加用户评论功能，介绍了一种可重用的内容小部件架构方法。我们添加了一个 RSS 网络订阅，并演示了在 Yii 应用程序中集成其他第三方工具和框架有多么容易。我们利用 Yii 的主题结构来帮助简化和设计应用程序，然后介绍 Yii 的国际化（`I18N`）功能，以便应用程序可以在不进行工程更改的情况下适应各种语言和地区。

在最后一章中，我们将把重点转向准备应用程序进行生产部署。我们介绍了优化性能和提高安全性的方法，以准备应用程序投入生产环境。

# 本书涵盖内容

第一章，*遇见 Yii*，为您提供了 Yii 的简要历史，介绍了**模型视图控制器**（**MVC**）应用程序架构，并向您介绍了典型的请求生命周期，从最终用户通过应用程序，最终作为响应返回给最终用户。

第二章，*入门*，致力于下载和安装框架，创建一个新的 Yii 应用程序外壳，并介绍 Gii，Yii 强大灵活的代码生成工具。

第三章，*TrackStar 应用程序*，介绍了 TrackStar 应用程序。这是一个在线项目管理和问题跟踪应用程序，您将在接下来的章节中构建。在这里，您将学习如何将 Yii 应用程序连接到底层数据库。您还将学习如何从命令行运行交互式 shell。本章的最后部分侧重于在 Yii 应用程序中提供单元测试和功能测试的概述，并提供了在 Yii 中编写单元测试的具体示例。

第四章，“项目 CRUD”，帮助您开始与数据库交互，开始向基于数据库的 Yii 应用程序 TrackStar 添加功能。您将学习如何使用 Yii 迁移进行数据库变更管理，我们使用 Gii 工具创建模型类，并使用模型类构建创建、读取、更新和删除（CRUD）功能。本章还向读者介绍了配置和执行表单字段验证。

第五章，“管理问题”，解释了如何向 TrackStar 应用程序添加其他相关的数据库表，并介绍了 Yii 中的关联 Active Record。本章还涵盖了使用控制器过滤器来利用应用程序生命周期，以提供前操作和后操作处理。我们介绍了官方 Yii 扩展库 Zii，并使用 Zii 小部件来增强 TrackStar 应用程序。

第六章，“用户管理和身份验证”，解释了如何在 Yii 中对用户进行身份验证。在为 TrackStar 应用程序添加管理用户功能的同时，读者学习如何利用 Yii 中的“行为”来实现对 Yii 组件之间共享通用代码和功能的极其灵活的方法。本章还详细介绍了 Yii 的身份验证模型。

第七章，“用户访问控制”，专门介绍了 Yii 的授权模型。首先，我们介绍了简单的访问控制功能，它允许您轻松地为基于多个参数的控制器操作配置访问规则。然后，我们看看在 Yii 中如何实现基于角色的访问控制（RBAC），它允许更健壮的授权模型，完全基于角色、操作和任务的分层模型进行完整的访问控制。将基于角色的访问控制实现到 TrackStar 应用程序中还向读者介绍了在 Yii 中使用控制台命令。

第八章，“添加用户评论”，帮助演示了如何实现允许用户在 TrackStar 应用程序中对项目和问题进行评论的功能；我们介绍了如何配置和使用统计查询关系，如何创建高度可重用的用户界面组件称为“小部件”，以及如何在 Yii 中定义和使用命名范围。

第九章，“添加 RSS Web Feed”，演示了在 Yii 应用程序中使用其他第三方框架和库有多么容易，并展示了如何使用 Yii 的 URL 管理功能来自定义应用程序的 URL 格式和结构。

第十章，“使其看起来不错”，帮助您更多地了解 Yii 中的视图，以及如何使用布局来管理跨应用程序页面共享的标记和内容。我们还介绍了主题化，展示了如何轻松地为 Yii 应用程序赋予全新的外观，而无需修改任何基础工程。然后，我们将介绍 Yii 中的国际化（`i18n`）和本地化（`l10n`），因为语言翻译被添加到我们的 TrackStar 应用程序中。

第十一章，“使用 Yii 模块”，解释了如何通过使用 Yii 模块向 TrackStar 网站添加管理功能。模块提供了一种非常灵活的方法来开发和管理应用程序中较大的、独立的部分。

第十二章，“生产就绪”，帮助我们准备我们的 TrackStar 应用程序投入生产。您将了解 Yii 的日志框架、缓存技术和错误处理方法，以帮助使您的 Yii 应用程序达到生产就绪状态。

# 本书所需软件

本书需要以下软件：

+   Yii 框架版本 1.1.12

+   PHP 5.1 或更高版本（建议使用 5.3 或 5.4）

+   MySQL 5.1 或更高版本

+   能够运行 PHP 5.1 的 Web 服务器；本书中提供的示例是在 Apache HTTP 服务器上构建和测试的，在这些服务器上 Yii 已经在 Windows 和 Linux 环境中进行了彻底测试

+   Zend 框架版本 1.1.12 或更高版本（仅需要第九章中的内容，*添加 RSS Web Feed*，以及此库的下载和配置，这在本章中有介绍）

# 这本书适合谁

如果您是具有面向对象编程知识的 PHP 程序员，并且希望快速开发现代、复杂的 Web 应用程序，那么这本书适合您。阅读本书不需要对 Yii 有任何先前的了解。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码词如下所示：“我们可以通过使用`include`指令来包含其他上下文。”

代码块设置如下：

```php
'components'=>array( 
'db'=>array(
    'connectionString' => 'mysql:host=localhost;dbname=trackstar',
    'emulatePrepare' => true,
    'username' => '[YOUR-USERNAME]',
    'password' => '[YOUR-PASSWORD]',
    'charset' => 'utf8',
  ),
),
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项会以粗体显示：

```php
'components'=>array( 
'db'=>array(
    **'connectionString' => 'mysql:host=localhost;dbname=trackstar',
    'emulatePrepare' => true,**
    'username' => '[YOUR-USERNAME]',
    'password' => '[YOUR-PASSWORD]',
    'charset' => 'utf8',
  ),
),
```

任何命令行输入或输出都以以下方式编写：

```php
**$ yiic migrate create <name>**
**%cd /WebRoot/trackstar/protected/tests**

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，如菜单或对话框中的单词，会以这样的方式出现在文本中：“点击**下一步**按钮会将您移动到下一个屏幕”。

### 注意

警告或重要说明会以这样的方式出现在框中。

### 提示

提示和技巧会以这样的方式出现。
