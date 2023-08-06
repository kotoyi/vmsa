# 场景
1. 指令流功能（TLBI+IC+CFP+CPP）
1. exception fault
1. xxx

# 分工
## page池  
（attribute，permission()，
fault：s1: af, translation, address size, s2:..., 
paging_pattern: coltl1, congtiguous）

## 指令池
### 指令类型
load，store，load/store，pre-index，post-index，
branchimm，branchany，
config——MSR，store，pre-index，post-index，branchimm，branchany，branchlink
check——load，load/store
### 指令特征
可跨页指令：size大于1B，可能不检查align
多uop指令

## context池


## 组合
1. 控制下的指令序
1. context序：config
1. 指令序：paging+config
1. PC序：paging+config



# library池
1. regime选择（仅用于main函数）：
1. target regime：
```
if target regime need switch EEL2 or E2H，then target vmcontext id need write eel2 or e2h
		（runSwitch need adapt the regime，first switch EEL2 and E2H——if EL2 switch E2H，then need mmu off， and check whether TCR_EL2 location）
```
1. context切换可能涉及到PC失控？

1. exception（？？）
1. always fault（与context配置无关） and 不改变pc流向：data always fault
1. maybe install tlb：permission fault，
1. update：af，ap，af&ap

# 多线程library
## BBM
## IC

# 单线程library

# 可重复library（作为subroutine body或者loop body）


# 用例分类
## precise pc and data:
不允许
## precise pc and unpredictable data:

same context：
context switch for va（允许）：
context switch for 


# 用例流程
## main函数
1. precise result（全部都可以与rm cosim的场景）：不允许多核unpredictable 场景（BBM）