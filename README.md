# Guide to Starting Research in Microarchitectural Security

This guide provides a structured approach to begin researching microarchitectural security.

## I. Build a Foundation (Essential Prerequisites)

Before diving into security, understand how computer architecture *works*.

1.  **Instruction Set Architecture (ISA):**
    *   Understand the basic concepts (x86, ARM, RISC-V).
    *   Know registers, instructions, memory addressing, and execution pipeline principles.
    *   Focus on how instructions interact with hardware.
2.  **Processor Pipeline:**
    *   Grasp stages (Fetch, Decode, Execute, Memory Access, Writeback).
    *   Understand instruction flow and *out-of-order execution* (critical for security).
3.  **Memory Hierarchy:**
    *   Familiarize yourself with the hierarchy (caches - L1, L2, L3, main memory, virtual memory).
    *   Understand cache lines, coherency protocols (e.g., MESI), and data movement.  Cache timing is crucial.
4.  **Operating System Concepts:**
    *   Understand process management, memory, and privileges.
    *   Know virtual memory, system calls, and kernel/user space distinction.

## II. Core Concepts in Microarchitectural Security

Focus on security-specific aspects.

1.  **Transient Execution Attacks:**  The *biggest* category.

    *   **Spectre:** Understand *all* variants (v1, v2, v1.1, v1.2, v4, etc.). Exploits speculative execution and branch prediction.  Leaks data across privilege boundaries.
    *   **Meltdown:** Exploits out-of-order execution to read kernel memory from user space.
    *   **Foreshadow (L1TF):** Targets L1 data cache, leaks from Intel SGX enclaves.
    *   **MDS Attacks (ZombieLoad, RIDL, Fallout):** Exploit microarchitectural data sampling (MDS) vulnerabilities.
    *   **Concept of "Transient Instructions":** Instructions executed speculatively, even if discarded, can leave traces (e.g., in cache).
    *   **Side Channels vs. Covert Channels:**  Side channels use unintended leakage (timing). Covert channels misuse communication mechanisms. Microarchitectural attacks are usually *side-channel* attacks.

2.  **Cache Attacks:**

    *   **Flush+Reload:** Classic cache timing attack. Monitor cache line access times.
    *   **Prime+Probe:** Fill cache (prime), measure access times to detect access (probe).
    *   **Evict+Time/Reload:** Similar to Flush+Reload, focusing on eviction.
    *   **Cache Coherency Attacks:** Exploit weaknesses in coherency protocols.

3.  **Rowhammer:**

    *   Rapidly accessing a DRAM row causes bit flips in adjacent rows.

4.  **Hardware Trojans:**

    *   Malicious modifications to hardware during design/manufacturing.

5.  **Other Areas:**

    *   **Power Analysis Attacks:** Exploit power consumption variations.
    *   **Electromagnetic (EM) Side Channels:** Use EM radiation.
    *   **Fault Injection Attacks:** Physically manipulate hardware.

## III. Tools and Techniques for Research

1.  **Simulators:**

    *   **gem5:** Powerful full-system simulator. *Essential*.
    *   **ChampSim:** Trace-based simulator for cache/branch predictor research.
    *   **MARSSx86:** Full-system simulator, x86 focused.
    *   **SimpleScalar:** Older, but useful for basics.

2.  **Performance Counters:**

    *   Use hardware performance counters to measure microarchitectural events.
    *   Tools like `perf` (Linux) are essential.
    *   Interpret data to identify vulnerabilities.

3.  **Programming Languages:**

    *   **C/C++:** Low-level programming and hardware interaction.
    *   **Assembly Language (x86, ARM, etc.):** Analyze disassembled code, write timing-sensitive code.
    *   **Python:** Scripting, data analysis, automation.

4.  **Reverse Engineering Tools:**

    *   **Disassemblers (IDA Pro, Ghidra):** Analyze compiled code.

## IV. Research Strategies

1.  **Read Papers:**

    *   **Top Security Conferences:**
        *   USENIX Security Symposium
        *   ACM CCS
        *   IEEE S&P (Oakland)
        *   NDSS
        *   ISCA
        *   MICRO
        *   HPCA
    *   **Google Scholar:** Keywords: "Spectre," "Meltdown," "cache attacks," etc.
    *   **Start with Survey Papers:** Overviews of specific areas.
    *   **Follow Citations:** Find related papers through references and "cited by."

2.  **Reproduce Existing Attacks:**

    *   *Crucial* for learning. Implement Spectre/Meltdown variants.

3.  **Identify Open Problems:**

    *   Limitations of attacks/defenses, unexplored features, new vectors.

4.  **Experimentation:**

    *   Simulators and counters to test hypotheses.
    *   Rigorous methodology.

5.  **Stay Updated:**

    *   Follow researchers, subscribe to mailing lists, check publications.

## V. Getting Started - A Concrete First Step

1.  **Focus on Spectre v1:** Fundamental transient execution attack.

2.  **Read the original Spectre paper:** [https://spectreattack.com/spectre.pdf](https://spectreattack.com/spectre.pdf)

3.  **Implement a simple proof-of-concept:** Adapt online examples (C).

4.  **Use `perf` to analyze:** Observe branch mispredictions and cache behavior.

5.  **Move to gem5:** Simulate the attack in gem5.
