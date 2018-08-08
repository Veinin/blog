---
title: 开发笔记：游戏 UI 半自动化开发流程
date: 2018-07-31 20:10:00
categories: 开发笔记
tags: [半自动化，游戏UI开发，Unity, MVC]
---

最近着手试了下我们客户端 UI 开发，发现整个流程对于开发人员来说并不是很友好，我们游戏客户端代码基本使用 Lua 语言进行，引擎则采用 Unity。对于 UI 模块目前有一套基本的 MVC 模式的开发流程，但这个开发模式的边界并没有处理很好，导致开发过程中异常艰难。编写代码的人员有时也会很懵逼，因为实现的方式可以有多种多样。

在了解了整个旧有的开发流程后，我发现个问题，其中由UI编辑到代码编写，这个流程中，大部分过程都是重复行工作，而针对这一部分重复行工作通过一些小工具可以让UI开发流程实现半自动化。

所谓半自动化，无非就是，开发人员不需要编写基本的UI代码，基础UI代码可以自动生成，包括整个开发流程中使用的各个UI窗口的组件都可以自动生成代码。

另外，因为UI编写过程中进程资源修改、代码修改，我希望都可以在修改完后立刻可以看到效果，而不是重启游戏。

下面是我对于一个半自动化的UI开发流程整理笔记。

<!--more-->

## UI 编辑流程

我们的 UI 编辑是一个独立的项目工程，通常一个功能的 UI 编辑会由策划完成一部分工作，程序人员拿过来，按需求再整理 UI 资源、编写代码即可。

1. 编辑器项目结构

在UI编辑器中，其文件结构看起来是这样的：

```shell
UIProject
    -> Assert
        -> PublicPrefabs - 公共UI
        -> PulibcResources - 公共资源
        -> Texture - 纹理
        -> UI
            -> LoginWindow - 登陆窗口UI
                -> Prefabs
            -> EquipWinodw - 装备窗口UI
                -> Prefabs
            -> TeamWindow  - 组队窗口UI
                -> Prefabs
                    -> TeamCreateView - 队伍创建窗口
                    -> TeamMemberView - 队员窗口
                    -> TeamMemberItem - 队员窗口队伍信息
                    -> ...
                ->  UI资源1.png
                ->  UI资源2.png
                ->  ...
            -> ...
```

比如在一个组队功能UI里面，其他包含一个文件夹（TeamWindow），该文件夹下包含了多个UI用到的私有美术资源图片、纹理等。Window 文件夹下，有一个 Prefabs 文件夹，用来保存该功能所用到的所有子UI。

比如上面队伍窗口（TeamWindow）在UI设计中看起来是这样的：

![mvc](/images/mvc/mvc_ui_design.png)

其包含3个View文件，两个主要窗口，CreateView用来创建队伍窗口，MemberView用来显示队伍成员窗口，而MemberItem用来显示队伍成员窗口下的队员详细。

2.UI文件打包

编辑好的UI会统一打包成一个文件，方便使用代码做资源一次性加载。比如上面TeamWindow，打包后一个统一的资源包文件，包含了上面所示的队伍UI下面的所有子窗口。

## MVC 设计模式

原有的UI开发流程是使用 MVC 设计模式的，采用这个模式，如果能处理好，开发起来也是会很顺畅的。

对于 MVC，我们先用一张图来展示：

![mvc](/images/mvc/mvc_pattern.png)

在 Unity UI 开发中引入 MVC 设计模式，它看起来是这样的：

### Model

- 其只是用来保存数据用的，其不能访问 View 或 Controller。
- 它可以被 Controller 和 View 直接访问。

### View

- 其只是游戏中能用户看到的 UI 布局。
- View 可以处理 UI 点击逻辑、处理外面传进来的数据，可以访问和修改 Model。
- 在 Unity 中，每个 UI 资源的 Prefab 文件都会生成一个与之对应的 View 源代码文件。

### Conroller

- 负责控制单个模块内所有 View 资源的加载、显示、关闭、数据更新、分组、层级控制等。
- 其持有所有 View 和 Model 对象。
- 其销毁后，所有View 和 Model 都会自动销毁。

