# CSAPP Bomb Lab 详细解析与复习笔记


<!--more-->
# CSAPP Bomb Lab 复习笔记

## 一、实验简介：不止于“拆弹”的底层修炼
Bomb Lab 是《深入理解计算机系统》（CSAPP）第三章「程序的机器级表示」的核心配套实验。实验核心是通过反汇编二进制程序 `bomb`，分析 x86-64 汇编代码的逻辑，推断出 6 个“炸弹阶段”的输入字符串，解除炸弹。

**实验价值**：  
- 不是死记硬背答案，而是掌握「汇编指令解析」「栈帧结构」「函数调用约定」「控制流（循环、switch、递归、链表）底层实现」等核心知识；  
- 熟练使用 GDB 调试工具，建立“代码-汇编-内存”的映射思维，为后续操作系统、编译原理学习打基础。

## 二、核心基础：拆弹前必须掌握的“内功心法”
### 2.1 x86-64 寄存器使用惯例（重中之重）
x86-64 寄存器共 16 个（%rax-%r15），按功能分为三类，**必须牢记函数调用中的角色**：

| 寄存器       | 功能说明                                                                 |
|--------------|--------------------------------------------------------------------------|
| %rdi/%rsi/%rdx/%rcx/%r8/%r9 | 函数第 1-6 个参数（顺序严格对应，超过 6 个参数通过栈传递）               |
| %rax         | 函数返回值；同时是临时寄存器（调用者无需保存）                           |
| %rsp         | 栈顶指针（Stack Pointer）：push/pop 时自动增减，指向当前栈顶             |
| %rbp         | 栈底/帧指针（Base Pointer）：固定指向当前函数栈帧底部，用于定位局部变量   |
| %rbx/%r12-%r15 | 被调用者保存寄存器（Callee-saved）：函数使用前必须 push 保存，退出前 pop 恢复 |
| %r10/%r11    | 临时寄存器（Caller-saved）：调用者无需保存，被调用函数可随意修改           |

### 2.2 GDB 调试工具：拆弹必备“神器”
无需盲目猜测代码逻辑，用 GDB 直接查看内存、寄存器、指令执行过程，高效定位答案。

#### （1）基础操作：启动与反汇编
```bash
# 1. 导出完整反汇编代码
objdump -d bomb > bomb.asm

# 2. 启动 GDB 调试
gdb bomb
```

#### （2）断点与执行控制
| 指令                  | 功能说明                                                                 |
|-----------------------|--------------------------------------------------------------------------|
| `b phase_1`           | 在函数 `phase_1` 入口打断点（直接按函数名打断点，最常用）               |
| `b *0x400ee0`         | 在内存地址 0x400ee0 打断点（函数过长时，精准定位某条指令）               |
| `r`（run）            | 运行程序，输入字符串后触发断点                                           |
| `ni`（nexti）         | 单步跳过指令（不进入函数调用，如 `callq strings_not_equal` 直接执行完）  |
| `si`（stepi）         | 单步进入指令（进入函数内部，如查看 `strings_not_equal` 的执行逻辑）      |
| `c`（continue）       | 继续执行到下一个断点                                                     |
| `q`（quit）           | 退出 GDB                                                                 |

#### （3）数据查看：核心中的核心
| 指令                  | 功能说明                                                                 |
|-----------------------|--------------------------------------------------------------------------|
| `x/s 0x402400`        | 以字符串（String）格式查看地址 0x402400 处的内容（Phase 1 直接用）       |
| `x/d $rsp`            | 以十进制（d）查看栈顶（%rsp 指向）的值                                   |
| `x/6wd $rsp`          | 以十进制（d）查看栈顶开始的 6 个“字（4字节）”（查看数组/输入的多个整数） |
| `x/8xg 0x402470`      | 以 8 字节（x）、十六进制（g）查看跳转表（Phase 3 用）                    |
| `i r`（info registers）| 查看所有寄存器当前值                                                     |
| `p $rax`              | 打印寄存器 %rax 的值（验证函数返回值、计算结果）                         |
| `p *(int*)0x6032d0`   | 以 int 类型查看地址 0x6032d0 处的值（查看链表节点、结构体字段）          |

