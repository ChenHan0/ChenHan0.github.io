---
title: "DirectX 11 教程一"
categories:
    - DirectX  
tag:
    - DirectX
    - Graphics
last_modified_at: 2018-04-09T23:56:00+08:00
---


# 使用Visual Studio 2015设置DirectX 11

在编写任何图形代码之前，我们需要具备这些工具。第一个工具是一个编译器，最好内置在一个漂亮的IDE中。我使用的是Visual Studio 2015。还有其他几种可用的，有些是免费的。我会让你决定选择哪一个。我个人使用Visual Studio 2015社区版，因为它是免费的，并且是一个出色的IDE。你可以从Visual Studio网站上下载Visual Studio社区版。当你在安装Visual Studio 2015时，请确保选择自定义并全选，以便Visual C++组件都已安装，否则它主要针对C#开发进行设置。  

<!-- more -->  

你需要的第二个工具是Window 10 SDK。Windows 10 SDK包含你需要编写的DirectX 11应用程序所需的所有DirectX 11头文件，库，DLL等。如果你安装了Visual Studio 2015，那么已经安装了SDK。否则，你可以从Windows开发人员中心(msdn)网站下载适用于Windows 10的Windows独立SDK。下载并安装SDK后，你将拥有编译DirectX 11程序所需的文件。Windows 10 SDK的文档也都位于Windows开发人员中心网站上。你将在那里找到Direct3D 11编程指南，其中包含所有DirectX 11文档以及一些示例代码。  

安装IDE和SDK后，现在可以将IDE设置为与Windows 10 SDK配合使用，以便编写DIrectX 11应用程序。请注意，在安装Windows 10 SDK之前，首先需要安装一些IDE。  

## <u>设置Visual Studio 2015</u>

在Visual Studio 2015中，我使用了以下步骤：  

首先，你需要创建一个空的Win32项目，然后选择File -> New -> Project。在新建项目菜单中选择Templates -> Visual C++ -> Win32。然后在选项中选择Win32 Project。然后给项目取一个名字（我将其命名为mine Engine）并选择储存位置，然后点击“Ok”。点击“Next”,你会得到另一个菜单。在“Addition options”下，在“Empty project”框中勾选复选标记，然后取消选中“Security Development Lifecycle(SDL) checks”。点击“Finish”，你现在应该有一个基本的Win32空项目设置完成了。  

然后在顶部栏上，你将在“Solution Platform”下拉列表中看到值“x86”，选择此选项并选择为“x64”。这会将你的项目设置为64位而不是默认的32位。  

## <u>总结</u>

我们现在应该为DirectX 11开发环境进行了设置并准备好开始编写DirectX 11应用程序了。  

