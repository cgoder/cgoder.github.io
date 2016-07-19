---
title: realtek TV RPC
date: 2016-07-15 14:50:37
categories:
 - work
tags:
 - realtek
 - linux
---
## 接口

1. RPC 客户端基类。
```cpp
class RpcClient
```

2. RPC类的实体。每个实体代表一个函数调用。其成员`m_StrFuncName`是原始调用函数的函数名。`m_CallCtxMap`为其map组合。
```cpp
struct RpcClient::CallContext
```

3. RPC回调类。
```cpp
class RpcClient::CallbackHandler
```

<!-- more -->

4. 数据序列化类(16KB缓存)。将client的指令(TYPE_INVOKE/TYPE_RESULT) 打包并序列化成数据流，提供给`IpcStreamer`传输。参见`RpcCommandType`结构体。
```cpp
class RpcCommandMuxer* m_pCmdMuxer
```

5. 数据序反列化类(16KB缓存)。将 IpcStreamer中得到的数据流反序列化成指令( TYPE_INVOKE/TYPE_RESULT/TYPE_CALLBACK_RESULT)。
```cpp
class RpcCommandDemuxer* m_pCmdDemuxer
```

6. 数据流传输通道。实际为FIFO式流。从管道里Read/Write数据，提供给`RpcCommandMuxer`及`RpcCommandDemuxer`序列化/反序列化数据。
```cpp
class rtk::ipc::IpcStreamer* m_pStreamer
```

7. callback的map集合。
```cpp
class CallbackMap m_CallbackMap
```

8. callback类。继承实现了一个`CommandProcessor`。其实类为`CallbackContext`。
```cpp
class RpcClient::CallbackHandler* m_pCallbackHandler
```

9. 回调函数存储的队列实例。实际将函数指针数据存储在一个list里。其中`struct CallbackContext`结构即为当前的`class CommandProcessor`实类。
```cpp
class CommandQueue<CallbackContext> m_CallbackCmdQue
```

10. 回调函数序列化的数据流结构体。其中`pData`指针指向已经序列化的指令（回调函数/组）数据流，iDataSize代表数据流数据大小。数据流经过`RpcCommandDemuxer`反序列化，即可解析出正确的指令（类型为TYPE_CALLBACK_RESULT的 回调函数）。
```cpp
struct CallbackContext
```

11. RPC客户端实例。
```cpp
class RpcClient& m_RpcClient
```

---

## 类体

1. 指令流头部数据结构体。内结构顺序为：
`RpcCommandHeader`+`RpcParamHeader1`+`data1`+`...`+`RpcParamHeaderN`+`dataN`。
```cpp
struct RpcCommandHeader
{
    unsigned int iRpcCommandType; //RCP类型。(TYPE_INVOKE/ TYPE_RESULT 参见RpcCommandType结构体)
    unsigned int iId; //唯一ID，用于指示哪个命令。
    unsigned int iTotalLen; //序列化的指令数据流总长度。并且包含 RpcCommandHeader结构体本身的长度在内。
    unsigned int iParamCounts; //数据流内单元（RpcParamHeader+data）个数。包括函数参数和函数名。结构及顺序为para1+para2+...paraN+FuncName。
}
```

2. 数据流单元结构体。函数名及参数均用此结构体序列化。
```cpp
struct RpcParamHeader
{
    unsigned int iRpcParamType; //RPC单元数据类型。参见 enum RpcParamType枚举类型。
    unsigned int iRpcParamLen; //RPC单元数据长度。此长度不包含 RpcParamHeader结构体本身长度在内。
};
```

3. RpcServer接口虚基类。
```cpp
class IpcServer
{
public: //子类必须实现这些纯虚接口，做具体事情。例如 IpcServerBase 类。
    virtual bool Start(const char* pStrServerName, void* pParam) = 0;
    virtual bool Stop() = 0;
    virtual bool RegisterObserver(IpcServerObserver* pObs) = 0;
    virtual void UnregisterObserver(IpcServerObserver* pObs) = 0;
    virtual const char* GetServerName() = 0;
    virtual void* GetParameter() = 0;

public:
    virtual ~IpcServer() { ; };
};
```

