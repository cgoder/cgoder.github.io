layout: tv
title: tv scan flow
date: 2016-07-22 13:58:45
categories:
 - work
tags:
 - DVB
---


翻出来以前随手写的一个文档，DTV搜索模块的简单抽象设计，原目的是为了设计一套中间层的，种种原因没能立项，这个半成品都不算的文档就丢尘埃里了。

<!-- more -->

### 1. 流程
分析频道搜索流程，大概分几个步骤。（以DTV为例）
1. 获取频率表；
2. 设置并锁定单个频率；
3. 获取单个频率上的数据（DTV为SI/PSI）；
4. 分析获取到的SI/PSI信息；
5. 保存SI/PSI中的信息到每个单独的分类数据库中；
6. 跳到第2步，循环，直至频率表中的频率全部扫完。


> `流程图如下：`

```flow
st=>start: 搜索开始
e=>end: 搜索结束
opGetFreqTable=>operation: 获取频率表
opGetNextFreq=>operation: 获取下一个频率
conGetFreqEnd=>condition: 频率表有数据？
opSetFreq=>operation: 设置频率
conLockTuner=>condition: 锁频成功？
opGetTable=>operation: 获取SI/PSI
conGetTable=>condition: 获取成功？
opParseTable=>operation: 分析SI/PSI
conPareTable=>condition: 分析成功？
opSaveData=>operation: 存储channel数据

st->opGetFreqTable->conGetFreqEnd->
conGetFreqEnd(no)->e
conGetFreqEnd(yes)->opGetNextFreq->opSetFreq->conLockTuner->
conLockTuner(yes)->opGetTable->conGetTable->
conLockTuner(no)->conGetFreqEnd->
conGetTable(yes)->opParseTable->conPareTable->
conGetTable(no)->conGetFreqEnd->
conPareTable(yes)->opSaveData(left)->conGetFreqEnd->
conPareTable(no)->conGetFreqEnd->
```


### 2. 抽象 

电视信号根据信号传输类型分类，有DTV和ATV；而其中DTV又可分为DVB/ATSC/ISDB等；而其中DVB根据协议不同传输标准又可分为常见的DVBC/DVBT/DVBS/DVBH以及国内的变种DTMB/CMMB等。
电视信号在搜索频道时，由Tuner/Demod/Demux等几个模块组合完成，但是所有的都可以抽象出几个通用的动作。例如设置频率参数，搜索方式等。其中：

* 与FE物理硬件相关的动作和数据可以抽象出`class IFrontEnd`，包含Tuner和Demod的相关方法；
* 与频率相关的动作和数据可以抽象出`class IFreqTable`，包含频率表相关的方法与数据；
* 与Demux和Filter相关的动作和数据可以抽象出`class iDemux`，包含TSD/Demux/Filter的相关方法和数据；
* 与用户逻辑直接关联的动作可以抽象出`class IChannelScan`，包含搜索操作相关的方法。
* 搜索模块的输出数据可以抽象出`class IChannelData`，包含与频道数据相关的方法及数据。

> `class IFrontEnd` Tuner/Demod相关

```cpp
class IFrontEnd:
{
    virtual int tv_frontEnd_init(unsigned char u8TunerId) = 0; //init
    virtual int tv_frontEnd_deInit(unsigned char u8TunerId) = 0; // deinit
    virtual int tv_frontEnd_setBandwidth(unsigned char u8TunerId, unsigned int u32Bandwidth) = 0; //设置频率带宽
    virtual int tv_frontEnd_setFreq(unsigned char u8TunerId, unsigned int u32freq) = 0; //设置主频率或者基础频率。
    virtual int tv_frontEnd_setModulation(unsigned char u8TunerId, unsigned int u32Modulation) = 0; //设置调制方式。如DVBS的QPSK，DVBC的QAM，DVBT的COFDM等
    virtual int tv_frontEnd_setSymbolrate(unsigned char u8TunerId, unsigned int u32SymbolRate) = 0; //设置符号率
    virtual int tv_frontEnd_getLockStatus(unsigned char u8TunerId) = 0; //
    virtual int tv_frontEnd_getSignalSNR(unsigned char u8TunerId) = 0; //Signal to Noise Ratio - which means signal quality
    virtual int tv_frontEnd_getSignalBER(unsigned char u8TunerId) = 0; //Bit Error Rate - which shows the error rate of the signal
    virtual int tv_frontEnd_getSignalAGC(unsigned char u8TunerId) = 0; // Automatic Gain Control - which means signal strength
}
```

