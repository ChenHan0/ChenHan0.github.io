---
title: "DirectX 11 教程二"
categories:
    - DirectX  
tag:
    - DirectX
    - Graphics
last_modified_at: 2018-04-09T23:56:00+08:00
---
# 创建一个框架和窗口

在开始使用DirectX 11进行编码之前，我建议构建一个简单的代码框架。这个框架将处理基本的窗口功能，并提供一种简单的方式来以有组织的和可读的方式扩展代码。由于这些教程的目的只是为了尝试DirectX 11的不同功能，我们将故意保持框架尽可能简洁。  

<!-- more -->  

## <u>框架</u>

框架工作将从四个方面开始。 它将有一个WinMain函数来处理应用程序的入口点。它还有一个system类，封装了将从WinMain函数中调用的整个应用程序。在system类中，我们将有一个用于处理用户输入的input类和一个用于处理DirectX图形代码的graphics类。以下是框架示意图：  

![](/assets/images/DirectX11/2-1.gif)  

现在我们看看如何构建框架，让我们先看看main.cpp文件中的WinMain函数。  

## <u>WinMain</u>

``` cpp
////////////////////////////////////////////////////////////////////////////////
// Filename: main.cpp
////////////////////////////////////////////////////////////////////////////////
#include "systemclass.h"


int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PSTR pScmdline, int iCmdshow)
{
    SystemClass* System;
    bool result;
    
    
    // 创建system对象.
    System = new SystemClass;
    if(!System)
    {
        return 0;
    }

    // 初始化和启动system对象.
    result = System->Initialize();
    if(result)
    {
        System->Run();
    }

    // 关闭和释放system对象.
    System->Shutdown();
    delete System;
    System = 0;

    return 0;
}
```

正如你所看到的，我们使WinMain功能保持非常简洁。我们创建system类并初始化它。如果初始化没有问题，那么我们调用system类Run函数。Run函数将运行它自己的循环并执行所有的应用程序代码，直到完成。Run函数完成后，我们关闭system对象并清理system对象。所以我们保持它非常简洁，并将整个应用程序封装在system类中。现在我们来看看system类的头文件。  

## <u>Systemclass.h</u>

``` cpp
////////////////////////////////////////////////////////////////////////////////
// Filename: systemclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _SYSTEMCLASS_H_
#define _SYSTEMCLASS_H_
```

这里我们定义WIN32_LEAN_AND_MEAN。我们这样做是为了加速build过程，它通过排除一些较少使用的API来减小Win32头文件的大小。  

``` cpp
///////////////////////////////
// PRE-PROCESSING DIRECTIVES //
///////////////////////////////
#define WIN32_LEAN_AND_MEAN
```

Windows.h被包含在内，因此我们可以调用这些函数来创建/销毁窗口并能够使用其他有用的win32函数。  

``` cpp
///////////////////////
// MY CLASS INCLUDES //
///////////////////////
#include "inputclass.h"
#include "graphicsclass.h"
```

这个类的定义非常简单。我们看到在此处定义的WinMain中调用的Initialize，Shutdown和Run函数。还有一些私有函数将在这些函数中被调用。我们还在该类中放置了一个MessageHandler函数来处理将在运行时发送给应用程序的Windows系统消息。最后我们有几个私有变量m_Input和m_Graphics，它们将指向处理图形和输入的两个对象。

``` cpp
////////////////////////////////////////////////////////////////////////////////
// Class name: SystemClass
////////////////////////////////////////////////////////////////////////////////
class SystemClass
{
public:
    SystemClass();
    SystemClass(const SystemClass&);
    ~SystemClass();

    bool Initialize();
    void Shutdown();
    void Run();

    LRESULT CALLBACK MessageHandler(HWND, UINT, WPARAM, LPARAM);

private:
    bool Frame();
    void InitializeWindows(int&, int&);
    void ShutdownWindows();

private:
    LPCWSTR m_applicationName;
    HINSTANCE m_hinstance;
    HWND m_hwnd;

    InputClass* m_Input;
    GraphicsClass* m_Graphics;
};


/////////////////////////
// FUNCTION PROTOTYPES //
/////////////////////////
static LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);


/////////////
// GLOBALS //
/////////////
static SystemClass* ApplicationHandle = 0;

#endif
```

