# 前言

几年前，如果你对某人说“游戏化”，你会得到一个奇怪的表情，好像你在编造一些新东西。也许你会得到一个快速的跟进问题：“嗯？那是什么？”

今天，那种遭遇会有所不同。尽管被人们误解，但企业正在非常认真地对待游戏化，因为他们发现越来越难以吸引并最终留住客户。

在这本书中，我们更仔细地研究了游戏化。我们假设您对游戏化一无所知，但对网站开发有一些背景。我们将带您了解本书中概述的游戏化设计框架，并在此过程中应用游戏化原则。

尽管不难理解，但游戏化需要一种专注于用户参与和乐趣的方法来开发系统。

完成后，您将拥有一个实时项目，以展示您对关键游戏化原则的理解。

# 本书涵盖的内容

第一章，*游戏化教育过程*，列举了教育环境中游戏化的当前用途。

第二章，*框架*，为读者提供了一个框架，可以应用到未来的游戏化项目中，以及我们将在本书的其余部分中构建的最终网站项目的视觉模拟。

第三章，*目标和目标行为*，带领读者通过游戏化过程的前几个实际步骤。我们为我们的项目定义了目标和目标行为。

第四章，*玩家*，帮助我们识别系统中的角色，用户和利益相关者（即玩家），以及他们的动机。

第五章，*活动*，概述了我们网站项目的游戏化系统的关键点，机制，元素，规则等。

第六章，*乐趣*，帮助读者理解内在动机和外在动机之间的区别，以及什么使一项活动有趣。

第七章，*总结*，总结了我们在前几章中所做的一切。读者通过游戏化设计框架进行迭代，实施更多的游戏元素来驱动用户行为。

附录，*表*，包含了 Bartle 玩家类型的参与循环以及我们的游戏化系统的游戏化设计框架概述。

# 这本书需要什么

要运行本书中的示例，需要最新版本的以下软件：

+   WAMP 服务器

+   PHP（5.0 版本或更高版本）

+   MySQL

+   MySQL Workbench（可选）

+   JQuery 库

+   文本编辑器

读者会发现这些材料很有用，如果他们对网站开发有基本的了解（HTML、JavaScript、CSS 和 PHP）。

# 这本书适合谁

这本书适用于想要构建游戏化应用程序的 IT 专业人员。您需要了解基本的 HTML 和网站工作的基本原理。如果您曾经构建过任何类型的网站，那么您已经走在了正确的道路上。

# 约定

在本书中，您会发现一些区分不同信息类型的文本样式。以下是这些样式的一些示例，以及它们的含义解释。

文本中的代码单词显示如下：“将数据库命名为`VuPoint`”。

代码块设置如下：

```php
[DELIMITER //
-- Procedure Name: SelectOnlinePlayers
-- Procedure Purpose: Returns a lists of players how currently logged in is true
CREATE PROCEDURE 'vupoint'.'SelectOnlinePlayers'()
BEGIN
Select  * from Player where CurrentlyLoggedIn=true;
END//
DELIMITER;
```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，菜单或对话框中的单词会在文本中出现，就像这样：“点击**数据库**菜单”。

### 注意

警告或重要提示会以这样的框出现。

### 提示

提示和技巧会以这种方式出现。
