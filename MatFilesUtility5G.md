## calculateNB_5G.m

这个函数的主要作用就是计算**NbeaconsF**和**NbeaconsT**。

因为5G没有相邻和非相邻的模式，所以SCI沿着传输块TB传输。

## findRBsBeaconSINRmin_5G.m

该函数主要是计算每个数据的子信道数量以及最小所需要的SINR。

## findTBS_5G.m

基于给到的一些参数去找到合适的TBS。

## fTBS_5G.m

这个函数的功能主要是在Ninfo<=3824的情况下返回TBS。

## getMCS.m

基于“MCS.txt”去获取到对应的MCS

