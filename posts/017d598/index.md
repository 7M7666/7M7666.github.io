# 8086 汇编系统笔记：内存分段、栈、寻址与控制转移


# 8086 汇编系统笔记：内存分段、栈、寻址与控制转移（超详细）

> 以 8086 为例，从“内存地址怎么算”一路讲到“多字节加法”“大小写转换”。

---

## 目录

1. [8086 的内存分段与物理地址](#1-8086-的内存分段与物理地址)
2. [栈（Stack）与 SS:SP 机制](#2-栈stack与-sssp-机制)
3. [内存寻址方式与 BX/BP 规则](#3-内存寻址方式与-bxbp-规则)
4. [CS:IP 与控制转移指令 JMP](#4-csip-与控制转移指令-jmp)
5. [标志寄存器 Flags](#5-标志寄存器-flags)
6. [CMP 与条件转移指令 Jxx](#6-cmp-与条件转移指令-jxx)
7. [多字节加减法：ADC / SBB](#7-多字节加减法adc--sbb)
8. [伪指令与汇编程序骨架](#8-伪指令与汇编程序骨架)
9. [例子：ASCII 字符大小写转换](#9-例子ascii-字符大小写转换)
10. [Debug 观察小技巧](#10-debug-观察小技巧)

---

## 1. 8086 的内存分段与物理地址

### 1.1 分段的原因

8086 寄存器宽度为 16 位，只能直接表示 0~65535（64KB）范围的地址。  
实际可访问内存是 1MB（20 位地址）。  
解决方案：**“段地址 × 16 + 偏移地址”**。

### 1.2 主要段寄存器

- `CS`：Code Segment，代码段
- `DS`：Data Segment，数据段
- `SS`：Stack Segment，栈段
- `ES`：Extra Segment，附加段

通用公式：

```text
物理地址 = 段寄存器值 × 16 + 偏移地址
````

常见几种场景：

* 访问数据：
  `物理地址 = DS × 16 + 偏移`
* 访问代码：
  `物理地址 = CS × 16 + IP`
* 使用 BX 做基址：
  `物理地址 = DS × 16 + BX + idata`
* 使用 BP 做基址：
  `物理地址 = SS × 16 + BP + idata`

> 结论：**只要括号里用了 BP，默认段就是 SS；否则通常默认段是 DS。**

---

## 2. 栈（Stack）与 SS:SP 机制

栈是一块专门区域，用来保存临时信息：返回地址、寄存器现场、局部变量等。
特点：**后进先出（LIFO）**。

### 2.1 栈相关寄存器

* `SS`：栈段基址
* `SP`：栈顶偏移地址（单位：字节）

栈顶元素的物理地址：

```text
栈顶物理地址 = SS × 16 + SP
```

8086 中，栈**从高地址向低地址生长**。

### 2.2 PUSH 入栈

以 `push ax` 为例：

1. `SP = SP - 2` （栈顶向低地址方向移动 2 个字节）
2. 将 `AX` 的 16 位内容写入到 `SS:SP` 处的内存

示例：

```asm
mov ax, 1000H
mov ss, ax
mov sp, 0010H     ; 初始化栈空（示例）

push bx           ; 第一次压栈，SP = 000EH
push ax           ; 第二次压栈，SP = 000CH
```

### 2.3 POP 出栈

以 `pop ax` 为例：

1. 从 `SS:SP` 处读取 16 位数据送入 AX
2. `SP = SP + 2` （栈顶向高地址方向移动 2 个字节）

示例：

```asm
pop ax           ; 先把栈顶数据给 AX，SP += 2
pop bx
```

说明：

* 栈中原来的数据在内存里还存在，但逻辑上已经“无效”，下一次 `push` 会覆盖。
* 8086 不会自动检查“栈满/栈空”，越界会破坏其他数据，由程序员自己负责。

### 2.4 栈空间例子：10000H~1000FH

假设用 `10000H ~ 1000FH` 这 16 个字节作为栈区（共 16B）：

* 栈底最高地址：`1000FH`
* 栈空时约定：**SP 指向栈底的下一单元**，即 `0010H`

初始化代码：

```asm
mov ax, 1000H
mov ss, ax
mov sp, 0010H    ; 栈空
```

连续压栈时 SP 变化：

```text
SP = 0010H  (空栈)
push ax  -> SP = 000EH
push bx  -> SP = 000CH
...
```

---

## 3. 内存寻址方式与 BX/BP 规则

8086 中，**只有 4 个寄存器可以出现在 `[...]` 内参与地址计算**：

* BX (Base)
* BP (Base Pointer)
* SI (Source Index)
* DI (Destination Index)

### 3.1 合法的组合规则

1. “基址”寄存器：BX 或 BP（二选一）
2. “变址”寄存器：SI 或 DI（二选一）
3. 可再加立即数 `idata`

合法示例（✅）：

```asm
[bx]
[bx+si]
[bx+di+idata]
[bp]
[bp+si]
[bp+di+idata]
[si+idata]
[di]
```

非法示例（❌）：

```asm
[bx+bp]    ; 两个基址寄存器，不允许
[si+di]    ; 两个变址寄存器，不允许
[ax]       ; AX 不能直接用于内存寻址
```

### 3.2 段寄存器的默认选择规则

* 只要地址表达式中出现了 `BP`，默认段寄存器 = `SS`
* 否则（使用 BX、SI、DI 等），默认段寄存器 = `DS`

对应的物理地址公式：

```text
mov ax, [bx+idata]  ; 物理地址 = DS × 16 + BX + idata
mov ax, [bp+idata]  ; 物理地址 = SS × 16 + BP + idata
```

可以强制指定段前缀：

```asm
mov ax, ds:[bp]     ; 强制使用 DS 段
mov ax, ss:[bx]     ; 强制使用 SS 段
```

### 3.3 DS 的常见赋值错误

段寄存器不能直接接立即数：

```asm
mov ds, 1000H   ; ❌ 错误
```

正确做法：

```asm
mov ax, 1000H
mov ds, ax      ; ✅ 先通过通用寄存器中转
```

---

## 4. CS:IP 与控制转移指令 JMP

### 4.1 正常的取指令流程

8086 执行一条指令大致流程：

1. 从 `CS:IP` 指向的内存单元取指令到指令缓冲器
2. `IP = IP + 指令长度`，指向下一条指令
3. 执行缓冲器中的指令

只要 **不修改 CS 和 IP**，程序就按照内存中的顺序向前执行。

### 4.2 JMP 指令：修改 CS/IP

`jmp` 的本质是：修改 `CS`、`IP` 或其中之一。

#### 4.2.1 段内转移（只改 IP）

```asm
jmp ax      ; IP = AX
jmp bx      ; IP = BX
```

段寄存器 CS 不变。

也可以写成“相对跳转”的形式（编译器生成偏移量）：

```asm
jmp short label
jmp near  label
```

#### 4.2.2 段间转移（同时改 CS 和 IP）

格式：

```asm
jmp 段地址:偏移
```

例：

```asm
jmp 1000:0003   ; CS = 1000H, IP = 0003H
```

执行后下一条指令来自物理地址：

```text
物理地址 = 1000H × 16 + 0003H = 10003H
```

---

## 5. 标志寄存器 Flags

标志寄存器并不存普通数据，而是存放各种“状态位”。

常用标志位及含义：

* `ZF`（Zero Flag）零标志：结果是否为 0
* `PF`（Parity Flag）奇偶标志：低 8 位中 1 的个数是奇数/偶数
* `SF`（Sign Flag）符号标志：结果最高位（符号位）
* `CF`（Carry Flag）进位标志：无符号运算中进位 / 借位
* `OF`（Overflow Flag）溢出标志：有符号运算中是否溢出

> 传送类指令（mov/push/pop 等）一般不影响标志位；
> 算术逻辑指令（add/sub/and/or/xor/cmp 等）会影响标志位。

### 5.1 ZF：零标志

* 结果为 0 → `ZF = 1`
* 结果不为 0 → `ZF = 0`

在调试器中通常显示为：

* `ZR`（Zero）表示 ZF=1
* `NZ`（Not Zero）表示 ZF=0

### 5.2 PF：奇偶标志

对“结果的低 8 位”统计 1 的个数：

* 若 1 的个数为偶数 → `PF = 1`（Parity Even，PE）
* 若 1 的个数为奇数 → `PF = 0`（Parity Odd，PO）

记忆：**P E = Parity Even = PF=1**

### 5.3 SF：符号标志

* 把结果的最高有效位复制给 SF：

  * 最高位 = 1 → SF=1（负数）
  * 最高位 = 0 → SF=0（非负）

CPU 不关心你把数据当有符号还是无符号，它总是设置 SF。
是否使用 SF 取决于程序逻辑。

### 5.4 CF & OF：进位 vs 溢出

**CF**

* 下溢 / 借位（无符号减法） → `CF = 1`
* 加法最高位产生进位（无符号加法） → `CF = 1`

**OF**

* 针对有符号数：

  * 正 + 正 → 得到负 → `OF = 1`
  * 负 + 负 → 得到正 → `OF = 1`
* 其它情况下 OF 一般为 0

例：8 位寄存器 AL 中执行 `98 + 99`

```asm
mov al, 98
add al, 99
```

二进制：

```text
0110 0010 (98)
+0110 0011 (99)
-----------
1100 0101 (结果)
```

* 作为无符号数：范围 0~255，结果 197，没越界 → CF=0
* 作为有符号数：98、99 都是正数，结果最高位为 1（负数）
  正 + 正 得到负数 → 溢出 → OF=1

结论：同一条指令，CF、OF 的含义取决于你是按无符号还是按有符号来解释。

---

## 6. CMP 与条件转移指令 Jxx

### 6.1 CMP：不留结果的 SUB

```asm
cmp ax, bx
```

本质上等价于：

```text
临时 = ax - bx
; 只根据这个结果更新 Flags，不把临时写回
```

根据结果不同，Flags 的典型变化：

* 若 `(ax) = (bx)` → 结果为 0 → `ZF = 1`
* 若 `(ax) < (bx)`（无符号） → 借位 → `CF = 1`
* 若 `(ax) > (bx)`（无符号） → 无借位且不为 0 → `CF=0, ZF=0`

### 6.2 常用无符号条件转移指令

以下几条通常配合 `cmp` 使用：

* `je / jz`：等于（ZF = 1）
* `jne / jnz`：不等于（ZF = 0）
* `jb / jc`：below / carry，小于（无符号，CF = 1）
* `ja`：above，大于（无符号，CF = 0 且 ZF = 0）
* `jbe`：below or equal，小于等于（CF = 1 或 ZF = 1）
* `jae / jnc`：above or equal，大于等于（CF = 0）

简单示例：

```asm
cmp ax, bx
ja  bigger     ; 若 ax > bx（无符号）跳转
jb  smaller    ; 若 ax < bx
je  equal      ; 若 ax == bx
```

---

## 7. 多字节加减法：ADC / SBB

16 位寄存器最多表示 0~65535（或有符号 -32768~32767）。
要处理 32 位、64 位、128 位等更大整数，就需要“拆分 + 进位/借位传递”。

核心：利用 `CF` 作为进位/借位在多字之间传递。

### 7.1 ADC：Add with Carry（带进位加法）

语法：

```asm
adc dest, src   ; dest = dest + src + CF
```

典型场景：两个 32 位数相加（每个拆为“高 16 位 + 低 16 位”）。

低地址存低位：

```asm
; 假设：
;   [si]   = A 的低 16 位
;   [si+2] = A 的高 16 位
;   [di]   = B 的低 16 位
;   [di+2] = B 的高 16 位

sub ax, ax        ; ax = 0，同时 CF = 0，清进位（关键一步）

; --- 低 16 位 ---
mov ax, [si]
adc ax, [di]      ; ax = A_low + B_low + CF(0)
mov [si], ax      ; 存回 A 的低位，也可以存到别处

; --- 高 16 位 ---
mov ax, [si+2]
adc ax, [di+2]    ; ax = A_high + B_high + 上一步产生的 CF
mov [si+2], ax
```

注意事项：

* 必须**在第一轮之前手动清 CF**（例如用 `sub ax, ax` 或 `clc`）。
* 在连续 `adc` 过程中，不能夹入会修改 CF 的指令（如 `add/sub/cmp` 等）。
  但 `inc/dec/loop` 等不会改 CF，可以安全使用。

### 7.2 SBB：Subtract with Borrow（带借位减法）

语法：

```asm
sbb dest, src   ; dest = dest - src - CF
```

用法与 ADC 类似，用来实现多字节减法的“高位减法”，CF 用来传递借位。

---

## 8. 伪指令与汇编程序骨架

汇编源文件中有两种东西：

1. **机器指令**：由 CPU 执行（mov、add、sub、jmp、push 等）
2. **伪指令**：只被汇编器处理，没有机器码（segment、ends、assume、end 等）

### 8.1 常见伪指令

* `segment / ends`：定义某个段的开始与结束
* `assume`：告诉汇编器，“某个段寄存器对应哪个段名”
* `end`：标记源程序结束，通常同时给出程序入口标号

### 8.2 典型程序模板
```asm
assume cs:codesg, ds:datasg

datasg segment
    ; 数据定义
    msg db 'Hello, world!$', 0
datasg ends

codesg segment

start:
    ; 初始化数据段
    mov ax, datasg
    mov ds, ax

    ; 程序主体
    mov dx, offset msg
    mov ah, 9
    int 21H          ; DOS 打印字符串

    ; 正常返回 DOS
    mov ax, 4c00H
    int 21H

codesg ends

end start
```

说明：

* `assume` 只是给汇编器看的“假设”，不会生成机器码。
* 运行时仍然需要代码（如 `mov ax, datasg / mov ds, ax`）真正设置段寄存器。
* `end start` 不仅告诉汇编器“到此结束”，还指定程序入口是 `start` 标号处。

---

## 9. 例子：ASCII 字符大小写转换

ASCII 中英文字母大小写有固定的位模式差异，可通过位运算实现大小写转换。

### 9.1 背景

* `'A' = 41H = 0100 0001B`
* `'a' = 61H = 0110 0001B`

差值：

```text
0110 0001B
0100 0001B
----------
0010 0000B = 20H
```

也就是说：**小写字母 = 大写字母 + 20H**。
对应到二进制，就是第 5 位（从 0 开始数）不同：

* 小写字母：bit5 = 1
* 大写字母：bit5 = 0

### 9.2 转大写：清零第 5 位（AND）

要把小写转成大写，需要把该位从 1 改为 0，其它位不变。
采用 AND 掩码：

```text
原：0110 0001 (a)
掩：1101 1111 (DFH)
AND:
   0100 0001 (A)
```

实现代码：

```asm
; al 中是小写字母
and al, 11011111B     ; 或 and al, 0DFH
; al 变成对应大写
```

### 9.3 转小写：将第 5 位置 1（OR）

把大写变成小写：只需把 bit5 置 1。
采用 OR 掩码：

```text
原：0100 0001 (A)
掩：0010 0000 (20H)
OR:
   0110 0001 (a)
```

实现代码：

```asm
; al 中是大写字母
or al, 00100000B      ; 或 or al, 20H
; al 变成对应小写
```

### 9.4 扫描字符串并大小写转换的示例

若内存中有一段以 0 结尾的字符串：

```asm
datasg segment
    str db 'AbCdEfG', 0
datasg ends
```

将全部字符转为大写：

```asm
mov ax, datasg
mov ds, ax

mov si, offset str

convert_loop:
    mov al, [si]
    cmp al, 0
    je  done          ; 遇到 0 结束

    ; 仅对小写字母操作
    cmp al, 'a'
    jb  skip
    cmp al, 'z'
    ja  skip

    and al, 11011111B ; 变成大写
    mov [si], al

skip:
    inc si
    jmp convert_loop

done:
    ; ...
```

---

## 10. Debug 观察小技巧

使用 DOS / Debug 或类似调试器时，可以通过命令观察寄存器和标志位变化。

### 10.1 查看寄存器与 Flags

常用命令：`r`

输出中右下角通常是标志位的简写，例如：

* `NZ` / `ZR`：ZF = 0 / 1
* `PO` / `PE`：PF = 0 / 1
* `PL` / `NG`：SF = 0（Plus）/ 1（Negative）
* `NC` / `CY`：CF = 0（No Carry）/ 1（Carry）

### 10.2 单步跟踪栈和跳转

* 执行 `t`（trace）或 `p`（step）观察：

  * `CS:IP` 如何随 `jmp/call/ret` 等指令变化
  * `SS:SP` 如何随 `push/pop` 变化
  * 算术运算后，Flags 的变化是否符合预期

示例：单步观察 `push ax / pop ax`：

```asm
mov ax, 1000H
mov ss, ax
mov sp, 0010H

mov ax, 1234H
push ax      ; 观察 SP = 000EH，内存 [SS:000EH] = 34H, [SS:000FH] = 12H
pop bx       ; 观察 BX = 1234H，SP 回到 0010H
```

在 debug 中逐步执行并配合 `d` 命令查看相关内存，可以帮助直观理解栈的方向和 PUSH/POP 的行为。

---

> 至此，一条完整知识线已经建立：
> **物理地址与分段 → 栈和 BP/BX 寻址 → CS:IP 与 JMP → Flags → CMP/Jxx → 多字节运算 → 实战例子（大小写转换）**。
> 复习时可以按章节跳读，重点记住：
>
> * 地址公式与 BP/SS 默认绑定
> * PUSH/POP 对 SP 的影响
> * CF/OF 区分（无符号 / 有符号）
> * ADC/SBB 多字节运算套路
> * ASCII 大小写转换的 AND/OR 掩码。




---

> Author: 7M7  
> URL: http://localhost:1313/posts/017d598/  

