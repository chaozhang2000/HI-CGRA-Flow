# Brief description

**HI-CGRA-Generator:** Use Chisel to generator and simulate CGRA hardware.  
**HI-CGRA-Mapper:** Use llvm to generator DFG for kernel program, map DFG node to MRRG, generate the bitstram.  
**HI-CGRA-Emu:** A C++ clock accurate CGRA Simulator.  
**HI-CGRA-Workbench:** A workbench where compile the kernel and use HI-CGRA-Mapper to generate DFG and bitstream.  

Except for HI-CGRA-Generator, which is currently under development, all others have already formed a demo,and detailed documentation work will begin after HI-CGRA-Generator demo is formed.

# Project configuration

Ubuntu 22.04

## HI-CGRA-Generator

1.  install dependency.
```
sudo apt-get install make curl openjdk-11-jdk
echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list
curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add
sudo apt-get update
sudo apt-get install sbt
```
[mill]: https://com-lihaoyi.github.io/mill/ 

## HI-CGRA-Mapper

1. install dependency
```
sudo apt-get install clang-12 make build-essential llvm llvm-12-dev graphviz bison flex
```
2. add MAPPER\_HOME to ~/.bashrc

## HI-CGRA-Workbench

1. add CGRA\_WORKBENCH to ~/.bashrc  

## HI-CGRA-Emu

1. install dependency
```
sudo apt-get install make build-essential bison flex libncurses-dev
```
2. add CGRA\_EMU\_HOME to ~/.bashrc

**For more information, please read the README.md file in this four folders**

# Demo

## Clone

```shell
git clone https://github.com/chaozhang2000/HI-CGRA-Flow.git
cd ./HI-CGRA-Flow.git
git submodule init
git submodule update
```

## Compile conv3 demo

You can read the code of conv3 kernel in "./HI-CGRA-Workbench/kernels/conv3/conv3.c".
```c
void kernel(int *line1,int *line2,int *line3,int *result, long k){
	int kernel[3][3]= {{1,2,3},{4,5,6},{7,8,9}};
	result[k] = line1[k-1]*kernel[0][0] + line1[k]*kernel[0][1] + line1[k+1]*kernel[0][2]
	+line2[k-1]*kernel[1][0] + line2[k]*kernel[1][1] + line2[k+1]*kernel[1][2]
	+line3[k-1]*kernel[2][0] + line3[k]*kernel[2][1] + line3[k+1]*kernel[2][2];
}
```

You need to compile the HI-CGRA-Mapper first, you should config the project at the first compilation.
```shell
cd ./HI-CGRA-Mapper
make defconfig # or make menuconfig
```

Compile the conv3 demo in HI-CGRA-Workbench.
```shell
cd ./HI-CGRA-Workbench
make KERNEL=conv3
```

The above command first use clang to compile conv3.c and generate kernel.bc, and then uses CGRA-Mapper to generate bitstreams and dataflow graphs.  
You can read LLVM IR by reading the kernel.ll file in "./HI-CGRA-Workbench/kernels/conv3/build".
```llvm
define dso_local void @kernel(i32* nocapture readonly %0, i32* nocapture readonly %1, i32* nocapture readonly %2, i32* nocapture %3, i64 %4) local_unnamed_addr #1 {
  %6 = add nsw i64 %4, -1
  %7 = getelementptr inbounds i32, i32* %0, i64 %6
  %8 = load i32, i32* %7, align 4, !tbaa !2
  %9 = getelementptr inbounds i32, i32* %0, i64 %4
  %10 = load i32, i32* %9, align 4, !tbaa !2
  %11 = shl nsw i32 %10, 1
  %12 = add nsw i32 %11, %8
  %13 = add nsw i64 %4, 1
  %14 = getelementptr inbounds i32, i32* %0, i64 %13
  %15 = load i32, i32* %14, align 4, !tbaa !2
  %16 = mul nsw i32 %15, 3
  %17 = add nsw i32 %12, %16
  %18 = getelementptr inbounds i32, i32* %1, i64 %6
  %19 = load i32, i32* %18, align 4, !tbaa !2
  %20 = shl nsw i32 %19, 2
  %21 = add nsw i32 %17, %20
  %22 = getelementptr inbounds i32, i32* %1, i64 %4
  %23 = load i32, i32* %22, align 4, !tbaa !2
  %24 = mul nsw i32 %23, 5
  %25 = add nsw i32 %21, %24
  %26 = getelementptr inbounds i32, i32* %1, i64 %13
  %27 = load i32, i32* %26, align 4, !tbaa !2
  %28 = mul nsw i32 %27, 6
  %29 = add nsw i32 %25, %28
  %30 = getelementptr inbounds i32, i32* %2, i64 %6
  %31 = load i32, i32* %30, align 4, !tbaa !2
  %32 = mul nsw i32 %31, 7
  %33 = add nsw i32 %29, %32
  %34 = getelementptr inbounds i32, i32* %2, i64 %4
  %35 = load i32, i32* %34, align 4, !tbaa !2
  %36 = shl nsw i32 %35, 3
  %37 = add nsw i32 %33, %36
  %38 = getelementptr inbounds i32, i32* %2, i64 %13
  %39 = load i32, i32* %38, align 4, !tbaa !2
  %40 = mul nsw i32 %39, 9
  %41 = add nsw i32 %37, %40
  %42 = getelementptr inbounds i32, i32* %3, i64 %4
  store i32 %41, i32* %42, align 4, !tbaa !2
  ret void
}
```