### 2.3 关键汇编指令解析：避坑指南
x86-64 汇编指令众多，重点掌握易混淆、高频出现的指令，避免因理解偏差踩坑。

#### （1）`lea` vs `mov`：地址 vs 数值
| 指令                  | 功能说明                                                                 |
|-----------------------|--------------------------------------------------------------------------|
| `mov (%rdi), %rax`    | 间接寻址：取 %rdi 指向的**内存地址中的值**，放入 %rax（访问内存）         |
| `lea (%rdi), %rax`    | 地址传送：将 %rdi 本身的**地址值**放入 %rax（不访问内存），常用于指针运算 |
| `lea (%rdi, %rdi, 2), %rax` | 计算 `%rdi * 3`（公式：基址 + 索引*比例，即 x + x*2 = 3x）               |

#### （2）`cmp` vs `test`：条件判断的核心
| 指令                  | 功能说明                                                                 |
|-----------------------|--------------------------------------------------------------------------|
| `cmp a, b`            | 计算 `b - a`，不保存结果，仅设置标志位（ZF=0 表示不等，ZF=1 表示相等）   |
| `test %eax, %eax`     | 计算 `%eax & %eax`，等价于检查 %eax 是否为 0（ZF=1 则为 0）              |
| `test %rdi, %rsi`     | 计算 `%rdi & %rsi`，检查两个值是否有共同的置位比特                       |

#### （3）跳转指令：标志位依赖
| 指令                  | 依赖标志位                                                               | 功能说明                                                                 |
|-----------------------|--------------------------------------------------------------------------|--------------------------------------------------------------------------|
| `je 0x400ef7`         | ZF=1                                                                     | 相等则跳转（jump equal）                                                 |
| `jne 0x400f3c`        | ZF=0                                                                     | 不等则跳转（jump not equal）                                             |
| `ja 0x400fad`         | CF=0 且 ZF=0                                                            | 无符号数大于则跳转（jump above）                                         |
| `jle 0x400f80`        | CF=1 或 ZF=1                                                            | 无符号数小于等于则跳转（jump less or equal）                              |

## 三、逐阶段拆弹：从易到难深度解析
### 3.1 Phase 1：字符串比较（热身题）
#### 核心考点：函数调用约定、字符串地址访问
#### 汇编逐行解析
```asm
phase_1:
  sub    $0x8,%rsp          ; 栈上分配 8 字节空间（对齐栈帧）
  mov    $0x402400,%esi     ; 第 2 个参数：目标字符串地址 0x402400 存入 %esi
  callq  401338 <strings_not_equal>  ; 调用字符串比较函数，第 1 个参数（输入字符串）在 %rdi
  test   %eax,%eax          ; 检查返回值：strings_not_equal 相等返回 0，不等返回 1
  je     400ef7             ; 若返回值为 0（ZF=1），跳转到安全区，炸弹不爆炸
  callq  40143a <explode_bomb>       ; 否则触发爆炸
```

#### 解法步骤
1. 启动 GDB：`gdb bomb`；
2. 打断点：`b phase_1`；
3. 运行程序：`r`，随便输入一个字符串（如 `test`），触发断点；
4. 查看目标字符串：`x/s 0x402400`，直接显示答案。

#### 答案验证
输入 GDB 查看到的字符串：`Border relations with Canada have never been better.`，Phase 1 解除。