## MVC 实现

对于上面所示的MVC模式，我们结合相关MVC特性，用实际代码来展示，下面实例统一采用 Lua 语言，并使用面向对象概念来设计。

### 实现一个 Controller 接口

Controller 是一个UI功能的控制中心，其控制当前Window下的所有UI的加载、显示、隐藏，以及数据保存、清理、传递工作。
默认情况下所有子UI（UIView子类）都是会自动被其销毁的，其资源、数据都会在最后一个窗口关闭后自动清理掉。

Controller 接口完整生命周期示例图如下：

![mvc](/images/mvc/mvc_controller_life_cycle.png)

实现的 Controller 接口代码如下：

``` lua
local UIController = Class('UIController')

function UIController:Ctor(opts)
    self.isInit         = false
    self.isPersistent   = opts.isPersistent -- 是否持久化 model 数据，如果为 true，不会销毁 model 数据
    self.prefabsName    = opts.prefabsName  -- UI 打包后的资源名称
    self.assets         = {}                -- 已经加载的资源

    self.model          = {}                -- model 数据

    self.viewClasses    = {}                -- view 资源对应的子类，未实例化
    self.openViews      = {}                -- 已经打开的 view
    self.openViewCount  = {}                -- 已经打开的 view 数量
    self.lastOpenData   = nil               -- 上次打开 view 时的数据（未初始化时需要先保存打开时数据，初始化完成后再打开）
end

function UIController:IsInit()
    return self.isInit
end

local function OnLoad(self, assets)
    for _, asset in ipairs(assets) do
        self.assets[asset.name] = asset
    end

    self.isInit = true

    if not next(self.viewClasses) then
        self:OnInitView()
    end

    if self.lastOpenData then
        self.Open(unpack(self.lastOpenData))
        self.lastOpenData = nil
    else
        self:OnDefaultOpen()
    end
end

function UIController:Init()
    if self:IsInit() then
        return
    end

    LoadManager:LoadPrefab(self.prefabsName, function(assets)
        OnLoad(self, assets)
    end)
end

function UIController:OnInitView()

end

function UIController:RegisterView(class)
    assert(self.viewClasses[class.name] == nil)
    self.viewClasses[class.name] = class
end

local function CreateView(self, name)
    local class = self.viewClasses[name]
    if not class then
        return
    end

    local view = class.New()
    view.controller = self
    view.model      = self.model

    self.openViews[name] = view
    self.openViewCount = self.openViewCount + 1

    return view
end

local function GetView(self, name)
    local view = self.openViews[name]
    if not view then
        view = CreateView(self, name)
    end

    if not view then
        return
    end

    if view:IsState(UIConst.LIFESTATE.NONE) then
        UIHelper.InitView(self, view)
    end

    return view
end

function UIController:Open(name, ...)
    if not self:IsInit() then
        self.lastOpenData = {name, ...}
        self:Init()
        return
    end

    local view = GetView(self, name)
    if not view then
        return
    end

    if view:IsState(UIConst.LIFESTATE.SHOW) then
        view:Update(...)
    else
        view:Open(...)
    end
end

function UIController:Close(view)
    if not view then
        return
    end

    GameObject.Destroy(view.gameObject)

    self.openViews[name] = nil
    self.openViewCount = self.openViewCount - 1

    if self.openViewCount == 0 then
        self:Dispose()
    end
end

function UIController:Dispose()
    self.isInit = false
    self.assets = {}

    LoadManager:RemovePrefab(self.prefabsName)

    if not self.isPersistent then
        self.model = {}
    end

    self:OnDispose()
end

function UIController:OnDispose()

end

return UIController
```

根据上面生命周期实例图和代码实现，我们可以看到 Contoller 入口有两个，一个是外部打开指定 View，一个是外部关闭指定 View。

当外部需要打开指定 View 时，我们先检查当前 Controller 是否初始化完成，如果未初始化，则先临时保存打开时的数据，然后再走初始化 Controller 逻辑，加载 UI 资源。

