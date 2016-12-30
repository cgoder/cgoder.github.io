---
title: Android学习笔记
date: 2016-12-30 16:16:39
categories:
 - study
tags:
 - Android
---
# [Android学习笔记](https://developer.android.com)
---

## 1. [Android四大组件](https://developer.android.com/guide/components/fundamentals.html#Components)
+ Activity `<activity>`
+ Service 服务 `<service>`
+ Provider 内容提供程序 `<receiver>`
+ BroadcastReceiver 广播接收器 `<provider>`

<!-- more -->

### 1.1 [Activity](https://developer.android.com/guide/components/activities.html)
Activity 表示具有用户界面的单一屏幕。例如，电子邮件应用可能具有一个显示新电子邮件列表的 Activity、一个用于撰写电子邮件的 Activity 以及一个用于阅读电子邮件的 Activity。 尽管这些 Activity 通过协作在电子邮件应用中形成了一种紧密结合的用户体验，但每一个 Activity 都独立于其他 Activity 而存在。 因此，其他应用可以启动其中任何一个 Activity（如果电子邮件应用允许）。 例如，相机应用可以启动电子邮件应用内用于撰写新电子邮件的 Activity，以便用户共享图片。

Activity 是一个应用组件，用户可与其提供的屏幕进行交互，以执行拨打电话、拍摄照片、发送电子邮件或查看地图等操作。 每个 Activity 都会获得一个用于绘制其用户界面的窗口。窗口通常会充满屏幕，但也可小于屏幕并浮动在其他窗口之上。
### 1.1.1 [三种状态](https://developer.android.com/guide/components/activities.html#Lifecycle)
1. Start 继续
    此 Activity 位于屏幕前台并具有用户焦点。（有时也将此状态称作“运行中”。）
2. Pause 暂停
    另一个 Activity 位于屏幕前台并具有用户焦点，但此 Activity 仍可见。也就是说，另一个 Activity 显示在此 Activity 上方，并且该 Activity 部分透明或未覆盖整个屏幕。 暂停的 Activity 处于完全活动状态（Activity 对象保留在内存中，它保留了所有状态和成员信息，并与窗口管理器保持连接），但在内存极度不足的情况下，可能会被系统终止。
3. Stop 停止
    该 Activity 被另一个 Activity 完全遮盖（该 Activity 目前位于“后台”）。 已停止的 Activity 同样仍处于活动状态（Activity 对象保留在内存中，它保留了所有状态和成员信息，但未与窗口管理器连接）。 不过，它对用户不再可见，在他处需要内存时可能会被系统终止。