### 3.2 Phase 2：循环与栈数组（序列推导）
#### 核心考点：栈上局部变量（数组）、循环结构、指针算术
#### 汇编逐行解析
```asm
phase_2:
  sub    $0x28,%rsp         ; 栈上分配 40 字节（6个int数组 + 栈对齐，6*4=24，补16字节对齐）
  lea    0x10(%rsp),%rsi    ; 第 2 个参数：数组起始地址（%rsp+16）存入 %rsi
  mov    %rsp,%rdi          ; 第 1 个参数：输入缓冲区地址存入 %rdi
  callq  40145c <read_six_numbers>  ; 读取 6 个整数，存入栈上数组（%rsp+16 开始）
  
  cmpl   $0x1,0x10(%rsp)    ; 检查数组第 1 个元素（%rsp+16）是否为 1
  jne    400f3c             ; 不是 1 则爆炸
  
  mov    $0x1,%ebx          ; %ebx = 1（循环变量 i=1，从第 2 个元素开始检查）
  mov    $0x18,%rbp         ; %rbp = 24（数组第 6 个元素地址：%rsp+16 + 5*4 = %rsp+36？不对，重新计算：
                            ; 数组起始地址 %rsp+16，元素索引 0-5，地址为 %rsp+16 + i*4
                            ; %rbp 是循环结束边界：%rsp+16 + 6*4 = %rsp+32 = 0x20？
loop_start:
  mov    -0x4(%rbx),%eax    ; 取前一个元素：数组[i-1]（%rbx 是当前元素指针，-4 是前一个元素地址）
  add    %eax,%eax          ; 前一个元素 *2（eax = eax * 2）
  cmp    %eax,(%rbx)        ; 比较当前元素（%rbx 指向）和前一个元素*2
  je     loop_continue      ; 相等则继续循环
  callq  <explode_bomb>     ; 不等则爆炸
loop_continue:
  add    $0x4,%rbx          ; 指针移动到下一个元素（int 占 4 字节）
  cmp    %rbp,%rbx          ; 检查是否到达循环边界
  jne    loop_start         ; 未到则继续循环
  add    $0x28,%rsp         ; 释放栈空间
  retq                      ; 函数返回，Phase 2 解除
```

#### 核心逻辑推导
- 输入要求：6 个整数（`read_six_numbers` 限制）；
- 第 1 个元素必须为 1；
- 从第 2 个元素开始，每个元素 = 前一个元素 × 2（等比数列）。

#### 解法步骤
1. 断点：`b phase_2`，运行 `r` 后输入 `1 2 4 8 16 32`；
2. 验证：GDB 中用 `x/6wd $rsp+16` 查看栈上数组，确认每个元素符合规律；
3. 执行 `c`，炸弹不爆炸，Phase 2 解除。

#### 答案
`1 2 4 8 16 32`

### 3.3 Phase 3：Switch 跳转表（多路分支）
#### 核心考点：switch-case 底层实现（跳转表）、多参数输入解析
#### 汇编逐行解析
```asm
phase_3:
  sub    $0x18,%rsp         ; 栈上分配 24 字节
  lea    0xc(%rsp),%rcx     ; 第 3 个参数：&y（第二个输入整数的地址）存入 %rcx
  lea    0x8(%rsp),%rdx     ; 第 2 个参数：&x（第一个输入整数的地址）存入 %rdx
  mov    $0x4025c3,%esi     ; 第 1 个参数：格式化字符串 "%d %d" 地址存入 %esi
  mov    %rsp,%rdi          ; 第 0 个参数：输入缓冲区地址存入 %rdi
  callq  401353 <__isoc99_sscanf>  ; 调用 sscanf(input, "%d %d", &x, &y)
  
  cmp    $0x1,%eax          ; sscanf 返回成功读取的参数个数，需 ≥1
  jbe    400fad             ; 读取失败（返回 0 或 1）则爆炸
  cmpl   $0x7,0x8(%rsp)     ; 检查 x（%rsp+8）是否 >7（无符号数）
  ja     400fad             ; x>7 则爆炸（switch default 分支）
  
  mov    0x8(%rsp),%eax     ; eax = x（作为跳转表索引）
  jmpq   *0x402470(,%rax,8) ; 关键：跳转表寻址！地址 = 0x402470 + x*8（x86-64 地址占 8 字节）

# 以下是 switch 的 case 分支
case_0:
  mov    $0xcf,%eax         ; eax = 207（十六进制 0xcf = 十进制 207）
  jmp    case_end
case_1:
  mov    $0x137,%eax        ; eax = 311（十六进制 0x137 = 十进制 311）
  jmp    case_end
...（省略 case_2 到 case_6）
case_7:
  mov    $0x147,%eax        ; eax = 327（十六进制 0x147 = 十进制 327）
  jmp    case_end

case_end:
  cmp    %eax,0xc(%rsp)     ; 比较 y（%rsp+c）和 eax（case 分支赋值的结果）
  je     400fc0             ; 相等则安全，不等则爆炸
  callq  <explode_bomb>
```

