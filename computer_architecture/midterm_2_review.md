# Midterm 2 Review - 4/30/2026

## 1. Five Stages of the RISC Pipeline

Understanding the sequential flow of instructions is critical for mastering processor architecture.

- **Instruction Fetch (IF)**: The **CPU** retrieves the next instruction from memory using the address stored in the **Program Counter (PC)**.
- **Instruction Decode (ID)**: The **Control Unit** analyzes the instruction bits to determine the operation (e.g., ADD, LOAD). It also reads source operands from the **Register File**.
- **Execute (EX)**: The **Arithmetic Logic Unit (ALU)** performs the core operation, such as mathematical calculations, logical comparisons, or memory address computation.
- **Memory Access (MEM)**: This stage is used primarily by memory-reference instructions:
    - **Read (Load)**: Data is fetched from **RAM** into the CPU.
    - **Write (Store)**: A value from a **Register** is written to **Data Memory**.
- **Write Back (WB)**: The final result, whether from the **ALU** or **Memory**, is written back to the destination **Register** for future use.

> **Note:** The **ID** and **MEM** stages are often considered "busy" stages because they involve complex hardware interactions (control logic and memory bus synchronization).

## 2. Multiplexers (MUX) in CPU Design

A **Multiplexer** (MUX) is a combinatorial logic device that selects one of several input signals based on a control signal.

- **Purpose**: MUXes allow the **Control Unit** to route data through different paths depending on the instruction.
- **Common Uses**:
    - Selecting between immediate values and register values for the **ALU**.
    - Selecting the source for the next **Program Counter** (e.g., PC+4 vs. Branch Target).

## 3. Clocking and Timing Signals

Processor operations are synchronized by a **Clock Signal**. 

- **Edge-Triggered Logic**: Modern CPUs typically read and write data at different clock edges (e.g., rising or falling) to ensure stability and prevent race conditions within a single clock cycle.

## 4. CPU Datapath and Functional Units

The **Datapath** is the collection of hardware elements that perform data processing operations.

### 4.1. Core Components
- **Registers**: High-speed storage within the CPU that holds data currently being processed. It is significantly faster than **RAM** but limited in capacity.
- **Control Unit**: The "brain" of the CPU. it decodes instructions and generates control signals to coordinate the **Registers**, **ALU**, and **Memory**. It does not perform calculations itself.
- **Arithmetic Logic Unit (ALU)**: The functional unit that executes all arithmetic (addition, subtraction) and logic (AND, OR, NOT) operations.

> **Teacher's Probing Question:** Why is the separation between the **Control Unit** and the **ALU** fundamental to the Von Neumann architecture?

## 5. Multi-cycle CPU Design

Unlike single-cycle designs where every instruction takes the same amount of time, **Multi-cycle Design** breaks instructions into smaller steps.

- **Benefits**:
    - **Efficiency**: Faster instructions (like branches) can finish in fewer cycles than slower ones (like loads).
    - **Resource Reuse**: Hardware components (like the ALU) can be reused in different cycles of the same instruction, reducing total hardware cost.

## 6. Exception and Interrupt Handling

**Exceptions** and **Interrupts** are events that disrupt the normal flow of instruction execution.

- **Definitions**:
    - **Exception**: Internal event (e.g., arithmetic overflow, undefined instruction).
    - **Interrupt**: External event (e.g., I/O device request).
- **Role of the Control Unit**: The **Control Unit** is responsible for detecting these events, saving the current state (PC), and branching to a specific **Exception Handler** routine in the OS.

---

# Pipelining

## Know the solutions to all types of hazards

- lecture slides 46 has a summary of these solutions
- you also need to know whether a solution can properly solve the hazards or not.
In particular,
    - Why only branch predictions is the only practical solution to control hazard? Why other solutions do not work well?
    - Does data bypassing/forwarding climinate stalls? And why?

## The definition of superscalar CPU

- lecture 48

## At least oen problem about pipelining 

- similar to those in assignment 3
- In particular, you need to know how the pipeline works when stalling is the only solution to the hazards

---

# Branch Preditions

## The basic two-bit saturated coutner for branch preditions

- You need to memorize the state machine

## Know the implementations of Brach Prediction Buffer and Branch Target Buffer

- What components do these two buffers have?
- How does a branch locate its entries in these two buffers?

## Correlating branch prediction

- Why does correcting branch predition work for some branches?
- Why does correclating branch prediction not work from some branches?
- The implementation of correlating brach preiction with Global Branch History Register (GBHR) and two-bit saturate counters.

## There will be one problem about branch preditions, similar to the last question of Assignment 3


