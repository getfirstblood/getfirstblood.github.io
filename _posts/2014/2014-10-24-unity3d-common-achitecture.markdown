# unity通用架构

标签（空格分隔）： unity3d

---

##通用ui部分：（mvc模块）
1. GUIManager部分：
```flow
st=>start: GUIManager
op1=>operation: Controller
op2=>operation: view
e=>end
st->op1->op2
```
MVC通用架构：
1. 该模块为表现层
2. 对外接口应该只有controller才算最解耦，具体流程为，当需要进行一个业务时，创建相应的controller，其负责管理对应的view与处理用户需求还有管理view之间的跳转的功能。
3. view层，负责用户输入时的界面表现，并将用户表现转给相应的controller
4. GUIManager管理所有的view，负责绘制界面，处理硬件的用户输入，根据逻辑需求转发给对应的view
5. UNITY中的应用：

> 当前使用的这个MVC区别：
> 由于使用unity3d引擎和ngui插件，guimanager的部分功能交给了ngui和unity3d来做，其中绘制部分交给了unity3d，用户源输入的管理分发交给了ngui。所以GUIManager这部分功能实现会比较乱。
> 具体实现为：
> view类该类的功能是从本地加载对应的prefab，初始化一个view对象。显示的时候将其加入GUIManager.并且设置自己的prefab显示
> unity3d引擎(GUIManager)记录了当前的view的结构（view字典）（对比通用架构，会发现那边应该会记录树结构）。绘制就有unity3d来完成（通用架构是通过view树结构调用绘制方法完成）。

如果能够通过xml来配置界面中的相应组件，然后通过解析代码生成对应的功能代码，这样就能够做到界面的可配置。
这部分功能的实现流程：
1、生成配置程序：生成配置界面xml
2、生成代码程序：根据相应的xml生成相应的界面代码（能够做到这一步是应为xml的对象代码都能理解，对于自定义的界面部分就无法实现自动生成对应代码的功能）
3、自定义的view部分，首先需要满足已有的框架基类，（比如cocoa中所有的view都有一个基类view），针对unity的平台，自定义的view可以先做成一个prefab然后通过resource.load来创建实例。(这部分其实原来通过生成代码完成的，现在因为是自定义的所以需要自己手动完成)
4、基类SnailComponent提供的功能有：
> panel的深度权值最大，及不同panel下的两个sprite的深度一个为1，一个为2，虽然1小于2但是如果panel1的深度大于那所有panel1里的sprite都要在panel2上层。
>ngui只是提供了panel此功能，snailcomponent抽象出来的一些组件也能够管理各个组件的深度，保证上级的组件深度权级最大。
>sample:snailbutton:SnailComponent，ISnailButton
>提供的组件之间的add，remove功能，管理子组件的功能
>提供了组件的外观控制功能
>提供了的事件监听转发的功能






##游戏逻辑模块（控制器与管理器）
1. 游戏对象控制器实现对象的某个特征
2. 业务控制器实现某一个功能
3. 管理器管理多个特征或多个功能的的一个中心

##手势识别架构
一个管理器（fingermanager)、一个控制器（fingercontroller）
fingermanager 负责初始化控制器，管理当前交互状态，分发交互事件。

fingercontroller 
负责交互捕获逻辑，将捕获的动作通知manager，由manager来分发。

##游戏状态架构层（状态机）
一个状态管理器GameStateManager、一个GameState基类
GameStateManager 负责
1. 保存所有gamestate实例
2. 当前状态引用
3. 初始默认的gamestatekey
4. 根据配置表初始化所有的gamestate
5. 每帧出发m_currentstate的update执行
6. 设置当前状态：启动、加载资源
7. 转换状态事件处理方法：将当前状态转为目标状态
GameState 负责
1. 保存资源id与对应资源列表的map
2. 资源id与对应资源场景的map
3. 默认使用的资源id
4. 保存允许的状态迁移映射关系
5. 当前状态是否含有地形
6. 转换状态方法，调用代理函数
7. 生命周期：
> start()
> load()
> init()

使用状态管理器能够很好的解耦游戏不同的游戏场景。使代码更容易阅读与维护。


##网路层架构
架构一：
UnitySocket.cs:
1. 连接服务器
2. 发送数据（由messageCode将message对象转换成stream发送）
3. 接收数据（将接收到的bytes转给SocketReceiveHandle）

messageCode.cs:
setMessageSteam：将message对象转换成stream。
GetMessageHead\GetMessageBody:将stream转换成message对象

SocketReceiveHandle.cs:
HandelMessage:将接收的数据通过messagecode转成message对象，并调用NetMessageDispather分发事件和messagebody。

EventDispatcher.cs:
1. 注册事件回调
2. 删除事件回调
3. 分发事件在相应update中调用

BaseMessageHandle.cs:
将基础类型写入stream

ClassMapping.cs:
类构建器，根据协议id找到对应的messagebody，创建messagebody对象返回

SocketMessageConfig.cs
1. 协议id与协议类对应解析
2. 获得协议数据类

架构二：
TCP/IP层:
1. gamesocket:管理接收发送模块，处理连接断开请求
2. gamereceiver:管理所有业务的回调，当接收到消息时，分发对应的回调。
3. gamesender:调用发送命令

协议层：
messagebase:定制返回消息的协议的组成，提供注册消息监听与移除监听的功能
commandbase:制定每条命令的协议的组成，提供回调监听的设置，提供回调数据的解析功能。

应用层：
1、继承messagebase、commandbase的消息命令
2、facade.cs外观模式：发送命令、监听消息、移除监听


