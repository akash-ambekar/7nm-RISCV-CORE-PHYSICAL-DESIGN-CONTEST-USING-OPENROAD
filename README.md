# 7nm-PHYSICAL-DESIGN-CONTEST-USING-OPENROAD
This repository gives brief review of Physical Design of RISC-V processor core using 7nm ASAP7 PDK. For this contest, the verilog code is provided and using OpenRoad EDA tool, we have performed RTLtoGDSII flow and improved the design PPA (Power, Performance, Area)

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

              ⇒   Minimum Clock Period      :   1760 ns
              ⇒   Maximum Clock Frequency   :   568.18 MHz
              ⇒   Core Utilization          :   45%
              ⇒   Design Area               :   2490 u^2
              ⇒   Power                     :   12.1 mW
              
              
              
              
# Proposed Methods to Improve fmax 

# 1) Modification in Clock Constraints 

File Location : ./flow/designs/asap7/ibex/constraints.sdc

As clock period is proportional to 1/fmax, reducing the clock frequency will result into enhanced fmax. But while doing so, one must take care that it should not cause any unrepairable setup violations when clock period is extreme reduced. We followed the step by step reduction and ended with best results at clock period = 850ps along with reducing clock to io delay to 0.01

![image](https://user-images.githubusercontent.com/100372947/228129497-7155a193-8138-4710-934a-1cd5939894ea.png)

# 2) Improving Core Utilization

Core utilization indicates the amount of core area used for cell placement. While improving the core utilization we have taken care that, core utilization mus
