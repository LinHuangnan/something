# 【Vivado开发教程】Vivado设计流程及使用模式

- [ ] Version
    * [x] linhuangnan
    * [x] 2024-02-16 
    * [x] Vivado开发教程
    * [ ] review

!!! info
    * The difference of FPGA design flow between ISE and Vivado
    * Vivado use modes
    * Demonstrate project mode design flow features

## 1、The difference of FPGA design flow between ISE and Vivado

### 1.1 ISE的设计流程

![213](../img/213.png)

ISE设计流程通常包括综合、翻译、映射、布局/布线、时序分析以及比特流生成等。下面分别解释这些步骤中的XST, NGDBuild, MAP, PAR, TRCE和BitGen：

#### XST (Xilinx Synthesis Technology)
XST是Xilinx提供的综合工具。综合是将硬件描述语言（如Verilog或VHDL）编写的设计转换成逻辑门电路的过程。XST允许设计者将他们的高层次设计描述转化为一个优化后的网表。这个网表列举了所有用于实现设计的基本元素，例如查找表（LUTs）、触发器（flip-flops）以及其他资源。

#### NGDBuild
NGDBuild是ISE工具链中的“翻译”步骤。它接受综合阶段产生的网表文件，并将其转换为Xilinx特定格式的网表文件（.ngd）。这个过程中，NGDBuild确保所有的硬件原语被正确地映射到FPGA上可用的物理元素上，并处理任何约束文件（如UCF文件，用户约束文件）以依据设计需求配置FPGA的资源。

#### MAP
MAP工具执行逻辑映射过程，它把NGDBuild生成的.ngd文件中描述的电路映射到FPGA内部的实际物理资源上。这个过程中，它会考虑各种实现选项和用户指定的约束条件，来优化逻辑与FPGA资源之间的映射，以达到最好的性能。

#### PAR (Place and Route)
PAR意为布局/布线。这个步骤接着MAP工具的输出并进行布局和布线，布局是指确定每个逻辑元素在FPGA芯片上的具体位置，布线则是创建连接这些元素的互联路径。PAR的目标是找到一个布局/布线解决方案，使得设计满足时钟频率要求并具有最佳的性能表现。

#### TRCE (Timing Report Constraint Editor)
TRCE是一个时序分析工具，它提供了静态时序分析功能。TRCE读取由PAR完成的布线信息，审核整个设计是否满足时序约束，如设置时间（setup time）、保持时间（hold time）和时钟转换时间。它会生成一个详细的时序报告，指出可能存在的问题区域，并帮助设计者做出调整以改善设计的时序性能。

#### BitGen (Bitstream Generation)
BitGen是位流文件生成器。它使用布线后的设计文件生成一个位流文件（.bit），该文件包含了配置FPGA的所有必要信息。这个位流文件可以通过下载电缆或其他方法传输到FPGA上，程序并配置FPGA以执行所设计的功能。

**ISE的每一个步骤都会有不同的数据模型生成**

### 1.2 Vivado的设计流程

![214](../img/214.png)

在整个设计流程中，Vivado生成的都是统一的设计模型（.dcp Design Checkpoint）

### 1.3 Vivado的系统级设计流程

![215](../img/215.png)

**因此Vivado是以IP为核心的设计**

### 1.4 什么是Design Checkpoint

![216](../img/216.png)

### 1.5 Design Database

![217](../img/217.png)

"Processes access the underlying database of your design" 指的是在数字芯片设计流程中，不同的处理步骤（processes）会读取和修改存储设计信息的基础数据库。这个基础数据库持有关于设计的所有信息，包括其结构、行为描述、时序信息和约束条件等。

在整个设计过程中，有不同类型的网表（netlists）会被使用和生成：

#### Elaborated Netlist
这个网表是指经过“展开”（elaboration）处理的网表，它是从高层次描述（比如Verilog或VHDL源代码）转换而来的，并且包含了所有的设计层次结构以及模块实例化信息。此时的网表尚未进行针对目标技术的优化。展开的网表通常用于初步的功能验证和时序分析。
#### Synthesized Netlist
综合处理将高层次的逻辑描述转换为与特定制造技术对应的逻辑门和其他原语的表示形式，生成所谓的“综合网表”。
这个网表代表了设计将如何在目标硬件（例如FPGA或ASIC）上物理实现，包括查找表（LUTs）、触发器（flip-flops）和逻辑门等。
综合网表可以进行时序分析，以确保设计能满足时序要求。
#### Implemented Netlist
在完成映射（MAP）、布局（Place）和布线（Route）工作后，生成的网表称为“实现网表”。
实现网表涉及更多的物理细节，如单元布局位置和互连路径，它反映了设计在实际硬件上的物理布局。
针对实现网表的时序分析将考虑到实际延迟和加载，这是最终确认是否满足性能指标的重要步骤。
在每一个步骤中，相关的处理流程都可能会修改当前网表或者根据需要创建新的网表。例如，在综合步骤之后，可能会产生一个新的综合网表；在布局/布线步骤之后，会创建一个反映了所有物理位置和互连的实现网表。