#### 核心逻辑推导
- 输入要求：2 个整数（x, y）；
- x 必须是 0-7 之间的整数（否则触发 default 爆炸）；
- switch 底层通过「跳转表」实现：`0x402470` 是跳转表起始地址，x 是索引，每个表项是 8 字节的代码地址；
- 每个 case 分支给 %eax 赋一个固定值，y 必须等于该值才能解除炸弹。

#### 解法步骤
1. 查看跳转表对应的 case 赋值：  
   在 GDB 中执行 `x/8xg 0x402470`，会显示 8 个地址（对应 x=0-7），但更简单的方式是直接看汇编中 case 分支的 `mov $0x..., %eax` 指令；
2. 任选一个 x（0-7），获取对应的 eax 值（即 y）；
3. 输入 x 和 y，验证是否通过。

#### 答案
- x=0 → y=207 → 输入 `0 207`；
- x=1 → y=311 → 输入 `1 311`；
- x=7 → y=327 → 输入 `7 327`；


### 3.4 Phase 4：递归函数（二分查找）
#### 核心考点：递归调用、栈帧生长、二分查找逻辑
#### 汇编逐行解析
```asm
phase_4:
  sub    $0x18,%rsp         ; 栈上分配 24 字节
  lea    0xc(%rsp),%rcx     ; 第 3 个参数：&y 存入 %rcx
  lea    0x8(%rsp),%rdx     ; 第 2 个参数：&x 存入 %rdx
  mov    $0x4025cf,%esi     ; 第 1 个参数：格式化字符串 "%d %d" 存入 %esi
  mov    %rsp,%rdi          ; 第 0 个参数：输入缓冲区存入 %rdi
  callq  <__isoc99_sscanf>  ; 调用 sscanf(input, "%d %d", &x, &y)
  
  cmp    $0x2,%eax          ; 必须成功读取 2 个整数，否则爆炸
  jne    401062
  cmpl   $0xe,0x8(%rsp)     ; 检查 x 是否 >14（0xe 是十进制 14）
  ja     401062             ; x>14 则爆炸
  
  mov    $0xe,%edx          ; 第 3 个参数：high=14 存入 %edx
  mov    $0x0,%esi          ; 第 2 个参数：low=0 存入 %esi
  mov    0x8(%rsp),%edi     ; 第 1 个参数：x 存入 %edi
  callq  401064 <func4>     ; 调用递归函数 func4(x, 0, 14)
  
  test   %eax,%eax          ; 检查 func4 返回值是否为 0
  je     40105b             ; 返回 0 则安全
  callq  <explode_bomb>
  
  ; 递归函数 func4 核心逻辑
func4:
  sub    $0x8,%rsp
  mov    %edx,%eax          ; eax = high
  sub    %esi,%eax          ; eax = high - low
  mov    %eax,%ecx          ; ecx = high - low
  shr    $0x1f,%ecx         ; 算术右移 31 位，处理负数（符号扩展）
  add    %ecx,%eax          ; eax = (high - low + ecx) → 等价于 (high - low) // 2（整除）
  add    %esi,%eax          ; eax = low + (high - low)//2 → mid 值（二分查找中间值）
  
  cmp    %edi,%eax          ; 比较 mid 和 x（参数 1）
  jle    func4_left         ; mid ≤ x → 递归左半区
  mov    %eax,%edx          ; mid > x → high = mid - 1
  dec    %edx
  callq  func4
  add    %eax,%eax          ; 返回值 *2
  jmp    func4_end
func4_left:
  cmp    %edi,%eax
  je     func4_end          ; mid == x → 返回 0
  mov    %eax,%esi          ; mid < x → low = mid + 1
  inc    %esi
  callq  func4
  lea    0x1(%rax,%rax,1),%eax  ; 返回值 = 2*ret +1
func4_end:
  add    $0x8,%rsp
  retq
```

