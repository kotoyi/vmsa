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
1. 采样函数接受regime和field value的确定值
## 历史Context记录与Switch处理<font color=yellow> TODO </font>
<font color=red>记录原则：根据VmContexId，计算对应的Context Pool Info, 并按顺序记录。</font>

<font color=red>处理原则：支持两类TLBI处理策略(immediately/lazy, default lazy)。
根据历史Context Pool Info, 仅处理最近一次相同的硬件Context Info，得到处理策略返回的指令并执行。</font>
### 实现方案
1. API每次switch前，检查该线程下的历史信息，若历史信息与target VmContextId must issue TLBI
    1. 如果switch前可以发target TLBI，如ASID
1. API每次switch成功，return一个VmContextId