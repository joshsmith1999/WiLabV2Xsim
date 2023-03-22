# mainV2X.m

![](https://image-kuang.oss-cn-shanghai.aliyuncs.com/typora/20220901152848.png)

## 1. 初始化

### 1.1 Settings

在MATLAB中关于上图中的settings存储在**ConfigFiles**中，该文件夹中存储了一系列配置文件，示例中采用的配置文件为**Highway3GPP.cfg**，这种文件后缀为cfg的文件其实就是存储了应用层、实施层、接入层、信道模型、移动数据等相关信息，接下来以**Highway3GPP.cfg**进行分析：

~~~ matlab
[seed]                  10 %用于随机数生成的种子
[simulationTime]        10 %仿真时间
rho]                   	35.5 - 61.5 - 122.5 %车辆密度
[rho]                   35.5
rho]                    12.5
vMean]                 	250 - 140 - 70 %车辆平均速度
Tbeacon]                0.1 - 0.1 - 0.2 %信标时间，默认值为0.1

[beaconSizeBytes]       190 %信标尺寸大小
[TypeOfScenario]        ETSI-Highway %场景类型
[Nlanes]                3 %车道数
[roadWidth]             4 %马路宽度
[roadLength]            2000 %马路长度
[vStDev]                0 %车速标准偏差，在Highway场景下默认为0

[F_dB]                  6 %接收器的噪声图
[Gt_dB]                 3 %发送机天线增益
[Gr_dB]                 3 %接收机天线增益

raw]                   100,200,300,400 %感知距离
[raw]                  300

folderPERcurves]	PERcurves/G5-HighwayLOS %PER vs SINR曲线对比所存储的文件

[MCS_LTE]               7 %LTE下编码调制方式
SINRthresholdLTE]      0.1 %用于LTE-V2X传输的SINR阈值
[BRAlgorithm]           18 %资源分配算法，因为添加了5G V2X所以数组设置为18            
[sizeSubchannel]        10 %在C-V2V子信道尺寸大小
[probResKeep]		0.4 %保持先前所选择的BR的概率
[FixedPdensity]		false %默认为false，如果为true，传输功率则会参考使用10MHz的功率，在信号只使用一部分的情况下进行缩放
[BRoverlapAllowed]  false %默认为false，如果为true，则允许信标资源重叠

rilModel11p]   true %当选用技术不是C-V2X时，集火一个模型来考虑ITS-G5中短强干扰的破坏性效应，默认为false

MCS_11p]               2 %802.11p下编码调制方式
SINRthreshold11p_LOS]      3.1 %对于IEEE 802.11p传输下的SINR阈值

[printPacketReceptionRatio]  true %记录从0到最大感知距离下的数据包接收率并打印出来
[printUpdateDelay]      true %记录连续成功接收信标之间的更新时延并但打印出来
[printPacketDelay]      true %记录连续成功接收信标之间的数据包延迟并打印出来
[printDataAge]          true %成功接收信标的数据寿命并打印出来
[printWirelessBlindSpot]	true %*这里文件里没有定义，等下再找一找

[printNeighbors]        true %打印邻居数量
[printPRRmap]           true %当场景类型为Traces，打印PRR映射
[printPowerControl] 	false %打印资源分配的相关文件，这里默认为false
[printHiddenNodeProb] 	false %打印
[printCBR]              true %打印CBR的累计分布函数图CDF

[mode5G]                1 %即支持5G下的分析模式
[ifAdjacent]            true %在LTE-V2V中允许PSCCH和PSSCH相邻
[SCS_NR]                15 %NR V2X的SCS设置，这里设置为15kHz
[nDMRS_NR]              24 %设置每个RB中每个slot使用的DMRS资源元素的数量
[MCS_NR]                7 %当所选技术为NR V2X时，调制编码方式，范围为 0~28
~~~

### 1.2 Initialization