#### 核心逻辑推导
- 输入要求：2 个整数（x, y）；
- 递归函数 `func4(x, low=0, high=14)` 的逻辑是**二分查找**：
  1. 计算 mid = (low + high) // 2；
  2. 若 mid > x：递归调用 func4(x, low, mid-1)，返回值 = 2 * 子调用返回值；
  3. 若 mid < x：递归调用 func4(x, mid+1, high)，返回值 = 2 * 子调用返回值 + 1；
  4. 若 mid == x：返回 0；
- 最终要求 `func4` 返回值为 0，且 y 必须为 0。

#### 解法步骤
1. 要让 func4 返回 0，必须触发 base case（mid == x）；
2. 二分查找的 base case 触发条件：x 是某轮递归的 mid 值，即 x 可以是 0、1、3、7、14（二分查找路径上的 mid）；
3. 选择最简单的 x=0，此时 func4 直接返回 0，y 必须为 0。

#### 答案
`0 0` 或 `1 0` 或 `3 0` 或 `7 0` 或 `14 0`

### 3.5 Phase 5：位操作与字符映射
#### 核心考点：位运算、数组索引、字符串映射
#### 汇编逐行解析
```asm
phase_5:
  sub    $0x20,%rsp         ; 栈上分配 32 字节
  mov    %rdi,%rbx          ; 保存输入字符串地址（%rdi 是输入）
  mov    $0x6,%edx          ; edx = 6（字符串长度计数器）
  mov    $0x0,%eax          ; eax = 0（循环变量 i=0）
len_check:
  movzbl (%rbx,%rax,1),%ecx ; 取输入字符串第 i 个字符（零扩展为 32 位）
  test   %cl,%cl            ; 检查是否为字符串结束符 '\0'
  je     len_confirm        ; 遇到 '\0' 则跳出长度检查
  inc    %eax               ; i++
  cmp    %edx,%eax          ; 检查 i 是否超过 6
  jne    len_check
len_confirm:
  cmp    $0x6,%eax          ; 输入字符串长度必须为 6，否则爆炸
  jne    4010d8
  
  mov    $0x0,%eax          ; i=0（循环变量）
map_loop:
  movzbl (%rbx,%eax,1),%ecx ; 取输入字符串第 i 个字符 c
  mov    %cl,(%rsp)         ; 保存 c 到栈
  and    $0xf,%ecx          ; 关键：c & 0xf（取低 4 位，掩码操作，范围 0-15）
  movzbl 0x4024b0(%rcx),%edx ; 查表：以低 4 位为索引，取 0x40240b 处数组的字符
  mov    %dl,0x10(%rsp,%eax,1) ; 保存映射后的字符到新字符串（栈上 0x10 开始）
  inc    %eax               ; i++
  cmp    $0x6,%eax          ; 循环 6 次（处理 6 个字符）
  jne    map_loop
  
  mov    $0x40245e,%esi     ; 目标字符串地址 0x40245e 存入 %esi
  lea    0x10(%rsp),%rdi    ; 映射后的新字符串地址存入 %rdi
  callq  <strings_not_equal> ; 比较新字符串和目标字符串
  test   %eax,%eax
  je     4010f0             ; 相等则安全，不等则爆炸
  callq  <explode_bomb>
```

