# 投稿指南

非常欢迎大家给Dronecode项目的相关指南积极投稿。 投稿内容包括PX4和QGroundControl 开发者和用户指南，以及MAVLink指南。 本篇章用来向投稿者们解释如何对项目指南进行更改、添加内容和创建翻译。

> **Note**你需要申请一个(免费的) [Github](http://github.com)帐户来向项目指南投稿。

## 快速更改

如需修改错误或编辑一个*现有页面*，请按如下步骤操作：

1. 点击指南相关页面顶部的工具栏图标**Edit**。
    
    ![Gitbook: 编辑页面按钮](../../assets/gitbook/gitbook_toolbar_icon_edit.png)
    
    打开编辑页面(Github)。

2. 进行修改。

3. 页面底部会提示投稿者创建一个单独的分支，然后 引导其提交一个*拉取请求*。

The documentation team reviews submitted pull requests and will either merge it or work with you to update it.

## 添加新内容 - 大更改

If you want to add new pages or images that can't easily be done through the Github interface. In this case you make changes in the same way as you would for code changes: use the *git* toolchain to get the code, modify it as needed, test that it renders properly using the Gitbook client, create a branch for your changes and create a pull request (PR) to pull it back into the documentation. The following instructions explain how.

### 内容分类

The *Developer Guide* is for documentation that is relevant to *software developers*. This includes users who need to:

* 添加或修改软件平台功能，如模块、飞行模式等。
* 添加新硬件支持/集成，如飞行控制器、外围、机型等。
* 从外部源与平台通信，例如一个配套的计算机。
* 了解软件架构

The *User Guide*, by contrast, is *primarily* for users who want to:

* 使用 PX4 控制飞行器
* 使用 PX4，基于已支持/现存的机型，构建、修改或配置类似载具。

> **Tip** 例如，关于如何构建/配置现有机架类型的详细信息在用户指南中，而 定义*新*机架类型的说明在开发者指南中。

### Gitbook 文档工具

The guide uses the [Legacy Gitbook Toolchain](https://legacy.gitbook.com/) toolchain.

Change requests can be either done on the Gitbook website using the [Gitbook editor](https://gitbookio.gitbooks.io/documentation/content/editor/index.html) or locally (more flexible, but less user-friendly).

In order to contribute many changes to the documentation, it is recommended that you follow these steps to add the changes locally and then create a pull request:

* 如果您还没有注册Github，请先[注册](https://github.com/join) Github 账户
* 点击[这里](https://github.com/PX4/px4_user_guide)查看PX4用户指南，点击[这里](https://github.com/PX4/Devguide)查看PX4开发者指南。 点击[这里](https://help.github.com/articles/fork-a-repo/#fork-an-example-repository)，查看如何分支git repository。
* 将分支克隆到本地计算机  
        sh
        cd ~/wherever/
        git clone https://github.com/<your git name>/px4_user_guide.git

* 通过 NPM 安装 gitbook。 在终端提示下，只需运行以下命令即可安装 GitBook：
    
    ```sh
    npm install gitbook-cli -g
    ```
    
    > **Note** 本地安装和构建Gitbook 的所需一切也会在[工具链文档](https://github.com/GitbookIO/gitbook/blob/master/docs/setup.md)中说明。

* 进入您的本地版本库并添加原始的上游版本：
    
    ```sh
    cd ~/wherever/px4_user_guide
    git remote add upstream https://github.com/PX4/px4_user_guide.git
    ```

* 现在您可以检出一个新分支，并向其中添加您的更改。 要构建您的卷册，请运行：
    
    ```sh
    gitbook build
    ```
    
    > **Note** 如果遇到报错: `/usr/bin/env: node：No such file or directory`, 请运行 `ln -s /usr/bin/nodejs /usr/bin/node`

* 要预览并服务您的卷册，请运行：
    
    ```sh
    gitbook serve
    ```
    
    > **Note** 运行 `gitbook install` 可安装缺失的插件。

* 现在您可以在 http://localhost:4000/ 上浏览您的本地卷册

* 在终端提示中使用`CTRL+c`退出。

* 除了端口4000，您也可以在另一个端口上作业：
    
    ```sh
    gitbook serve --port 4003
    ```

* 您也可以以 html 、pdf、epub 或 mobi 格式输出： 
        sh
        gitbook help

* 预览过您的更改后，只要您感觉满意，您便可以添加并提交这些更改：
    
    ```sh
    git add <file name>
    git commit -m "<your commit message>"
    ```
    
    需进一步了解提交信息, 请参阅 [Contributing](../contribute/README.md) 部分。

* 现在, 您可以将本地提交推送到分支版本库
    
    ```sh
    git push origin <your feature branch name>
    ```

* 您可以通过浏览器访问分支版本库来验证推送是否成功： ```https://github.com/<your git name>/px4_user_guide.git```  
    您应该会看到一条告知消息：一个新分支已被推送到您的分支版本库。
* 现在是时候创建一个拉取请求 (PR) 了。 在 "新分支消息" 的右侧 (请参阅前面的一个步骤), 您应该看到一个绿色按钮, 上面写着 "Compare & Create Pull Request"。 然后, 它应该列出你的更改，你必须添加一个有意义的标题 (在提交 PR 的情况下, 它通常是提交消息) 和消息 (<span style="color:orange">解释你做了这些更改的原因 </span>， 检查 [其他拉取请求 ](https://github.com/PX4/px4_user_guide/pulls) 进行比较)。
* 搞定！ PX4 指南项目的负责人现在将看到您的投稿, 并决定是否要整合稿件内容。 每过一段时间，他们会检查你的更改，以确保没有疑义。

In overview:

* 用多个单独文件编写的页面，使用 markdown \(与Github wiki中的语法几乎一致\)。 
* 卷册的*架构*定义，详见**SummMMARY.md**文件。
* 这是一本[多语种](https://github.com/GitbookIO/gitbook/blob/master/docs/languages.md)的卷册。 所以在根目录中有一个**LANGS.md**文件来定义支持哪些语言。 每种语言的页面都存储在用相关语言代码命名的文件夹\(例如，中文的“zh”、 英文的“en”\)。 
* 一个名为**book.json**的文件定义了此构建的所有依赖关系。
* 网络钩子会被用来跟踪是否有文件合并到此版本库上的主分支，如果有，卷册将重新构建。

## 文档规范指南

1. 文件/文件名

* 将新文件放入相应的子文件夹
* 使用描述性名称。 特别是图像类型文件，文件名应简要描述其中的内容。
* 命名中使用小写，并用下划线"\_"分割单词

2. 图片

* 使用能保证图像使用价值的最小尺寸和最小分辨率。
* 默认情况下，新图像应在 **/assets/** 的子文件夹中创建 (这样翻译之间可共享图像资源)。

3. 内容:

* 始终如一地使用 "样式" (如bold, emphasis等) 按钮和菜单定义，请使用样式**Bold**。 工具名称，请使用样式*Emphasis*。 其他情况，尽量少用样式。
* 标题和页面标题应该遵从"第一字母大写"
* 页面标题应该是一级标题 。 所有其他标题应该是二级或更低级别的标题。
* 不要在标题中添加任何样式。
* 不要翻译注释、提示或警告声明的*第一部分* （例如`**Note**` ），因为这部分是用来向读者表达注解内容级别的。

## 翻译 {#translation}

We'd love your help to translate *QGroundControl* and our guides for PX4, *QGroundControl* and MAVLink. For more information see: [Translation](../contribute/translation.md).

## 许可证

All PX4/Dronecode documentation is free to use and modify under terms of the permissive [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) licence.