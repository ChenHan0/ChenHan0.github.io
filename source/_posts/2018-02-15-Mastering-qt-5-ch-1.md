---
title: "Mastering Qt 5 翻译 第一章 Qt入门基础"
categories:
    - Cpp  
tag:
    - Cpp  
    - Qt  
last_modified_at: 2018-02-15T11:26:00+08:00
---

# 第一章 Qt入门基础  

如果你知道C++都没有接触过Qt，或者您已经制作了一些中级的Qt应用程序，这章将确保你的Qt基础是足以学习接下来的章节中的高级概念的。  

<!-- more -->  

我们将教你使用Qt Creator创建一个简单的Todo应用程序。这个应用程序将展示一个你可以创建/更新/删除的任务列表。我们将介绍Qt Creator和Qt Designer接口，signal/slot(信号槽)机制，创建包含自定义的signal/slot的自定义widget并将其集成到您的应用程序中。  

您将使用新的C++14语义实现一个todo应用程序：lambdas、自动变量和for循环。这些概念每一个都将被详细介绍，并将在整本书中使用。  

在本章的最后，您将能够使用Qt Widget和新的C++语义创建一个具有灵活UI的桌面应用程序。  

在本章中我们将介绍一下主题：  
- Qt项目的基本结构
- Qt Designer接口
- UI基础
- signal和slot
- 自定义`QWidget`
- C++14 lambda, auto, for each

## 创建一个项目  

该做的第一件事就是开启Qt Creator。  

在Qt Creator中，你可以通过**File/New File or Project/Application/Qt Widgets Application/Choose**。  

该向导将会引导您完成四个步骤：  
1. **位置**：你必须选择一个项目名称和一个位置。
2. **套件**：你项目的目标平台（桌面、安卓等）。
3. **细节**：生成的类的基类信息和名称。
4. **概要**：允许你将项目设置成子项目并自动将其添加到版本控制系统。

即使所以的设置都使用默认的值，至少请设置一个有用的项目名称，如“todo”或“TodoApp”。如果你想把他命名成“Untitled”或“Hello world”，我们也不会怪你。  

一旦完成后，Qt Creator将生成几个文件，你可以在Project结构层次视图中看到这些文件：  

![](/assets/images/mastering-qt-ch-1/1.jpg)  

`.pro`文件是Qt的项目配置文件。由于Qt添加了特定的文件格式和C++关键字，因此会执行中间构建步骤，解析所有文件以生成最终文件。这个过程通过`qmake`——一个来自Qt SDK的可执行程序——来完成。它还会为你的项目生成最终的生成文件。  

一个基本的`.pro`文件通常包含：  

- 使用的Qt模块（`core`，`gui`等）
- 目标名称（`todo`，`todo.exe`等）
- 项目模板(`app`，`lib`等)
- 源文件，头文件和格式等

Qt和C++14有一些很棒额功能。这本书将在所有的项目中展示它们。对于`GCC`和`CLANG`编译器，你必须在`.pro`文件中添加`CONFIG += c++14`以在Qt工程中开启C++14，如下面代码所示：  

``` cpp
QT       += core gui 
CONFIG   += c++14 
 
greaterThan(QT_MAJOR_VERSION, 4): QT += widgets 
 
TARGET = todo 
TEMPLATE = app 
 
SOURCES += main.cpp \ 
           MainWindow.cpp 
 
HEADERS  += MainWindow.h \ 
 
FORMS    += MainWindow.ui \ 
```

`MainWindow.h`和`MainWindow.cpp`文件是`MainWindow`类的头文件/源文件。这些文件含有向导生成的默认的GUI。  

`MainWindow.ui`是你的XML格式的UI设计文件。它能够更简单地在Qt Designer中编辑。这个工具是一个所见即所得的编辑器，这可以帮助你添加和调整图形组件（widgets）。  

这是`main.cpp`文件，它具有众所周知的功能：  

``` cpp
#include "MainWindow.h" 
#include <QApplication> 
 
int main(int argc, char *argv[]) 
{ 
    QApplication a(argc, argv); 
    MainWindow w; 
    w.show(); 
 
    return a.exec(); 
} 
```