#### 核心逻辑推导
- 输入要求：长度为 6 的字符串；
- 映射规则：输入字符的**低 4 位**作为索引，在一个硬编码数组（地址 0x4024b0）中查找对应的字符，生成新字符串；
- 新字符串必须与目标字符串（地址 0x40245e）完全一致。

#### 解法步骤
1. 查看目标字符串：`x/s 0x40245e` → 假设结果是 `flyers`；
2. 查看映射数组：`x/s 0x4024b0` → 假设结果是 `maduiersnfotvbyl`；
3. 逆向映射：目标字符 → 查找在映射数组中的索引→ 构造输入字符；

| 目标字符 | 映射数组索引 | 输入字符低 4 位 | 合法输入字符 |
|----------|--------------|------------------|----------------------|
| f        | 9            | 0x9              | '9'（0x39）、'i'（0x69） |
| l        | 15           | 0xf              | 'o'（0x6f）、'O'（0x4f） |
| y        | 14           | 0xe              | 'n'（0x6e）、'N'（0x4e） |
| e        | 5            | 0x5              | 'e'（0x65）、'5'（0x35） |
| r        | 6            | 0x6              | 'f'（0x66）、'6'（0x36） |
| s        | 7            | 0x7              | 'g'（0x67）、'7'（0x37） |

4. 构造输入字符串：从每个位置的合法字符中任选一个，组成 6 字符字符串。

#### 答案
`ionefg`（低 4 位分别为 9、0xf、0xe、5、6、7，映射后为 `flyers`）

### 3.6 Phase 6：链表排序
#### 核心考点：结构体指针、链表遍历、指针重排、排序逻辑
#### 汇编逐行解析
```asm
phase_6:
  sub    $0x50,%rsp         ; 栈上分配 80 字节（存储 6 个指针 + 局部变量）
  lea    0x18(%rsp),%rsi    ; 第 2 个参数：&array 存入 %rsi（array 存储 6 个输入整数）
  mov    %rsp,%rdi          ; 第 1 个参数：输入缓冲区存入 %rdi
  callq  <read_six_numbers> ; 读取 6 个整数到 array（%rsp+24 开始）
  
  ; 第一步：检查输入合法性（1-6 的全排列）
  mov    $0x0,%ebx          ; i=0
check_duplicate:
  mov    0x18(%rsp,%rbx,4),%eax ; array[i]
  cmp    $0x6,%eax          ; array[i] >6 → 爆炸
  ja     4011c8
  cmp    $0x1,%eax          ; array[i] <1 → 爆炸
  jb     4011c8
  mov    $0x0,%ecx          ; j=0
check_inner:
  cmp    %rbx,%ecx          ; j == i → 跳过
  je     check_inner_end
  mov    0x18(%rsp,%ecx,4),%edx ; array[j]
  cmp    %edx,%eax          ; array[i] == array[j] → 爆炸（重复）
  je     4011c8
  inc    %ecx               ; j++
  jmp    check_inner
check_inner_end:
  inc    %ebx               ; i++
  cmp    $0x6,%ebx          ; 检查 6 个元素
  jne    check_duplicate
  
  ; 第二步：反转映射（陷阱！）
  mov    $0x6,%ebx          ; i=6
reverse_map:
  dec    %ebx               ; i 从 5 到 0
  mov    0x18(%rsp,%ebx,4),%eax ; array[i]
  mov    $0x7,%edx
  sub    %eax,%edx          ; array[i] = 7 - array[i]（反转：1→6，2→5，...，6→1）
  mov    %edx,0x18(%rsp,%ebx,4)
  cmp    $0x0,%ebx
  jne    reverse_map
  
  ; 第三步：构建链表节点指针数组
  mov    $0x0,%ebx          ; i=0
build_ptr_array:
  mov    0x18(%rsp,%ebx,4),%eax ; array[i]（反转后的值）
  lea    0x6032d0(%rip),%rdx ; 链表头节点地址（假设 0x6032d0）
  mov    $0x0,%ecx          ; k=0
find_node:
  cmp    %ecx,%eax          ; 找到第 array[i] 个节点
  je     find_node_end
  mov    (%rdx),%rdx        ; 指针移动到下一个节点（rdx = node->next）
  inc    %ecx               ; k++
  jmp    find_node
find_node_end:
  mov    %rdx,0x30(%rsp,%ebx,8) ; 存储节点指针到 ptr_array[i]（8字节指针）
  inc    %ebx
  cmp    $0x6,%ebx
  jne    build_ptr_array
  
  ; 第四步：重组链表（按 ptr_array 顺序串联）
  mov    $0x1,%ebx          ; i=1
link_list:
  mov    0x30(%rsp,%ebx,8),%rdx ; ptr_array[i]
  mov    0x30(%rsp,%rbx,8),%rax ; ptr_array[i-1]
  mov    %rdx,0x8(%rax)     ; ptr_array[i-1]->next = ptr_array[i]
  inc    %ebx
  cmp    $0x6,%ebx
  jne    link_list
  movq   $0x0,0x8(%rdx)     ; 最后一个节点的 next = NULL
  
  ; 第五步：检查链表是否降序排列
  lea    0x30(%rsp),%rbx    ; 链表头指针（ptr_array[0]）
check_sorted:
  mov    (%rbx),%rax        ; current = head
  mov    0x8(%rbx),%rbx     ; head = head->next
  test   %rbx,%rbx          ; 到达链表尾 → 检查通过
  je     401234
  mov    0x4(%rax),%edx     ; current->value
  mov    0x4(%rbx),%ecx     ; next->value
  cmp    %ecx,%edx          ; current->value >= next->value → 降序
  jge    check_sorted       ; 符合则继续检查
  callq  <explode_bomb>     ; 不符合则爆炸
```