WndProc函数和ApplicationHandle指针也包含在这个类文件中，所以我们可以将windows系统消息传递到system类中的MessageHandler函数中。  

现在让我们看看system类的源文件：  

## <u>Systemclass.cpp</u>

``` cpp
////////////////////////////////////////////////////////////////////////////////
// Filename: systemclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "systemclass.h"
```

在类构造函数中，我将对象指针初始化为null。这很重要，因为如果这些对象的初始化失败，那么Shutdown函数会继续尝试清理这些对象。如果对象不为空，那么它会认为它们是有效的创建对象，并且需要清理它们。在你的应用程序中将所有指针和变量初始化为空也是一种好习惯。如果你不这样做，某些release版本将build失败。  

``` cpp
SystemClass::SystemClass()
{
    m_Input = 0;
    m_Graphics = 0;
}
```

在这里我创建一个空的拷贝构造函数和空的类析构函数。在这个类中，我不需要它们，但如果没有定义，一些编译器会为你生成它们，在这种情况下，我宁愿它们是空的。

你也会注意到我不会在类的析构函数中清理任何对象。我会在下面的Shutdown函数中清除所有对象。 原因是我不相信它会被调用。某些Windows函数(如ExitThread())不会调用析构函数而导致内存泄漏。  

``` cpp
SystemClass::SystemClass(const SystemClass& other)
{
}


SystemClass::~SystemClass()
{
}
```

下面的Initialize函数完成应用程序的所有设置。它首先调用InitializeWindows，它将创建我们的应用程序使用的窗口。它还创建和初始化应用程序用于处理用户输入和将图形渲染到屏幕的输入和图形对象。

``` cpp
bool SystemClass::Initialize()
{
    int screenWidth, screenHeight;
    bool result;


    // 在将变量发送到函数之前，把初始化屏幕的宽度和高度为零.
    screenWidth = 0;
    screenHeight = 0;

    // 初始化窗口API.
    InitializeWindows(screenWidth, screenHeight);

    // 创建 input对象. 该对象将用于处理读取用户的键盘输入.
    m_Input = new InputClass;
    if(!m_Input)
    {
        return false;
    }

    // 初始化input对象.
    m_Input->Initialize();

    // 创建graphics对象. 该对象将为此应用程序渲染所有图形.
    m_Graphics = new GraphicsClass;
    if(!m_Graphics)
    {
        return false;
    }

    // 初始化graphics对象.
    result = m_Graphics->Initialize(screenWidth, screenHeight, m_hwnd);
    if(!result)
    {
        return false;
    }
    
    return true;
}
```

Shutdown函数可以执行清理功能。 它关闭并释放与图形和输入对象相关的所有内容。它也会关闭窗口并清理与之关联的手柄。  

``` cpp
void SystemClass::Shutdown()
{
    // 释放graphics对象.
    if(m_Graphics)
    {
        m_Graphics->Shutdown();
        delete m_Graphics;
        m_Graphics = 0;
    }

    // 释放input对象.
    if(m_Input)
    {
        delete m_Input;
        m_Input = 0;
    }

    // 关闭窗口.
    ShutdownWindows();
    
    return;
}
```

Run函数是我们的应用程序将循环并执行所有应用程序处理直到我们决定退出的地方。应用程序处理在每个循环中都被调用的Frame函数中完成。这是一个重要的概念，因为现在我们的应用程序的其余部分必须写在它的里面。伪代码如下所示：  
while 没有完成
     检查Windows系统消息
     处理系统消息
     处理应用程序循环
     检查用户是否在帧处理期间退出

``` cpp
void SystemClass::Run()
{
    MSG msg;
    bool done, result;


    // 初始化message结构体.
    ZeroMemory(&msg, sizeof(MSG));
    
    // 循环，直到窗口或用户发出退出消息.
    done = false;
    while(!done)
    {
        // 处理windows消息.
        if(PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
        {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }

        // 如果窗口示意结束应用程序，则退出.
        if(msg.message == WM_QUIT)
        {
            done = true;
        }
        else
        {
            // 否则，执行帧处理.
            result = Frame();
            if(!result)
            {
                done = true;
            }
        }

    }

    return;

}
```

