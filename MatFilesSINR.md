# MatFilesSINR文件夹

这个文件夹里主要由四个函数组成：

## calculateNLOSv.m

顾名思义就是计算NLOS(Non-Line Of Sight)的函数：

### What Does Non-Line Of Sight (NLOS) Mean?

非视距(NLOS)是指无线电频率(RF)的传播路径被障碍物遮挡(部分或完全) ，从而使无线电信号难以通过。无线电发射机和无线电接收机之间常见的障碍是高层建筑、树木、自然景观和高压电源导体。有些障碍物吸收无线电信号，有些障碍物反射无线电信号，这些障碍物都限制了信号的传输能力。

这个函数的输入主要是X和Y即在坐标系下车辆的x坐标和y坐标；输出为NLOSv即指的是大小等于车辆数量的平方矩阵，表示有多少车辆阻碍了与给定行和列对应的车辆的视线（矩阵始终与主对角线对称，主对角线上为零）

**ps：**车辆数量由X的长度决定。

这个因为不会用到它的细节，知道函数的作用即可。

## computeChannelGain.m

这个函数的作用是计算信道增益的。具体来说就是用来计算接收功率和创建RXpower矩阵的函数。

## deriveRanges.m

这个函数的主要作用是根据所选算法推导最大感知范围和其他范围。

这里主要针对C-V2X的最大感知距离计算进行调研，主要是找到推导公式：

~~~ matlab
phyParams.RawMaxLOSCV2X = ((phyParams.P_ERP_MHz_CV2X*phyParams.Gr)/(phyParams.sinrThresholdCV2X_LOS*phyParams.L0_far*phyParams.Pnoise_MHz))^(1/phyParams.b_far);
    phyParams.RawMaxNLOSCV2X = ((phyParams.P_ERP_MHz_CV2X*phyParams.Gr)/(phyParams.sinrThresholdCV2X_NLOS*phyParams.L0_NLOS*phyParams.Pnoise_MHz))^(1/phyParams.b_NLOS);
    % Compute maximum range with 2 times standard deviation of shadowing in LOS (m)
    phyParams.RawMaxCV2X =  phyParams.RawMaxLOSCV2X * 10^((2*phyParams.stdDevShadowLOS_dB)/(10*phyParams.b_far));
~~~

这里分LOS和NLOS的：

| 参数                  | 定义               |
| --------------------- | ------------------ |
| P_ERP_MHz_CV2X        | C-V2X每MHz的功率   |
| Gr                    | 接收天线增益       |
| sinrThresholdCV2X_LOS | 对于CV2X的SINR阈值 |
| L0_far                |                    |
| Pnoise_MHz            |                    |
| b_far                 |                    |



## readPERtable.m

读取PER的表格，PER指的是接收错误的数据包