4. 继承自`IpcServer`，RpcServer基础实现类。
```cpp
class IpcServerBase: public IpcServer
{
public: // 接口定义在IpcServer类中，这里是具体实现。
    bool Start(const char* pStrServerName, void* pParam);
    bool Stop();
    bool RegisterObserver(IpcServerObserver* pObs);
    void UnregisterObserver(IpcServerObserver* pObs);
    const char* GetServerName();
    void* GetParameter();

public:
    IpcServerBase();
    ~IpcServerBase();
    IpcServerBase (const IpcServerBase &) {}
    IpcServerBase &operator= (const IpcServerBase&) { return *this; }

protected: //子类必须实现这些纯虚接口，做具体事情。例如 IpcServerPipeImpl类。
    virtual bool DoStart(const char* pStrServerName, void* pParam) = 0;
    virtual bool DoStop() = 0;
    virtual IpcStreamer* DoAccept() = 0;
    virtual void FreeIpcStreamer(IpcStreamer* pStreamer) = 0;
    virtual bool IsShutdownServer() = 0;

private: // Used to notify observers
    void NotifyOpened(const char* pStrServerName);
    bool NotifyAccepted(IpcStreamer* pStreamer);
    void NotifyClosed();

private:
    class PrivateImpl;
    PrivateImpl* m_pImpl;
};
```

5. 继承自`IpcServerBase`，FIFO类型RpcServer主实现类。
```cpp
class IpcServerPipeImpl: public IpcServerBase
{
public:
    IpcServerPipeImpl();
    ~IpcServerPipeImpl();
    IpcServerPipeImpl(const IpcServerPipeImpl &param) {}
    IpcServerPipeImpl &operator = (const IpcServerPipeImpl&) { return *this; }

private: //接口定义在 IpcServerBase 类中，这里是具体实现。
    bool DoStart(const char* pStrServerName, void* pParam);
    bool DoStop();
    IpcStreamer* DoAccept();
    void FreeIpcStreamer(IpcStreamer* pStreamer);
    bool IsShutdownServer();

private:
    struct PrivateImpl;
    PrivateImpl* m_pImpl;
};
```

6. IPC服务器端接口类。管理 RpcServer集合等map。
```cpp
class IpcServerManager
{
public:
    enum
    {
        TIME_OUT_INFINITY = -1 //Time out infinity
    };

public:
    static IpcServerManager& GetInstance();
    bool StartServer(const char* pStrServerName, int iIpcImplMode, void* pParam);
    bool StopServer(const char* pStrServerName);
    void StopAllServers();
    void WaitForAllServerStop(int iTimeout);
    bool RegisterObserver(const char* pStrServerName, IpcServerObserver* pObs);
    void UnregisterObserver(const char* pStrServerName, IpcServerObserver* pObs);

private:
    IpcServerManager();
    IpcServerManager(const IpcServerManager&);
    IpcServerManager& operator=(const IpcServerManager&);
    ~IpcServerManager();

private:
    struct PrivateImpl;
    PrivateImpl* m_pImpl;
};
```

7. RPC服务端接口类。实际调用IpcServerManager类方法。
```cpp
class RpcServer
{
public:
    RpcServer();
    ~RpcServer();
    bool StartServer(const char* pStrServerName, void* pParam);
    bool StopServer();
    const char* GetServerName() const;

private:
    class PrivateImpl;
    PrivateImpl* m_pImpl;

private:
    RpcServer(const RpcServer &);
    RpcServer &operator=(const RpcServer &);
};
```