当加载 UI 资源完成后，我们通过 View 的名称，调用 `GetView()` 函数获取一个 View 对象，如果 View 对象未创建，则走创建对象逻辑 `CreateView()`，并调用 View 初始化逻辑（下一节详解）。

获取到 View 对象后，如果 View 是打开状态则调用 `UIView.Update(...)` 函数更新，关闭状态则调用 `UIView:Open(...)` 函数打开。

UIController 对外暴露一个公共接口：

- `UIController.Open(name, ...)`，用于外部打开指定 View 。
- `UIController.RegisterView(class)`，用于注册一个 View 子类，通过其来实例化相应的 UI 窗口。
- `UIController.Close(view)`，用于关闭一个 View，这个接口用于 View 为了关闭自己而调用。

UIController 作为一个父类，需要子类重写以下接口：

- `UIController.OnInitView()`，初始化所有View，调用 `RegisterView` 注册指定 View。
- `UIController.OnDispose()`，销毁整个 Controller 的后续处理。

### 实现一个 View 接口

View 接口是所有UI资源窗口的父类，子类通过继承方式实现一个UI子窗口显示。
View 子类可以直接访问和修改 model 数据，另外，我希望在 View 中能间接调用 Controller 打开其他窗口。

View 接口完整生命周期示例图如下：

![mvc](/images/mvc/mvc_view_life_cycle.png)

根据上面生命周期实例图，我们可以看出整个 View 对象从初始化到关闭的完整生命周期流程。我们把 View 生命周期状态分为3部分：

- NONE，未初始化状态
- HIDE，隐藏状态
- SHOW，打开状态

根据上面的状态图，我们先实现 View 接口代码如下：

``` lua
local UIView = Class('UIView')

function UIView:Ctor()
    self.controller    = nil -- 持有 View 的 UIController 对象
    self.model         = nil -- View 持有的 Model 数据，与 UIController 共享

    self.gameObject    = nil   -- 引擎对象，该对象保存着UI资源
    self.isAutoDestroy = false -- 是否自动销毁，默认关闭时直接销毁对象，如果不自动销毁，关闭动作时隐藏
    self.layer         = UIConst.LAYER.MIDDLE   -- UI 层级（高、中、低）
    self.lifeState     = UIConst.LIFESTATE.NONE -- UI 生命周期状态
end

function UIView:IsState(state)
    return self.lifeState == state
end

function UIView:Init()
    self.gameObject:SetActive(false)
    self.lifeState = UIConst.LIFESTATE.HIDE
    self:OnInit()
end

function UIView:OpenView(name, ...)
    self.controller:Open(name, ...)
end

function UIView:Open(...)
    if not self:IsState(UIConst.LIFESTATE.HIDE) then
        return
    end

    self.lifeState = UIConst.LIFESTATE.SHOW
    self.gameObject:SetActive(true)
    self:OnShow(...)
end

function UIView:Update(...)
    if not self:IsState(UIConst.LIFESTATE.SHOW) then
        return
    end

    self:OnUpdate(...)
end

function UIView:Close()
    if not self:IsState(UIConst.LIFESTATE.SHOW) then
        return
    end

    self:OnHide()

    if self.isAutoDestroy then
        self.controller:Close(self)
    else
        self.gameObject:SetActive(false)
        self.lifeState = UIConst.LIFE_STATE.HIDE
    end
end

function UIView:OnInit()

end

function UIView:OnShow(...)

end

function UIView:OnUpdate(...)

end

function UIView:OnHide()

end

return UIView
```

View 初始化时，会从引擎底层加载 UI 编辑器指定的 UI 组件到 View 对象，这一步是 Contoller 在创建 View 时执行的，我们可以看到 `UIContoller.GetView()` 函数调用时，判断当前 View 对象如果是未初始化状态则调用帮助函数 `UIHelper.InitView()` 加载所有 UI 组件对象，其加载代码看起来时这样的：

