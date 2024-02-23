# 前言

PHP 是一种免费开源的编程语言，正在持续复兴，而 Laravel 处于前沿。Laravel 5 被证明是最适合新手和专家程序员的可用框架。遵循现代 PHP 的面向对象最佳实践，可以减少上市时间，并构建强大的 Web 和 API 驱动的移动应用程序，可以自动测试和部署。

您将学习如何使用 Laravel 5 PHP 框架快速开发软件应用程序。

# 这本书涵盖了什么

第一章，*使用 phpspec 进行正确设计*，讲述了如何配置 Laravel 5 以使用 phpspec 进行现代单元测试，如何使用 phpspec 设计类，以及执行单元和功能测试。

第二章，*自动化测试-迁移和填充数据库*，涵盖了数据库迁移，其背后的机制以及如何为测试创建种子。

第三章，*构建服务、命令和事件*，讨论了 Model-View-Controller 以及它如何演变为服务、命令和事件，以解耦代码并实践关注点分离。

第四章，*创建 RESTful API*，带您了解如何创建 RESTful API：基本的 CRUD 操作（创建、读取、更新和删除），以及讨论一些最佳实践和超媒体控制（HATEOAS）。

第五章，*使用表单生成器*，带您进入 Web 界面的一面，展示如何利用 Laravel 5 的一些最新功能来创建 Web 表单。这里还将讨论反向路由。

第六章，*使用注解驯服复杂性*，专注于注解。当应用程序变得复杂时，`routes.php`文件很容易变得混乱。在控制器内部使用注解，可以大大提高代码的可读性；然而，除了优点之外，还存在一些缺点。

第七章，*使用中间件过滤请求*，向您展示如何创建可在控制器之前或之后调用的可重用过滤器。

第八章，*使用 Eloquent ORM 查询数据库*，帮助您学习如何以一种方式使用 ORM 来减少编码错误的概率，增加安全性并减少 SQL 注入的可能性，以及学习如何处理 Eloquent ORM 的限制。

第九章，*扩展 Laravel*，讲述了如何将应用程序扩展到基于云的架构。讨论了读写主/从配置，并引导读者进行配置。

第十章，*使用 Elixir 构建、编译和测试*，介绍了 Elixir。Elixir 基于 gulp，是一个任务运行器，是一系列构建脚本，可以自动化 Laravel 软件开发工作流程中的常见任务。

# 这本书需要什么

我们需要以下软件：

+   Apache/Nginx

+   PHP 5.4 或更高版本

+   MySQL 或类似软件

+   Composer

+   phpspec

+   Node.js

+   npm

# 这本书适合谁

如果您是一位经验丰富的新手或者是一位有能力的 PHP 程序员，对现代 PHP（至少版本 5.4）的概念有基本的了解，那么这本书非常适合您。

需要基本的面向对象编程和数据库知识。您应该已经熟悉 Laravel，或者至少已经尝试过这个框架。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些示例以及它们的含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下：“新的`artisan`命令如下运行”

代码块设置如下：

```php
protected function schedule(Schedule $schedule)
    {
        $schedule->command('inspire')
             ->hourly();
        $schedule->command('manage:waitinglist')
            ->everyFiveMinutes();

    }
```

任何命令行输入或输出都是这样写的：

```php
**$ php artisan schedule:run**

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会以这种方式出现在文本中：“如下截图所示，**迁移**表现在这里。”

### 注意

警告或重要说明会出现在这样的框中。

### 提示

提示和技巧会出现在这样。