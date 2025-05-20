---
layout:     post
title:      "Tomasulo's algorithm with speculation stages"
date:       2025-05-20 09:00:00 +0200
categories: computer-architecture
background: '/assets/figures/2025-05-20-tomasulo-with-speculation-11.png'
---

Continuing with the articles on computer architecture today we are going to delve into the out-of-order instructions execution and more specifically in Tomasulo's algorithm with speculation stages. If the concepts here seem very complex to you, I recommend you to read my [article](https://www.linkedin.com/pulse/cooking-up-computer-architecture-kitchen-rush-analogy-iker-pedrosa-ejvlf) in which I make an analogy between a modern computer architecture and the Kitchen Rush game.

Before proceeding, let us review three key concepts:

* **Dynamic scheduling** is a technique used in computer architecture to improve processor performance by allowing instructions to be executed out-of-order.
* **Tomasulo's algorithm** is a specific dynamic scheduling technique used in modern processors.
* **Speculation** is a technique used in computer architecture to improve performance optimization by making a prediction about the outcome of a branch operation and executes instructions based on that prediction before the actual outcome is known.

All of this together helps to maximize the utilization of functional units and improve performance. They allow the CPU to execute instructions out-of-order. This is a fundamental aspect of pipelining, which significantly improves throughput.

# Additional concepts

* The **instruction queue** acts as a buffer that holds instructions fetched from memory before they are decoded and executed. By prefetching and queuing instructions, the processor can overlap the fetching of subsequent instructions with the execution of current ones.
* **Functional units** are dedicated hardware components designed to execute particular types of instructions. Some examples: Arithmetic Logic Unit (ALU), Floating-Point Unit (FPU) like FP adders and FP multipliers, and Memory Unit.
* **Reservation stations** are used to buffer instructions, their operands, and information about which functional units will produce the needed data. They play a key role in handling data dependencies and resolving hazards that can occur during instruction execution.
* The **Reorder Buffer (ROB)** is a buffer (often circular) that holds information about instructions that have been issued but not yet committed. It tracks the state of each instruction, including whether it has completed execution and whether it has encountered any exceptions. Instructions are entered into the ROB in the order they are issued and they are commited in the same order. A ROB entry stores various attributes, notably a tag that uniquely identifies an instruction within the pipeline, and a value field that holds the instruction's computed result before it's committed.
* The **Common Data Bus (CDB)** is essentially a broadcast bus that allows functional units to transmit their results to all reservation stations simultaneously.
* **Load buffers** often used to hold load operations that have had their addresses calculated, but are waiting to have the data returned from the memory system, or are waiting for memory ordering issues to be resolved.
* **Control entries** refer to the data structures that track the status and allocation of resources within the reservation stations and the ROB.

# The stages of Tomasulo's algorithm with speculation

We've laid the groundwork, exploring the fundamental principles of dynamic scheduling and out-of-order execution. Now, let's dive into the heart of sophisticated processor design: Tomasulo's algorithm, enhanced with speculation. Get ready to dissect the intricate stages of speculative execution and see how they dramatically improve performance.

## Issue

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-1.png){: .img-fluid}
</div>

1. Get an instruction from the instruction unit.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-2.png){: .img-fluid}
</div>

2. If there is an empty reservation station and an empty slot in the ROB, then issue the instruction. If either all reservations are full or the ROB is full, then the instruction issue is stalled until both have available entries.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-3.png){: .img-fluid}
</div>

3. If the operands are available in either the registers or the ROB (from a previous instruction that hasn't yet commited), send them to the reservation station.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-4.png){: .img-fluid}
</div>

4. Update the control entries in the reservations stations and the ROB to indicate the buffers are in use.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-5.png){: .img-fluid}
</div>

5. The number of the ROB entry allocated for the result is also sent to the reservation station, so that the number can be used to tag the result when it is placed on the CDB.

## Execute

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-6.png){: .img-fluid}
</div>

6. If one or more of the operands is not yet available, monitor the CDB while waiting for the register to be computed. This step is the one that checks for RAW hazards.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-7.png){: .img-fluid}
</div>

7. When both operands are available at a reservation station, execute the operation. Instructions may take multiple clock cycles in this stage.

## Write result

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-8.png){: .img-fluid}
</div>

8. When the result is available, write the ROB tag and the result on the CDB, and from the CDB into the ROB and to any reservation stations waiting for this result.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-9.png){: .img-fluid}
</div>

9. Mark the reservation station as available.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-10.png){: .img-fluid}
</div>

10. Special actions are required for store instructions. If the value to be stored is available, it is written into the `Value` field of the ROB entry for the store. If the value to be stored is not available yet, the CDB must be monitored until that value is broadcast, at which time the `Value` field of the ROB entry of the store is updated.

## Commit

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-05-20-tomasulo-with-speculation-11.png){: .img-fluid}
</div>

11. There are three different sequences of actions:
    1. Normal commit: the instruction reaches the head of the ROB and its result is present in the buffer, the processor updates the register with the result and removes the instruction from the ROB.
    2. Store commit: instead of updating the result register the memory is updated.
    3. Branch with incorrect prediction: wrong speculation. The ROB is flushed and execution is restarted at the correct successor of the branch.

# Conclusion

It's clear that Tomasulo's algorithm, especially with the addition of speculation, presents a significant level of complexity. However, as we've demonstrated, breaking down the process into manageable stages and utilizing diagrams can greatly simplify the understanding of its operation. This exploration underscores the remarkable engineering that powers our modern computing devices.