## 2、Vivado use modes

![218](../img/218.png)

可以把不同的设计输入放在不同的文件夹当中，比如verilog和VHDL代码放在src文件夹中，ip放在ip文件夹中，约束放在xdc文件

vivido的使用方式可以是`Project Mode`，也可以是`Non-Project mode`。其中Project Mode可以支持图形界面的方式，也可以支持Tcl脚本的方式，而Non-Project mode只能以Tcl脚本的方式运行。

### 2.1 Project Mode

![219](../img/219.png)

![220](../img/220.png)

### 2.2 Non-Project Mode

![221](../img/221.png) 

## 3、Demo：Project Mode Design Flow

### 3.1 GUI界面方式

准备下面几个文件夹

![222](../img/222.png) 

创建工程后文件结构如下

![222](../img/222.png) 

（1）打开vivado，创建一个新的工程

![224](../img/224.png) 

（2）选择next

![225](../img/225.png) 

（3）选择工程名和工程路径

![226](../img/226.png) 

（4）选择工程类型

![227](../img/227.png) 

（5）选择芯片型号

![228](../img/228.png) 

（6）界面如下图所示

![229](../img/229.png) 

（7）添加设计源文件

![230](../img/230.png) 

（8）添加RTL代码

![231](../img/231.png) 

（9）将整个RTL代码目录添加到工程，并且拷贝到工程目录下

![232](../img/232.png) 

（10）在design source中可以看到文件已经加入vivado工程中

![233](../img/233.png) 

（11）添加相应的IP

![234](../img/234.png) 

![235](../img/235.png) 

（12）添加相应的约束

![236](../img/236.png) 

![237](../img/237.png) 

（13）添加相应的仿真文件

![238](../img/238.png) 

![239](../img/239.png) 

（14）运行综合

![240](../img/240.png) 

![241](../img/241.png) 

（15）综合完成后可以查看综合后的原理图

![242](../img/242.png) 

（16）接下来进行实现，可以打开实现的配置

![243](../img/243.png) 

（17）Run Implementation

![244](../img/244.png) 

（18）生成比特文件

![245](../img/245.png) 

### 3.2 Tcl脚本方式

（1）打开Tcl Shell

![245](../img/245.png) 

（2）切换到相应工作目录，然后source TCL脚本

```sh
  # Set project directory and create it
  set workdir {F:\Vivado\Tutorial\design_flow}
  set workdir [file normalize $workdir]
  cd $workdir
  set prjname wave_gen_project
  file mkdir ./$prjname
  set part xc7k325tffg900-2
  create_project $prjname ./$prjname -part $part
  # Add design source to this project
  add_files -fileset sources_1 -norecurse -scan_for_includes -quiet ./src
  update_compile_order -fileset sources_1
  add_files -fileset sources_1 ./ip/char_fifo/char_fifo.xci
  add_files ./ip/clk_core/clk_core.xci
  update_compile_order -fileset sources_1
  add_files -fileset constrs_1 -quiet ./xdc
  add_files -fileset sim_1 -quiet ./sim
  update_compile_order -fileset sim_1
  puts "Create project correctly!"
  # Synthesis, implementation and bitgen
  launch_runs synth_1
  wait_on_run synth_1
  launch_runs impl_1 -to_step write_bitstream
  wait_on_run impl_1
  open_run impl_1
  start_gui
```


## 4、Summary

![223](../img/223.png) 


### IP-Centric system level design integration (基于IP的系统级设计集成)
- Create and package IP and add it to IP catalog (创建和封装IP，然后添加到IP目录中)
创建自定义的知识产权(IP)模块，并将其打包，以便可以将其添加到工具的IP目录中，供设计者使用。

- Create IP subsystems with IP Integrator (使用IP集成器创建IP子系统)
使用一个特定的集成工具（如Xilinx Vivado中的IP Integrator），可以快速地将多个IP核组合成一个功能更为复杂的IP子系统。 这些子系统可能包括嵌入式处理器、数字信号处理(DSP)单元或高级综合(HLS)生成的组件。

### System design entry through hardware validation flows (通过硬件验证流程进行系统设计输入)
- Advanced design, analysis and process management capabilities (先进的设计、分析和过程管理能力) 提供一套完整的工具来进行复杂设计的输入、分析与管理，帮助设计者完成从概念到最终产品的全过程。

### Flexible use models (灵活的使用模型)

- Tcl accessible common data model throughout the flow (在整个流程中都可以用Tcl访问的通用数据模型)
在设计流程的每个步骤中，都可以使用工具命令语言(Tcl)脚本来访问和操作设计数据，这提供了极大的灵活性和可自定义性。

- Project and compilation style flows (项目和编译风格的流程)
支持传统的以项目为中心的工作流程，也支持类似于软件开发中的编译流程，方便不同习惯的工程师选择适合自己的工作方式。

- Support for third party software tools (支持第三方软件工具)
集成环境不仅支持自身的工具和库，还允许接入第三方软件工具，使得用户可以利用外部资源来增强设计能力。

!!! More Info
    Ug888: design flows overview