If successful, you will find the following files in "./HI-CGRA-Workbench/outputs/conv3\_0"  
1. bitstream.bin: the CGRA config bitstream.
2. conv3\_0.png and kernel.dot: the DFG of the conv3 kernel function.

conv3\_0.png is displayed below
<div style="text-align:center;">
    <img src="./doc/demodfg.png" alt="demodfg" style="max-width:100%; height:auto; display:inline-block;">
</div>

* Each black DFGnode in DFG is an instruction in LLVM IR. 
* Each blue DFGNode in DFG is a const in instruction. 
* Each read DFGNode in DFG is an argument of the kernel function.The parameters of the function include the loop number i labeled "loop:0" in the graph, as well as other parameters such as line1, line2, line3 and result  which are also labeled "%0","%1","%2" and "%3" in the graph.
* The number 0 or 1 on the DFGEdge indicates which operand this is.0 represents the first operand, 1 represents the second operand.
* The Orange DFGEdges marks the longest path in the DFG.
* We give a level to each DFGNode,the DFGNode with lower level executes earlier and is mapped earlier.

After generating the DFG,HI-CGRA-Mapper map the DFG to MRRG, a simple MRRG is shown in the following figure.
<div style="text-align:center;">
    <img src="./doc/mrrgsimple.png" alt="mrrgsimple" style="max-width:100%; height:auto; display:inline-block;">
</div>

This MRRG shows the resources that CGRAs of five PEs can use within two clock cycles.Each block in this figure represents the PE of a certain clock cycle, which can perform an operation this cycle and transfer data to neighbor PEs in next clock cycle.The red Edge in this figure indicates the transmission of date.The vertical downward arrow indicates that the data is saved in PE for one clock cycle.

This demo's CGRA Hardware is shown in this following figure.MRRG for this CGRA is bigger.
```
  the default CGRA

          col0   col1   col2   col3
  |----| |----| |----| |----| |----|
  |Mem3| |    |-|    |-|    |-|    | row3
  |----| |----| |----| |----| |----|
           |      |      |      |
  |----| |----| |----| |----| |----|
  |Mem2| |    |-|    |-|    |-|    | row2
  |----| |----| |----| |----| |----|
           |      |      |      |
  |----| |----| |----| |----| |----|
  |Mem1| | 4  |-|....|-|    |-|    | row1   ^ y
  |----| |----| |----| |----| |----|        |
           |      |      |      |           |
  |----| |----| |----| |----| |----|        |
  |Mem0| | 0  |-| 1  |-| 2  |-| 3  | row0   |
  |----| |----| |----| |----| |----|        |

  --------------------------------> x
```
In this demo,CGRA is 4x4.Each PE is connected to the upper,lower,left,and right PEs.Each PE in a row can access a datamem,for example,PE0,1,2,3 can access Datamem0.You can read CGRA params in "./HI-CGRA-Workbench/kernel/conv3/param.json". The partial content of this file is as follows.
```json
{
  	"kernel"                :"kernel",
	"rows"			:4,
	"cols"			:4,
	"instmemsize"		:10,
	"constmemsize"		:8,
	"shiftconstmemsize"	:8,
	"datamemsize"		:400,
	"optlatency"		:{"load" : 1,"store":1},
	"optpipeline"		:["load" ,"store"],
	"datamemnum"		:4,
	"datamemaccess"		:{
	 				"0" : [0,1,2,3],
					"1" : [4,5,6,7],
					"2" : [8,9,10,11],
					"3" : [12,13,14,15]
	 			}
}
```
* **kernel:** name of kernel function
* **rows:** rows of cgra.
* **instmemsize:** size of instmem in each PE .
* **datamemsize:** size of each datamem.
* **optlatency:** define which operations require multiple cycles.""load":1" mean load opt need one cycle.Operations that can get result within the current clock cycle have a latency value of 0.
* **optpipeline:** define which operations is pipelined,but not implemented yet.Currently,only pipelined multi-cycle operations are supported for multi-cycle operations, so just need to add them to optlatency.
* **datamemnum:** num of datamems.should >= mems define in datamemaccess.
* **datamemaccess:** define which PEs can access each memory,""0":[0,1,2,3]" mean PE 0,1,2,3 can access datamem 0. datamem id start from 0.
You can change rows and cols to map conv3 to different cgra to start. We have test rows and cols in range of 4 to 9.You can also change datamemaccess to test.You can also change opt's latency,for example add ""mul":1" to test.