像往常一样，`main.cpp`文件包含程序入口点。默认情况下，它将执行两个操作：  

- 实例化并显示你的主窗口
- 实例化一个`QApplication`并执行阻塞的主事件循环

这是Qt Creator的左下角工具栏：  

![](/assets/images/mastering-qt-ch-1/2.jpg)  

使用它在Debug模式下构建并执行你的`todo`应用程序：  

1. 检查该项目是否处于Debug构建模式
2. 使用锤子图标按钮构建你的项目
3. 使用带有蓝色小虫子的开始按钮开始调试

你会发现一个漂亮的空窗口。在解释完这个`MainWindow`是如何构建的之后，我们将解决这个问题。  

![一个空的MainWindow截图](/assets/images/mastering-qt-ch-1/3.jpg)  


    Qt tip
    1. 按**ctrl + B**(Windows/Linux)或**Command + B**(Mac)来构建你的工程
    2. 按**F5**(Windows/Linux)或**Command + R**(Mac)来在Debug模式下运行你的项目

## MainWindow结构

这个生成的类是Qt框架使用的完美但简单的例子；我们将一起解剖它。像之前提到的一样，`MainWindow.ui`文件描述了你的UI设计，`MainWindow.h`/`MainWindow.cpp`是C++对象，你可以使用代码操作UI。  

看一下头文件`MainWindow.h`是非常重要的。咱们的`MainWindow`对象继承着Qt的`QMainWindow`类：  

``` cpp
#include <QMainWindow> 
 
namespace Ui { 
class MainWindow; 
} 
 
class MainWindow : public QMainWindow 
{ 
    Q_OBJECT 
 
public: 
    explicit MainWindow(QWidget *parent = 0); 
    ~MainWindow(); 
private: 
    Ui::MainWindow *ui; 
}; 
```

因为我们的类继承自`QMainWindow`类，所有我们在头文件的顶部添加了相应的include。第二部分是`UI::MainWindow`的前向声明，因为我们只需要声明一个指针。  

`Q_OBJECT`对于非Qt开发人员来说，可能看起来有点奇怪。这个宏指令允许这个类定义它自己的signal/slot和更多的全局的元对象系统。这些功能将在本章后面介绍。  

这个类定义了一个共有的构造函数和析构函数。后者很常见。但构造函数需要一个参数parent。这个参数一个默认为空的`QWidget`指针。  

`Qwidget`是一个UI组件。它可以是一个label，一个textbox，一个button等等。如果你在你的window、layout和其他的UI组件之间定义了父子关系，那么你的应用程序的内存管理将非常简单。确实，在这种情况下，删除掉父物体就足够了，应为他的析构函数也会将子物体一并删除，而子物体又会删除掉子物体。  

我们的`MainWindow`扩展了来自Qt框架的`QMainWindow`。我们在私有字段中有一个`ui`成员变量。它的类型是`Ui::MainWindow`的指针，它在由Qt生成的`ui_MainWindow.h`中定义。它是UI设计文件`MainWindow.ui`的C++转录。该ui成员变量允许你从C++与你的UI组件（`QLabel`，`QPushButton`等）进行交互，如下图所示：  

![](/assets/images/mastering-qt-ch-1/4.jpeg)  

    C++ tip
    如果你的类只使用类类型的引用或者指针，则可以通过使用前向申明来避免包含头文件。这将大大减少编译时间。

现在头文件的部分讲完了，我们可以来讲讲源文件`MainWindow.cpp`。  

在下面的代码片段中，第一个include的是我们的头文件。第二个是生成的类`Ui::MainWindow`所需要的include。这是必须的，因为我们只在头文件中使用前向声明。  

``` cpp
#include "MainWindow.h" 
#include "ui_MainWindow.h" 
 
MainWindow::MainWindow(QWidget *parent) : 
    QMainWindow(parent), 
    ui(new Ui::MainWindow) 
{ 
    ui->setupUi(this); 
}
```

