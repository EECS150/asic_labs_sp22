# EECS 151/251A ASIC Project Specification: Checkpoint 4
<p align="center">
Prof. Sophia Shao
</p>
<p align="center">
TAs (ASIC): Dima Nikiforov
</p>
<p align="center">
Department of Electrical Engineering and Computer Science
</p>
<p align="center">
College of Engineering, University of California, Berkeley
</p>

---

## 1 Synthesis, PAR, & Power

### 1.1 Performing Synthesis and PAR
Make sure your design is backed up at this point.

The setup for Synthesis, and PAR is the similar to what we have used in the labs during the class,
with some formatting differences.  In `par.yml`, there is extra guidance 
for how to do placement constraints. Based on how you implemented the caches from the previous
checkpoint, you will need to modify these constraints to match the master SRAM cell used as well as
the path. Now you should be ready to proceed to Synthesis and PAR. As in the previous labs, execute
the following:
```shell
export HAMMER_HOME=$PWD/hammer
source hammer/sourceme.sh
```
The first thing you should do before simulating, is to make the SRAM libraries, with the command:
```
make srams
```
If you want to make sure the RTL has been pointed to correctly, you can try running the asm tests
from this environment. To do so, type the following commands:
```
make sim-rtl test_asm=all
```
The command generates the `simv` file, which is the simulation executable, then iterates through all
the asm tests using the root `Makefile`. If everything looks fine, you can proceed to Synthesis and
PAR:
```
make clean
make srams
make syn
make par
```
If everything went smoothly, you should now have a circuit laid out. To view the layout, go to
`build/par-rundir/` directory and type
```
./generated_scripts/open_chip
```
You are expected to record and document your area, power and clock frequency performance (as
determined by your critical path). To verify that your design works after PAR, use the following commands:
```
make sim-gl-par test_asm=all
```
Some final notes:
* You may also want to generally make sure that the post-synthesis netlist passes tests before moving onto post-PAR simulation, because the latter can be slower and will complicate your debugging with any PAR-related failures you may have (e.g. incomplete wiring of signal or clock nets
due to a bad floorplan).
* There is a new constraint added to `syn.tcl` under the key `vlsi.inputs.delays`. 
The external memory model in `riscv_test_harness.v` generates a delayed version of the signals
going into your CPU (see the parameter `INPUT_DELAY`). Annotating these delays for synthesis/PR 
is necessary in order capture this effect when the tools perform timing analysis. If you are
curious, this gets translated into `build/syn-rundir/pin_constraints_fragment.sdc`
as input to synthesis. After synthesis, the relevant pin delays are encoded in
`build/syn-rundir/riscv_top.mapped.sdc`. These are Synopsys Design Constraint
format files. Do not touch this delay constraint except to update the value as your clock period
divided by 5.
* As described in Lab 6, the ASAP7 dummy SRAMs do not have complete timing information.
This is most apparent in gate-level simulations because the SRAMs do not provide any SDF
timing annotation. You may find that despite meeting timing in synthesis and PAR, you will
likely need to increase the gate-level simulation clock period for the benchmarks to pass.

### 2 Checkpoint 4 Deliverables
*Checkoff due: May 6 (Friday), 2022*


1. Show that all of the assembly tests and final pass using the cache in a post-par simulation

2. Show your layout, and explain your design considerations when creating the floorplan

3. Show your final pipeline diagram, updated to match the code

---

## 3 Beyond Checkpoint 4: CPU Optimization

### 3.1 Optimizing for frequency
Beyond functionality, your final project grade will be determined by the maximum operating frequency
of your processor, determined by the critical path. You will also want to optimize for the number of
cycles that your processor takes to execute certain programs, more on that later. The critical path will
be dependent on how aggressively you ask the tools to optimize the design, by changing the target clock
period in the `syn.yml` file.

When Innovus is finished, look at the timing report for the critical path. In some cases, it is possible
to modify your Verilog to improve the critical path by moving pipeline stage registers. However in other
cases, timing can only be improved by tweaking settings in `syn.yml` and `par.yml`.

Be sure to backup (meaning check in or branch) your working design before attempting to move
logic, because functionality is worth much more of your grade than maximum frequency.

You are allowed to add additional pipeline stages, but remember that you will need to deal with the additional hazards that accompany them.
Be careful that adding additional stages does not increase the overall execution.
Your final performance metric is not only based on the clock speed at which your design will run, so keep
that in mind before heavily modifying your design.

