# EECS 151/251A ASIC Project Specification RISC-V Processor Design: Overview
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
## 1. Introduction

The primary goal of this project is to familiarize students with the methods and tools of digital design. In order to make the project both interesting and useful, we will guide you through the implementation of a CPU that is intended to be integrated on a modern SoC. Working alone or in teams of 2, you will be designing a simple 3-stage CPU that implements the RISC-V ISA, developed here at UC Berkeley. If you work in a team, you must both have a complete understanding of your entire project code, and you will both receive the same grade.

Your first and most important goal is to write a functional implementation of your processor. To better expose you to real design decisions, you will also be tasked with improving the performance of your processor. You will be required to meet a minimum performance to be specified later in the project.

You will use Verilog HDL to implement this system. You will be provided with some testbenches to verify your design, but you will be responsible for creating additional testbenches to exercise your entire design. Your target implementation technology will be the ASAP7 7nm Educational PDK, a predictive model technology used for instruction. The project will give you experience designing synthesizeable RTL (Register Transfer Level) code, resolving hazards in a simple pipeline, building interfaces, and approaching system-level optimization.

Your first step will be to map our high level specification to a design which can be translated into a hardware implementation. You will then generate and debug that implementation in Verilog. These steps may take significant time if you do not put effort into your system architecture before attempting implementation. After you have built a working design, you will be optimizing it for speed in the 7nm technology that we have been using this semester.