首先使用了一个**mainInit.m**进行初始化，所以接下来继续学习**MainFiles**文件夹下的**mainInit.m**函数，在**mainV2X.m**中Initialization部分为：

~~~ matlab
%% Initialization
[appParams,simParams,phyParams,outParams,simValues,outputValues,...
    sinrManagement,timeManagement,positionManagement,stationManagement] = mainInit(appParams,simParams,phyParams,outParams,simValues,outputValues,positionManagement);

% The simulation starts at time '0'
timeManagement.timeNow = 0;

% The variable 'timeNextPrint' is used only for printing purposes
timeNextPrint = 0;

% The variable minNextSuperframe is used in the case of coexistence
minNextSuperframe = min(timeManagement.coex_timeNextSuperframe);
~~~

现在开始看一下**mainInit.m**：

先观察函数包含哪些参数：

~~~ matlab
function [appParams,simParams,phyParams,outParams,simValues,outputValues,...
    sinrManagement,timeManagement,positionManagement,stationManagement] = mainInit(appParams,simParams,phyParams,outParams,simValues,outputValues,positionManagement)
~~~

初始化使用的一些参数分析：

| 参数               | 解释         |
| ------------------ | ------------ |
| appParams          | 应用层参数   |
| simParams          | 仿真参数     |
| phyParams          | 物理层参数   |
| outParams          | 输出参数     |
| simValues          | 仿真值       |
| outputValues       | 输出值       |
| positionManagement | 位置管理     |
| sinrManagement     | 信干噪比管理 |
| timeManagement     | 时间管理     |
| stationManagement  | 站点管理     |

#### 1.2.1 对于活动车辆和状态进行初始化

~~~ matlab
%% Init of active vehicles and states
% Move IDvehicle from simValues to station Management
stationManagement.activeIDs = simValues.IDvehicle;
simValues = rmfield(simValues,'IDvehicle');

% The simulation starts at time '0'
timeManagement.timeNow = 0;

% State of each node
% Discriminates C-V2X nodes from 11p nodes
if simParams.technology==1
    
    % All vehicles in C-V2X (lte or 5g) are currently in the same state
    % 100 = LTE TX/RX
    stationManagement.vehicleState = 100 * ones(simValues.maxID,1);
 
elseif simParams.technology==2
   
    % The possible states in 11p are four:
    % 1 = IDLE :    the node has no packet and senses the medium as free
    % 2 = BACKOFF : the node has a packet to transmit and senses the medium as
    %               free; it is thus performing the backoff
    % 3 = TX :      the node is transmitting
    % 9 = RX :      the node is sensing the medium as busy and possibly receiving
    %               a packet (the sender it firstly sensed is saved in
    %               idFromWhichRx)
    stationManagement.vehicleState = ones(simValues.maxID,1);

else % coexistence
    
    % Init all as LTE
    stationManagement.vehicleState = 100 * ones(simValues.maxID,1);
    %Then use simParams.numVehiclesLTE and simParams.numVehicles11p to
    %initialize
    for i11p = 1:simParams.numVehicles11p
        stationManagement.vehicleState(simParams.numVehiclesLTE+i11p:simParams.numVehiclesLTE+simParams.numVehicles11p:end) = 1;
    end
        
%     % POSSIBLE OPTION FOR DEBUG PURPOSES
%     % First half 11p, Second half LTE
%     stationManagement.vehicleState = 100*ones(simValues.maxID,1);
%     stationManagement.vehicleState(1:1:end/2) = 1;

end

% RSUs technology set
if appParams.nRSUs>0
    if strcmpi(appParams.RSU_technology,'11p')
        stationManagement.vehicleState(end-appParams.nRSUs+1:end) = 1;
    elseif strcmpi(appParams.RSU_technology,'LTE')
        stationManagement.vehicleState(end-appParams.nRSUs+1:end) = 100;
    end
end

~~~

#### 1.2.2 对每个技术的活动车辆矩阵进行初始化