在很多情况下，Qt使用初始化器生成了一段代码。参数parent用于调用超类的构造方法`QMainWindow`。我们的私有成员变量`ui`也是在现在初始化的。  

现在，`ui`已经初始化了，我们必须调用`setupUi`函数来初始化`MainWindow.ui`设计文件中所使用的widget。  

像我们在构造方法中初始化指针一样，它必须在析构函数中被清除：  

``` cpp
MainWindow::~MainWindow() 
{ 
    delete ui; 
} 
```

## Qt Designer

Qt Designer是开发Qt应用程序的主要工具。这个所见即所得的编辑器将帮助你更轻松地开发你的GUI。如果你在MainWindow.ui文件的编辑模式跟设计模式之间切换，你将看到真正的XML内容和设计器：  
![](/assets/images/mastering-qt-ch-1/5.jpeg)  

设计器显示了以下几个部分：  

- 表单编辑器：这是表单的可视化显示（现在是空的）
- Widget框：这包含了所有能用于你的表单的widget
- 对象检查器：这将你的表单以分层树的形式表示
- 属性编辑器：这列举了所选widget的属性
- 对象编辑器/信号槽编辑器：这处理你对象之间的连接

是时候开始润色这个空白窗口了。让我们从**显示widgets**部分拉一个**Label** widget到表单上。你可以在属性编辑器中修改name和text属性。  

由于我们正在制作`todo`应用程序，所以我们建议使用一下属性：  

- objectName : statusLabel
- text : Status: 0 todo/0 done

该label稍后将显示`todo`任务的计数与已完成的任务计数。好了，保存，编译并运行你的应用程序。你现在应该在窗口中看到你的新Label了。  

你现在可以添加一个带有以下属性的push button了：  

- objectName : addTaskbutton
- text : Add task

你应该会得到一个跟下面接近的结果：  

![](/assets/images/mastering-qt-ch-1/6.jpeg)  

    Qt tip
    你可以通过双击，直接在窗体上编辑widget的text属性！

## 信号槽

Qt框架通过三个概念引入了一个弹性的信息交换机制：信号、槽和连接：

- 信号是由对象发送的信息
- 槽是一个函数，当信号被触发时将调用这个函数
- 连接函数是指定哪个信号连接到哪一个slot

Qt已经为它的类提供了信号槽，你可以在你的应用程序中使用它们。例如，`QPushButton`拥有一个`signal clicked()`，当用户点击按钮时会触发这个信号。`QApplication`类拥有一个`slot quit()`函数，当你想要终止你的应用程序时你可以调用它。  

这是为什么你将会喜欢上Qt的信号槽：

- 槽函数仍是一个普通的函数，你可以自己手动调用它
- 单个信号可以被连接到不同的槽
- 单个槽可以被不同的已连接的信号调用
- 可以在不同对象的信号和槽之间建立连接，甚至可以在不同线程的对象之间建立连接！

请记住，为了确保信号能够连接到槽，它们方法的签名必须匹配。参数的数量，顺序和类型必须相同。请记住，信号和槽永远没有返回值。  

这是一个Qt连接的语法：  

``` cpp
connect(sender, &Sender::signalName, 
    receiver, &Receiver::slotName); 
```

我们可以做的第一个使用这个奇妙机制的实验就是将现有的信号跟现有的槽连接起来。我们将这个连接调用添加到`MainWindow`构造函数中：

``` cpp
MainWindow::MainWindow(QWidget *parent) : 
    QMainWindow(parent), 
    ui(new Ui::MainWindow) 
{ 
    ui->setupUi(this); 
    connect(ui->addTaskButton, &QPushButton::clicked, 
    QApplication::instance(), &QApplication::quit); 
} 
```

让我们来分析一下一个连接是怎么工作的：  

- `sender`：这是将发送信号的对象。在我们的例子中，它是从UI designer中添加的名为`addTaskButton`的`QPushButton`
- `&Sender::signalName`：这是指向成员信号函数的指针。这里我们想要在点击信号被触发时执行一些事情。  
- `receiver`：这是将要接收和处理信号的对象。在我们的案例中，它是在`main.cpp`中创建的对象`QApplication`。  
- `&Receiver::slotName`：这是指向其中一个接受者的成员槽函数的指针。在本例中，我们将使用内建在`QApplication`中的`quit()`槽函数，它将终止应用程序。  