#### 1.1.2 [Activity生命周期](https://developer.android.com/guide/components/activities.html#ImplementingLifecycleCallbacks)
+ Activity 的**整个生命周期**发生在 `onCreate()` 调用与`onDestroy()`调用之间。您的 Activity 应在 `onCreate()` 中执行“全局”状态设置（例如定义布局），并释放 `onDestroy()`中的所有其余资源。例如，如果您的 Activity 有一个在后台运行的线程，用于从网络上下载数据，它可能会在 `onCreate()` 中创建该线程，然后在 `onDestroy()` 中停止该线程。
+ Activity 的**可见生命周期发**生在 `onStart()` 调用与 `onStop()` 调用之间。在这段时间，用户可以在屏幕上看到 Activity 并与其交互。 例如，当一个新 Activity 启动，并且此 Activity 不再可见时，系统会调用 `onStop()`。您可以在调用这两个方法之间保留向用户显示 Activity 所需的资源。 例如，您可以在 `onStart()` 中注册一个 BroadcastReceiver 以监控影响 UI 的变化，并在用户无法再看到您显示的内容时在 `onStop()` 中将其取消注册。在 Activity 的整个生命周期，当 Activity 在对用户可见和隐藏两种状态中交替变化时，系统可能会多次调用 `onStart()` 和 `onStop()`。
+ Activity 的**前台生命周期**发生在 `onResume()` 调用与 `onPause()` 调用之间。在这段时间，Activity 位于屏幕上的所有其他 Activity 之前，并具有用户输入焦点。 Activity 可频繁转入和转出前台 — 例如，当设备转入休眠状态或出现对话框时，系统会调用 `onPause()`。 由于此状态可能经常发生转变，因此这两个方法中应采用适度轻量级的代码，以避免因转变速度慢而让用户等待。
![](https://developer.android.com/images/activity_lifecycle.png)

### 1.2 [Service 服务](https://developer.android.com/guide/components/services.html)
服务是一种在后台运行的组件，用于执行长时间运行的操作或为远程进程执行作业。 服务不提供用户界面。 例如，当用户位于其他应用中时，服务可能在后台播放音乐或者通过网络获取数据，但不会阻断用户与 Activity 的交互。 诸如 Activity 等其他组件可以启动服务，让其运行或与其绑定以便与其进行交互。

Service 是一个可以在后台执行长时间运行操作而不提供用户界面的应用组件。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。 此外，组件可以绑定到服务，以与之进行交互，甚至是执行进程间通信 (IPC)。 例如，服务可以处理网络事务、播放音乐，执行文件 I/O 或与内容提供程序交互，而所有这一切均可在后台进行。
#### 1.2.1 两种形式
1. Start 启动
    当应用组件（如 Activity）通过调用 `startService()` 启动服务时，服务即处于“启动”状态。一旦启动，服务即可在后台无限期运行，即使启动服务的组件已被销毁也不受影响。 已启动的服务通常是执行单一操作，而且不会将结果返回给调用方。例如，它可能通过网络下载或上传文件。 操作完成后，服务会自行停止运行。
2. Bind 绑定
    当应用组件通过调用 `bindService()` 绑定到服务时，服务即处于“绑定”状态。绑定服务提供了一个客户端-服务器接口，允许组件与服务进行交互、发送请求、获取结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。 仅当与另一个应用组件绑定时，绑定服务才会运行。 多个组件可以同时绑定到该服务，但全部取消绑定后，该服务即会被销毁。

> **注意：** 
Service服务在其托管进程的主线程中运行，它既不创建自己的线程，也不在单独的进程中运行（除非另行指定）。 这意味着，如果服务将执行任何 CPU 密集型工作或阻止性操作（例如 MP3 播放或联网），则应在服务内创建新线程来完成这项工作。通过使用单独的线程，可以降低发生“应用无响应”(ANR) 错误的风险，而应用的主线程仍可继续专注于运行用户与 Activity 之间的交互。

> **如何决定使用服务还是线程？**
简单地说，服务是一种即使用户未与应用交互也可在后台运行的组件。 因此，您应仅在必要时才创建服务。 如需在主线程外部执行工作，不过只是在用户正在与应用交互时才有此需要，则应创建新线程而非服务。 例如，如果您只是想在 Activity 运行的同时播放一些音乐，则可在 onCreate() 中创建线程，在 onStart() 中启动线程，然后在 onStop() 中停止线程。您还可以考虑使用 AsyncTask 或 HandlerThread，而非传统的 Thread 类。如需了解有关线程的详细信息，请参阅进程和线程文档。 请记住，如果您确实要使用服务，则默认情况下，它仍会在应用的主线程中运行，因此，如果服务执行的是密集型或阻止性操作，则您仍应在服务内创建新线程。

#### 1.2.2 [Service生命周期](https://developer.android.com/guide/components/services.html#Lifecycle)
+ 服务的**整个生命周期**从调用 `onCreate()` 开始起，到 `onDestroy()` 返回时结束。与 Activity 类似，服务也在 `onCreate()` 中完成初始设置，并在 `onDestroy()` 中释放所有剩余资源。例如，音乐播放服务可以在 `onCreate()` 中创建用于播放音乐的线程，然后在 `onDestroy()` 中停止该线程。
无论服务是通过 `startService()` 还是 `bindService()` 创建，都会为所有服务调用 `onCreate()` 和 `onDestroy()` 方法。
+ 服务的**有效生命周期**从调用 `onStartCommand()` 或 `onBind()` 方法开始。每种方法均有 {Intent 对象，该对象分别传递到 `startService()` 或 `bindService()`。 对于启动服务，有效生命周期与整个生命周期同时结束（即便是在 `onStartCommand()` 返回之后，服务仍然处于活动状态）。对于绑定服务，有效生命周期在 `onUnbind(`) 返回时结束。
![](https://developer.android.com/images/service_lifecycle.png)

#### 1.2.3 [bindService 绑定服务](https://developer.android.com/guide/components/bound-services.html)
绑定服务是客户端-服务器接口中的服务器。绑定服务可让组件（例如 Activity）绑定到服务、发送请求、接收响应，甚至执行进程间通信 (IPC)。 绑定服务通常只在为其他应用组件服务时处于活动状态，不会无限期在后台运行。
> 注：只有 Activity、服务和内容提供程序可以绑定到服务 — 您无法从广播接收器绑定到服务。

**[bindService绑定服务的生命周期](https://developer.android.com/guide/components/bound-services.html#Lifecycle)**
![](https://developer.android.com/images/fundamentals/service_binding_tree_lifecycle.png)

### 1.3 [Provider 内容提供程序](https://developer.android.com/guide/topics/providers/content-providers.html)
内容提供程序管理一组共享的应用数据。您可以将数据存储在文件系统、SQLite 数据库、网络上或您的应用可以访问的任何其他永久性存储位置。 其他应用可以通过内容提供程序查询数据，甚至修改数据（如果内容提供程序允许）。 例如，Android 系统可提供管理用户联系人信息的内容提供程序。 因此，任何具有适当权限的应用都可以查询内容提供程序的某一部分（如 ContactsContract.Data），以读取和写入有关特定人员的信息。 内容提供程序也适用于读取和写入您的应用不共享的私有数据。 例如，记事本示例应用使用内容提供程序来保存笔记。 内容提供程序作为 ContentProvider 的子类实现，并且必须实现让其他应用能够执行事务的一组标准 API。 如需了解详细信息，请参阅内容提供程序开发者指南。

内容提供程序管理对结构化数据集的访问。它们封装数据，并提供用于定义数据安全性的机制。 内容提供程序是连接一个进程中的数据与另一个进程中运行的代码的标准界面。
#### 1.3.1 [内容提供程序基础](https://developer.android.com/guide/topics/providers/content-provider-basics.html)
提供程序是 Android 应用的一部分，通常提供自己的 UI 来使用数据。 但是，内容提供程序主要旨在供其他应用使用，这些应用使用提供程序客户端对象来访问提供程序。 提供程序与提供程序客户端共同提供一致的标准数据界面，该界面还可处理跨进程通信并保护数据访问的安全性。
#### 1.3.2 [创建内容提供程序](https://developer.android.com/guide/topics/providers/content-provider-creating.html)
您将提供程序作为 Android 应用中的一个或多个类（连同清单文件中的元素）实现。 其中一个类会实现子类 ContentProvider，即您的提供程序与其他应用之间的接口。 尽管内容提供程序旨在向其他应用提供数据，但您的应用中必定有这样一些 Activity，它们允许用户查询和修改由提供程序管理的数据。

### 1.4 BroadcastReceiver 广播接收器
广播接收器是一种用于响应系统范围广播通知的组件。 许多广播都是由系统发起的 — 例如，通知屏幕已关闭、电池电量不足或已拍摄照片的广播。应用也可以发起广播 — 例如，通知其他应用某些数据已下载至设备，并且可供其使用。 尽管广播接收器不会显示用户界面，但它们可以创建状态栏通知，在发生广播事件时提醒用户。 但广播接收器更常见的用途只是作为通向其他组件的“通道”，设计用于执行极少量的工作。 例如，它可能会基于事件发起一项服务来执行某项工作。

## 2. 其它重要概念
### 2.1 [任务和返回栈](https://developer.android.com/guide/components/tasks-and-back-stack.html?hl=zh-cn) 
应用通常包含多个 Activity。每个 Activity 均应围绕用户可以执行的特定操作设计，并且能够启动其他 Activity。 例如，电子邮件应用可能有一个 Activity 显示新邮件的列表。用户选择某邮件时，会打开一个新 Activity 以查看该邮件。
一个Activity可以启动设备上的其它应用中的Activity，它们之间可以通过`Intent`来沟通。

    任务是指在执行特定作业时与用户交互的一系列 Activity。 这些 Activity 按照各自的打开顺序排列在堆栈（即返回栈）中。

Activity在任务栈中以下图中的方式存在。
![](https://developer.android.com/images/fundamentals/diagram_backstack.png?hl=zh-cn)
> 图示：显示任务中的每个新 Activity 如何向返回栈添加项目。 用户按“返回”按钮时，当前 Activity 随即被销毁，而前一个 Activity 恢复执行。
**流程如下：**
+ 设备主屏幕是大多数任务的起点。当用户触摸应用启动器中的图标（或主屏幕上的快捷方式）时，该应用的任务将出现在前台。 如果应用不存在任务（应用最近未曾使用），则会创建一个新任务，并且该应用的“主”Activity 将作为堆栈中的根 Activity 打开。
+ 当前 Activity 启动另一个 Activity 时，该新 Activity 会被推送到堆栈顶部，成为焦点所在。 前一个 Activity 仍保留在堆栈中，但是处于停止状态。Activity 停止时，系统会保持其用户界面的当前状态。 用户按“返回”按钮时，当前 Activity 会从堆栈顶部弹出（Activity 被销毁），而前一个 Activity 恢复执行（恢复其 UI 的前一状态）。 堆栈中的 Activity 永远不会重新排列，仅推入和弹出堆栈：由当前 Activity 启动时推入堆栈；用户使用“返回”按钮退出时弹出堆栈。 因此，返回栈以“后进先出”对象结构运行。
+ 如果用户继续按“返回”，堆栈中的相应 Activity 就会弹出，以显示前一个 Activity，直到用户返回主屏幕为止（或者，返回任务开始时正在运行的任意 Activity）。 当所有 Activity 均从堆栈中移除后，任务即不复存在。

**Activity 和任务的默认行为总结如下：**
+ 当 Activity A 启动 Activity B 时，Activity A 将会停止，但系统会保留其状态（例如，滚动位置和已输入表单中的文本）。如果用户在处于 Activity B 时按“返回”按钮，则 Activity A 将恢复其状态，继续执行。
+ 用户通过按“主页”按钮离开任务时，当前 Activity 将停止且其任务会进入后台。 系统将保留任务中每个 Activity 的状态。如果用户稍后通过选择开始任务的启动器图标来恢复任务，则任务将出现在前台并恢复执行堆栈顶部的 Activity。
+ 如果用户按“返回”按钮，则当前 Activity 会从堆栈弹出并被销毁。 堆栈中的前一个 Activity 恢复执行。销毁 Activity 时，系统不会保留该 Activity 的状态。
+ 即使来自其他任务，Activity 也可以多次实例化。

---

### 2.2 [进程和线程](https://developer.android.com/guide/components/processes-and-threads.html)
当某个应用组件启动且该应用没有运行其他任何组件时，Android 系统会使用单个执行线程为应用启动新的 Linux 进程。默认情况下，同一应用的所有组件在相同的进程和线程（称为“主”线程）中运行。 如果某个应用组件启动且该应用已存在进程（因为存在该应用的其他组件），则该组件会在此进程内启动并使用相同的执行线程。 但是，您可以安排应用中的其他组件在单独的进程中运行，并为任何进程创建额外的线程。

#### 2.2.1 [进程](https://developer.android.com/guide/components/processes-and-threads.html#Processes)
**android:process**
默认情况下，同一应用的所有组件均在相同的进程中运行，且大多数应用都不会改变这一点。

**生命周期**
1. 前台进程
    用户当前操作所必需的进程。如果一个进程满足以下任一条件，即视为前台进程。
    - 托管用户正在交互的 Activity（已调用 Activity 的 `onResume()` 方法）
    - 托管某个 Service，后者绑定到用户正在交互的 Activity
    - 托管正在“前台”运行的 Service（服务已调用 `startForeground()`）
    - 托管正执行一个生命周期回调的 Service（`onCreate()`、`onStart()` 或 `onDestroy()`）
    - 托管正执行其 `onReceive()` 方法的 BroadcastReceiver
2. 可见进程
    没有任何前台组件，但仍然会影响用户在屏幕上所见内容的进程。如果某一进程满足以下任一条件，即视为可见进程。
    - 托管不在前台、但仍对用户可见的 Activity（已调用其 `onPause()` 方法）。例如，如果前台 Activity 启动了一个对话框，允许在其后显示上一 Activity，则有可能会发生这种情况。
    - 托管绑定到可见（或前台）Activity 的 Service。
3. 服务进程
    - 正在运行已使用 `startService()` 方法启动的服务且不属于上述两个更高类别进程的进程。尽管服务进程与用户所见内容没有直接关联，但是它们通常在执行一些用户关心的操作（例如，在后台播放音乐或从网络下载数据）。因此，除非内存不足以维持所有前台进程和可见进程同时运行，否则系统会让服务进程保持运行状态。
4. 后台进程
    - 包含目前对用户不可见的 Activity 的进程（已调用 Activity 的 `onStop()` 方法）。这些进程对用户体验没有直接影响，系统可能随时终止它们，以回收内存供前台进程、可见进程或服务进程使用。 通常会有很多后台进程在运行，因此它们会保存在 LRU （最近最少使用）列表中，以确保包含用户最近查看的 Activity 的进程最后一个被终止。如果某个 Activity 正确实现了生命周期方法，并保存了其当前状态，则终止其进程不会对用户体验产生明显影响，因为当用户导航回该 Activity 时，Activity 会恢复其所有可见状态。 有关保存和恢复状态的信息，请参阅 Activity文档。
5. 空进程
    - 不含任何活动应用组件的进程。保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。 为使总体系统资源在进程缓存和底层内核缓存之间保持平衡，系统往往会终止这些进程。

> 由于运行服务的进程其级别高于托管后台 Activity 的进程，因此启动长时间运行操作的 Activity 最好为该操作启动服务，而不是简单地创建工作线程，当操作有可能比 Activity 更加持久时尤要如此。例如，正在将图片上传到网站的 Activity 应该启动服务来执行上传，这样一来，即使用户退出 Activity，仍可在后台继续执行上传操作。使用服务可以保证，无论 Activity 发生什么情况，该操作至少具备“服务进程”优先级。 同理，广播接收器也应使用服务，而不是简单地将耗时冗长的操作放入线程中。

####  2.2.2 [线程](https://developer.android.com/guide/components/processes-and-threads.html#Threads)
+ **UI线程**
当应用启动时，系统会为主应用启动一个默认的名为“主线程”的执行线程。主要负责UI相关事务，所以也被称为UI线程。
    - 不要阻塞UI线程。ANR（应用无响应）即由此而产生。
    - 不要在UI线程之外访问Android UI工具包。
+ **[工作线程](https://developer.android.com/guide/components/processes-and-threads.html#WorkerThreads)**
如果有繁重的工作，应该交给工作线程来做。但是工作线程不应访问或者修改UI相关的数据。
使用[AsyncTask]()可以允许用户对UI进行异步操作。它允许先阻塞工作线程中的操作，然后在UI线程中发布结果。
+ [线程安全方法](https://developer.android.com/guide/components/processes-and-threads.html#ThreadSafe)

#### 2.2.3 [进程间通信IPC](https://developer.android.com/guide/components/processes-and-threads.html#IPC)
Android实现进程间通信IPC的机制是通过远程过程调用RPC来实现。通过RPC，由Activity或者其它应用组件调用的方法将命令及数据传送给其它进程中的组件进行远程调用执行，再将结果通过命令返回给调用方。
> 要执行IPC，必须使用`bindService()`将应用绑定到服务上。可参见[服务](https://developer.android.com/guide/components/services.html)的详细介绍。

### 2.3 [Intent及Intent过滤器](https://developer.android.com/guide/components/intents-filters.html) 
Intent是一个消息传递对象。可以用它来进行消息传递。如不同Activity之间的相互启动。常见的基本使用实例如下三个：
+ 启动Activity
    通过将Intent传递给`startActivity()`，可以启动新的Activity实例。其中Intent描述了要启动的Activity并携带了必要的数据。如果需要返回结果，可以通过`startActivityForResult()`来实现。
+ 启动服务
    通过将Intent传递给`startService()`，可以启动在后台执行操作的服务，服务执行一次性操作，如下载文件 。同时Intent描述了要启动的服务，并携带了必要的数据。如果服务指在使用客户端-服务器接口，则通过将Intent传递给`bindService()`来实现。
+ 传递广播
    广播是任何应用可接收的消息。系统将针对系统事件通过将Intent传递给`sendBroadcast()/sendOrderedBroadcast()/sendStickyBroadcast()`来实现。

**Intent分2种类型：**
#### 2.3.1 显式Intent
显式Intent按名称（完全限定类名）指定要启动的组件。 通常，您会在自己的应用中使用显式 Intent 来启动组件，这是因为您知道要启动的 Activity 或服务的类名。例如，启动新 Activity 以响应用户操作，或者启动服务以在后台下载文件。
创建显式 Intent 启动 Activity 或服务时，系统将立即启动 Intent 对象中指定的应用组件。即立即启动Activity或服务。

#### 2.3.2 隐式Intent
隐式Intent不会指定特定的组件，而是声明要执行的常规操作，从而允许其他应用中的组件来处理它。 例如，如需在地图上向用户显示位置，则可以使用隐式 Intent，请求另一具有此功能的应用在地图上显示指定的位置。
创建隐式 Intent 时，Android 系统通过将 Intent 的内容与在设备上其他应用的清单文件中声明的 Intent 过滤器进行比较，从而找到要启动的相应组件。 如果 Intent 与 Intent 过滤器匹配，则系统将启动该组件，并向其传递 Intent 对象。 如果多个 Intent 过滤器兼容，则系统会显示一个对话框，支持用户选取要使用的应用。
> **注意：** 为了确保应用的安全性，启动 Service 时，请始终使用显式 Intent，且不要为服务声明 Intent 过滤器。使用隐式 Intent 启动服务存在安全隐患，因为您无法确定哪些服务将响应 Intent，且用户无法看到哪些服务已启动。从 Android 5.0（API 级别 21）开始，如果使用隐式 Intent 调用 bindService()，系统会引发异常。

#### 2.3.3 Intent过滤器
Intent 过滤器是应用清单文件中的一个表达式，它指定该组件要接收的 Intent 类型。 例如，通过为 Activity 声明 Intent 过滤器，您可以使其他应用能够直接使用某一特定类型的 Intent 启动 Activity。同样，如果您没有为 Activity 声明任何 Intent 过滤器，则 Activity 只能通过显式 Intent 启动。
> 例如分享一张图片，可能会有多个APP能支持这种操作，系统就会弹出选择项来让用户选择使用哪个APP来分享。
![](
https://developer.android.com/images/components/intent-filters@2x.png)

#### 2.3.4 [Intent构建](https://developer.android.com/guide/components/intents-filters.html#Building)
Intent 对象携带了 Android 系统用来确定要启动哪个组件的信息（例如，准确的组件名称或应当接收该 Intent 的组件类别），以及收件人组件为了正确执行操作而使用的信息（例如，要采取的操作以及要处理的数据）。包含的主要信息如下：
+ 组件名称
要启动的组件名称。可选。如果有名称，是显式的Intent调用；如果没有名称，则是隐式的Intent调用。可以用`setComponent()`/`setClass()`/`setClassName()`设置组件。
    注意：启动`Service`时，应该始终指定组件名称。否则将无法确定哪项服务会响应Intent。
+ 操作
指定要执行的操作的符串。如`ACTION_VIEW`和`ACTION_SEND`、`ACTION_EDIT`等。使用`setAction()`来为Intent指定操作。
+ 数据
引用 待操作数据和/或该数据MIME类型的URI。提供的数据类型通常由Intent的操作决定。仅设置URI调用`setData()`，仅设置MIME调用`setType()`，同时设置URI/MIME调用`setDataAndType()`。
    注意：若要同时设置 URI 和 MIME 类型，请勿调用 `setData()` 和 `setType()`，因为它们会互相抵消彼此的值。请始终使用 `setDataAndType()` 同时设置 URI 和 MIME 类型。
+ 类别
一个包含应处理Intent组件类型的附加信息的字符串。可以将任意数量的类别描述放入一个Intent中。但是大多数Intent不需要这个。如`CATEGORY_BROWSABLE`、`CATEGORY_LAUNCHER`。可以调用`addCategory()`设置。
+ 扩展Extra
携带完成请求操作所需的附加信息的键值对。调用`putExtra()`方法添加extra数据。
    例如，使用 ACTION_SEND 创建用于发送电子邮件的 Intent 时，可以使用 EXTRA_EMAIL 键指定“目标”收件人，并使用 EXTRA_SUBJECT 键指定“主题”。
+ 标志
在Intent类中定义的，充当Intent元数据的标志。标志可以指示Android系统如何启动Activity（如Activity应属于哪个任务），以及启动后如何处理（如它是否属于最近的Activity列表）。调用`setFlags()`方法来设置标志。

#### 2.3.5 [Intent解析](https://developer.android.com/guide/components/intents-filters.html#Resolution)
当系统收到隐式Intent以启动Activity时，它根据以下三个方面将该Intent与Intent过滤器进行比较，以便搜索匹配到该Intent最佳Activity：
1. Intent操作
2. Intent数据（URI和数据类型）
3. Intent类别
通过 Intent 过滤器匹配 Intent，这不仅有助于发现要激活的目标组件，还有助于发现设备上组件集的相关信息。 例如，主页应用通过使用指定 ACTION_MAIN 操作和 CATEGORY_LAUNCHER 类别的 Intent 过滤器查找所有 Activity，以此填充应用启动器。