Note for bonus grading: due to the SRAM timing issue described above, the maximum frequency
you achieved in PAR (not gate-level simulation) is most accurate and should be what you report for
frequency.

### 3.2 Optimizing for number of cycles
We are providing you tests that are the output of example C programs to run for your processor. They
are meant to be a representative example of different types of programs that each have different reasons
why they may take extra cycles to execute, for a variety of reasons including, but not limited to cache
misses, and branch/jump stalls. A more complicated cache structure may be able to reduce some of the
time spent waiting for memory accesses, but it may not be optimal for all cases. If you implement a
configurable cache you are allowed to set the cache settings differently on a per test basis, you will need
to add those pins to the top level Riscv151 file as well as the testbench with compile flags for VCS. In
terms of dealing with branching and jumping, you can implement any type of branch predictor that you
want to. A branch predictor in its simplest form will always choose to take (or not take) the branch and
then figure out if it was incorrect, and if so go back to where the instruction memory should have gone,
making sure that any additional instructions that were started do not change the state of the CPU. This
means that there should be no writes to memory or any registers for those instructions.

The list of final tests are contained within the Makefile under the variable `bmark_tests`, which
include a few tests that are meant to actually test the performance of your design. These tests are longer
C programs that are meant to test different aspects of your design and how you handle different types
of hazards. To run these longer tests you can run the following commands, like in checkpoint #3:
```
make sim-rtl test_bmark=all
```
You may need to increase the number of cycles for timeout for some of the longer tests (like sum,
replace and cachetest) to pass.

### 3.3 Optimizing for power
**DISCLAIMER:** The infrastructure to do power analysis in this project is very different from gate-level
simulation and power analysis so far. Doing this optimization is *purely optional* and should only be
tried after you can pass the benchmarks normally! **Proceed at your own risk!**

You have the ability to also find out the power consumption of your processor for the various provided benchmarks. 
The value of this is to figure out whether the way you wrote your logic is efficient
and avoids extra switching activity. Simplify instruction decode logic, forwarding paths, etc. can result
in lower power consumption!

Near the bottom of `sim-gl-par.yml`, you will see a few lines:
```
execute_sim: false
# Below is for power analysis. See the spec for instructions!
# execution_flags_append:
# - "+loadmem=../../tests/asm/addi.hex"
# - "+max-cycles=10000"
```
If you reverse the comments (i.e. comment out `execute_sim`: false and uncomment the
rest), this tells Hammer to run the `simv` executable with the addi test, instead of having the `Makefile`
in the root folder run the executable. This is currently the only way that we can get Hammer currently
to generate the SAIF file with our benchmark hex files. To proceed with the simulation of addi in this
case:
```
make sim-gl-par test_asm=addi.out
```
You will find that it will do a simulation twice due to how the root `Makefile` is configured. The
first one should pass (after a lot of printing each cycle number), while the second one should also pass
like you have seen so far—ignore this second simulation. You should now see an `ucli.saif` file in
`build/sim-rundir`. Then, as in previous labs, run Voltus:
```
make power-par
```
And you should get static and dynamic power reports in `build/power-rundir`.

Some closing recommendations:

* This infrastructure only allows us to run one benchmark at a time. To run a different benchmark,
replace the hex file in the `execution_flags_append` list, and alter the `max-cycles` value
as necessary (see the `*_timeout_cycles_variables` in the root Makefile for the numbers).
* Due to the ASAP7 PDK’s dummy SRAMs, we can’t measure SRAM power, and thus can’t find
out how power-efficient our caching is. Therefore, the best benchmarks to run would be an
arithmetic-heavy one that relies heavily on the register file (but the provided benchmarks require
memory accesses). If you have lots of time on your hand, we encourage you to find power
numbers for the **final** benchmark, but **you will not be graded on power performance.**

---


## Acknowledgement

This project is the result of the work of many EECS151/251 GSIs over the years including:
Written By:
- Nathan Narevsky (2014, 2017)
- Brian Zimmer (2014)
Modified By:
- John Wright (2015,2016)
- Ali Moin (2018)
- Arya Reais-Parsi (2019)
- Cem Yalcin (2019)
- Tan Nguyen (2020)
- Harrison Liew (2020)
- Sean Huang (2021)
- Daniel Grubb, Nayiri Krzysztofowicz, Zhaokai Liu (2021)
- Dima Nikiforov (2022)