>  `class IFreqTable` 频率表相关

```cpp
class IFreqTable:
{
public:
    virtual int tv_freqTable_set(char* pFilePath) = 0;
    virtual int tv_freqTable_set(struct FreqNode* freqList) = 0;
    virtual struct FreqNode* tv_freqTable_get(void) = 0;
    virtual struct FreqNode* tv_freqTable_getFreq(unsigned int u32FreqIndex) = 0;
private:
    struct FreqNode* m_freqList;
};

typedef struct FreqNode 
{
    unsigned int u32Freq;
    unsigned int u32Modulation;
    unsigned int u32SymbolRate;
    struct FreqNode* prev;
    struct FreqNode* next;
};
```

> `class IDemux` TSD/Demux/Filter相关

```cpp
class IDemux:
{
public:
    tv_demux_init() = 0;
    tv_demux_deInit() = 0;
    tv_demux_open(unsigned char u8DemuxId) = 0;
    tv_demux_close(unsigned char u8DemuxId) = 0;
    tv_demux_parseSi(unsigned char* siBuffer) = 0;
    tv_demux_parsePsi(unsigned char* psiBuffer) = 0;
    //TODO:
private:
    //TODO:
}
```

> `class IChannelScan` 搜索逻辑相关

搜索的动作，分为几种。

```cpp
//自动搜索: `
int tv_channel_scan_auto(unsigned int u32StartFreq, unsigned int u32EndFreq); //full模式。根据频率带宽在全频率表范围内自动递增搜索。
int tv_channel_scan_auto(unsigned int u32mainFreq); //quick模式。以中心主频率上的NIT表数据为准。

//手动搜索：
int tv_channel_scan_manual(unsigned int u32StartFreq, unsigned int u32EndFreq); 
int tv_channel_scan_manual(unsigned int u32StartFreq, unsigned int u32EndFreq, unsigned int u32Modulation, unsigned int u32SymbolRate);
```

抽象几种搜索方式，可以得到如下接口类：

```cpp
class IChannelScan:
{
public:
    virtual int tv_channelScan(class FreqTable freqTable);
    virtual int tv_channelScan_auto(unsigned int u32mainFreq) = 0;
    virtual int tv_channelScan_auto(bool bNit) = 0;
    virtual int tv_channelScan_auto(unsigned int u32StartFreq, unsigned int u32EndFreq) = 0;
    virtual int tv_channelScan_manual(unsigned int u32StartFreq, unsigned int u32EndFreq) = 0;
    virtual int tv_channelScan_auto_stop() = 0;
    virtual int tv_channelScan_auto_pause() = 0;
    virtual int tv_channelScan_manual_stop() = 0;
    virtual int tv_channelScan_complete() = 0;
private:
    class IFreqTable m_freqTable;
};
```

> `class IChannelData` 频道数据相关

```cpp
class IChannelData:
{
public:
    vritual int tv_ChannelData_saveData() = 0;
private:
    //TODO:
}
```


### 3.  接口 
抽象类的实际对象接口设计

```cpp
class DtvChannelScan: public IChannelScan, public IFrontEnd, public IDemux, public IChannelScan, public IChannelData
{
public:
    int tv_channelScan(class FreqTable freqTable) {
        assert(freqTable);
        assert(tv_frontEnd_init());
        
        struct FreqNode* freqList = m_freqTable::tv_freqTable_get();
        //TODO:
        foreach freqList[] {
            tv_frontEnd_setFreq(u8TunerIndex, u32Freq);
            if (tv_frontEnd_getLockStatus(u8TunerIndex)) {
                assert(tv_demux_init());
                tv_demux_open();
                tv_demux_SetFilter();
                while(--timeout) {
                    if (SI) {
                        tv_demux_parseSi(SDT);
                        tv_demux_parseSi(BAT);
                        tv_demux_parseSi(EIT);
                        tv_demux_parseSi(TDT);
                        tv_demux_parseSi(TOT);
                    }
                    if (PSI) {
                        tv_demux_parsePsi(NIT);
                        tv_demux_parsePsi(PAT);
                        tv_demux_parsePsi(PMT);
                        tv_demux_parsePsi(CAT);
                    }
                    tv_ChannelData_saveData();
                }
            }
        }
    }
    int tv_channelScan_auto(unsigned int u32mainFreq);
    int tv_channelScan_manual();
private:
    class IFreqTable m_freqTable;

};
```

### 4. TODO：
