# Computer Architecture - 4/30/26

## 1. Five Stages of the RISC Pipeline

Understanding the sequential flow of instructions is critical for mastering processor architecture. The standard RISC pipeline breaks instruction execution into five distinct stages to increase throughput.

- **Instruction Fetch (IF)**: The **CPU** retrieves the next instruction from memory using the address stored in the **Program Counter (PC)**. The PC is usually incremented by 4 immediately after.
- **Instruction Decode (ID)**: The **Control Unit** analyzes the instruction bits to determine the operation (e.g., ADD, LOAD). It reads source operands from the **Register File** and performs sign-extension for immediate values.
- **Execute (EX)**: The **Arithmetic Logic Unit (ALU)** performs the core operation. This includes:
    - Arithmetic/Logical operations (e.g., add, or).
    - Calculating the Effective Address for memory access.
    - Comparing operands for branch instructions.
- **Memory Access (MEM)**: This stage is used primarily by memory-reference instructions:
    - **Read (Load)**: Data is fetched from **RAM** into the CPU.
    - **Write (Store)**: A value from a **Register** is written to **Data Memory**.
- **Write Back (WB)**: The final result, whether from the **ALU** or **Memory**, is written back to the destination **Register** in the **Register File**.

> **Note:** The **ID** and **MEM** stages are often considered "busy" stages because they involve complex hardware interactions (control logic and memory bus synchronization).

---

## 2. Multiplexers (MUX) in CPU Design

A **Multiplexer (MUX)** is a combinatorial logic device that selects one of several input signals based on a control signal provided by the **Control Unit**.

- **Purpose**: MUXes allow the **Datapath** to be reconfigured dynamically for different instruction types.
- **Critical MUX Locations**:
    - **ALU Source MUX**: Selects between the second register operand (for R-type instructions) and a sign-extended immediate value (for I-type instructions like addi or lw).
    - **Next PC MUX**: Selects between the sequential PC (PC+4) and the branch target address or jump target.
    - **RegWrite Source MUX**: Selects whether the data written back to the register file comes from the ALU or from Memory (Load instructions).

---

## 3. Clocking and Timing Signals

Processor operations are synchronized by a **Clock Signal**, ensuring that data transitions occur in a predictable manner.

- **Edge-Triggered Logic**: Modern CPUs typically use **edge-triggered** updates.
    - **Rising Edge**: Often used to update state elements like the **Program Counter** and **Registers**.
    - **Falling Edge**: Sometimes used for specific internal transitions, though most modern designs are single-edge.
- **Setup and Hold Times**: Hardware must ensure that data is stable before the clock edge (setup) and remains stable briefly after (hold) to prevent metastability.

---

## 4. CPU Datapath and Functional Units

The **Datapath** consists of the functional units and the buses that carry data between them.

### 4.1. Core Components
- **Registers**: A small, high-speed storage array (Register File). It is significantly faster than **RAM** but has very limited capacity.
- **Control Unit**: The "brain" of the CPU. It decodes the opcode and generates control signals (e.g., `RegWrite`, `ALUSrc`, `MemRead`) to coordinate the hardware.
- **Arithmetic Logic Unit (ALU)**: The functional unit that executes arithmetic (addition, subtraction) and logic (AND, OR, NOT) operations.

> **Teacher's Probing Question:** Why is the separation between the **Control Unit** and the **ALU** fundamental to the Von Neumann architecture?

---

## 5. Multi-cycle CPU Design

Unlike single-cycle designs where every instruction takes the same amount of time as the slowest instruction (usually lw), **Multi-cycle Design** breaks instructions into smaller steps.

- **Key Benefits**:
    - **Variable Execution Time**: Faster instructions (like branches) can finish in 3 cycles, while slower ones (like loads) take 5.
    - **Resource Reuse**: A single ALU can be used for both PC incrementing and actual computation in different cycles, reducing the total transistor count.
- **Control Implementation**: Usually implemented using a **Finite State Machine (FSM)** or microcode.

---

## 6. Exception and Interrupt Handling

**Exceptions** and **Interrupts** are events that disrupt the normal flow of instruction execution.

- **Definitions**:
    - **Exception**: Internal event (e.g., arithmetic overflow, undefined instruction, system call).
    - **Interrupt**: External event (e.g., I/O device request, timer expiration).
- **Handling Procedure**:
    1. Save the address of the offending/interrupted instruction in the **EPC** (Exception Program Counter).
    2. Save the cause of the exception.
    3. Jump to a predefined address (the **Exception Handler** in the OS).
    4. Restore state and return to the original program using eret.

---

## 7. Pipelining Hazards and Solutions

Pipelining increases throughput but introduces **Hazards**—situations where the next instruction cannot execute in the next clock cycle.

### 7.1. Types of Hazards and Primary Solutions