#### 核心逻辑推导
- 输入要求：6 个整数，必须是 1-6 的全排列（无重复、无超出范围）；
- 关键陷阱：输入的每个数会被转换为 `7 - x`（如输入 1→6，输入 6→1）；
- 链表结构：内存中存在一个链表，每个节点包含 `value`（4字节）、`id`（4字节）、`next`（8字节指针）；
- 重组规则：根据转换后的输入数，从原链表中取出对应节点，按输入顺序串联成新链表；
- 最终要求：新链表的 `value` 必须**降序排列**。

#### 解法步骤（核心三步）
1. 查看原链表节点的 `value`：  
   链表头地址通常在 `0x6032d0`（汇编中 `lea 0x6032d0(%rip),%rdx`），用 `x/20dw 0x6032d0` 查看节点值：  
   假设节点 1-6 的 value 为：`168, 332, 728, 516, 471, 224`（需按实际地址查询）；
2. 对节点 value 降序排序，记录对应的**原节点编号**（1-6）：  
   降序 value：728（节点3）→ 516（节点4）→ 471（节点5）→ 332（节点2）→ 224（节点6）→ 168（节点1）；
3. 应对反转映射：输入数 = `7 - 原节点编号`（因为代码中 `array[i] = 7 - array[i]`）：  
   原节点编号：3 → 输入 4（7-3=4）；  
   原节点编号：4 → 输入 3（7-4=3）；  
   原节点编号：5 → 输入 2（7-5=2）；  
   原节点编号：2 → 输入 5（7-2=5）；  
   原节点编号：6 → 输入 1（7-6=1）；  
   原节点编号：1 → 输入 6（7-1=6）；

#### 答案（需按实际链表值调整）
`4 3 2 5 1 6`


---

> Author: 7M7  
> URL: http://localhost:1313/posts/bomblab/  