~~~ matlab
%% Initialization of the vectors of active vehicles in each technology, 
% which is helpful to work with smaller vectors and matrixes
stationManagement.activeIDsCV2X = stationManagement.activeIDs.*(stationManagement.vehicleState(stationManagement.activeIDs)==100);
stationManagement.activeIDsCV2X = stationManagement.activeIDsCV2X(stationManagement.activeIDsCV2X>0);
stationManagement.activeIDs11p = stationManagement.activeIDs.*(stationManagement.vehicleState(stationManagement.activeIDs)~=100);
stationManagement.activeIDs11p = stationManagement.activeIDs11p(stationManagement.activeIDs11p>0);
stationManagement.indexInActiveIDs_ofLTEnodes = zeros(length(stationManagement.activeIDsCV2X),1);
for i=1:length(stationManagement.activeIDsCV2X)
    stationManagement.indexInActiveIDs_ofLTEnodes(i) = find(stationManagement.activeIDs==stationManagement.activeIDsCV2X(i));
end
stationManagement.indexInActiveIDs_of11pnodes = zeros(length(stationManagement.activeIDs11p),1);
for i=1:length(stationManagement.activeIDs11p)
    stationManagement.indexInActiveIDs_of11pnodes(i) = find(stationManagement.activeIDs==stationManagement.activeIDs11p(i));
end
~~~

上面这段代码主要的作用就是将两种不同技术的车辆区分开来并且进行初始化。

#### 1.2.3 数据包管理初始化

有两种数据类型，分别为CAM和DEMN。

#### 1.2.4 数据包生成

#### 1.2.5 传播初始化

~~~ matlab
%% Initialize propagation
% Tx power vectors
if isfield(phyParams,'P_ERP_MHz_CV2X')
    if phyParams.FixedPdensity
        % Power density is fixed, must be scaled based on subchannels
        phyParams.P_ERP_MHz_CV2X = phyParams.P_ERP_MHz_CV2X * (phyParams.NsubchannelsBeacon/phyParams.NsubchannelsFrequency);
    end %else % Power is fixed, independently to the used bandwidth
    phyParams.P_ERP_MHz_CV2X = phyParams.P_ERP_MHz_CV2X*ones(simValues.maxID,1);
else
    phyParams.P_ERP_MHz_CV2X = -ones(simValues.maxID,1);
end
if isfield(phyParams,'P_ERP_MHz_11p')
    phyParams.P_ERP_MHz_11p = phyParams.P_ERP_MHz_11p*ones(simValues.maxID,1);
else
    phyParams.P_ERP_MHz_11p = -ones(simValues.maxID,1);
end

% Vehicles in a technology have the power to -1000 in the other; this is helpful for
% verification purposes
phyParams.P_ERP_MHz_CV2X(stationManagement.vehicleState~=100) = -1000;
phyParams.P_ERP_MHz_11p(stationManagement.vehicleState==100) = -1000;
~~~

#### 1.2.6 信道初始化

~~~ matlab
%% Channels
stationManagement.vehicleChannel = ones(simValues.maxID,1);
% NOTE: sinrManagement.mcoCoefficient( RECEIVER, TRANSMITTER) 
sinrManagement.mcoCoefficient = ones(simValues.maxID,simValues.maxID);
if phyParams.nChannels>1
    [stationManagement,sinrManagement,phyParams] = mco_channelInit(stationManagement,sinrManagement,simValues,phyParams);
end

% Shadowing matrix
sinrManagement.Shadowing_dB = randn(length(stationManagement.activeIDs),length(stationManagement.activeIDs))*phyParams.stdDevShadowLOS_dB;
%triu()返回矩阵上三角元素
sinrManagement.Shadowing_dB = triu(sinrManagement.Shadowing_dB,1)+triu(sinrManagement.Shadowing_dB)';
~~~

#### 1.2.7 坐标和距离管理