| Hazard Type | Description | Solution |
| :--- | :--- | :--- |
| **Structural** | Two instructions need the same hardware resource. | Duplicate hardware (e.g., separate I-Cache and D-Cache). |
| **Data** | An instruction depends on a previous instruction's result. | **Forwarding (Bypassing)**, Stalling (Bubbles), or Compiler Scheduling. |
| **Control** | The next instruction is unknown due to a branch/jump. | **Branch Prediction**, Stalling, or Delayed Branching. |

---

## 8. Data Hazards: Forwarding and Stalls

### 8.1. Data Forwarding (Bypassing)
Forwarding routes the result directly from the pipeline registers (EX/MEM or MEM/WB) back to the ALU inputs, skipping the Write Back stage.
- **Does it eliminate all stalls?** **No.**
- **The Load-Use Hazard**: If an instruction immediately follows a load and uses its result, a **1-cycle stall** is mandatory. This is because the loaded data is only available at the end of the **MEM** stage, but the next instruction needs it at the start of its **EX** stage.

### 8.2. Stalling (Inserting Bubbles)
If forwarding cannot resolve the hazard, the hardware inserts a "bubble" (a NOP), holding the IF and ID stages constant while allowing the later stages to progress.

---

## 9. Branch Prediction Mechanisms

Control hazards are the most significant bottleneck in deep pipelines. **Branch Prediction** is the only practical solution because stalling is too slow and delayed branching (like in MIPS) doesn't scale to modern pipeline depths.

### 9.1. Two-Bit Saturated Counter
To prevent a single outlier (like the exit of a loop) from flipping the prediction, we use a 2-bit state machine. It requires **two consecutive mispredictions** to change the global prediction.

### 2-Bit Saturating Counter State Table

| Current State | Binary | Prediction | If Actual: **Taken (T)** | If Actual: **Not Taken (NT)** |
| :--- | :--- | :--- | :--- | :--- |
| **Strongly Not Taken** | `00` | Not Taken | Move to `01` (Weakly NT) | Stay at `00` (Strongly NT) |
| **Weakly Not Taken** | `01` | Not Taken | Move to `10` (Weakly T) | Move to `00` (Strongly NT) |
| **Weakly Taken** | `10` | Taken | Move to `11` (Strongly T) | Move to `01` (Weakly NT) |
| **Strongly Taken** | `11` | Taken | Stay at `11` (Strongly T) | Move to `10` (Weakly T) |

### 9.2. BPB vs. BTB

| Feature | Branch Prediction Buffer (BPB) | Branch Target Buffer (BTB) |
| :--- | :--- | :--- |
| **Goal** | Predict Direction (Taken/Not Taken) | Predict Target Address |
| **Lookup** | Indexed by lower bits of PC (no tag). | Search by full PC (tag match). |
| **Stage** | Usually **ID** (Decode) stage. | **IF** (Fetch) stage. |
| **Benefit** | Low hardware cost, high accuracy. | Allows zero-delay fetching of target. |

---

## 10. Correlating Branch Prediction

Basic predictors only look at the history of the current branch. **Correlating Predictors** (also known as (m,n) predictors) use the behavior of *other* recent branches to improve accuracy.

- **Mechanism**: A **Global Branch History Register (GBHR)** shifts in a '1' for taken and '0' for not taken for every branch encountered.
- **Implementation**: The GBHR is combined with the PC bits to index into a large table of 2-bit counters.
- **Why it works**: Many branches are logically dependent (e.g., if `(a > 0)` is true, then `(a > -1)` is also true).

---

## 11. Superscalar CPU Architecture

A **Superscalar** CPU can execute more than one instruction per clock cycle (**IPC > 1**).

- **Key Features**:
    - **Instruction-Level Parallelism (ILP)**: Exploiting independent instructions to run them simultaneously.
    - **Multiple Issue**: Fetching and dispatching multiple instructions at once.
    - **Multiple Execution Units**: Having multiple ALUs, FPUs, and Load/Store units.
    - **Out-of-Order Execution**: Instructions may finish out of order, but they are always **committed** in order to maintain logical correctness.

---

## 12. Pipelining with Stalling-Only Solutions

In scenarios where **Stalling** is the only solution (no forwarding, no prediction), the performance penalty is severe:

- **Data Hazard Penalty**: Usually requires a **2-cycle stall** to allow the first instruction to reach the WB stage and write to the register file before the second instruction reads it in the ID stage.
- **Control Hazard Penalty**: Usually requires a **3-cycle stall** until the branch outcome and target are resolved in the MEM stage.

> **Teacher's Probing Question:** If a program has 20% branches and every branch causes a 3-cycle stall, what is the Maximum Speedup compared to a non-pipelined processor (assuming a 5-stage pipeline)?
