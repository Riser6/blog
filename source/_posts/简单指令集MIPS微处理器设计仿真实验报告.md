---
title: 简单指令集MIPS微处理器设计仿真实验报告
date: 2020-04-23 08:06:43
author: Riser
top: true
cover: true
password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
toc: false
mathjax: ture
summary: 简单指令集mips微处理器基于硬件描述语言Verilog的设计与仿真实验
categories: 实验报告
tags:
  - mips微处理器
  - Verilog
---

### 简单指令集MIPS微处理器设计仿真实验报告

---

#### 实验任务

​		全部采用Verilog 硬件描述语言设计实现简单指令集MIPS 微处理器，并且要求微处理器支持：

- 算术、逻辑运算指令：add、sub、and、or、slt
- 数据传送指令：lw、sw
- 程序控制指令:beq、j

​		要求指令存储器在时钟上升沿读出指令，指令指针的修改、寄存器文件写入、数据存储器数据写入都在时钟下降沿完成。完成完整设计代码输入、各模块完整功能仿真，整体仿真，验证所有指令执行情况。

​		且假定所有通用寄存器复位时取值都为各自寄存器编号乘以4；PC寄存器初始值为0；数据存储器和指令存储器容量大小为32*32，且地址都从0开始，指令存储器初始化时装载测试MIPS汇编程序的机器指令，数据存储器所有存储单元的初始值为其对应地址的取值。需要注意的是数据存储器的地址呈现以下规则：都是4的整数倍。

​		仿真以下MIPS汇编语言程序段的执行流程：

```verilog
main:	add $4,$2,$3
        lw $4,4($2)
        sw $5,8($2)
        sub $2,$4,$3
        or $2,$4,$3
        and $2,$4,$3
        slt $2,$4,$3
        beq $3,$3,equ
        w $2,0($3)
equ:    beq $3,$4,exit
        sw $2,0($3)
exit:   j main
```

---

#### 实验目的

-  熟悉微处理器的基本构成
- 掌握哈佛结构的计算机工作原理
- 学会设计简单指令集的mips微处理器
- 了解软件控制硬件工作的基本原理

---

#### 实验环境

- Windows10操作系统
- 编辑工具：Notepad++,ModelSim编辑器
- Verilog仿真平台:ModelSim

---

#### 基本原理

​		本实验设计的简单指令集的MIPS微处理器完整框图如下所示。由图可知，该微处理器由寄存器文件模块、ALU模块、指令存储器模块、数据存储器模块、主控制器译码模块、ALU控制模块，并将各模块按照下图级联方式进行连接，再加上时钟信号CLK和复位信号Reset，就能模拟MIPS微处理器在上电情况在时钟节拍的调剂下执行指令。

