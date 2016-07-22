---
title: mstar6586 supernova note
date: 2016-07-22 14:12:52
categories:
 - work
tags:
 - mstar
 - supernova
---
前阵子参与了目标市场为海外的，搭载Google Cast应用的TV项目。基于Mstar6586芯片，Linux做的案子。有很多亮点，感觉不错，特别是Mstar新的Supernova，对比之前Turnkey方案下的Pure Supernova进步很大。把过程中的学习笔记放上来记录一下。

这个案子对比传统Linux的案子，比较特别的是IPC部分。它把Android的Binder机制裁剪一下引入进来了。这种IPC机制一下子就方便多了。把自己的Supernova中底层TvService进程与客户订制化要求很高的UI进程之间剥离开来，大大简化了底层逻辑的上层UI之间的耦合度。另外，它的APM部分也不错，有空再另说。

<!-- more -->

## Configuration
**PATH**:`\Supernova\MStarSDK\release\lib.macan_build_Macan_093B_ROM_NAND_DVB_SERVICE.sh`
### `\Supernova\out\buildenv\env.cfg`
- CHIP=macan
- BOARD=093B_DVB
- TARGET_CPU=arm
- TOOLCHAIN=arm-gnueabi
- PROJ_MODE=dvb (support dvb/atsc/isdb)
- LINK_TYPE=dynamic
- TV_SYSTEM=dvbt2
- EXT4=false

### `Supernova\out\buildenv\config\buildenv.mk`
- UI_PATH = ui/jarves/dvbt
- UI_RESOLUTION = _FHD

### `\Supernova\out\buildenv\config\sw_cfg\tvsystem\dtv\dvb.mk`
- DVB_ENABLE=1 
    + DVBT=1
    + DVBC=1
    + DVBS=1
    + DTMB=0
    + ISDB=0
- CI_ENABLE=1
- CI_PLUS_ENABLE=1
- HBBTV_ENABLE=1
- MHEG5_ENABLE = 1
- SUBTITLE_ENABLE = 1

### `Supernova\out\buildenv\config\chips\macan\MST093B_10APY_15445_DVB\pcb_config.mk`
- ENABLE_BACKEND = 1
- CHINA_ATV=0
- CHINA_ENABLE=0

### `Supernova\out\buildenv\target\default_setting.mk`
- STORAGE_TYPE = nand
    + UBIFS = true
    + ext4 = false
    + FS_TYPE = ubifs
- BOOTLOGO_DELAY = 0
- BOOTLOGO_DFBLAYER = 0
- KERNEL_COMPRESS = 0

### `Supernova\out\buildenv\target\dvb.macan\image_setup.mk`
- IMG_FOLDER = Supernova\target\dvb.macan\images
- APP_EXE_FILE =tvos
- APP_PADDING_SIZE    = 0x01400000
- CONFIG_PADDING_SIZE = 0x00100000
- KERNEL_PADDING_SIZE = 0x00800000
- MSLIB_PADDING_SIZE  = 0x00C00000
- RFS_PADDING_SIZE = 0x00900000
- CUS_PADDING_SIZE    = 0x00400000
- CUSBK_PADDING_SIZE  = 0x00400000
- ROOTFS_PADDING_SIZE = 0x00400000

### `Supernova\out\buildenv\config\devices\device_option.mk`
- TUNER_AV2012 = 1
- TUNER_MXL661 = 1
- DEMOD_macan = 1

### `Supernova\out\buildenv\config\sw_cfg\ext_devices.mk`
- FLOATING = hardfloat
- WIFI_80211CFG_ENABLE = 1
- WIFI_DONGLE_RALINK_7632_ENABLE = 1
- LIBDIR = lib_ipc.macan

### `Supernova\out\buildenv\config\sw_cfg\platform\platform.mk`
- MSTAR_IPC = 1
    + MM_IPC_SERVICE_ENABLE = 1
    + RVU_IPC_SERVICE_ENABLE = 1
    + APP_IPC_SERVICE_ENABLE = 1
    + MOUNTNOTIFIER_IPC_SERVICE_ENABLE = 1
    + SECURE_IPC_SERVICE_ENABLE = 1
    + TV_IPC_SERVICE_ENABLE = 1
    + WEBAPPMGR_IPC_SERVICE_ENABLE = 1
    + BROWSER_IPC_SERVICE_ENABLE = 1
    + COMMUI_IPC_SERVICE_ENABLE = 1
    + WIDI_IPC_SERVICE_ENABLE = 1