## 2. Action

上面主要就是针对mainInit.m函数进行了解释，现在进入mainV2X函数的下一部分，即Simulation Cycle，而Simulation Cycle主要是分为三部分：

- Position Update
- Quality assessment
- Radio resources reassignment and blocking events

### 2.1 Position Update

现在分析Position Update相关函数**mainPositionUpdate.m**,首先看函数申明部分：

~~~ matlab
function [appParams,simParams,phyParams,outParams,simValues,outputValues,timeManagement,positionManagement,sinrManagement,stationManagement] = ...
    mainPositionUpdate(appParams,simParams,phyParams,outParams,simValues,outputValues,timeManagement,positionManagement,sinrManagement,stationManagement)
~~~

这上面的参数其实已经在**mainInit.m**中进行了解释，这里就不过多赘述。在该函数中，主要使用了一个函数**updatePosition.m**，该函数的主要功能为实时更新车辆的位置。

这里根据伪代码来解释这个Action的Simulation Cycle:

~~~ matlab
%当时间管理记录现在的时间小于仿真的时间的话，那么执行这个while语句
while timeManagement.timeNow < simParams.simulationTime
	%找到那个最小的时刻开始，然后记录下此时那个event下的车辆ID，这样也获取了车辆矢量的索引indexEvent
	[timeEvent, indexEvent] = min(timeManagement.timeNextEvent(stationManagement.activeIDs))；
	%如果timeEvent晚于下列event的发生，则需要把timeEvent设置成下列event的时间，例如：
	if timeEvent >= timeManagement.timeNextCV2X
       timeEvent = timeManagement.timeNextCV2X;
       %fprintf('LTE subframe %.6f\n',timeEvent);
    end
    %代码中还有很多timeEvent与其它event的比较，这么做的主要目的就是为了校正timeEvent设定时间：如果时间事件晚于下一个LTE事件，就设定时间为LTE时间；如果时间事件晚于下一次位置更新，则设置时间为位置更新；如果仿真时间到了，就结束，即：
    if round(timeManagement.timeNow*1e10)/1e10>=round(simParams.simulationTime*1e10)/1e10
        break;
    end
    %接下来就是真正执行Action的步骤，首先是Positon Update：
    %如果timeEvent等于下一位置更新的时间，则执行以下位置更新函数：
    if timeEvent==timeManagement.timeNextPosUpdate
    	mainPositionUpdate();
    %然后重新设定一下下一位置更新的时间；
    %如果timeEvent等于下一CBR更新的时间则：
    elseif timeEvent == timeManagement.timeNextCBRupdate
    	cbrUpdate11p()；
    	cbrUpdateCV2X();
    %如果timeEvent等于下一超级帧，则：
    elseif timeEvent == minNextSuperframe
    	superframeManagement();
    %如果timeEvent等于C-V2X时间，则：
    elseif timeEvent == timeManagement.timeNextCV2X
    	mainCV2XttiStarts();
    %接下来就是Packet generation：
    elseif timeEvent == timeManagement.timeNextPacket(idEvent)
    	%根据vehiclestate区分情况，100为LTE
    	bufferOverflowLTE();
    	%如果state不为100，则：
    	 newPacketIn11p();
    	 %针对NR V2X非周期性流量的生成，所以hi需要加一个随机的元素进去：
    	 generationInterval = 		                 timeManagement.generationIntervalDeterministicPart(idEvent)+exprnd(appParams.generationIntervalAverageRandomPart);
    	 %exprnd()服从指数分布的随机数
~~~

上面就是简单地学习了一下**mainV2X.m**这个函数，后面还是需要进一步仔细学习的，现在是把它整体框架根据开头那张图结合代码了解了一遍，通过未来的学习进一步加强对该工具的使用操作。



**注：这次在2022/11文件的测试日志中对mainV2X.m结合实验进一步分析了这个主函数，具体细节详细看2022.11文件**