### 1.1 RISC-V
The final project for this class will be a VLSI implementation of a RISC-V (pronounced risk-five) CPU. RISC-V is an instruction set architecture (ISA) developed here at UC Berkeley. It was originally developed for computer architecture research and education purposes, but recently there has been a push towards commercialization and industry adoption. For the purposes of this lab, you don’t need to delve too deeply into the details of RISC-V. However, it may be good to familiarize yourself with it, as this will be at the core of your final project. Check out the official [RISC-V Instruction Set Manual](https://riscv.org/technical/specifications/) (Volume 1, Unprivileged Spec) and explore http://riscv.org for more information.
- Read sections 2.2 and 2.3 to understand how the different types of instructions are encoded. 
- Read sections 2.4, 2.5, 2.6, and 9.1 and think about how each of the instructions will use the ALU

### 1.2 Project phases
Your project will consist of two different phases: front-end and back-end. Within each phase, you will have multiple checkpoints that will ensure you are making consistent progress. These checkpoints will contribute (although not significantly) to your final grade. You are free to make design changes after they have been checked off.

In the first phase (front-end), you will design and implement a 3-stage RISC-V processor in Verilog, and run simulations to test for functionality. At this point, you will only have a functional description of your processor that is independent of technology (there are no standard cells yet). You are highly encouraged to finish each checkpoint early, and each checkpoint will be released before the due date of the ongoing one. Everything will take much longer than you expect, and finishing early gives you more time to improve your QoR (Quality of Results, e.g. clock period).

In the second phase (back-end), you will implement your front-end design in the ASAP7 7nm kit using the VLSI tools you used in lab. When you have finished phase 2, you will have a design that could move onto fabrication if this were a real technology process. You will have about 2 weeks to complete the second phase after its release.

### 1.3 Philosophy
This document is meant to describe a high-level specification for the project and its associated support hardware. You can also use it to help lay out a plan for completing the project. As with any design you will encounter in the professional world, we are merely providing a framework within which your project must fit.

You should consider the GSI(s) a source of direction and clarification, but it is up to you to produce a fully-functional design and its physical implementation. Ultimately the responsibility of designing and debugging your solution lies on you.

### 1.4 General Project Tips
Be sure to use top-down design methodologies in this project. We began by taking the problem of designing a basic computer system, modularizing it into distinct parts, and then refining those parts into manageable checkpoints. You should take this scheme one step further; we have given you each checkpoint, so break each into smaller, manageable pieces.

As with many engineering disciplines, digital design has a normal development cycle. In the norm, after modularizing your design, your strategy should roughly resemble the following steps:

- **Design** your modules well, make sure you understand what you want before you begin to code.

- **Code** exactly what you designed; do not try to add features without redesigning.

- **Simulate** thoroughly; writing a good testbench is as much a part of creating a module as actually coding it.
- **Debug** completely; anything which can go wrong with your implementation will.

Some general tips when designing complex RTL modules:

* Document your project thoroughly as you go
  * comment your Verilog
  * before making any RTL changes, **modify your pipeline diagram first to visualize this change**, doing this:
    * may reveal the change is actually infeasible
    * ensures that you and your partner have the same view of your processor's operation
* Split the module operation into data/control paths and design each separately
  * Start with the simplest possible implementation
  * Make changes incrementally and always test your module after each change, no matter how small
  * Finish the required features first before attempting any extra features
* Use github version control features like commits, branches, etc.
* Save your work often and rely on redundancy (e.g. copy files from `/scratch` to your home directory often to ensure they're backed up)
* Parallelize work as much as possible (e.g. start writing CPU RTL as you finish your diagram, work on CPU and Cache in parallel, start physical design as you finish your cache)


This project is divided into checkpoints. Each checkpoint will be due 1 to 2 weeks after its release, but the next checkpoint will be released early. Use this to your advantage- try to get ahead so that you have additional time to debug. Your TA will clarify the specific timeline for your semester.

The most important goal is to design a functional processor- this alone is 50-60% of the final grade, and you must have it **working completely** to receive any credit for performance.

---

## 2. Front-end design (Phase 1)

The first phase in this project is designed to guide the development of a three-stage pipelined RISC-V CPU that will be used as a base system for your back-end implementation.
Phase 1 will last for 5 weeks and has weekly checkpoints.

- Checkpoint 1: ALU design and pipeline diagram
- Checkpoint 2: Core implementation
- Checkpoint 3: Core + memory system implementation 
- Checkpoint 4: Physical Design


### 2.1 Adding SSH Key
First you must add an SSH key to your Github account, to allow you to push to your project repo from the instructional machines without entering your Github password each time. You may run these commands in any location on any instructional machine (the SSH key will be stored in your home directory and thus work on all machines).
```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
# hit Enter to each prompt (leave response blank)
cat ~/.ssh/id_ed25519.pub
# Then select and copy the contents of the id_ed25519.pub file
# displayed in the terminal to your clipboard
```

In your browser, navigate to [https://github.com/settings/ssh/new](https://github.com/settings/ssh/new) (log into your Github account if needed). You should see the `SSH Keys / Add New` page. Enter the following values:
* Title: `something descriptive (ex. eecs151)`
* Key: `paste the contents of the id_ed25519.pub file`

Then click the green `Add SSH key` button.

### 2.2 Project Git Repo
The skeleton files for the project will be delivered as a git repository provided by the staff. You should clone this repository as follows. It is highly recommended to familiarize yourself with git and use it to manage your development.


```shell
git clone /home/ff/eecs151/labs/project_skeleton /path/to/my/project
```

To get a team repo, fill out the google sheet via the link on Piazza with your team information. Please do this even if you are working alone, as these git repos will be used for version control and as part of the final checkoff. You will receive an email with an invite link to your project repo, which you should click to join before following the directions below. 

An example working flow to be able to pull from the skeleton as well as push/pull with your team repository is shown below:


```shell
cd /path/to/my/project
git remote add myOrigin git@github.com:EECS150/fa21_asic_teamXX
```

Then to pull changes from the skeleton, you would need to type:
```shell
git pull origin master
```

To pull changes from your team repository you would type:
```shell
git pull myOrigin master
```

And to push changes to your team repository (please do not attempt to push to the skeleton repository), you would usually want to pull first (above) and then type:
```shell
git push myOrigin master
```

---

## 3. Grading

### EECS 151:
|                   |           |
|-------------------|---------|
|  **70%**          |   Functionality at project due date: Your design will be subjected to a comprehensive test suite and    your score will reflect how many of the tests your implementation passes.
|  **25%**          |   Final Report and Final Interview: If your design is not 100% functional, this is your opportunity  explain your bugs and recoup points.
|  **5%**           |   Checkpoints: Each check-off is worth 1.25%. If you accomplished all of your checkpoints on time, you will receive full credit in this category.
|  **Bonus 5%**     |   Performance at project due date: You must have a fully working design to score points in this section. You will receive up to 5 bonus points as your performance improves relative to your peers. Performance will be calculated using the Iron Law: IPC * F

### EECS 251A:
|                 |           |
|-----------------|---------|
|   **60%**       |  Functionality at project due date: Your design will be subjected to a comprehensive test suite and your score will reflect how many of the tests your implementation passes.
|   **10%**       |  Set-Associative Cache: Implementation and performance of the configurable set-associative cache.
|   **25%**       |  Final Report and Final Interview: If your design is not 100% functional, this is your opportunity explain your bugs and recoup points.
|   **5%**        |  Checkpoints: Each check-off is worth 1.25%. If you accomplished all of your checkpoints on time,  you will receive full credit in this category.
|   **Bonus 5%**  |  Performance at project due date: You must have a fully working design to score points in this section. You will receive up to 5 bonus points as your performance improves relative to your peers. Performance will be calculated using the Iron Law: IPC * F

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
