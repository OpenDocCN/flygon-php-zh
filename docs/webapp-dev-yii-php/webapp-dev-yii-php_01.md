# 第一章：见识 Yii

网络开发框架通过立即提供核心基础和所需的基础设施，帮助您快速将白板上的想法转化为功能齐全的生产就绪代码，从而帮助您快速启动应用程序。有了今天网络应用程序所需的所有常见功能，并且有可用的框架选项来满足这些期望，几乎没有理由从头开始编写下一个网络应用程序。现代、灵活、可扩展的框架几乎和编程语言本身一样是今天网络开发人员的必备工具。当两者特别互补时，结果是一个非常强大的工具包——比如 Java 和 Spring、Ruby 和 Rails、C#和.NET 以及 PHP 和 Yii。

Yii 是创始人薛强的心血结晶，他于 2008 年 1 月 1 日开始开发这个开源框架。薛强在开始这个项目之前，曾经多年开发和维护 PRADO 框架。从 PRADO 项目中积累的多年经验和用户反馈，巩固了对一个更易于扩展、更高效的基于 PHP5 的框架的需求，以满足应用开发人员日益增长的需求。Yii 的初始 alpha 版本于 2008 年 10 月正式发布，与其他基于 PHP 的框架相比，其极其出色的性能指标立即引起了极大的关注。2008 年 12 月 3 日，Yii 1.0 正式发布，截至 2012 年 10 月 1 日，最新的生产就绪版本为 1.1.12。它拥有一个不断壮大的开发团队，并且每天都在 PHP 开发人员中不断增加其知名度。

**Yii**这个名字是**Yes, it is**的缩写，发音为*Yee*或(*ji:*)。Yii 是一个高性能、基于组件的、用 PHP5 编写的 Web 应用程序框架。这个名字也代表了最常用来描述它的形容词，比如易用、高效和可扩展。让我们快速看一下 Yii 的每个特点。逐个进行。

# 易用

要运行基于 Yii 1.x 的 Web 应用程序，您只需要核心框架文件和支持 PHP 5.1.0 或更高版本的 Web 服务器。要使用 Yii 进行开发，您只需要了解 PHP 和面向对象编程。您不需要学习任何新的配置或模板语言。构建 Yii 应用程序主要涉及编写和维护自己的自定义 PHP 类，其中一些将扩展自核心 Yii 框架组件类。

Yii 吸收了许多其他知名 Web 编程框架和应用程序的优秀理念和工作。因此，如果您从其他网络开发框架转向 Yii，您很可能会发现它很熟悉并且易于操作。

Yii 还秉承着“约定优于配置”的理念，这有助于其易用性。这意味着 Yii 对于几乎所有用于配置应用程序的方面都有合理的默认值。遵循规定的约定，您可以编写更少的代码，并花费更少的时间开发应用程序。但是，Yii 并不强迫您。它允许您自定义所有默认值，并且很容易覆盖所有这些约定。我们将在本章和整本书中介绍一些这些默认值和约定。

# 高效

Yii 是一个高性能的基于组件的框架，可用于开发任何规模的 Web 应用程序。它鼓励在 Web 编程中最大程度地重用代码，并且可以显著加速开发过程。正如之前提到的，如果您遵循 Yii 的内置约定，您可以几乎不需要手动配置就能让应用程序立即运行起来。

Yii 还设计为帮助您进行*DRY*开发。**DRY**代表**不要重复自己**，这是敏捷应用程序开发的一个关键概念。所有 Yii 应用程序都是使用**模型-视图-控制器**（**MVC**）架构构建的。Yii 通过提供一个地方来保存您的 MVC 代码的每一部分来强制执行这种开发模式。这最小化了重复，并有助于促进代码重用和易于维护。您需要编写的代码越少，将应用程序推向市场所需的时间就越少。应用程序越容易维护，它在市场上的时间就越长。

当然，这个框架不仅使用高效，而且速度非常快，性能优化。Yii 从一开始就考虑了性能优化，并且结果是 PHP 框架中最高效的之一。因此，Yii 增加到其上的应用程序的任何额外开销都是极其微不足道的。

# 可扩展