8. RPC调用执行管理类。
```cpp
class RpcExecutorManager
{
public:
    enum ErrCode
    {
        ERR_FAILED = -1,
    };
public:
    static RpcExecutorManager& GetInstance();
    bool RegisterExecutor(RpcExecutor* pExecutor);
    bool UnregisterExecutor(RpcExecutor* pExecutor);
    int Execute(RpcCommandDemuxer* pCmdDemuxer,  RpcCallback* pRpcCallback,  char* pBuffer, int iBufSize);

private:
    struct PrivateImpl;
    PrivateImpl* m_pImpl;

private:
    RpcExecutorManager();
    ~RpcExecutorManager();
    RpcExecutorManager(const RpcExecutorManager&);
    RpcExecutorManager& operator=(const RpcExecutorManager&);
};
```

---

## 原理及过程分析

- ### RPC机制原理

　　RealTek的TvService模块的RPC使用Unix基础的FIFO（命名管道）来实现。实际项目中创建了1+2N个FIFO来完成RPC。因为FIFO是单工的，所以只能单向传输。
　　
　　RpcServer模块在随TvService开机启动运行后，即创建1个FIFO，用于监听并接收RpcClient模块传输过来的`IpcPipeImplHeader`数据，并根据`IpcPipeImplHeader`数据中的参数`iID`来获知与此RpcClient交互的一对R/W属性的FIFO。这对FIFO由连接的RpcClient创建。
　　
　　RpcClient创建了2个FIFO，用于RpcClient与RpcServer的数据传输 。因为带有 RpcClient的`iID`，此参数`iID`保证系统内随机性和唯一性，所以RpcServer能通过此参数来识别对应的RpcClient,并与之交互。RpcClient将参数存储在`IpcPipeImplHeader`结构体中，通过RpcServer已经创建好的FIFO传输给RpcServer。这里，RpcClient知道RpcServer所创建的FIFO，是因为在编码里就已经约定好的FIFO文件路径，所以RpcClient只需要按此路径去打开这个FIFO，往里面写入数据即可。而且，因为RpcServer是跟随TvService一起启动的，而TvService是一个单独的进程，很早就启动完成，所以能够保证在RpcClient打开这个FIFO并写入文件时，对应的RpcServer已经创建成功这个FIFO了。
　　
　　同理，系统里可能存在多个此对R/W属性的FIFO。因为可能有多个RpcClient同时存在。如果有其它RpcClient连接RpcServer的话，则同样有一对FIFO提供给RpcServer，提供方法是通过向RpcServer创建的FIFO写入`IpcPipeImplHeader`数据来实现沟通。

- ### RpcServer启动过程 

　　构造RpcServer时，通过调用`IpcServerManager`接口类初始化来创建实例，并`StartServer`。而在`StartServer`方法中调用了`IpcServerPipeImpl`类构造函数来实例化 **IPC_PIPE** 型RpcServer对象，并`Start`服务。`IpcServerPipeImpl`接口类继承自`IpcServerBase`接口类，`Start`方法在`IpcServerBase`接口类中，而`Start`方法又通过虚接口调用了子类`IpcServerPipeImpl`类中的`DoStart`方法来创建FIFO，然后调用`IpcServerBase`接口类中的`StartAccetpClient`方法来启动接收RpcClient数据的服务。`IpcServerBase`接口类中的`PrivateImpl`成员类是继承自`CommandProcessor`类的对象，用于开始接收并通知streamer的接收数据。在`StartAccetpClient`方法中创建了一个`CommandQueue`类，通过调用此类的`AddCommand`方法并用 **ACCEPT_COMMAND** 参数（此参数目前不起任何作用）来创建一个等待指令的线程`Policy1Proc`。如果监测到有事件发生，此线程调用`IpcServerBase::PrivateImpl`这个 `CommandProcessor`对象中的`Execute`方法来执行。而`Execute`方法里又调用了`IpcServerPipeImpl`接口类中的`DoAccept`方法接收所有来自RpcServer创建的FIFO的数据。而RpcClient在启动连接RpcServer时会向这个FIFO传输`IpcPipeImplHeader`结构的数据。
　　
　　RpcServer接收到`IpcPipeImplHeader`结构的数据后，调用`CreateStreamerByCommand`方法来分析数据，然后像RpcClient的初始化动作一样，通过 调用`IpcStreamerPipeImpl`类创建streamer(`IpcStreamerPipeImpl::Create`)，并放到streamer类map中存储。其中，因为接收到的数据中有`iID`这个成员是表示其唯一性的（RpcClient的connnect流程中是通过rand方法来获取的），而且当RpcServer接收到此数据时，RpcClient端已经创建了2个FIFO，所以RpcServer后面会直接使用这2个FIFO向RpcClient端回传数据。