你可以编译和运行这个简短的例子。当你点击在`MainWindow`上的`addTaskbutton`时应用程序将会被终止。  

    Qt tip
    你可以将一个信号和另一个信号连接在一起。当第一个信号被触发时，第二个信号将被发射出去。

你已经懂得了如何将一个信号与现有的槽函数连接起来，现在让我们来看看如何在我们的`MainWindow`类中声明和实现自定义的`addTask()`槽函数。当用户点击`ui->addTaskButton`时，这个槽函数将被调用。  

这是更新后的`MainWindow.h`：  

``` cpp
class MainWindow : public QMainWindow 
{ 
    Q_OBJECT 
 
public: 
    explicit MainWindow(QWidget *parent = 0); 
    ~MainWindow(); 
 
public slots: 
    void addTask(); 
 
private: 
    Ui::MainWindow *ui; 
}; 
```

Qt使用特定的`slot`关键字来标识槽函数。由于槽函数还是一个函数，所以你可以随时根据需要调整其可见性（`public`，`protected`，`private`）。

在`MainWindow.cpp`中添加此槽函数的实现：  

``` cpp
void MainWindow::addTask() 
{ 
    qDebug() << "User clicked on the button!"; 
} 
```

Qt提供了一个使用`QDebug`类显示调试信息的有效方式。获取`QDebug`对象的简单方法就是调用`QDebug()`函数。然后，你就可以使用流操作符发送你的调试信息。  

像这样更新文件的头部：  

``` cpp
#include <QDebug> 
 
MainWindow::MainWindow(QWidget *parent) : 
    QMainWindow(parent), 
    ui(new Ui::MainWindow) 
{ 
    ui->setupUi(this); 
    connect(ui->addTaskButton, &QPushButton::clicked, 
    this, &MainWindow::addTask); 
} 
```

由于我们现在要在外部的槽函数中使用`QDebug()`，所以我们必须要include `<QDebug>`。更新后的连接现在讲调用我们的自定义槽函数而不是退出应用程序。  

编译和运行应用程序。如果你点击按钮，你将在Qt Create中的`Application Output`选项卡中看到你的调试信息。

## 自定义QWidget  

现在我们必须创建一个保存我们的数据（任务名称和完成状态）的`Task`类。这个类将它的表单文件从`MainWindow`中分离。Qt Creator提供一个自动化的工具用来生成一个基类和相关表单。  

点击**File/New File or Project/Qt/Qt Designer Form Class**。这有几种表单模板；你将认识到Qt Create在我们开始`todo`项目时为我们创建的主窗口。选择**Widget**并命名类为`Task`，然后单击**Next**。下面是Qt Creator将要做的事情：  

1. 创建一个`Task.h`文件和`Task.cpp`文件。
2. 创建关联的`Task.ui`并将其关联到`Task.ui`。
3. 将三个新文件添加到`todo.pro`，以便于它们可以被编译。

完成，然后，瞧，`Task`类已经准备好被填充了。首先，我们将进入到`Task.ui`中。我们从拖拽一个`Check Box`（将`objectName`改为`checkBox`）和`Push Button`（`objectName` = `removeButton`）开始。  

![](/assets/images/mastering-qt-ch-1/7.jpg)  

除非你有一双精确到像素的眼睛，不然你的物品经常是不会有很好的对齐的。你需要指出你的widget将如何被布置，及当窗口发生几何改变时（例如，用户改变窗户大小时）该如何反应。对此，Qt有几个默认布局类：  

- `Vertical Layout`：在这种布局中，widget是垂直堆叠的
- `Horizontal Layout`：在这种布局中，widget是水平堆叠的
- `Grid Layout`：在这种布局中，widget被安排在可以被细分为更小的单元格网络中
- `Form Layout`：在这种布局中，widget就像网页表单，标签或者输入框一样排列

