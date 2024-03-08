# Brief description

**HI-CGRA-Generator:** Use Chisel to generator and simulate CGRA hardware. 
**HI-CGRA-Mapper:** Use llvm to generator DFG for kernel program, map DFG node to MRRG, generate the bitstram.
**HI-CGRA-Emu:** A C++ clock accurate CGRA Simulator
**HI-CGRA-Workbench:** A workbench where compile the kernel and use HI-CGRA-Mapper to generate DFG and bitstream.

Except for HI-CGRA-Generator, which is currently under development, all others have already formed a demo,and detailed documentation work will begin after HI-CGRA-Generator demo is formed

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