下面的Frame函数是我们应用程序的执行所有处理的地方。到目前为止，它非常简洁，我们检查输入对象以查看用户是否按下了escape键并想要退出。如果他们不想退出，那么我们调用图形对象来完成它的帧处理，这将处理该帧的图形。随着应用程序的增长，我们会在这里放置更多的代码。  

``` cpp
bool SystemClass::Frame()
{
    bool result;


    // 检查用户是否按下了escape键并想要退出应用程序.
    if(m_Input->IsKeyDown(VK_ESCAPE))
    {
        return false;
    }

    // 为graphics对象执行帧处理.
    result = m_Graphics->Frame();
    if(!result)
    {
        return false;
    }

    return true;
}
```

MessageHandler函数是我们接收Windows系统消息的地方。通过这种方式，我们可以监听我们感兴趣的某些信息。目前，我们只会读取是否按下了某个键或是否松开了某个键，并将该信息传递给input对象。所有其他信息我们将传回给Windows默认消息处理程序。  

``` cpp
LRESULT CALLBACK SystemClass::MessageHandler(HWND hwnd, UINT umsg, WPARAM wparam, LPARAM lparam)
{
    switch(umsg)
    {
        // 检查键盘上是否有按键被按下.
        case WM_KEYDOWN:
        {
            // 如果按下某个键，则将其发送到input对象，以便它可以记录该状态.
            m_Input->KeyDown((unsigned int)wparam);
            return 0;
        }

        // 检查键盘上是否有键被松开.
        case WM_KEYUP:
        {
            // 如果一个键被松开了，然后将其发送到输入对象，以便它可以取消该键的状态.
            m_Input->KeyUp((unsigned int)wparam);
            return 0;
        }

        // 任何其他消息发送给默认消息处理程序，因为我们的应用程序不会使用它们.
        default:
        {
            return DefWindowProc(hwnd, umsg, wparam, lparam);
        }
    }
}
```

InitializeWindows函数是我们放置代码来构建我们将用于渲染的窗口的位置。它将screenWidth和screenHeight返回给调用函数，以便我们可以在整个应用程序中使用它们。我们使用一些默认设置创建窗口来初始化一个没有边框的普通黑色窗口。该函数将创建一个小窗口或全屏窗口，这将取决于名为FULL_SCREEN的全局变量。如果这设置为true，那么我们使屏幕覆盖整个用户的桌面窗口。如果它设置为false，我们只需在屏幕中间制作一个800x600的窗口。我将FULL_SCREEN全局变量放在graphicsclass.h文件的顶部，以防万一您想修改它。稍后你将明白为什么我将全局置于该文件中而不是此文件的头文件中。

