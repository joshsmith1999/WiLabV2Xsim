## findRCintervalAutonomous.m

根据RRI的大小确定RC的随机范围应该设置在那里：

| RRI           | RC range |
| ------------- | -------- |
| $$ >=100ms $$ | [5,15]   |
| $$ >=50ms $$  | [10,30]  |
| else          | [25,75]  |

## CV2XsensingProcedure.m

这个函数显而易见表示的是半持续调度算法SPS中的sensing过程，在这里简述一下SPS算法的感知过程是如何运行的。

首先感知过程是在一个感知窗口（SW：Sensing Window）中进行的，窗口的持续时间为**100ms~1100ms**，在这个过程中，UE会去感知和解码其它UE发过来的SCI消息，解码后的SCI与RSRP的测量一起存储，该信息用于确定当需要进行新的资源选择时必须排除哪些资源。在模式2中，资源可以被排除在可能的候选资源之外，当且仅当相关信息不可用(例如,由于测站发射而没有进行测量)或者它们被具有高于给定阈值的相关RSRP的先前SCI保留。在进行了排除资源这一个步骤后，资源会随机从剩下的可用候选资源中进行随机的选择。

这个函数里一个很关键的点是去根据情况更新**感知矩阵sensing matrix**，它具有三维信息：

- 1st D: 储存在时域的值的数目，对应于1s的标准持续时间，为ceil(1/Tbeacon);
- 2nd D: BRid;
- 3rd D: 车辆ID



- Part 1 更新感知矩阵Sensing Matrix

首先将LTE-LTE的感知功率保存到**sensedPowerCurrentSF_MHz**，然后将那些在当前子帧感知的功率低于功率门限值的数抹零。然后就可以更新感知矩阵Sensing matrix了。

- Part 2 更新knownUsedMatrix

即从SCI消息读取的状态。

RC=1时，knowUsedMatrix的值设置为0并且告知资源已经不能为下次传输周期进行预留了，RC大于1时，knowUesdMatrix进行更新。

## BRreassignment3GPPautonomous.m

这个函数主要讲的就是3GPP定义的Mode 4，然后现在支持了5G的Mode 2。

- Part 1 检查这个站点是否需要重选
- Part 2 执行重选

其中，LTE-V2X中计算调度的感知功率是使用平均RSSI，而5G-V2X中，使用的是感知仅仅是解码SCI消息的基础之上进行。还有5G Mode 2允许所有的资源是非半双工的或者预留RSRP高于阈值的，并且L2表被移除了。