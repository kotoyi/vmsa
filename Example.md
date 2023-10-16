# Arsenal — API
batchReload

xxxIsEnable

# Library — 功能序列
## long latency
### iex long latency
DIV
### fsu long latency
FDIV-256
### lsu long latency
ld device

## Power - Noise
1. def \_\_call__
选择power策略： 如果isWfiEnabled, 选择一个power函数执行

## Wfx - Noise
1. def \_\_call__
选择wfx策略：根据不同的context选择不同的wfx map,如果有wfxt, 则随机xt寄存器的值，
1. config：可以更新寄存器配置
<font color=red> 
## Branch - 需要作为整体用例考虑 </font>
1. def \_\_call__
选择branch指令策略？
1. branchTo(va): 如果va有效(verifyVirtualAddress检查的是什么内容？)，则生成该va
1. binaryTree: 有条件taken和有条件不taken？

## Flush - Noise
1. def \_\_call__
选择flush策略，可以接受传参
1. robFlush: 共多种flush，通过eval flush type map得到robFlush的指令
1. bruFlush: 并未填充branch pattern，仅仅靠BLR和RET不匹配完成功能