- MSTAR_TVOS = 1
    + SQL_DB_ENABLE = 1
- MALI_ENABLE = 0
- MSTAR_WEBUI = 0
- PLATFORM_TYPE = MSTAR_PURESN

### `Supernova\out\buildenv\config\sw_cfg\common_feature\common_feature.mk`
+ KEYPAD_ENABLE = 1
+ ENCRYPTED_NETWORK_UPGRADE_ENABLE = 1
+ EPG_ENABLE = 1
+ EPG_EED_ENABLE = 1
+ PVR_ENABLE = 1
+ PIP_ENABLE = 0
+ TIMESHIFTSEAMLESS_ENABLE = 0
+ BGPVR_ENABLE = 0
+ OAD_ENABLE = 1 (DVB OAD Software System Update)
+ SAMBA_CLIENT_ENABLE = 0
+ PREVIEW_MODE_ENABLE = 0
+ WIFI_ENABLE = 1 (Enable to use wifi function)
+ BLUETOOTH_ENABLE = 0
+ ACTIVE_STANDBY_MODE_ENABLE = 1 (Background recording function, for recording after TV turn off)
+ ENABLE_DYNSCALING = 1 (enable dynamic scaling.)

## Architecture
![](http://ww3.sinaimg.cn/large/772d7a33gw1f62o7mbac8j20go0frq6e.jpg)

### Supernova
- ATV Player: to provide the ATV basic functions: Display/Signal Monitor/Auto Tuning/Manual Tuning/Channel Up, down/...
- MM Player: to provide the Multi Media basic functions: Play/Stop/...
- DTV Player: to provide the DTV basic functions:Display/Signal Monitor/Scan/Channel Change/EPG/CI/MHEG5/PVR...
- All other Player: to provide the basic functions for various input source: Display/Signal Monitor/...
- Database: to provide the information saved in the EEprom. For example: Program Info and OSD Info
- CIMMI: to provide CIMMI display information for CI card ui
- Timer: to provide all time related functions
- Channel Manager: wrapper for channel manage functions for ATV/DTV
- Factory Mode: to provide factory setting interface to factory ui
- Picture Quality: to provide the Picture basic functions. For example: Brightness, contrast, Color,...etc
- Control Class(MSrv_Control): interface to get all kinds of MSrv objects
- TTX class: to provide the TTX basic functions.


### Msrv class Hierachy
![](http://ww2.sinaimg.cn/large/772d7a33gw1f62o8212wdj20l70elabp.jpg)


### Msrv_Control
总控类，是父类的**实例对象**。
继承自`class MSrv_Control_DVB/MSrv_Control_ATSC`，
再继承自`class MSrv_Control_TV/MSrv_Control_STB`，
再继承自`class MSrv_Control_common`。

例子：
Change Input Source: MSrv_Control(MSrv_Control_DVB/MSrv_Control_common) class
->SetInputSourceCmd(); 
继承关系图如下：
![](http://ww1.sinaimg.cn/large/772d7a33gw1f62o8gmbv6j20qb08lq58.jpg)

### TVOS Call Flow
主要逻辑关系调用流程如下：
![](http://ww4.sinaimg.cn/large/772d7a33gw1f62o9qqyk2j21kw1gj1jo.jpg)



## EventManager
事件管理器类。基于Vector的消息队列，异步事件池。允许消息发送者`class TvOSEventSender`异步（发送到`class EventManager`内部的Vector消息队列后再一个个调用发送者的发送消息函数）及同步（立即直接调用发送者自己的消息发送函数）消息发送。
### Dispatcher
`class EventManager`创建了一个主检测发送线程`threadEventDispatcher()`来启动发送事件；同时初始化一个最大个数为4的线程池数组存储一系列发送事件任务（分别由各自的线程负责，在`StartPostThreadPool()`中创建），事件发送任务经由`class mapi_event`来完成发送任务启动消息，最终由线程池中的各个发送线程收到启动发送的消息后，自己调用发送者的发送消息函数来发送消息（在发送者各自的子类service中完成）。
### Dispatch process
当发送者`class TvOSEventSender `发送消息到`class EventManager`时，事件消息将存储在` vector<ST_TVOS_EVENT> tvos_event_queue`中。发送事件的工作线程`threadEventDispatcher()`中将根据`m_bPoolAvaliable`状态来决定是否进行分发工作，并检测`m_PostTaskCount`空闲的发送任务个数，来循环发送已经被填充的`tvos_event_dipatchqueue`事件队列，通过调用赋值的线程池数组`m_postevent_thread[i]`的成员`mapi_event`中的`send()`来发送`**POST_START**`消息给各自的线程发送处理函数`PostTask()`。当各自的线程接收到`**POST_START**`消息后，通过调用发送者`class TvOSEventSender`的成员函数`_PostEventToClient()`实现最终的消息发送。此消息发送即通过调用每个发送者模块自己的子类service中的`PostEventToClient()`函数来实现。如`bool CaManagerService::PostEventToClient(U32 nEvt, U32 wParam, U32 lParam)`。
```cpp
class EventManager //事件管理器
{
public:
    EventManager();
    ~EventManager();
//获取实例对象
    static EventManager * GetInstance();
//销毁实例对象
    static void Destroy();
//初始化，并创建事件发送线程threadEventDispatcher()。最大4个发送事件线程空间。
    bool Init(int paralTaskCount = MAX_POST_THREAD);
    bool Finalize();
//发送消息到EM。存储到tvos_event_queue中。默认不重复尝试。
    bool PostEvent(ST_TVOS_EVENT s_Event, bool retry=false);
//发送优先消息到EM。存储到优先队列tvos_event_tempqueue中。
    bool PostEventToQueueForBoot(ST_TVOS_EVENT s_Event);
//注册发送类型/者。枚举type与sender类一一对应。
    bool RegisterSender(EN_POSTEVENT_SERVICE servicetype, TvOSEventSender* pSender);
//取消注册发送类型/者
    bool UnRegisterSender(EN_POSTEVENT_SERVICE servicetype);
//通过发送消息类型找到发送者
    TvOSEventSender * GetEventSender(EN_POSTEVENT_SERVICE servicetype);
//初始化事件发送任务线程池数组，并创建对应的线程。创建m_PostTaskCount个mapi_event实例及各状态参数。
    bool StartPostThreadPool();
//关闭所有的事件发送任务及相关数据。关闭线程池，退出m_PostTaskCount mapi_event。
    bool StopPostThreadPool();
    bool IsTvListenerReady();
    void SetTvListenerReady(bool isReady);
//分发事件函数。
//循环从tvos_event_dipatchqueue队列中事件通过空闲的m_postevent_thread[i]调用m_postevent_thread[i].NotifyEvt->Send()发送启动消息。
//实际发送工作由线程池数组元素中的线程通过调用发送者自己的子类service的发送消息函数来完成发送。
    void DispatchEvent()
//线程池任务锁
    pthread_mutex_t m_ThreadPoolLock;
//线程池是否可用标志
    bool m_bPoolAvaliable;

private:
//事件发送线程池中线程数据结构体
    typedef struct {
            pthread_t postevent_thread_handle;
            pthread_cond_t postevent_thread_condition;
            pthread_mutex_t postevent_thread_mutex;
            ST_TVOS_EVENT a_Event;
            mapi_event<POST_TASK_NOTIFY_EVENT> *NotifyEvt;
            bool inited;
            bool isBusy;
            EventManager * pEventManager;
    } postevent_thread_data;
//发送事件线程数据状态存储数组。存储了MAX_POST_THREAD个mapi_event实例及线程。
    postevent_thread_data m_postevent_thread[MAX_POST_THREAD];
//发送事件线程实际计数
    int m_PostTaskCount;
//分发事件线程，调用DispatchEvent()函数分发事件。
    static void* threadEventDispatcher(void *arg);
//线程池中的线程处理函数。
    static void* PostTask(void *pdata);
//判断事件是否需要重发。
    static bool EventNeedRetry(U32 evt, U32 wPparam);
//事件管理器EM的实例
    static EventManager *m_pInstance ;
    static bool m_bInit;
//事件注册列表，以<类型，发送者>存储。
    std::map<int, TvOSEventSender* > m_ServiceMap;
//普通事件发送队列。
    vector<ST_TVOS_EVENT> tvos_event_queue;
//实际事件发送空队列。发送时将tvos_event_tempqueue和tvos_event_queue中的事件加入并发送。
    vector<ST_TVOS_EVENT> tvos_event_dipatchqueue;
//存储优先发送的事件队列。
    vector<ST_TVOS_EVENT> tvos_event_tempqueue;
    pthread_mutexattr_t m_ServiceMapLockAttr;
    pthread_mutex_t m_ServiceMapLock;
    pthread_mutexattr_t m_QueueLockAttr;
    pthread_mutex_t m_QueueLock;
    pthread_mutexattr_t m_ThreadPoolAttr;
    pthread_t m_threadEventDispatcher_id;
    bool m_bDisPatchEnable;
    bool m_TvListenerReady;
};

class TvOSEventSender //事件发送者
{
public:
    mutable Mutex m_Lock;
public:
    TvOSEventSender():m_Lock(Mutex::RECURSIVE){ m_ServiceType = E_MANAGER_NULL; m_bRegistered = false; m_bIsPosting = false;}
    ~TvOSEventSender();
//将发送者自己注册到事件管理器
    bool RegisterToEM(EN_POSTEVENT_SERVICE servicetype);
//取消注册
    bool UnRegisterToEM();
//发送消息到事件管理器。syncchronous参数提供同步与异步方式。默认为异步，即发送到EM事件管理器事件队列。
    bool PostEventToEM(U32 nEvt, U32 wParam, U32 lParam, int flag=EVENT_FLAG_NONE, U32 togglenEvt=NO_TOGGLE_EVT, bool syncchronous=false);
//直接调用发送者自己的service中发送消息函数将消息立即发送出去。不经过事件管理器EM。即同步发送方式。
    bool _PostEventToClient(U32 nEvt, U32 wParam, U32 lParam);
//纯虚函数。要求继承此类的各种Service实现此消息发送函数，供同步发送方式使用。
    virtual bool PostEventToClient(U32 nEvt, U32 wParam, U32 lParam) = 0;
    void Clear();

protected:
    bool m_bIsPosting;

private:
//把事件消息发送到事件管理器EM。
    bool _PostEventToEM(U32 nEvt, U32 wParam, U32 lParam, int flag, U32 togglenEvt);
//发送事件的类型模块枚举定义（见如下枚举）
    EN_POSTEVENT_SERVICE m_ServiceType;
//是否注册到事件管理器EM的标志
    bool m_bRegistered;
};

typedef enum //发送事件的类型模块。每个类型都有一个class TvOSEventSender类的sender注册到事件管理器EM上。
{
    E_TVMANAGER,
    E_PLAYERIMPL,
    E_AUDIOMANAGER,
    E_PICTUREMANAGER,
    E_PIPMANAGER,
    E_3DMANAGER,
    E_TIMERMANAGER,
    E_CAMANAGER,
    E_CIMANAGER,
    E_CECMANAGER,
    E_MHLMANAGER,
    E_THIRDPARTYMANAGER,
    E_USBMASSSTORAGEMANAGER,
    E_NETWORKMANAGER,
    E_FACTORYMANAGER,
    E_MANAGER_MAX,
    E_MANAGER_NULL = E_MANAGER_MAX,
}EN_POSTEVENT_SERVICE;

typedef struct //消息事件结构体
{
    U32 u32Evt;
    U32 u32ToggleEvt;
    U32 u32wParam;
    U32 u32lParam;
    EN_POSTEVENT_SERVICE enPostSer;//发送者。
    U16 u16Flag;
}ST_TVOS_EVENT;
```