![图1 加上控制器的简单指令集MIPS微处理器完整框图](https://gitee.com//Riser_Wu/images/raw/master/img/20200420155555.png)

---

#### 实验模块设计

##### 寄存器文件模块的设计

寄存器文件模块保存寄存器操作数数据，这里使用32个32位宽的寄存器存储数据（1个寄存器存储1个寄存器操作数），指令执行过程中需要区分指令Rs,Rt,Rd域（提供寄存器地址）对应的寄存器，该模块支持读出Rs,Rt域对应的寄存器的值，在时钟信号clk的上升沿，并且能够在写信号regwr有效、复位信号reset无效情况下，将数据写入Rd域对应的寄存器，并且在复位信号reset有效时，将所有寄存器置0。

模块verilog代码设计如下：

```verilog
module regFile(
    input [4:0] RsAddr,
    input [4:0] RtAddr,
    input [4:0] WriteAddr,
    input [31:0] WriteData,
    input regwr,
    input clk,
    input reset,
    output [31:0] RsData,
    output [31:0] RtData
);
    reg [31:0] regs[0:31];
    assign RsData=(RsAddr==5'b0)?32'b0:regs[RsAddr];
    assign RtData=(RtAddr==5'b0)?32'b0:regs[RtAddr];
    integer i;
    always @(negedge clk)
        if(!reset & regwr)

```

##### ALU模块的设计

在mips简单指令集下，ALU单元需支持对输入的两个有符号操作数in1、in2做加、减、与、或、小于设置5种运算，输出运算结果ALURes，并且有输入的ALU控制信号ALUCtr决定执行何种操作，其对应关系如下图所示：

![图2 对应加、减、与、或、小于设置5种运算的ALU 控制信号编码](https://gitee.com//Riser_Wu/images/raw/master/img/20200420172425.png)

输出ALU运算结果ALURes以及判断两操作数是否相等的标志Zero（若两操作数相等）用于程序跳转指令通路的控制。

模块verilog代码设计如下：

```verilog
module ALU(
    input signed [31:0] in1,
    input signed [31:0] in2,
    input [3:0] ALUCtr,
    output reg [31:0] ALURes,
    output  reg Zero
    );
    always @(in1 or in2 or ALUCtr)
    begin
        case(ALUCtr)
            4'b0000: begin
                ALURes=in1 & in2;
                Zero=0;
            end
            4'b0001: begin
                ALURes=in1 | in2;
                Zero=0;
            end
            4'b0010: begin
                ALURes=in1 + in2;
                Zero=0;
            end
            4'b0110: begin
                ALURes=in1 - in2;
                Zero=(ALURes == 0) ?1:0;
            end
            4'b0111: begin
                ALURes=(in1 < in2) ?1:0;
                Zero=0;
            end
            default: begin
                ALURes=0;
                Zero=0;
            end
        endcase
    end
endmodule
```

##### 指令存储器模块的设计

指令存储器存储待执行的指令，这里用32个32位宽的寄存器保存指令（1个寄存器存储1条指令），通过传入指令的序号地址addr（顶层模块中PC寄存器右移两位的结果得到的序号地址），得到待执行指令的机器码instr。这里提前使用MARS得到测试汇编代码的机器码，使用readmemh函数将机器码从文件中读取到寄存器内。

模块verilog代码设计如下：

```verilog
module instruct_rom(
    input [4:0] addr,
output reg [31:0] instr,
input clk
    );
reg [31:0]  regs[0:31];
always @(posedge clk)
        assign instr = regs [addr];
    initial
    $readmemh("D:/inter/mips_cpu/test.txt",regs,0,10);
endmodule
```

##### 数据存储器模块的设计

数据存储器存储存储器操作数，这里用32个32位宽的寄存器保存数据（1个寄存器存储1个存储器操作数），通过输入地址addr，输出对应存储单元的数据readdata，并且在clk时钟上升沿，并且在写控制信号MemWR有效地情况下，可以将数据writedata写入地址addr对应存储单元。初始时，按照测试要求，数据存储器存储初始值对应的地址。

模块verilog代码设计如下：

```verilog
module data_rom(
    input [4:0] addr,
    output [31:0] readdata,
    input [31:0] writedata,
    input MemWR,
    input clk
    );
    reg [31:0]  regs[0:31];
    assign readdata=regs[addr];
    always @(negedge clk)
    if(MemWR)
    regs[addr]=writedata;
    integer i;
    initial
    for(i=0;i<32;i=i+1)
        regs[i]=i;
endmodule
```

##### 主控制器译码模块的设计

主控制模块输入指令操作码opCode，产生ALU控制信号ALUop，电路中5个复用器的通道选择信号RtDst、Imm、M2R 、B、J，以及寄存器文件、数据存储器写控制信号regwr、memwr。具体译码规则如下图：

![图3 主控制器译码规则](https://gitee.com//Riser_Wu/images/raw/master/img/20211939.png)

模块verilog代码设计如下：

```verilog
module master_Ctr(
    input [5:0] opCode,
    output [1:0]ALUop,
    output RtDst,
    output regwr,
    output Imm,
    output memwr,
    output B,
    output J,
    output M2R
    );
    reg[8:0] temp_output;
    assign {RtDst,Imm,M2R,regwr,memwr,B,J,ALUop}=temp_output;
    always @ (opCode)
        case (opCode)
            6'b000010:temp_output=9'bxxx0001xx;
            6'b000000:temp_output=9'b100100010;
            6'b100011:temp_output=9'b011100000;
            6'b101011:temp_output=9'bx1x010000;
            6'b000100:temp_output=9'bx0x001001;
            default:temp_output=9'b000000000;
        endcase
endmodule
```

##### ALU控制模块的设计

ALU控制模块输入主控制器产生的ALUop信号和指令功能码func，输出ALU控制信号ALUCtr。具体译码规则如下图：

![20213520](https://gitee.com//Riser_Wu/images/raw/master/img/20213520.png)

模块verilog代码设计如下：

```verilog
module ALU_Ctr(
    input [1:0] ALUop,
    input [5:0] func,
    output reg [3:0] ALUCtr
    );
    always @ (ALUop or func)
    casex({ALUop,func})
        8'b00xxxxxx: ALUCtr=4'b0010;
        8'b01xxxxxx: ALUCtr=4'b0110;
        8'b10xx0000: ALUCtr=4'b0010;
        8'b10xx0010: ALUCtr=4'b0110;
        8'b10xx0100: ALUCtr=4'b0000;
        8'b10xx0101: ALUCtr=4'b0001;
        8'b10xx1010: ALUCtr=4'b0111;
        default: ALUCtr=4'b0000;
    endcase
endmodule
```

##### Mips微处理器顶层模块设计

顶层模块将之前的多个模块实例化，并且按照图5中将各个模块通过复用器、移位、符号拓展、拼接单元连接起来。

![20214745](https://gitee.com//Riser_Wu/images/raw/master/img/20214745.png)

模块verilog代码设计如下（中间信号见图5）：

```verilog
module main_mips_cpu(
    input Clk,
    input Reset
    );
    wire [31:0] 		TempPC,MuxPC,JumpPC,BranchPC,SquencePC,Imm32,ImmL2,RegWD,RsData,RtData,ALUIn2,ALURes,MemRD,Instr;
    wire [4:0] RegWA;
    wire [27:0] PsudeoPC;
    wire BranchZ,J,B,Zero,RegDst,RegWr,ALUSrc,MemWR,Mem2Reg;
    wire [1:0] ALUOp;
    wire [3:0] ALUCtr;
    reg [31:0] PC;
    assign PsudeoPC={Instr[25:0],2'b00};
    assign JumpPC={SquencePC[31:28],PsudeoPC};
    assign SquencePC=PC+4;
    assign BranchPC=ImmL2+SquencePC;
    assign MuxPC=BranchZ?BranchPC:SquencePC;
    assign TempPC=J?JumpPC:MuxPC;
    assign BranchZ=Zero&B;
    assign ImmL2={Imm32[29:0],2'b00};
    assign Imm32={Instr[15]?16'hffff:16'h0,Instr[15:0]};
    assign ALUIn2=ALUSrc?Imm32:RtData;
    assign RegWA=RegDst?Instr[15:11]:Instr[20:16];
    assign RegWD=Mem2Reg?MemRD:ALURes;
    data_rom U_data_rom(ALURes[6:2],MemRD,RtData,MemWR,Clk);
    ALU U_ALU(RsData,ALUIn2,ALUCtr,ALURes,Zero);
    instruct_rom U_instruct_rom(PC[6:2],Instr,Clk);
    regFile 	       U_regFile(Instr[25:21],Instr[20:16],RegWA,RegWD,RegWr,Clk,Reset,RsData,RtData);
    master_Ctr U_master_Ctr(Instr[31:26],ALUOp,RegDst,RegWr,ALUSrc,MemWR,B,J,Mem2Reg);
    ALU_Ctr U_ALU_Ctr(ALUOp,Instr[5:0],ALUCtr);
    always @ (negedge Clk)
    begin
    if(Reset)
    PC<=0;
    else
    PC<=TempPC;
    end
endmodule

```

---

#### 实验步骤

1. 使用MARS获取测试程序的机器码，并以test.txt保存在D:/inter/mips_cpu路径下。

2. 按照上述编写各个子模块，并且对各子模块分级调试仿真。

3. 编写testbench对顶层模块实例化，进而对整个微处理器进行仿真，并且对仿真结果进行分析。

   testbench如下：

   ```verilog
   `timescale 1ns/1ns
   module test_main(
       );
       reg Clk,Reset;
       main_mips_cpu U0(Clk,Reset);
       parameter PERIOD=10;
       always begin
           Clk=1'b0;
           #(PERIOD/2) Clk=1'b1;
           #(PERIOD/2);
       end
       initial
       begin
       Reset=1;
       #20
       Reset=0;
       #2000
       $stop;
       end
   endmodule
   ```

---

#### 实验结果及分析

##### 测试的程序

```verilog
main:	add $4,$2,$3
        lw $4,4($2)
        sw $5,8($2)
        sub $2,$4,$3
        or $2,$4,$3
        and $2,$4,$3
        slt $2,$4,$3
        beq $3,$3,equ
        w $2,0($3)
equ:    beq $3,$4,exit
        sw $2,0($3)
exit:   j main
```

##### 实验仿真波形图

实验仿真结果波形图的重要信号如下图（其中存储器和寄存器数据只选择了前5组，因为只有前5个单元的存储器和寄存器数据才有变化）

![图6 实验仿真波形图](https://gitee.com//Riser_Wu/images/raw/master/img/22234421.png)

##### 实验结果分析

###### 测试信号

-  时钟信号Clk是以10ns为周期的正方波
-  复位信号Reset在20ns前为1即有效20ns后为0即无效

符合我们测试信号的预置。

###### 预置和边沿要求

- 寄存器的初始值全为0，存储器的初始值为各自的编号
- 指令读取在时钟上升沿完成，PC指针的修改，寄存器文件和数据存储器的写入在时钟下降沿完成。

符合实验要求

###### 测试执行的逻辑正确性

1.  PC的跳转情况

   通过指令指针PC的跳转情况，判断整个程序执行的整体跳转情况。我们发现PC的值通过一次递增4在0~32间来回循环，说明程序是在0到8号指令来回循环执行，这符合程序执行的跳转逻辑，我们结合后面的分析判断此处跳转的合理性。

2. 第一个循环的执行情况

   - 复位信号失效后的第一个时钟下降沿，先看到PC的值从0跳变到4，并且在下一个时钟上升沿第2条指令的机器码读入，再看这个时钟周期内，RegDst、RegWr为1，其余控制信号为0，表明ALU执行Rs、Rt两寄存器操作数的运算，并将ALU结果写入Rd寄存器，与add指令相符，时钟下降沿时寄存器和存储器值并未发生变化，因为第一条指令是将2号寄存器和3号寄存器的值相加赋予4号寄存器，这里所有寄存器的值都为0，所有没改变4号寄存器的值。

   - 时钟的第二个有效下降沿，PC的值从4变为8，并且在下一个时钟上升沿第3条指令的机器码读入，再看这个时钟周期内，ALUSrc、Mem2Reg、RegWr为1，其余控制信号为0，表明ALU执行Rs寄存器操作数和立即数的运算，并将结果作为地址得到对应存储器数据写入Rt寄存器，这与lw指令相符，并且在时钟下降沿4号寄存器变为1，我们看源指令lw $4,4($2)，将存储单元地址为4的存储单元值装载到4号寄存器，按照实验要求中数据存储器的地址规范要求，那么这里存储单元为4的就是1号存储单元，它的值为1,4号寄存器变为1，执行正确。

   - 时钟的第三个有效下降沿，PC的值从8变为12，并且此时第4条指令的机器码读入，再看这个时钟周期内，ALUSrc、MemWR为1，RegDst、Mem2Reg为x，其余控制信号为0，表明ALU执行Rs寄存器操作数和立即数的运算，得到存储器地址，并将Rt寄存器的值存入对应存储单元，这与sw指令相符，并且在时钟下降沿2号存储单元的值变为0，执行正确。

   - 接下来4个时钟周期，RegDst、RegWr为1，其余控制信号为0，表明ALU执行Rs寄存器操作数和Rt数的运算，并将得到的结果写入Rd寄存器，这是由于这4条指令执行的都是算术逻辑运算类指令，并将值赋予2号寄存器，容易发现2号寄存器值的改变也是正确的。

   - 时钟的第8个有效下降沿，PC的值从28变为32，说明第7条指令执行完毕，并且在下一个时钟上升沿第8条指令的机器码读入，这个时间周期内，B为1，RegDst、Mem2Reg为x，其余控制信号为0，PC的跳转由ALU单元的zero信号决定，这与beq指令译码相符，容易知道3号寄存器和4号寄存器的值不相等，那么程序往下执行，所以执行正确。

   - 时钟的第9个有效下降沿，PC的值从32变为0，说明第8条指令执行完毕，并且下一时钟上升沿第0条指令的机器码读入，这个时间周期内，BranchZ为1，RegDst、Mem2Reg、ALUSrc为x，其余控制信号为0，这与j指令译码相符，到8号指令调回0号指令，对应波形中PC从32跳回0，所以执行正确。

     综上，这个微处理器的仿真结果是正确的。

---

#### 实验总结

这次简单指令集MIPS微处理器设计仿真实验对微处理器的模块结构及功能有了更清晰的认识，对哈佛结构计算机的工作原理也有了更加深刻的理解，达到了巩固课内学习的效果，并且掌握了Verilog仿真平台的使用方法。

在这次实验中，感谢老师为我们提供了丰富的实验资源，实验过程中目的性非常明确，尽管还是难免遇到了不少问题，通过联系课内知识冷静思考，并且利用老师开设的讨论平台积极与同学们讨论，将这些问题逐一解决，这个过程也极大地锻炼了我。