- ### RpcClient启动过程

　　与RpcServer类似，不赘述。

- ### RpcClient与RpcServer交互过程

　　RpcServer先启动，创建好监听RpcClient的FIFO。
　　
　　调用`IpcClientFactory`类创建实例对像，实际调用`IpcClientPipeImpl`类来构造RpcClient。并加入到`ClientList`vctor容器中。
　　
　　RpcClient构造后开始连接RpcServer(`IpcClientPipeImpl::Connect`)。先`open`打开RpcServer创建好的FIFO，并向FIFO`write`一个`IpcPipeImplHeader`结构体的数据后关闭FIFO。然后调用`IpcStreamerPipeImpl`类创建streamer(`IpcStreamerPipeImpl::Create`)，并`open`打开(`IpcStreamerPipeImpl::DoOpen`)，此方法中将创建2个FIFO，并将其设为一读一写（因为FIFO是单工）。

- ### INVOKE远程调用执行过程

　　当RpcClient类有一个函数需要invoke调用底层TvService里的函数时，需要将invoke参数顺序`push`，最后`push`函数名，最后`finish`。然后调用`RpcCommandMuxer`类对数据序列化并流化，通过RpcClient创建的具有W属性的FIFO传输出去。RpcServer会通过具有此PIPE(对应到RpcServer端是R属性)读取到数据流，并在server端为此新生成一个新的streamer及线程等相关资源，专门用来接收此次通信的流数据，并调用`RpcCommandDemuxer`类反序列化数据。并调用`RpcExecutorManager`类里的方法来执行相关动作。
　　
　　其中`RpcExecutorManager`是一个专用于管理`RpcExecutor`类对象的管理器。同样的还有`IpcServerManager`。
　　
　　而`RpcExecutor`是真正执行函数调用的类。它被用于模块初始化时调用，通过`RegisterExecutor`/`UnregisterExecutor`方法注册/反注册到`RpcExecutorManager`类中的list中。它的`Execute`方法即是真正执行invoke函数的地方，执行完返回的结果是一个已经序列化的以`RpcCommandMuxer`类表示的数据。
　　　　
　　RpcServer回传函数调用结果的方式与RpcClient远程调用函数的传输方式一样，不同点在于调用`RpcCommandMuxer`序列化数据时，头部`RpcCommandHeader`结构体数据里面填充的`iRpcCommandType`参数数据不同。
　　
　　TvService模块初始化时调用`RpcExecutor`类来注册供RpcClient远程调用invoke的native函数集合，其成员`m_FuncMap`是一个`RpcCommandMuxer* (PrivateImpl::*fpFunction)(RpcCommandDemuxer*, RpcCallback*);`类型的函数指针成员的map。通过查找map中的映射关系，最终找到native函数，并执行。执行完将结果封装成`RpcCommandMuxer`流数据返回。
　　
> **例如:** `TvChannelApiExecutor`类。
> 
```cpp
class PrivateImpl(): m_CmdMuxer(m_Buffer, sizeof(m_Buffer)),
                   m_bFirstPlay(false)
    {
        m_FuncMap["TvAutoScanStart"] = &PrivateImpl::TvAutoScanStart;
        m_FuncMap["TvAutoScanStop"] = &PrivateImpl::TvAutoScanStop;
        m_FuncMap["TvAutoScanComplete"] = &PrivateImpl::TvAutoScanComplete; 
        m_FuncMap["TvAutoScanStartWithRange"] = &PrivateImpl::TvAutoScanStartWithRange;
        m_FuncMap["TvAutoScanStopWithRange"] = &PrivateImpl::TvAutoScanStopWithRange;
        m_FuncMap["TvAutoScanCompleteWithRange"] = &PrivateImpl::TvAutoScanCompleteWithRange;  
        m_FuncMap["TvSeekScanStart"] = &PrivateImpl::TvSeekScanStart;
        m_FuncMap["TvSeekScanStop"] = &PrivateImpl::TvSeekScanStop;
        m_FuncMap["TvScanManualStart"] = &PrivateImpl::TvScanManualStart;
        m_FuncMap["TvScanManualStop"] = &PrivateImpl::TvScanManualStop;
        m_FuncMap["TvScanManualComplete"] = &PrivateImpl::TvScanManualComplete;
        m_FuncMap["TvScanInfo"] = &PrivateImpl::TvScanInfo;
        m_FuncMap["IsTvScanning"] = &PrivateImpl::IsTvScanning;
        m_FuncMap["GetAtvSeqScanStartFreq"] = &PrivateImpl::GetAtvSeqScanStartFreq;
        m_FuncMap["GetAtvSeqScanEndFreq"] = &PrivateImpl::GetAtvSeqScanEndFreq;
        m_FuncMap["SetDtvScanType"] = &PrivateImpl::SetDtvScanType;
        m_FuncMap["GetDtvScanType"] = &PrivateImpl::GetDtvScanType;
        m_FuncMap["PlayNextChannel"] = &PrivateImpl::PlayNextChannel;
        m_FuncMap["PlayPrevChannel"] = &PrivateImpl::PlayPrevChannel;
        m_FuncMap["PlayFirstChannel"] = &PrivateImpl::PlayFirstChannel;
        m_FuncMap["PlayHistoryChannel"] = &PrivateImpl::PlayHistoryChannel;
        m_FuncMap["DumpTvChannelList"] = &PrivateImpl::DumpTvChannelList;
        m_FuncMap["SetDefaultFilter"] = &PrivateImpl::SetDefaultFilter;
        m_FuncMap["GetDefaultFilter"] = &PrivateImpl::GetDefaultFilter;
        m_FuncMap["GetCurChannel"] = &PrivateImpl::GetCurChannel;
        m_FuncMap["GetChannelInfoByIndex"] = &PrivateImpl::GetChannelInfoByIndex;
        m_FuncMap["GetChInfoArray"] = &PrivateImpl::GetChInfoArray;
        m_FuncMap["GetChannelBandwidth"] = &PrivateImpl::GetChannelBandwidth;
        m_FuncMap["GetChannelName"] = &PrivateImpl::GetChannelName;
        m_FuncMap["GetChannelCount"] = &PrivateImpl::GetChannelCount;
        m_FuncMap["SortChannel"] = &PrivateImpl::SortChannel;
        m_FuncMap["SaveChannel"] = &PrivateImpl::SaveChannel;
        m_FuncMap["PlayChannelByIndex"] = &PrivateImpl::PlayChannelByIndex;
        m_FuncMap["PlayChannelByNum"] = &PrivateImpl::PlayChannelByNum;
        m_FuncMap["PlayChannel"] = &PrivateImpl::PlayChannel;       
        m_FuncMap["PlayChannelByLCN"] = &PrivateImpl::PlayChannelByLCN;
        m_FuncMap["PlayFirstChannelInFreq"] = &PrivateImpl::PlayFirstChannelInFreq;
        m_FuncMap["PlayChannelByChnumFreq"] = &PrivateImpl::PlayChannelByChnumFreq;
        m_FuncMap["SwapChannelByIdxEx"] = &PrivateImpl::SwapChannelByIdxEx;
        m_FuncMap["SwapChannelByNumEx"] = &PrivateImpl::SwapChannelByNumEx;
        m_FuncMap["ReloadLastPlayedSource"] = &PrivateImpl::ReloadLastPlayedSource;
        m_FuncMap["SetCurChannelSkipped"] = &PrivateImpl::SetCurChannelSkipped;
        m_FuncMap["SetCurAtvSoundStd"] = &PrivateImpl::SetCurAtvSoundStd;
        m_FuncMap["FineTuneCurFrequency"] = &PrivateImpl::FineTuneCurFrequency;
        m_FuncMap["SetCurChAudioCompensation"] = &PrivateImpl::SetCurChAudioCompensation;
        m_FuncMap["SetSource"] = &PrivateImpl::SetSource;
        m_FuncMap["SetBootSource"] = &PrivateImpl::SetBootSource;
        m_FuncMap["GetCurChannelSkipped"] = &PrivateImpl::GetCurChannelSkipped;
        m_FuncMap["GetCurAtvSoundStd"] = &PrivateImpl::GetCurAtvSoundStd;
        m_FuncMap["GetCurChAudioCompensation"] = &PrivateImpl::GetCurChAudioCompensation;
        m_FuncMap["GetSourceList"] = &PrivateImpl::GetSourceList;
        m_FuncMap["GetSourceListCnt"] = &PrivateImpl::GetSourceListCnt;
        m_FuncMap["GetCurSourceType"] = &PrivateImpl::GetCurSourceType;
        m_FuncMap["GetBootSource"] = &PrivateImpl::GetBootSource;
#if defined (TVSERVER_IDTV)
        m_FuncMap["GetIDTVSubSource"] = &PrivateImpl::GetIDTVSubSource;
#endif
        m_FuncMap["GetCurTvType"] = &PrivateImpl::GetCurTvType; 
        m_FuncMap["GetChannelNameList"] = &PrivateImpl::GetChannelNameList;    
        m_FuncMap["GetCurrentProgramInfo"] = &PrivateImpl::GetCurrentProgramInfo;                   
        m_FuncMap["GetCurrentProgramDescription"] = &PrivateImpl::GetCurrentProgramDescription;                         
        m_FuncMap["GetCurrentProgramRating"] = &PrivateImpl::GetCurrentProgramRating;   
        m_FuncMap["HasCurrentProgramWithSubtitle"] = &PrivateImpl::HasCurrentProgramWithSubtitle;           
        m_FuncMap["GetCurAtvSoundSelect"] = &PrivateImpl::GetCurAtvSoundSelect;
        m_FuncMap["GetCurDtvAudioPID"]=&PrivateImpl::GetCurDtvAudioPID;
        m_FuncMap["GetCurDtvVideoPID"]=&PrivateImpl::GetCurDtvVideoPID;
        m_FuncMap["GetCurDtvTSID"]=&PrivateImpl::GetCurDtvTSID;
        m_FuncMap["GetCurDtvServiceID"]=&PrivateImpl::GetCurDtvServiceID;
        m_FuncMap["GetCurDtvPCR"]=&PrivateImpl::GetCurDtvPCR;
        m_FuncMap["SetCurDtvSoundSelectByIndex"] = &PrivateImpl::SetCurDtvSoundSelectByIndex;
        m_FuncMap["GetCurDtvSoundSelectIndex"]=&PrivateImpl::GetCurDtvSoundSelectIndex;     
        m_FuncMap["GetCurrentAudioLang"] = &PrivateImpl::GetCurrentAudioLang;
        m_FuncMap["GetCurInputInfo"] = &PrivateImpl::GetCurInputInfo;
        m_FuncMap["GetCurrentSetting_tv"] = &PrivateImpl::GetCurrentSetting_tv;
        m_FuncMap["GetChannelFreqCount"] = &PrivateImpl::GetChannelFreqCount;
        m_FuncMap["GetChannelFreqByTableIndex"] = &PrivateImpl::GetChannelFreqByTableIndex;   
        m_FuncMap["GetChannelchannelNumByTableIndex"] = &PrivateImpl::GetChannelchannelNumByTableIndex;      
        m_FuncMap["GetChannelCountByFreq"] = &PrivateImpl::GetChannelCountByFreq;     
        m_FuncMap["GetCurChannelIndex"] = &PrivateImpl::GetCurChannelIndex;
        m_FuncMap["PlayNumberChannel"] = &PrivateImpl::PlayNumberChannel;
        m_FuncMap["GetChannelListChannelCount"] = &PrivateImpl::GetChannelListChannelCount;
        m_FuncMap["GetChannelDataList"] = &PrivateImpl::GetChannelDataList;
        m_FuncMap["GetCurDtvSoundSelectList"] = &PrivateImpl::GetCurDtvSoundSelectList;
        m_FuncMap["GetCurDtvSoundSelectCount"] = &PrivateImpl::GetCurDtvSoundSelectCount;
        m_FuncMap["GetCurAtvSoundSelectList"] = &PrivateImpl::GetCurAtvSoundSelectList;
        m_FuncMap["GetCurAtvSoundSelectCount"] = &PrivateImpl::GetCurAtvSoundSelectCount;
        m_FuncMap["SetCaptionMode"] = &PrivateImpl::SetCaptionMode;
        m_FuncMap["GetCaptionMode"] = &PrivateImpl::GetCaptionMode;
        m_FuncMap["SetAnalogCaption"] = &PrivateImpl::SetAnalogCaption;
        m_FuncMap["GetAnalogCaption"] = &PrivateImpl::GetAnalogCaption;
        m_FuncMap["SetDigitalCaption"] = &PrivateImpl::SetDigitalCaption;
        m_FuncMap["SetChannelFav"] = &PrivateImpl::SetChannelFav;
        m_FuncMap["SetChannelSkip"] = &PrivateImpl::SetChannelSkip;
        m_FuncMap["SetChannelBlock"] = &PrivateImpl::SetChannelBlock;       
        m_FuncMap["SetChannelDel"] = &PrivateImpl::SetChannelDel;
        m_FuncMap["GetChannelFav"] = &PrivateImpl::GetChannelFav;
        m_FuncMap["GetChannelSkip"] = &PrivateImpl::GetChannelSkip;     
        m_FuncMap["GetChannelBlock"] = &PrivateImpl::GetChannelBlock;
        m_FuncMap["QueryTvStatus"] = &PrivateImpl::QueryTvStatus;
        m_FuncMap["StartRecordTs"] = &PrivateImpl::StartRecordTs;
        m_FuncMap["StopRecordTs"] = &PrivateImpl::StopRecordTs;
        m_FuncMap["GetEpgDailyListCountByChIdx"] = &PrivateImpl::GetEpgDailyListCountByChIdx;   
#ifdef DVB_T                                
        m_FuncMap["GetEpgDailyListByChIdx"] = &PrivateImpl::GetEpgDailyListByChIdx;
#endif
#ifdef ENABLE_FACE_DETECTION_FOR_MAGELLAN
        m_FuncMap["StartDetection"] = &PrivateImpl::StartDetection; 
        m_FuncMap["StopDetection"] = &PrivateImpl::StopDetection;   
#endif
#ifdef QAM_MODE_SETTING
        m_FuncMap["GetTvQamConst"] = &PrivateImpl::GetTvQamConst;
        m_FuncMap["SetTvQamConst"] = &PrivateImpl::SetTvQamConst;
#endif
#ifdef SYMBOL_RATE_SETTING_BY_VAL
        m_FuncMap["GetTvSymbolRateValue"] = &PrivateImpl::GetTvSymbolRateValue;
        m_FuncMap["SetTvSymbolRateValue"] = &PrivateImpl::SetTvSymbolRateValue;
#endif
        m_FuncMap["SetSubtitleEnable"] = &PrivateImpl::SetSubtitleEnable;
        m_FuncMap["GetSubtitleEnable"] = &PrivateImpl::GetSubtitleEnable;
        m_FuncMap["SetDtvSubtitleByIndex"] = &PrivateImpl::SetDtvSubtitleByIndex;
        m_FuncMap["GetDtvSubtitleIndexList"] = &PrivateImpl::GetDtvSubtitleIndexList;
        m_FuncMap["GetCurDtvSubtitleIndex"] = &PrivateImpl::GetCurDtvSubtitleIndex;
        m_FuncMap["GetDtvSubtitleIndexListCount"] = &PrivateImpl::GetDtvSubtitleIndexListCount;
        m_FuncMap["GetDtvSubtitleIndexListCountByCategory"] = &PrivateImpl::GetDtvSubtitleIndexListCountByCategory;
        m_FuncMap["SetDTVAudioType"] = &PrivateImpl::SetDTVAudioType;
        m_FuncMap["GetDTVAudioType"] = &PrivateImpl::GetDTVAudioType;
        m_FuncMap["SetDTVAudioPrimaryLang"] = &PrivateImpl::SetDTVAudioPrimaryLang;
        m_FuncMap["GetDTVAudioPrimaryLang"] = &PrivateImpl::GetDTVAudioPrimaryLang;
        m_FuncMap["SetDTVAudioSecondaryLang"] = &PrivateImpl::SetDTVAudioSecondaryLang;
        m_FuncMap["GetDTVAudioSecondaryLang"] = &PrivateImpl::GetDTVAudioSecondaryLang;
#ifdef ENABLE_NEW_DVB_2
        m_FuncMap["SetDTVSubtitleType"] = &PrivateImpl::SetDTVSubtitleType;
        m_FuncMap["GetDTVSubtitleType"] = &PrivateImpl::GetDTVSubtitleType;
#endif
#if defined(DVB_SUBTITLE)
        m_FuncMap["SetDTVSubtitlePrimaryLang"] = &PrivateImpl::SetDTVSubtitlePrimaryLang;
        m_FuncMap["GetDTVSubtitlePrimaryLang"] = &PrivateImpl::GetDTVSubtitlePrimaryLang;
        m_FuncMap["SetDTVSubtitleSecondaryLang"] = &PrivateImpl::SetDTVSubtitleSecondaryLang;
        m_FuncMap["GetDTVSubtitleSecondaryLang"] = &PrivateImpl::GetDTVSubtitleSecondaryLang;
#endif //defined(DVB_SUBTITLE)
        m_FuncMap["SetATVTableScan"] = &PrivateImpl::SetATVTableScan;
        m_FuncMap["GetATVTableScan"] = &PrivateImpl::GetATVTableScan;
        m_FuncMap["GetIsNoSignal"] = &PrivateImpl::GetIsNoSignal;
        m_FuncMap["GetEpgData"] = &PrivateImpl::GetEpgData;
        m_FuncMap["GetEpgDataByLCN"] = &PrivateImpl::GetEpgDataByLCN;
        m_FuncMap["GetEpgListEpgCount"] = &PrivateImpl::GetEpgListEpgCount; 
        m_FuncMap["GetAllLCNByCurFreq"] = &PrivateImpl::GetAllLCNByCurFreq;
        m_FuncMap["GetEpgDataList"] = &PrivateImpl::GetEpgDataList;
        m_FuncMap["SetTVStopMode"] = &PrivateImpl::SetTVStopMode;
        m_FuncMap["GetTVStopMode"] = &PrivateImpl::GetTVStopMode;
    }
```


- ### 传输数据的序列/流化

　　类`RpcCommandMuxer`和类`RpcCommandDemuxer`是用于RpcClient与RpcServer之间沟通数据的序列化与反序列化。
　　
　　类`IpcStreamerPipeImpl`是用于数据的流化处理。

---

## 示例

参见 **`RpcTestJni.cpp`** 源文件。

---
