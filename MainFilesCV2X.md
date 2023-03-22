# MainFilesCV2X文件夹

## updateSINRCV2X.m

- **该函数的功能主要是计算每个接受邻居的平均SINR。**

## updateKPICV2X.m

- **该函数主要是更新CV2X的KPI指标。**

在函数里，它分别计算了出差错的数量以及正确传输信标的数量。

## sensedPowerCV2X.m

- **该函数是通过在子帧上的LTE节点去计算功率。**

## mainCV2XttiStarts.m

- **该函数是一个C-V2X TTI(time transmission interval)的开始。**

与此对应的就是**mainCV2XttiEnds.m**，它是一个C-V2X TTI的结束函数。这里简单分析一下**mainCV2XttiEnds.m**:

~~~ matlab
%首先找出现在正在LTE进行传输的车辆索引值
activeIDsTXLTE = stationManagement.transmittingIDsCV2X;
indexInActiveIDsOnlyLTE = stationManagement.indexInActiveIDsOnlyLTE_OfTxLTE;
%在第一个BR分配后执行KPIs的计算
updateSINRCV2X();
%检查SCI消息的正确性
 if simParams.BRAlgorithm==18
        % correctSCImatrix is nTXLTE x nNeighblors
        stationManagement.correctSCImatrixCV2X = (sinrManagement.neighborsSINRsciAverageCV2X > phyParams.minSCIsinr);
        if simParams.BRAlgorithm==18
        	stationManagement.correctSCImatrixCV2X = (sinrManagement.neighborsSINRsciAverageCV2X > phyParams.minSCIsinr);
~~~

然后接着就是KPI的计算：

## initLastPowerCV2X.m

- **该函数是初始化上一次功率分配的函数。**

SINR计算为：

$$ SINR(i,j)=C/(PnRB+selfI+Itot)$$

## elaborateFateRxCV2X.m

- **检测正确解码的信标以及创建正确传输的列表。**

## counterTX.m

- 主要是用来计算在Raw范围内邻居正确传输的信标资源

- 获取的矩阵matrix总共有三维：

**matrix=[Correctly transmitted beacons, Errors, Neighbors]**