``` cpp
void SystemClass::InitializeWindows(int& screenWidth, int& screenHeight)
{
    WNDCLASSEX wc;
    DEVMODE dmScreenSettings;
    int posX, posY;


    // 获取一个指向这个对象的外部指针.
    ApplicationHandle = this;

    // 获取此应用程序的实例.
    m_hinstance = GetModuleHandle(NULL);

    // 给应用程序一个名字.
    m_applicationName = L"Engine";

    // 使用默认设置设置Windows类.
    wc.style = CS_HREDRAW | CS_VREDRAW | CS_OWNDC;
    wc.lpfnWndProc = WndProc;
    wc.cbClsExtra = 0;
    wc.cbWndExtra = 0;
    wc.hInstance = m_hinstance;
    wc.hIcon = LoadIcon(NULL, IDI_WINLOGO);
    wc.hIconSm = wc.hIcon;
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.hbrBackground = (HBRUSH)GetStockObject(BLACK_BRUSH);
    wc.lpszMenuName = NULL;
    wc.lpszClassName = m_applicationName;
    wc.cbSize = sizeof(WNDCLASSEX);
    
    // 注册Windows类.
    RegisterClassEx(&wc);

    // 确定客户端桌面屏幕的分辨率.
    screenWidth  = GetSystemMetrics(SM_CXSCREEN);
    screenHeight = GetSystemMetrics(SM_CYSCREEN);

    // 根据是全屏模式还是窗口模式来设置屏幕设置.
    if(FULL_SCREEN)
    {
        // 如果是全屏将屏幕设置为用户桌面的最大尺寸和32位.
        memset(&dmScreenSettings, 0, sizeof(dmScreenSettings));
        dmScreenSettings.dmSize       = sizeof(dmScreenSettings);
        dmScreenSettings.dmPelsWidth  = (unsigned long)screenWidth;
        dmScreenSettings.dmPelsHeight = (unsigned long)screenHeight;
        dmScreenSettings.dmBitsPerPel = 32;         
        dmScreenSettings.dmFields     = DM_BITSPERPEL | DM_PELSWIDTH | DM_PELSHEIGHT;

        // 将显示设置更改为全屏.
        ChangeDisplaySettings(&dmScreenSettings, CDS_FULLSCREEN);

        // 将窗口的位置设置为左上角.
        posX = posY = 0;
    }
    else
    {
        // 如果是窗口化，则将其设置为800x600分辨率.
        screenWidth  = 800;
        screenHeight = 600;

        // 将窗口放在屏幕中间.
        posX = (GetSystemMetrics(SM_CXSCREEN) - screenWidth)  / 2;
        posY = (GetSystemMetrics(SM_CYSCREEN) - screenHeight) / 2;
    }

    // 用屏幕设置创建窗口并获取它的句柄.
    m_hwnd = CreateWindowEx(WS_EX_APPWINDOW, m_applicationName, m_applicationName, 
                WS_CLIPSIBLINGS | WS_CLIPCHILDREN | WS_POPUP,
                posX, posY, screenWidth, screenHeight, NULL, NULL, m_hinstance, NULL);

    // 将窗口放在屏幕上并将其设置为主要焦点.
    ShowWindow(m_hwnd, SW_SHOW);
    SetForegroundWindow(m_hwnd);
    SetFocus(m_hwnd);

    // 隐藏鼠标光标.
    ShowCursor(false);

    return;
}

```

ShutdownWindows就是这样做的。它将屏幕设置恢复到正常状态并释放窗口和与其关联的句柄。

``` cpp
void SystemClass::ShutdownWindows()
{
    // 显示鼠标光标.
    ShowCursor(true);

    // 如果退出全屏模式，请将显示设置改回.
    if(FULL_SCREEN)
    {
        ChangeDisplaySettings(NULL, 0);
    }

    // 删除窗口.
    DestroyWindow(m_hwnd);
    m_hwnd = NULL;

    // 删除应用程序实例.
    UnregisterClass(m_applicationName, m_hinstance);
    m_hinstance = NULL;

    // 释放指向该类的指针.
    ApplicationHandle = NULL;

    return;
}
```

WndProc函数是Windows发送消息的地方。当我们在上面的InitializeWindows函数中用wc.lpfnWndProc = WndProc初始化窗口类时，您会注意到我们告诉Windows它的名字。我将它包含在这个类文件中，因为我们通过将所有消息发送到SystemClass中定义的MessageHandler函数，将它直接绑定到system类中。这使我们可以将消息传递功能直接绑定到我们的类中并保持代码整洁。  

``` cpp
LRESULT CALLBACK WndProc(HWND hwnd, UINT umessage, WPARAM wparam, LPARAM lparam)
{
    switch(umessage)
    {
        // 检查窗口是否将被摧毁.
        case WM_DESTROY:
        {
            PostQuitMessage(0);
            return 0;
        }

        // 检查窗口是否将被关闭.
        case WM_CLOSE:
        {
            PostQuitMessage(0);     
            return 0;
        }

        // 所有其他消息传递给system类中的消息处理程序.
        default:
        {
            return ApplicationHandle->MessageHandler(hwnd, umessage, wparam, lparam);
        }
    }
}
```

## <u>Inputclass.h</u>

为了保持教程简单，我暂时使用了Windows输入，直到我在DirectInput上做了一个教程。输入类处理来自键盘的用户输入。这个类从SystemClass :: MessageHandler函数获得输入。输入对象将每个键的状态存储在键盘数组中。当被查询时，它会告诉调用函数是否按下了某个键。 这是头文件：  