每一种布局都将尝试去限制所有的widget占据一样的面积。它要么改变widget的形状，要么增加额外的空白，这取决于每一种widget的约束。一个`Check Box`不能被拉长，而`Push Button`可以。  

在我们的`Task`对象中，我们希望它们是水平堆叠的。在`Form Editor`标签栏中，右键点击窗口并选择**Lay out/Lay out Horizontally**。每当你添加一个新的widget到这个布局中，它将会是水平排列的。  

现在在刚才的`checkBox`对象后面再添加一个`Push Button（objectName = editButton）`。  

`Form Editor`窗口提供了一个关于你的UI将如何渲染的实时预览。如果你现在拉伸窗口，你讲注意到每一个widget将对这个事件产生怎样的反应。当改变水平方向的大小时，你可以注意到按钮被拉伸了。这看起来很糟糕。我们需要某些东西来“暗示”布局，这些按钮不应该被拉伸。进入到`Spacer`widget，并在widget box中将`Horizontal Spacer`拉出来，并放置在`checkBox`物体后面：  

![](/assets/images/mastering-qt-ch-1/8.jpg)  

spacer是特殊的widget，它尝试推动（水平或竖直方向上）相邻的widget，以迫使他们占用尽可能小的空间。`editButton`和`removeButton`对象现在只占用其文本的空间，并在调整大小时将其推到窗口的边缘。  

你可以在表单中添加任何类型的子布局（垂直，水平，网格，表单），并使用widget，spacer和布局组合成一个复杂的应用程序。这些工具的目标是设计一个外观漂亮的桌面应用程序，并在窗口发生几何变换时做出正确的反应。  

Designer部分已经完成了，我们可以切换到`Task`的源代码了。由于我们创建了一个Qt Designer表单类，`Task`已经与其UI紧密相关了。我们将利用他将我们的模型储存在一个地方。当我们创建一个`Task`对象时，它必须有一个名字：  

``` cpp
#ifndef TASK_H 
#define TASK_H 
 
#include <QWidget> 
#include <QString> 
 
namespace Ui { 
class Task; 
} 
 
class Task : public QWidget 
{ 
    Q_OBJECT 
 
public: 
    explicit Task(const QString& name, QWidget *parent = 0); 
    ~Task(); 
 
    void setName(const QString& name); 
    QString name() const; 
    bool isCompleted() const; 
     
private: 
    Ui::Task *ui; 
}; 
 
#endif // TASK_H 
```

构造函数指定了一个名字，正如你所见的，并没有一个专门的字段用来储存对象的任何状态。所有的这些工作将在表单部分完成。我们还添加了一些与表单交互的setter和getter。最好是将模型和UI完全分开，但我们的例子十分简单，可以将它们合并。而且，`Task`实现的细节对于外部是隐藏的，并且可以在以后重构。这是`Task.cpp`文件的内容：  

``` cpp
#include "Task.h" 
#include "ui_Task.h" 
 
Task::Task(const QString& name, QWidget *parent) : 
        QWidget(parent), 
        ui(new Ui::Task) 
{ 
    ui->setupUi(this); 
    setName(name); 
} 
 
Task::~Task() 
{ 
    delete ui; 
} 
 
void Task::setName(const QString& name) 
{ 
    ui->checkbox->setText(name); 
} 
 
QString Task::name() const 
{ 
    return ui->checkbox->text(); 
} 
 
bool Task::isCompleted() const 
{ 
   return ui->checkbox->isChecked(); 
} 
```

我们将信息储存在我们的`ui->checkbox`，`name()`和`isCompleted()`中，getter从`ui->checkbox`中获取它们的数据。  

## 添加一个任务

现在我们将重新排列`MainWindow`布局，以显示我们的待办任务。现在，没有widget可以显示我们的任务。打开`MainWindow.ui`，编辑它以得到以下的结果：  

![](/assets/images/mastering-qt-ch-1/9.jpg)  

如果详细说明，我们有：  