Yii 经过精心设计，几乎可以扩展和定制其代码的每一部分，以满足任何项目需求。事实上，很难不利用 Yii 的可扩展性，因为开发 Yii 应用程序的主要活动之一就是扩展核心框架类。如果您想将扩展的代码转化为其他开发人员有用的工具，Yii 提供了易于遵循的步骤和指南，帮助您创建这样的第三方扩展。这使您能够为 Yii 不断增长的功能列表做出贡献，并积极参与扩展 Yii 本身。

值得注意的是，这种易用性、卓越性能和深度的可扩展性并不是以牺牲其功能为代价的。Yii 充满了功能，帮助您满足当今 Web 应用程序所面临的高要求。支持 AJAX 的小部件，RESTful 和 SOAP Web 服务集成，MVC 架构的强制执行，DAO 和关系 ActiveRecord 数据库层，复杂的缓存，分层基于角色的访问控制，主题，国际化（I18N）和本地化（L10N）只是 Yii 冰山一角。从 1.1 版本开始，核心框架现在打包了一个官方扩展库，称为*Zii*。这些扩展由核心框架团队成员开发和维护，并继续扩展 Yii 的核心功能集。并且随着一个庞大的用户社区，他们也通过编写 Yii 扩展来贡献，Yii 应用程序的整体功能集每天都在增长。在 Yii 框架网站上可以找到可用的用户贡献的扩展列表，网址为[`www.yiiframework.com/extensions`](http://www.yiiframework.com/extensions)。还有一个*非官方*的扩展库，其中包含了一些很棒的扩展，网址为[`yiiext.github.com/`](http://yiiext.github.com/)，这真正展示了社区的力量和这个框架的可扩展性。

# MVC 架构

正如前面提到的，Yii 是一个 MVC 框架，并为每个模型、视图和控制器代码的每个部分提供了明确的目录结构。在我们开始构建第一个 Yii 应用程序之前，我们需要定义一些关键术语，并了解 Yii 如何实现和强制执行这种 MVC 架构。

## 模型

通常在 MVC 架构中，*模型*负责维护状态，并应该封装适用于定义此状态的数据的业务规则。在 Yii 中，模型是框架类`CModel`或其子类的任何实例。模型类通常由可以具有单独标签（用于显示目的的用户友好内容）的数据属性组成，并且可以根据模型中定义的一组规则进行验证。构成模型类属性的数据可以来自数据库表的一行，也可以来自用户输入表单中的字段。

Yii 实现了两种模型，即表单模型（`CFormModel`类）和活动记录（`CActiveRecord`类）。它们都是从同一个基类`CModel`继承而来。`CFormModel`类表示收集 HTML 表单输入的数据模型。它封装了所有表单字段验证的逻辑，以及可能需要应用于表单字段数据的任何其他业务逻辑。然后它可以将这些数据存储在内存中，或者借助活动记录模型将数据存储在数据库中。

活动记录（AR）是一种设计模式，用于以面向对象的方式抽象数据库访问。Yii 中的每个 AR 对象都是`CActiveRecord`或其子类的实例，它包装数据库表或视图中的单行数据，封装了所有与数据库访问相关的逻辑和细节，并包含了大部分需要应用于该数据的业务逻辑。表行中每个列的数据字段值都表示为活动记录对象的属性。稍后将更详细地介绍活动记录。

## 视图

通常*视图*负责呈现用户界面，通常基于模型中的数据。Yii 中的视图是一个包含用户界面相关元素的 PHP 脚本，通常使用 HTML 构建，但也可以包含 PHP 语句。通常，视图中的任何 PHP 语句都非常简单，是条件或循环语句，或者引用其他 Yii UI 相关元素，如 HTML 助手类方法或预构建小部件。更复杂的逻辑应该与视图分离，并适当放置在模型中（如果直接处理数据）或控制器中（用于更一般的业务逻辑）。

## 控制器

*控制器*是我们路由请求的主要指挥官，负责接收用户输入，与模型交互，并指示视图更新和适当显示。Yii 中的控制器是`CController`或其子类的实例。当控制器运行时，它执行请求的操作，然后与必要的模型交互，并呈现适当的视图。一个操作，简单来说，就是一个以*action*开头的控制器类方法。

# 将这些连接在一起：Yii 请求路由

在 MVC 实现中，Web 请求通常具有以下生命周期：

+   浏览器向托管 MVC 应用程序的服务器发送请求

+   调用控制器操作来处理请求

+   控制器与模型交互

+   控制器调用视图

+   视图呈现数据（通常为 HTML）并将其返回给浏览器显示

Yii 的 MVC 实现也不例外。在 Yii 应用程序中，来自浏览器的传入请求首先由路由器接收。路由器分析请求以决定应将其发送到应用程序的何处进行进一步处理。在大多数情况下，路由器会识别控制器类中的特定操作方法，将请求传递给该方法。这个操作方法将查看传入的请求数据，可能与模型交互，并执行其他所需的业务逻辑。最终，这个操作方法将准备好响应数据并发送给视图。视图将格式化这些数据以符合所需的布局和设计，并返回给浏览器显示。

## 博客发布示例

为了更好地理解所有这些，让我们看一个虚构的例子。假设我们使用 Yii 构建了一个新的博客网站`http://yourblog.com`。这个网站与大多数典型的博客网站类似。主页显示最近发布的博客文章列表。每篇博客文章的名称都是超链接，可以将用户带到显示完整文章的页面。以下图表说明了 Yii 如何处理从点击这些假想博客文章链接发送的传入请求：

![博客发布示例](img/9584_01_01.jpg)

该图表跟踪了用户点击链接时发出的请求：`http://yourblog.com/post/show/id/99`

首先，请求被发送到路由器。路由器解析请求，决定将其发送到何处。URL 的结构对路由器将做出的决定至关重要。默认情况下，Yii 识别以下格式的 URL：

`http://hostname/index.php?r=ControllerID/ActionID`

`r`查询字符串变量指的是 Yii 路由器分析的路由。它将解析此路由以确定适当的控制器和操作方法，以进一步处理请求。现在您可能立即注意到我们上面的示例 URL 不遵循此默认格式。这只是一个非常简单的配置应用程序以识别以下格式的 URL：

`http://hostname/ControllerID/ActionID`

我们将继续使用这种简化的格式来进行示例。URL 中的`ControllerID`名称指的是控制器的名称。默认情况下，这是控制器类名称的第一部分，直到单词`Controller`。例如，如果您的控制器类名称是`TestController`，`ControllerID`名称将是*test*。`ActionID`类似地指的是由控制器定义的操作的名称。如果操作是在控制器内定义的简单方法，那么它将是方法名称中跟在单词`action`后面的任何内容。例如，如果您的操作方法名为`actionCreate()`，`ActionID`名称就是`create`。

### 注意

如果省略了`ActionID`，控制器将采取默认操作，按照约定是控制器中称为`actionIndex()`的方法。如果还省略了`ControllerID`，应用程序将使用默认控制器。Yii 默认控制器称为`SiteController`。

回到示例，路由器将分析 URL`http://yourblog.com/post/show/id/99`，并将 URL 路径的第一部分`post`作为`ControllerID`，将第二部分`show`作为`ActionID`。这将转换为将请求路由到`PostController`类中的`actionShow()`方法。URL 的最后部分，`id/99`部分，是一个*名称/值*查询字符串参数，在处理过程中将可用于该方法。在这个示例中，数字`99`代表所选博客文章的唯一内部 ID。

在我们虚构的博客应用程序中，`actionShow()`方法处理特定博客文章条目的请求。它使用查询字符串变量`id`来确定请求的特定文章。它要求模型检索有关博客文章条目编号 99 的信息。模型 AR 类与数据库交互以检索所请求的数据。在从模型检索数据后，我们的控制器通过使其可用于视图来进一步准备数据以供显示。视图然后负责处理数据布局，并向浏览器提供响应以供用户显示。

这种 MVC 架构允许我们将数据呈现与数据操作、验证和其他应用程序业务逻辑分开。这使得开发人员非常容易改变应用程序的各个方面而不影响 UI，UI 设计人员也可以自由地进行更改而不影响模型或业务逻辑。这种分离还使得非常容易提供同一模型代码的多个呈现方式。例如，您可以使用驱动`http://yourblog.com`的 HTML 布局的相同模型代码来驱动 RIA 呈现、移动应用程序、Web 服务或命令行界面。最终，遵循这些约定并分离功能将导致一个更容易扩展和维护的应用程序。

Yii 做了很多工作来帮助您执行这种分离，不仅仅提供一些命名约定和代码放置建议。它有助于处理所有需要将所有部分粘合在一起的低级"胶水"代码。这使您能够在不必自己编写所有细节的情况下获得严格的 MVC 设计应用程序的好处。让我们来看看其中一些低级细节。

# 对象关系映射和 Active Record

在很大程度上，我们构建的 Web 应用程序将其数据存储在关系数据库中。我们在上一个示例中使用的博客帖子应用程序将博客帖子内容存储在数据库表中。然而，Web 应用程序需要将持久数据库存储中的数据映射到定义域对象的内存类属性中。**对象关系映射**（**ORM**）库提供了将数据库表映射到域对象类的功能。

处理 ORM 的大部分代码都是关于描述数据库中的字段如何对应到我们内存对象的属性，并且编写起来是乏味和重复的。幸运的是，Yii 通过提供 Active Record（AR）模式的 ORM 层来拯救我们，使我们免受这种重复和乏味。

## Active Record

如前所述，AR 是一种用于以面向对象的方式抽象数据库访问的设计模式。它将表映射到类，行映射到对象，列映射到类属性。换句话说，每个 Active Record 类的实例代表数据库表中的一行。然而，AR 类不仅仅是一组属性，这些属性映射到数据库表中的列。它还包含应用于该数据的必要业务逻辑。最终结果是一个定义了如何写入和从数据库中读取的类。

通过依赖约定并坚持合理的默认设置，Yii 对 AR 的实现将节省开发人员大量时间，否则可能会花在配置上，或者编写创建、读取、更新和删除数据所需的乏味重复的 SQL 语句上。它还允许开发人员以面向对象的方式访问存储在数据库中的数据。为了说明这一点，让我们再次以我们虚构的博客示例为例。以下是一些使用 AR 操作特定博客帖子的示例代码，其内部 ID（也用作表的主键）为`99`。它首先通过使用主键检索帖子。然后更改标题并更新数据库以保存更改：

```php
$post=Post::model()->findByPk(99);
$post->title='Some new title';
$post->save();
```

Active Record 完全解除了我们编写任何 SQL 代码或以其他方式处理底层数据库的乏味。

实际上，Yii 中的 Active Record 甚至做得更多。它与 Yii 框架的许多其他方面无缝集成。有许多"活动"HTML 助手输入表单字段直接与它们各自的 AR 类属性相关联。通过这种方式，AR 直接提取输入表单字段的值到模型中。它还支持复杂的自动数据验证，如果验证失败，Yii 视图类可以轻松地将验证错误显示给最终用户。我们将在本书中多次重新访问 AR 并提供具体示例。

# 视图和控制器

视图和控制器非常密切相关。控制器使数据可供视图显示，视图生成页面触发事件，将数据发送到控制器。

在 Yii 中，视图文件属于呈现它的控制器类。通过这种方式，我们可以在视图脚本中简单地引用`$this`来访问控制器实例。这种实现方式使视图和控制器非常密切。

当涉及到 Yii 控制器时，故事远不止调用模型和渲染视图那么简单。控制器可以管理服务，以提供对请求的复杂预处理和后处理，实现基本的访问控制规则以限制对某些操作的访问，管理应用程序范围的布局和嵌套布局文件的渲染，管理数据的分页，以及许多其他幕后服务。再次感谢 Yii，让我们不必为这些混乱的细节而费心。

Yii 有很多内容。探索它所有美丽的最佳方式就是开始使用它。现在我们已经掌握了一些非常基本的想法和术语，我们有很好的条件来做到这一点。

# 总结

在本章中，我们在很高的层次上介绍了 Yii PHP Web 应用程序框架。我们还涵盖了 Yii 所采用的许多软件设计概念。如果你对这次初步讨论的抽象性有些困惑，不要担心。一旦我们深入具体的例子，一切都会变得清晰起来。但总结一下，我们具体涵盖了：

+   应用程序开发框架的重要性和实用性

+   Yii 是什么，以及使 Yii 变得非常强大和有用的特点。

+   MVC 应用程序架构以及在 Yii 中实现此架构

+   典型的 Yii Web 请求生命周期和 URL 结构

+   Yii 中的对象关系映射和 Active Record

在下一章中，我们将通过简单的 Yii 安装过程，并开始构建一个工作应用程序，以更好地阐述所有这些想法。
