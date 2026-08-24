# 8086 汇编与 Debug 笔记：分段、栈、寻址与调试


这篇把原来的两篇 8086 笔记合成一篇，保留复习时最常用的内容：分段地址、栈、寻址规则、跳转、标志位、多字节运算，以及 DOS Debug 的常用命令。

<!--more-->

## 1. 8086 的基本模型

8086 是 16 位 CPU，常用寄存器也是 16 位。

通用寄存器：

- `AX`、`BX`、`CX`、`DX`
- 每个都能拆成高 8 位和低 8 位，例如 `AX = AH + AL`

段寄存器：

- `CS`：代码段
- `DS`：数据段
- `SS`：栈段
- `ES`：附加段

指令指针：

- `IP`：下一条指令在代码段中的偏移地址

常用标志位：

| 标志位 | 含义 |
| --- | --- |
| `ZF` | 结果是否为 0 |
| `CF` | 无符号运算是否进位或借位 |
| `OF` | 有符号运算是否溢出 |
| `SF` | 结果最高位是否为 1 |

## 2. 分段地址与物理地址

8086 有 20 根地址线，可以访问 1MB 内存。但寄存器只有 16 位，单个寄存器最多表示 64KB。解决办法是“段地址 + 偏移地址”。

公式：

```text
物理地址 = 段地址 × 16 + 偏移地址
```

也就是：

```text
Physical = Segment << 4 + Offset
```

例子：

```text
1000:0000 -> 1000H × 10H + 0000H = 10000H
1000:0009 -> 1000H × 10H + 0009H = 10009H
```

常见场景：

| 访问内容 | 默认公式 |
| --- | --- |
| 取指令 | `CS × 16 + IP` |
| 访问普通数据 | `DS × 16 + 偏移` |
| 栈顶 | `SS × 16 + SP` |

## 3. 栈与 `SS:SP`

栈用于保存临时信息，比如返回地址、寄存器现场、局部数据。8086 的栈从高地址向低地址增长。

入栈 `push ax`：

```text
SP = SP - 2
把 AX 写入 SS:SP
```

出栈 `pop ax`：

```text
从 SS:SP 读 16 位数据到 AX
SP = SP + 2
```

示例：

```asm
mov ax, 1000H
mov ss, ax
mov sp, 0010H

push bx      ; SP = 000EH
push ax      ; SP = 000CH
pop ax       ; SP = 000EH
```

注意：8086 不会自动检查栈越界，栈空间要自己规划。

## 4. 寻址方式与 `BX/BP/SI/DI`

8086 中，只有 `BX`、`BP`、`SI`、`DI` 可以直接参与内存寻址。

合法示例：

```asm
[bx]
[si]
[bx+si]
[bx+di+idata]
[bp]
[bp+si]
[bp+di+idata]
```

非法示例：

```asm
[ax]       ; AX 不能直接寻址
[bx+bp]    ; 两个基址寄存器不能同时用
[si+di]    ; 两个变址寄存器不能同时用
```

默认段寄存器规则：

- 地址表达式里有 `BP`，默认段寄存器是 `SS`。
- 其他情况通常默认段寄存器是 `DS`。

```asm
mov ax, [bx+idata]  ; DS × 16 + BX + idata
mov ax, [bp+idata]  ; SS × 16 + BP + idata
```

如果要改默认段，可以显式写段前缀：

```asm
mov ax, ds:[bp]
mov ax, ss:[bx]
```

## 5. `CS:IP` 与跳转

CPU 正常执行时：

1. 从 `CS:IP` 指向的位置取指令。
2. `IP` 增加当前指令长度。
3. 执行指令。

`jmp` 的本质就是修改 `IP`，或者同时修改 `CS` 和 `IP`。

段内跳转，只改 `IP`：

```asm
jmp ax
jmp short label
jmp near ptr label
```

段间跳转，同时改 `CS:IP`：

```asm
jmp 1000:0000
```

## 6. `CMP` 与条件转移

`cmp a, b` 本质上做 `a - b`，但不保存结果，只影响标志位。

常见条件跳转：

| 指令 | 含义 |
| --- | --- |
| `je` / `jz` | 相等 / 结果为 0 |
| `jne` / `jnz` | 不相等 / 结果不为 0 |
| `ja` | 无符号大于 |
| `jb` | 无符号小于 |
| `jg` | 有符号大于 |
| `jl` | 有符号小于 |

例子：

```asm
cmp ax, bx
je equal
jmp done

equal:
    mov cx, 1

done:
    nop
```

## 7. 多字节加减法：`ADC` 与 `SBB`

`ADC` 会把进位标志 `CF` 一起加进去，适合多字节加法。

```asm
add ax, bx      ; 低 16 位相加
adc dx, cx      ; 高 16 位相加，再加低位产生的 CF
```

`SBB` 会把借位标志 `CF` 一起减掉，适合多字节减法。

```asm
sub ax, bx
sbb dx, cx
```

## 8. DOS Debug 常用命令

进入 Debug：

```text
C:\> debug
-
```

常用命令可以记成 `TRUEAD + GQ`：

| 命令 | 作用 |
| --- | --- |
| `T` | 单步执行 |
| `R` | 查看或修改寄存器 |
| `U` | 反汇编 |
| `E` | 修改内存 |
| `A` | 在内存中写汇编指令 |
| `D` | 查看内存 |
| `G` | 连续执行 |
| `Q` | 退出 |

## 9. 常用 Debug 流程

写入一段简单程序：

```text
-a 1000:0000
1000:0000 mov ax, 1000
1000:0003 mov bx, 2000
1000:0006 add ax, bx
1000:0008 int 3
1000:0009
```

反汇编查看：

```text
-u 1000:0000
```

修改 `CS:IP`：

```text
-r cs
1000
-r ip
0000
```

单步执行：

```text
-t
```

查看内存：

```text
-d 1000:0000
```

退出：

```text
-q
```

## 10. 汇编程序骨架

一个基本 MASM 程序通常长这样：

```asm
assume cs:code, ds:data

data segment
    db 'hello$'
data ends

code segment
start:
    mov ax, data
    mov ds, ax

    ; 主体代码

    mov ax, 4c00h
    int 21h
code ends

end start
```

要点：

- `assume` 告诉汇编器段寄存器和段名的对应关系。
- `mov ds, data` 不能直接写，必须先经过通用寄存器。
- 程序结束常用 `mov ax, 4c00h` + `int 21h`。

## 11. ASCII 大小写转换

ASCII 中，大写字母和小写字母相差 `20H`。

```asm
; 大写转小写
or al, 20H

; 小写转大写
and al, 0DFH
```

也可以用加减：

```asm
add al, 20H    ; A -> a
sub al, 20H    ; a -> A
```

## 12. 小结

8086 复习可以按这条线走：

1. 先会算 `段地址 × 16 + 偏移地址`。
2. 再理解 `CS:IP`、`SS:SP`、`DS` 的默认作用。
3. 然后记住 `BX/BP/SI/DI` 的寻址规则。
4. 条件跳转看 `cmp` 后的标志位。
5. 用 Debug 的 `R`、`U`、`T`、`D` 观察寄存器、代码和内存。

## 相关文章

- [GDB x 命令详解：格式、单位与内存查看]({{< relref "posts/xnotes.md" >}})：对比现代 GDB 查看内存时的格式和单位写法。


---

> 作者: 7M7  
> URL: https://7m7666.github.io/posts/017d599/  

