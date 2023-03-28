# 7nm-PHYSICAL-DESIGN-CONTEST-USING-OPENROAD
This repository gives brief review of PPA Improvement by Physical Design of RISC-V processor core using 7nm ASAP7 PDK. For this contest, the verilog code is provided and using OpenRoad EDA tool, we have performed RTLtoGDSII flow and improved the design PPA (Power, Performance, Area)

![image](https://user-images.githubusercontent.com/100372947/228320761-19c35794-58bf-4d11-a124-4978eca0c164.png)

# Contents

- [OpenROAD EDA Tool](#openroad-eda-tool)
- [Proposed Work & Initial Design Constraints](#proposed-work-&-initial-design-constraints)
- [Reference Circuit Diagram](#reference-circuit-diagram)
- [Circuit Details](#circuit-details)
- [Resolution of ADC](#resolution-of-adc)
- [Proposed Methodology](#proposed-methodology)
- [EDA Tools Used](#eda-tools-used)
- [Verilog Code](#verilog-code)
- [Makerchip](#makerchip)
- [Creating model of 8:3 Encoder using NgVeri](#creating-model-of-8-3-encoder-using-ngveri)
- [Schematics](#schematics)
- [Netlist](#netlist)
- [Output Waveforms](#output-waveforms)
- [GAW Waveforms](#gaw-waveforms)
- [Author](#author)
- [Acknowledgements](#acknowledgements)
- [References](#references)


# OpenROAD EDA Tool
OpenROAD is an integrated chip physical design tool that takes a design from synthesized Verilog to routed layout.

• An outline of steps used to build a chip using OpenROAD is shown below:

• Initialize floorplan - define the chip size and cell rows

• Place pins (for designs without pads )

• Place macro cells (RAMs, embedded macros)

• Insert substrate tap cells

• Insert power distribution network

• Macro Placement of macro cells

• Global placement of standard cells

• Repair max slew, max capacitance, and max fanout violations and long wires

• Clock tree synthesis

• Optimize setup/hold timing

• Insert fill cells

• Global routing (route guides for detailed routing)

• Antenna repair

• Detailed routing

• Parasitic extraction

• Static timing analysis

OpenROAD uses the OpenDB database and OpenSTA for static timing analysis.

ORFS Documentation & Installation Guide : https://openroad.readthedocs.io/en/latest/main/README.html



# Proposed Work & Initial Design Constraints

In this contest, we have provided with a verilog code for RISC-V processor core (Code Link : https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/master/flow/designs/src/ibex) and we are expected to run the RTLtoGDSII flow for given design and improve the PPA with respect to any one parameter. Considering the code, we have decided to improve the performance of the design by improving fmax of design.

The initial design constraints are given as follows :

              ⇒   Minimum Clock Period      :   1760 ps
              ⇒   Maximum Clock Frequency   :   568.18 MHz
              ⇒   Core Utilization          :   45%
              ⇒   Design Area               :   2490 u^2
              ⇒   Power                     :   12.1 mW
              
              
              
              
# Proposed Methods to Improve fmax 

  1) Modification in Clock Constraints 
  2) Improving Core Utilization\
  3) Replacing RVT Cells by LVT Cell & using FF Library
  4) Modification of Supply Voltage
  5) Modification of RC Parasitics File
  6) Increasing Metal Layer for Routing
  7) Modification in Placement Density
  8) Modification in CTS Buffer Distance
  
# 1) Modification in Clock Constraints 

File Location : ./flow/designs/asap7/ibex/constraints.sdc

As clock period is proportional to 1/fmax, reducing the clock frequency will result into enhanced fmax. But while doing so, one must take care that it should not cause any unrepairable setup violations when clock period is extreme reduced. We followed the step by step reduction and ended with best results at clock period = 850ps along with reducing clock to io delay to 0.01

![image](https://user-images.githubusercontent.com/100372947/228129497-7155a193-8138-4710-934a-1cd5939894ea.png)

# 2) Improving Core Utilization

File Location : ./flow/designs/asap7/ibex/config.mk

Core utilization indicates the amount of core area used for cell placement (in %). While improving the core utilization we have taken care that, high core utilization must not create NP routing difficulty. Hence, we tune the core utilization within the range of 40 to 60 and iterated the design to get best results.

# 3) Replacing RVT Cells by LVT Cell & using FF Library

File Location : ./flow/platforms/asap7/config.mk

During synthesis, we are provided with various options to use Slow-Slow/Fast-Fast library & RVT/LVT/SLVT library cells. Considering improving fmax as utmost goal, we used FF library & replaced all RVT cells by LVT cells. LVT cells have lower threshold voltage which results into high current comparing with RVT cells having high threshold. Dur to large current, faster switching will be achieved which results into improvement of fmax. No doubt LVT cells may cause higher leakage power than RVT but no significant changes in power has been observed after replacement.

![image](https://user-images.githubusercontent.com/100372947/228182826-bb63eb6e-742d-4b11-a2bb-d10cee2d064f.png)

# 4) Modification of Supply Voltage

File Location : ./flow/platforms/asap7/config.mk

The supply voltage is provied in 3 formats namely Best, Typical & Worst. Initially it was 0.77/0.7/0.63 V i.e. +-10% tolerance band. But if we increase the supply voltage, drain current will rise and results in faster switching which ultimately improve fmax. Considering this, we increased value of supply voltage 1.3/1
27/1.2 V (Best/Typical/Worst) and observed the improvements.

![image](https://user-images.githubusercontent.com/100372947/228216576-5d552642-9aac-4635-9b4e-75b9836edfcd.png)


# 5) Modification of RC Parasitics File

File Location : ./flow/platforms/asap7/setRC.tcl

Parasitics are the components that degrade the performance of the design. Generally the metal resistances & stray capacitances are the most dominant causes for this RC parasitics drop which causes larger delay. In order to reduce this delay, we reduce the metal & via resistance along with metal capacitances.


# 6) Increasing Metal Layer for Routing

File Location : ./flow/platforms/asap7/config.mk

In initial config file, the max metal layer for routing is set to M7 where as upto M9 layers are provided in the PDK. So, increasing metal layers reduces the routing congestions.

![image](https://user-images.githubusercontent.com/100372947/228185474-b25bd738-67bd-4fc0-b3cc-9a09d16e6ff0.png)

# 7) Modification in Placement Density

File Location : ./flow/platforms/asap7/config.mk

Placement density is the utilization of the cells in your design. In practical, the equation is the placement density should be less than 70%. so that remaining 30% can be utilized for routing. We have tuned the placement density in range of 50% to 65% and found best results at 55% placement density

# 8) Modification in CTS Buffer Distance

File Location : ./flow/platforms/asap7/config.mk

While doing CTS, the tool periodically insert buffers to avoid degradation of signal but inserting multiple buffers at very small distances will result into unnecessary rise in overall latency. To avoid this, we finely tuned and increased the CTS Buffer Distance.

# Final Layout

![image](https://user-images.githubusercontent.com/100372947/228250952-3a71721b-d7d1-4e80-9312-049f152e24ec.png)

# Clock Tree

![Screenshot (87)](https://user-images.githubusercontent.com/100372947/228317757-abf9e95a-1b53-4691-b34e-2dda3e6a61f9.png)

# DRC 

As the OpenROAD tool itself iterate the design routing unless all DRC violations gets removed, the design ensures 0 DRC violations

# Results & Design Achievements

![image](https://user-images.githubusercontent.com/100372947/228320811-a0426bab-5ec5-4051-8ce8-c03e8b342289.png)

![image](https://user-images.githubusercontent.com/100372947/228317683-f4211932-5c46-4728-b044-ddfee0050a98.png)

The biggest achievement of the design is to run the design at maximum operating frequency of 1.17GHz. Along with it, there is a slight improvement in design area along with core utilization. Due to trade off, we see the enhancement in power which can be reduced by further modifications.

The final modified design constraints are given as follows :

              ⇒   Minimum Clock Period      :   850 ps
              ⇒   Maximum Clock Frequency   :   1.176 GHz
              ⇒   Core Utilization          :   55%
              ⇒   Design Area               :   2417 u^2
              ⇒   Power                     :   32.3 mW
              

# Acknowledgements

1.	Kunal Ghosh (Co-Founder, VLSI System Design Pvt. Ltd.)

2.	Dr. Hemanta Kumar Mondal (Professor, Dept. of ECE, NIT Durgapur)	
	
3.	Sumanto Kar (Sr. Project Technical Assistant, IIT Bombay)

4.  Vijayan Krishnan


# References 

OpenROAD Flow Scripts & Documentation : https://openroad.readthedocs.io/en/latest/main/README.html