In this demo we constrain load and store opts which access the first row of image to datamem0 ,sencond row to datamem1, third row to datamem2 and result to datamem3.You can see this constraint in "./HI-CGRA-Workbench/kernel/conv3/mapconstraint.json".This constraint means that a specific memory access operation will only be mapped to specific PEs,for example,load opt that access the first row of image,will only by mapped to the PE can access Datamem0, in this example is PE0,1,2,3.The mapconstraint.json is as follows.
```json
{
  "DFGNodeIDCGRANodeID"		: [[0,5]],
  "DFGNodeIDMemID"		: [[2,0],[4,0],[9,0],[13,1],[17,1],[21,1],[25,2],[29,2],[33,2],[37,3]]
}
```
* **DFGNodeIDCGRANodeID:** "[a,b]" This parameter specifies that a DFGNode with ID=a will be forcibly mapped to a CGRANode with ID=b.
* **DFGNodeIDMemID:** "[a,b]" This parameter specifies that a DFGNode with ID=a will be mapped to a CGRANode which can access Datamem b.Every mem access opt should be bound to a specific CGRANode.If a DFGNode is not mem access opt, it will be ignore.A mem access opt can also be bound to a certain CGRANode using DFGNodeIDCGRANodeID.

The demo conv3's DFG mapped in the MRRG is shown in the following figure.This result is not the latest version. In this picture,we bound mem access opts to PE0,1,2,3. Each PE can access one datamem.And all opts latency is consider 0.The DFGNode ID and operation name in the PE indicate which DFGNode occupies the PE in a certain clock cycle.Operands of DFGNodes which comes from the calculation results of other DFGNodes are represented by edges in this figure.Operands of DFGNodes which comes from const or function param is put in PE's constmem or control regs,this is not show in this figure.
<div style="text-align:center;">
    <img src="./doc/convmrrg1.png" alt="convmrrg1" style="max-width:100%; height:auto; display:inline-block;">
</div>

HI-CGRA-Mapper successfully maps with II = 6.We need 6 clock cycles to calculate the kernel function for one time.This is shown in following figure.In our conv3 application,we need to run this kernel function for many times.
<div style="text-align:center;">
    <img src="./doc/convmrrg2.png" alt="convmrrg2" style="max-width:100%; height:auto; display:inline-block;">
</div>

## Run conv3 demo in HI-CGRA-Emu

You need to compile the HI-CGRA-Emu first, config the project before the first compilation..
```shell
cd ./HI-CGRA-Emu
make defconfig # or make menuconfig
```

Then you can compile run the Emu to simulate.In "./HI-CGRA-Workbench". make sure you have compiled the kernel, the bitstream.bin and param.json generated by HI-CGRA-Mapper will be pass to HI-CGRA-Emu.
```shell
cd ./HI-CGRA-Workbench
make sim KERNEL=conv3
```
When run a applicate in HI-CGRA-Emu,you need load data to DataMems first. This part of code is put in "./HI-CGRA-Workbench/kernel/conv3/kernel.h". for a new applicate, you need write your own kernel.h, but don't modify the function name.
Code of HI-CGRA-Emu is simple now, it's also a good way to understand our hardware's behavior.

You can see the result shown in following figure.
<div style="text-align:center;">
    <img src="./doc/result.png" alt="result" style="max-width:100%; height:auto; display:inline-block;">
</div>

# TODO