``` lua
local UIHelper = {}

-- Lua 封装的 UI 组件对象
local UI_TYPES = {
    ['UIButton']        = require("game.ui.component.UIButton"),
    ['UILabel']         = require("game.ui.component.UILabel"),
    ['UIInputField']    = require("game.ui.component.UIInputField"),
    -- 其他 UI 组件...
}

-- 创建引擎 UI 对象
local function CreateCSObject()
    -- TODO 创建引擎UI对象，设置其层级关系
end

-- 初始化所以 UI 组件Lua对象
local function InitComponents(view)
    local components = view.gameObject.components

    for _, component in ipairs(components) do
        local uiLuaClass = UI_TYPES[component.typeName]
        if uiLuaClass then
            view[component.key] = uiLuaClass.New(component.value)
        end
    end
end

-- 初始化 UIView 对象
function UIHelper.InitView(view)
    view.gameObject = CreateCSObject(view)

    InitComponents(view)

    view:Init()
end

return UIHelper
```

在我们可以看到所有 UI 组件都在 Lua 层有一个简单的封装对象，我们在 UI 编辑器指定 UI 组件名称和类型后，在加载完 View 对象后，其 UI 组件就被赋予给了 View 对象。下面给出了一个按钮组件的简单封装实例：

``` lua
local UIButton = Class('UIButton', UIBehaviour)

function UIButton:SetText(name)
    self.core.text = name
end

function UIButton:SetEnable(isEnable)
    self.core:SetEnable(isEnable)
end

function UIButton:AddListener(callback, delayTime)
    delayTime = delayTime or 0.3
    self.event.onClickLua = callback
    self.event.clickDelayTime = delayTime
end

return UIButton
```

View 初始化完成后，每次打开时，都会传入打开时需要的数据，我们根据状态，把数据传给不同的处理函数。当前 UI 处于打开状态时调用 `UIView.Open()`；处于显示状态，则调用 `UIView.Update()`。最后可以使用 `UIView.Close()` 关闭 View。

UIView 对外暴露一个公共接口：

- `UIView.Init()`，初始化 View，初始化完成后调用一次 `UIView.OnInit()`。
- `UIView.Open(...)`，当 View 处于隐藏状态时，打开 View，最终会调用 `UIView.OnShow()` 处理。
- `UIView.Update(...)`，当 View 处于打开状态时，更新 View 数据，最终会调用 `UIView.OnUpdate()` 处理。
- `UIView.Close(...)`，用于关闭当前 UI，并调用 `UIView.OnHide()` 处理 View 隐藏后逻辑，其资源清理逻辑最终会交给 Controller 执行。
- `UIView.OpenView(name)`，用于打开其他 View，其打开逻辑会交给 Controller 执行。

UIView 作为一个父类，需要子类重写以下接口：

- `UIView.OnInit()`，初始化View，我们需要做一些 UI 组件事件注册逻辑处理，如果某个按钮点击后的逻辑，因为这些逻辑固定不变的，整个 View 生命周期只会调用过一次。
- `UIView.OnShow()`，如果 View 隐藏状态，在打开 View 时，会调用此函数，并传入打开 UI 时需要处理的数据。
- `UIView.OnUpdate()`，如果 View 打开状态，在打开 View 时，会调用此函数，并传入打开 UI 时需要处理的数据。
- `UIView.OnHide()`，当 View 被调用 UIView.Close(...) 时，在销毁前，会被调用，需要在此函数内处理关闭 View 时的逻辑。

## View 分组

在复杂的游戏 UI 功能开发时，常常遇到这些的问题：

- 同一个 UI 窗口内，我们希望某一组 UI 是互斥的，即同一时间内，Controller 只能打开该组内的某一个 View，如果同组有其他 View 打开则先关闭一打开的 View。
- 有一些 UI 分组，在某个指定 UI 关闭后，其分组内的任何 UI 只要打开都会自动关闭。

看到这些需求后，我们很快想到使用一颗树来管理这些 UI View 分组。其分组示例图如下：

![mvc](/images/mvc/mvc_view_group.png)

从图中我们可以看到，这是一个树形结构的 UI 分组，有一个根节点，子节点之间是分层的，每一层节点之间可以设置未互斥状态，即同一时间内，只能被打开一个。
有了这个树形结构，当某个节点被关闭后，我们可以很快获取到这个节点的所有子节点，并关闭它们。

于是可以设计一个 UI 分组代码：

