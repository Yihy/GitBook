---
date : 2017-11-26T13:47:08+02:00
tags : 
  - idea
  - 翻译
title : Idea  本地化指南
---


标题：Idea  本地化指南
---

翻译自 [IntelliJ平台SDK 开发指南](http://www.jetbrains.org/intellij/sdk/docs/reference_guide/localization_guide.html)

该文档的目的是描述创建IDEA的本地化版本所需的步骤。
——————
## 应用程序包 布局

关于本地化目的，需要翻译的所有资源（英文）位于一个jar文件中
***resources_en.jar***。
这个jar文件用于IDEA核心功能
***%INST_HOME%\lib\resources_en.jar***
和每个捆绑的插件都有一个资源位于
***%INST_HOME%\plugins\$Plugin$\lib\resources_en.jar***.


翻译的资源应该是相似的，并放置在源jar一样的目录。
所以本地化包应该具有完全相同数量的jar文件，并且它们必须以与原始jar文件相同的目录结构。
为了在每次安装时启用多个本地化，而本地化包覆盖彼此，我们建议将该区域的名称添加到jar名称中（例如，***resources_cn.jar***）。

## resources_en.jar 的内容和布局

属性文件通常包含消息，菜单项，对话标签文本等。
对于每个这样的文件，本地化的jar应该包含相对于jar包位于完全相同路径的翻译版本，并且具有与原始文件加上区域设置标识符完全相同的名称。

例如  ***resources_en.jar***中***messages/ActionsBundle.properties*** 文件  应该有其翻译版本 ***messages/ActionsBundle_cn.properties*** 文件在 ***resources_cn.jar***中。

所有属性文件都应使用 *\uXXXX* 序列进行ASCII编码，这些序列用于没有ASCII表示形式的字符。
请[native2ascii](http://docs.oracle.com/javase/7/docs/technotes/tools/solaris/native2ascii.html)参阅工具了解更多详情。

在线转码工具：[在线编码转换--OSChina](http://tool.oschina.net/encode?type=3)


属性值主要遵循MessageFormat规则。

>  **注意** 由于历史原因，主菜单、工具栏、弹出菜单和其他操作的助记符以 `\_` （下划线）为前缀，而所有其他助记符（如复选框，按钮等）使用 `&` （和号）标志为同一目的。 此外，在某些地方可以遇到 `&&` （双和号），它们表示在MacOS X下使用的替代助记符（助记符映射到 `U`, `I`, `O`, `N` 字符，是不工作的）。 一般来说，使用与原始属性值相同的助记符，一切都可以。

## 本地化组件

* **检查说明（Inspection descriptions）** 出现在 Settings|Errors 中，并表示有关每个检测工具打算做什么的简短描述信息。
每个描述由 ***/inspectionDescriptions/*** 文件夹下的单个html文件表示，应以UTF-8编码编码。
应将本地化版本存储在后缀为locale的文件夹中。 例如***resources_en.jar*** 中的***/inspectionDescriptions/CanBeFinal.html*** 翻译后应该存在***resources_cn.jar***中***/inspectionDescriptions_cn/CanBeFinal.html*** 。

* **意图描述和样本（Intention descriptions and samples）**与检查说明非常相似，但布局更为先进。
每个意图都有一堆文件，位于 /intentionDescriptions/ 中的intent的短名称的文件夹中。
这些文件包括 description.html，其中包含与检查一样的描述，另外还有几个模板文件，说明了样本的意图。
这些模板是可选的翻译。 与检查描述类似，整个intentDescriptions文件夹应该带有区域设置标识符。
例如 ***/intentionDescriptions/AddOnDemandStaticImportAction/description.html*** 翻译应放在 ***/intentionDescriptions_cn/AddOnDemandStaticImportAction/description.html***
所有的HTML文件必须使用UTF-8编码

* **Tips of the day** 遵循相同的模式检查和意图。
例如，***/tips/AdaptiveWelcome.html*** 的翻译将存到***/tips_cn/AdaptiveWelcome.html***。
关于提示的唯一特别之处在于它们使用特殊模式来表示快捷键，如 *EnterAction*;
将被替换为在运行时映射到 *EnterAction* 的当前使用的键映射中的点击。
所以请确保您在翻译时保留完整的序列。
记住UTF-8编码。

* **文件模板（File templates）**再次以同样的方式（如果一切都应该被翻译）。
***/fileTemplates/Singleton.java.ft*** 存到 ***/fileTemplates_cn/Singleton.java.ft***。

按照Sun的属性绑定规则，无论何时某些资源在本地化版本中找不到，将使用  ***resources_en.jar***  的默认版本。

本人空闲时间会进行idea本地化。

## 项目地址
| 网站 | 地址 |
| :-:     |  :-:   |
|Github  |  [https://github.com/Yihy/resources_cn](https://github.com/Yihy/resources_cn)  |
| 码云 | [https://gitee.com/yihyforever/resources_cn](https://gitee.com/yihyforever/resources_cn) |