``` cpp
////////////////////////////////////////////////////////////////////////////////
// Filename: inputclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _INPUTCLASS_H_
#define _INPUTCLASS_H_


////////////////////////////////////////////////////////////////////////////////
// Class name: InputClass
////////////////////////////////////////////////////////////////////////////////
class InputClass
{
public:
    InputClass();
    InputClass(const InputClass&);
    ~InputClass();

    void Initialize();

    void KeyDown(unsigned int);
    void KeyUp(unsigned int);

    bool IsKeyDown(unsigned int);

private:
    bool m_keys[256];
};

#endif
```

## <u>Inputclass.cpp</u>

``` cpp
////////////////////////////////////////////////////////////////////////////////
// Filename: inputclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "inputclass.h"


InputClass::InputClass()
{
}


InputClass::InputClass(const InputClass& other)
{
}


InputClass::~InputClass()
{
}


void InputClass::Initialize()
{
    int i;
    

    // Initialize all the keys to being released and not pressed.
    for(i=0; i<256; i++)
    {
        m_keys[i] = false;
    }

    return;
}


void InputClass::KeyDown(unsigned int input)
{
    // If a key is pressed then save that state in the key array.
    m_keys[input] = true;
    return;
}


void InputClass::KeyUp(unsigned int input)
{
    // If a key is released then clear that state in the key array.
    m_keys[input] = false;
    return;
}


bool InputClass::IsKeyDown(unsigned int key)
{
    // Return what state the key is in (pressed/not pressed).
    return m_keys[key];
}
```

## <u>Graphicsclass.h</u>

graphics类是由system类创建的另一个对象。此应用程序中的所有图形功能都将封装在此类中。我还将在这个文件中使用头文件来显示我们可能想要改变的所有图形相关的全局设置，例如全屏或窗口模式。目前这个类将是空的，但将来的教程将包含所有的图形对象。

``` cpp
////////////////////////////////////////////////////////////////////////////////
// Filename: graphicsclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _GRAPHICSCLASS_H_
#define _GRAPHICSCLASS_H_


//////////////
// INCLUDES //
//////////////
#include <windows.h>


/////////////
// GLOBALS //
/////////////
const bool FULL_SCREEN = false;
const bool VSYNC_ENABLED = true;
const float SCREEN_DEPTH = 1000.0f;
const float SCREEN_NEAR = 0.1f;
```

我们需要这四个全局变量来开始其他工作。  

``` cpp
////////////////////////////////////////////////////////////////////////////////
// Class name: GraphicsClass
////////////////////////////////////////////////////////////////////////////////
class GraphicsClass
{
public:
    GraphicsClass();
    GraphicsClass(const GraphicsClass&);
    ~GraphicsClass();

    bool Initialize(int, int, HWND);
    void Shutdown();
    bool Frame();

private:
    bool Render();

private:

};

#endif
```

## <u>Graphicsclass.cpp</u>

由于我们正在为本教程构建框架，因此我暂时将这个类保留为空。  

``` cpp
////////////////////////////////////////////////////////////////////////////////
// Filename: graphicsclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "graphicsclass.h"


GraphicsClass::GraphicsClass()
{
}


GraphicsClass::GraphicsClass(const GraphicsClass& other)
{
}


GraphicsClass::~GraphicsClass()
{
}


bool GraphicsClass::Initialize(int screenWidth, int screenHeight, HWND hwnd)
{

    return true;
}


void GraphicsClass::Shutdown()
{

    return;
}


bool GraphicsClass::Frame()
{

    return true;
}


bool GraphicsClass::Render()
{

    return true;
}
```

## <u>总结</u>

所以现在我们有一个框架和一个可以在屏幕上弹出的窗口。这个框架工作现在将成为所有未来教程的基础，因此理解这个框架是相当重要的。在进入下一个教程之前，请尝试进行练习以确保代码编译并可以正常工作。如果你不理解这个框架，你可以继续学习其他教程，并且在框架完成后将更容易理解它。