``` lua
local Tree = require "game.foundation.Tree"

local UIGroupTree = Class('UIGroupTree', Tree)

function UIGroupTree:Ctor()
    self.isExclusion = true -- 子节点是否互斥，默认未互斥
end

function UIGroupTree:SetExclusion(isExclusion)
    self.isExclusion = isExclusion
end

function UIGroupTree:IsExclusion()
    return self.isExclusion
end

function UIGroupTree:AddChildUI(view)
    return self.NewChild(view.name)
end

return UIGroupTree
```

代码中树形结构 `Tree` 实现请参考外部 [链接](https://gist.github.com/Veinin/a9fbf71f6a82938100dff7fbff501495)。

然后我们拓展 Contoller 接口实现：

- 加入新的成员变量 group (UIGroupTree类型)
- 加入创建 UI 分组对象函数，`UIController.NewGroup(view)`，用于创建一个分组。
- 加入新的抽象函数 `UIController.OnInitGroup()`，子类如需要UI分组，则重写该函数，该函数在 Controller 初始化时被调用。
- 打开某个 UI 时检查 View 对象是否分组、分组是否互斥、是否有互斥 View 已打开，如果打开则关闭互斥 View。
- 关闭某个 UI 时检查 View 对象是否分组，当前节点是否存在子节点，子节点是否有打开情况，如果打开则关闭 View。

扩展 Controller 的代码实现如下：

``` lua
local UIGroupTree = require "game.ui.group.UIGroupTree"

local UIController:Ctor()
    -- 上面已实现代码省略...
end

local function OnLoad(self, assets)
    -- 上面已实现代码省略...

    if not next(self.viewClasses) then
        -- ...
        self:OnInitGroup()
    end

    -- ...
end

function UIController:OnInitGroup()
    -- 子类实现
end

-- 创建一个UI分组
function UIController:NewGroup(view)
    assert(self.group == nil, "不允许创建多个分组")
    self.group = UIGroupTree.New(view.name)
    return self.group
end

-- 检查互斥
local function CheckExclusion(self, view)
    if not self.group or self.openViewCount == 1 then
        return
    end

    local name = view.name
    local node = self.group:FindChild(name)
    if not node then
        return
    end

    local parent = node:GetParent()
    if #parent == 0 or not parent:IsExclusion() then
        return
    end

    local childName
    local childView
    for _, child in ipairs(parent) do
        childName = child.name
        childView = self.openViews[childName]
        if childName ~= name and childView then -- 如果打开则关闭
            self:Close(childView)
        end
    end
end

function UIController:Open(name, ...)
    -- ...

    -- 打开前检查分组互斥
    CheckExclusion(self, view)

    if view:IsState(UIConst.LIFESTATE.SHOW) then
        view:Update(...)
    else
        view:Open(...)
    end
end

-- 检查子节点关闭
local function CheckCloseChild(self, view)
    if not self.group then
        return
    end

    local name = view.name
    local node = self.group:FindChild(name)
    if not node then
        return
    end

    local childView
    node:IterativeChildren(function(child)
        childView = self.openViews[child.value]
        if childView then  -- 如果子节点打开则关闭
            self:Close(childView)
        end
    end)
end

function UIController:Close(view)
    -- ...

    if self.openViewCount == 0 then
        self:Dispose()
    else
        -- 还有打开的 View，检查是否其子节点，并关闭
        CheckCloseChild(self, view)
    end
end

return UIController
```

有了上面扩展实现的代码，我们就实现分组示例图中展示的分组结构：

``` lua
function MyTestController:OnInitGroup()
    local root = self:NewGroup(UIView1)
    root:AddChild(UIView2)
    root:AddChild(UIView3)

    local child = root:AddChild(UIView4)
    child:AddChild(UIView5)
    child:AddChild(UIView6)
end
```

## UI 开发半自动化

我们知道，游戏 UI 开发，很多固定的流程其实是可以省略的：

- UI 编辑完成后，开始写业务代码时，可以将所有基础代码全部生成，比如上面的 Controller 子类和每个窗口的 View 子类。
- 每个 View 类型，我们设计时其会对应 UI 编辑器的一个界面，且 UI 编辑器里面的所有组件，都是可以读取到的，在 Unity 中，我们保存为 Prefab 文件，而这个文件可以帮助我们设计一个自动生成代码的工具。
- 每次修改 UI 增加了新的组件后，我们希望原有的已经编辑的代码可以保留，在编辑器中刷新一下代码，能将新增加的控件基础代码直接加入到相应的代码文件里面。
- 每次修改完 UI 资源后，然后对应改完代码后，不希望在游戏中重启客户端才能看到效果，所以，我们加入了热更机制，任何代码都是可以热更新的，那么 UI 开发人员编码阶段将会非常方便。

综上考虑，因为我们项目很多人使用 **IntelliJ IDEA** 配合一个不错的 Lua 插件 **EmmyLua**， 相对来说使用 Lua 开发还是比较顺畅的。所以决定在 **IntelliJ IDEA** 平台上开发一个为项目定制的 UI 开发插件（当然现在一些主流的代码编辑器都支持插件编写，你可以很容易在其他编辑器中实现这些内容）。

在UI编辑完成后，开发人员，建立 UI 功能开发文件夹后，右键菜单点击生成代码，插件自动生成 UIController 和 UIView 对应子类。

比如一个组队功能包含以下 UI 文件:

``` shell
-> TeamWindow
    -> Prefabs
        -> CreateView.prefab
        -> MemberView.prefab
        -> MemberItem.prefab
```

生成的代码文件夹：

``` shell
-> team
    -> contoller
        -> TeamController.lua
    -> views
        -> CreateView.lua
        -> MemberView.lua
    -> protocol
        -> TeamProtocol.lua -- 协议文件，项目定制
```

下面是 Contoller 实例 `TeamController.lua` 文件内容，该文件自动生成了很多基础的代码，需要填写的地方只剩下分组（如需要）：

``` lua
local UIController = require "game.ui.core.UIController"
local CreateView = require "game.ui.team.views.CreateView"
local MemberView = require "game.ui.team.views.MemberView"

local TeamController = Class('TeamController', UIController)

function TeamController:OnInitView()
    self:RegisterView(CreateView)
    self:RegisterView(MemberView)
end

funciton TeamController:OnInitGroup()

end

return TeamController
```

下面是一个 View 实例 CreateView.lua 文件内容，其基本的组件代码全部通过编辑器UI文件自动生成，开发人员只需要在对应的 UI 组件填入相对于的业务逻辑即可：

``` lua
local UIView = require "game.ui.core.UIView"

local CreateView = Class('CreateView', UIView)

function CreateView:Ctor()

end

function CreateView:OnInit()
    self.closeButton:AddListener(function()
        self:Close()
    end)

    self.cofirmButton:AddListener(function()
        -- 开发人员实现创建队伍逻辑
    end)
end

function CreateView:OnShow(...)
    self.nameInputField:SetText()
    self.fightINputField:SetText()
    self.levelInputField:SetText()
end

function CreateView:OnUpdate(...)

end

function CreateView:OnHide()

end

return CreateView
```

为了对比，下面展示以下 `CreateView.lua` 对应 `CreateView.prefab` 文件结构：

![mvc](/images/mvc/create_view.png)

可以看出这个 UI 有5个需要处理的组件，两个按钮（关闭、确定），3个输入文本框（队伍名称、队伍战力、最低等级）。我们可以很容易从编辑的 prefab 文件中提取到里面的 UI 组件，然后通过插件直接生成代码文件。

有了上面的这些自动生成的代码，剩下的工作，其实就是填写一些基础的业务逻辑，那些重复性的工作工具已经可以帮你很好的完成了。

## 总结

本文阐述了一个半自动化工作的游戏 UI 编写流程，并给出了实现步骤。
游戏 UI 开发流程存在大部分重复工作，我们稍微花点时间，其实是可以把这些重复工作自动化的，剩下的工作，无非就是填写下业务逻辑代码。
可以预见的是，开发这么一套半自动化的工具其实不到一周时间，其实是可以很容易搞定的，并且后续的 UI 开发效率其实是提升了一个档次。