- 一个包含`toolbarLayout`文件和`tasksLayout`文件的名为`centralWidget`的垂直布局
- 一个垂直的spacer将这些布局推向顶部，迫使它们占据尽可能小的空间
- 我们去掉了`menuBar`，`mainToolBar`和`statusBar`。Qt Creator自动创建了它们，而我们根本不需要它们。你可以从它们的名字猜出它们的作用。 

不要忘记通过在**Object Inspector**窗口中选中**MainWindow**并编辑**Qwidget/windowTitle**属性来将`MainWindow`的标题重命名为`Todo`。你的应用程序应被正确命名。  

    Qt tip
    在Designer模式下按Shift+F4可在表单编辑器和源文件之间切换。

现在，MainWindow的UI已经准备好迎接任务了，让我们切换到代码部分。应用程序必须跟踪新的任务。在`MainWindow.h`文件中添加一下内容：  

``` cpp
#include <QVector> 
 
#include "Task.h" 
 
class MainWindow : public QMainWindow 
{ 
    // MAINWINDOW_H 
 
public slots: 
    void addTask(); 
 
private: 
    Ui::MainWindow *ui; 
    QVector<Task*> mTasks; 
}; 
```

`QVector`是Qt容器类，它提供了一个动态数组，这相当于`std::vector`。作为基本规则，STL容器相对于Qt容器更加具有可定制性但可能会遗漏一些功能。如果你使用C++11智能指针，你应该更偏爱`std`容器，但我们将在稍后讨论。在`QVector`的Qt文档中，你可能会偶然发现下面的语句：“**对于大多数用途来说，`QList`是适合使用的类**”。Qt社区对此有一些辩论：  

- 你是否经常需要在数组的开头或中间插入大于指针的对象？请使用`QList`类
- 需要连续的内存分配？较少的CPU和内存开销？请使用`QVector`类

每次我们想要将新的`Task`对象添加到`mTask`函数时，现在都会调用已添加的槽函数`addTask()`。  

每次单击`addTaskButton`时，让我们填我们的`QVector`任务。首先，我们在`MainWindow.cpp`文件中连接`clicced()`信号：  

``` cpp
MainWindow::MainWindow(QWidget *parent) : 
    QMainWindow(parent), 
    ui(new Ui::MainWindow), 
    mTasks() 
{ 
    ui->setupUi(this); 
    connect(ui->addTaskButton, &QPushButton::clicked,  
    this, &MainWindow::addTask); 
}; 
```

    C++ tip
    作为最佳实践，尝试在初始化列表中初始化成员变量，并遵守变量声明的顺序。你的代码将运行得更快，将避免不必要的变量副本。

`addTask()`函数的函数体将会是这样：  

``` cpp
void MainWindow::addTask() 
{ 
        qDebug() << "Adding new task"; 
        Task* task = new Task("Untitled task"); 
        mTasks.append(task); 
        ui->tasksLayout->addWidget(task); 
} 
```

我们创建一个新的任务，并将其添加到`mTask`vector中。因为任务是一个`QWidget`，我们也直接将其添加到`tasksLayout`。这里需要注意的一点是，我们从来没有管理过这个新任务的内存。`delete task`指令在哪呢？这是我们在本章前面提到的Qt框架的一个重要特性；`QObject`类的父类会自动处理对象的销毁。  

在我们的案例中，`ui->tasksLayout->addWidget(task)`的调用有一个有趣的副作用；任务的所有权被转移到了`tasksLayout`。在`Task`构造函数中定义的`QObject * parent`现在是`tasksLayout`，并且当`tasksLayout`通过迭代遍历子元素并调用其析构函数释放自己的内存时，也将调用`Task`的析构函数。  

这将发生在精确的时刻：  

``` cpp
MainWindow::~MainWindow() 
{ 
    delete ui; 
} 
```

当`MainWindow`被释放时（记住，它是一个在`main.cpp`中分配的堆栈变量），它将调用`delete ui`，这将反过来降低整个`QObject`的层次结构。此功能具有有趣的后果。首先，如果你在应用程序中使用`QObject`父子模型，你将拥有更少的内存来管理。其次，它可能会碰到一些新的C++11语义，特别是智能指针。我们将在后面的章节中讨论。  

## 使用QDialog











