# 流程
1. 根据初始化，选定Context策略
1. 采样Context并存储到Context pool中

# 实现
## 注册Context策略
1. regime similar(pair类型，每轮选择两个regime（可能是相同的），可能引入0/1次el switch，可能引入0/1次tlbi过程)
    1. regime select
        >```
        >                    vas==2
        >                    e2h==1
        >                    tge==0             vas==2
        >                  +---------> nsel20 <-------> nsel20
        >                  |             ^
        >                  |  vas==1     | e2h=1
        >                  |  e2h==0     | vas==1
        >                  |  tge==0     v
        >                  +---------> nsel2             sel20 <---->sel20
        >                  |             ^                 ^    vas==2
        >                  |    vas==1   |     vas==1      | e2h=1
        >                  v    tge=1    v     eel2=1      v vas==1
        >                nsel1 <-----> sel3 <----------> sel2
        > nsel10 <-----> nsel10          ^                 ^
        >         vas==2                 |     vas==1      |
        >                                v     eel2==1     |
        >                              sel1 <--------------+
        >                sel10 <-----> sel10
        >                       vas==2
        >```
    1. similar field
        1. same field
            1. total same
            1. same effective field and random ignore field
        1. different field
            1. pick random num - 1, 2, no constraint
                1. different exception report syndrome(paging enabled - M,EPD,NFD)
                1. different hit/miss tlb(asid, vmid)
    1. 对应的VmContextId数量上限计算
        1. 每个regime下Context上限为2^(∑(field num)bits value option per field),如field的num为2，分别为1bit和2bit，那么上限为2^(1+2)=8
        1. 满足regime silmilar策略的组合上限为2*(单regime VmContextId上限^2)
1. regime couterpart(pair类型，每轮选择两个regime，可能引入超过一个el switch过程，不会引入tlbi过程)
    1. same el and stage, different ns, others same
    1. same ns and stage, different el, others same
1. part stage share（pair类型，每轮选择1个regime，但是一定有一个stage状态不同，可能引入0/2次el switch（switch higher to switch stage and switch back），可能引入0/1次tlbi）
    1. same el and ns, different stage, share stage on context
        > support single stage
        > ```
        > off<---->on
        > ```
        > support two stage
        > ```
        > offoff<---->onoff
        >    ^          ^
        >    |          |
        >    v          v
        > offon<----->onon
        > ```
 
## 采样
### 参数
1. 轮数，每轮选择一个策略（random，similar，counterparts，stageshare）
1. 每轮的数量，与策略相关，策略决定了数量的上限
1. 采样函数接受regime和field value的确定值, 确定值来自于这一轮的策略选择及数目的随机
## 历史Context记录与Switch处理<font color=yellow> TODO </font>
<font color=red>记录原则：根据VmContexId，计算对应的Context Pool Info, 并按顺序记录。</font>

<font color=red>处理原则：支持两类TLBI处理策略(immediately/lazy, default lazy)。
根据历史Context Pool Info, 仅处理最近一次相同的硬件Context Info，得到处理策略返回的指令并执行。</font>
### 实现方案
#### 记录
1. switch函数完成后，得到当前的VmContextId，如果有两级stage，则每个stage的VmContextId都需要被记录，且记录前检查历史是否有重复，有则不再进行记录。
1. switch函数完成后，得到delay的TLBI instr，并以Regime为key进行记录。
#### TLBI处理 
1. 判断是否需要issue TLBI，API每次switch前，检查该线程下的历史信息。（<font color=red>target的每个stage的VmContextId都要查</font>）
    1. 若历史信息与target VmContextId同regime，且delta非空，则根据delta issue TLBI（py/arm下默认检查场景）
    1. 若历史信息与target VmContextId regime仅有Security不一致，且如果同Security需要issue TLBI，则此时也必须issue TLBI（<font color=red> 新增场景1</font>）
    1. 若历史信息与target VmContext为同EL的同Secure的不同Regime（EL2/EL20），则必须issue TLBI，TLBI ALLE2（<font color=red> 新增场景2</font>）
    1. 若历史信息与target VmContextId同Regime，且有同VMID的delta非空的场景，且target VmContextId涉及VMID切换（VS/VMID），则必须发TLBI，且该TLBI必须在VMID切换完成后发（<font color=red> 新增场景3</font>）
1. issue TLBI时机
    1. immediately类型，一般场景在switch完成后issue TLBI；对于新增场景3，需要在切换完VMID后发。不会记录到delay的TLBI instr。
    1. delay类型，如果target VmContextId的Regime和Switch后停留的regime不同，则记录到delay TLBI instr；如果相同，则先计算此次是否需要TLBI，并和历史记录的TLBI instr合并在switch完成后发送。特殊情况，如果通过enRegimeSwitch的方式，当下并没有实质Context切换的场景下，也需要查看历史信息发TLBI。