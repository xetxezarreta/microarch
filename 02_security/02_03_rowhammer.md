# Rowhammer: Exploiting DRAM Vulnerabilities for Bit Flips

This document provides a detailed examination of Rowhammer, a critical hardware vulnerability affecting Dynamic Random Access Memory (DRAM) chips. Rowhammer allows attackers to induce bit flips (change a 0 to a 1, or a 1 to a 0) in memory locations they shouldn't be able to access directly, potentially leading to privilege escalation, data corruption, and system compromise. This is achieved by repeatedly accessing specific memory locations at high frequency, exploiting the physical characteristics of DRAM technology.

## 1. Introduction: DRAM Fundamentals and the Rowhammer Phenomenon

Dynamic Random Access Memory (DRAM) is the most common type of RAM used in computers, servers, and mobile devices. DRAM stores data as electrical charges in tiny capacitors, one capacitor per bit. These capacitors are arranged in a two-dimensional array of rows and columns.

**Key DRAM Concepts:**

*   **DRAM Cell:** The basic unit of DRAM, consisting of a capacitor and a transistor. The capacitor stores the bit value (charged or discharged), and the transistor controls access to the capacitor.
*   **Wordline (Row):** A horizontal line in the DRAM array that connects all the transistors in a row. Activating a wordline (applying a voltage) opens the transistors, allowing data to be read from or written to the capacitors in that row.
*   **Bitline (Column):** A vertical line in the DRAM array that connects to the capacitors in a column. The bitline carries the data signal to and from the sense amplifier.
*   **Sense Amplifier:** A circuit that detects the small voltage difference on the bitline when a row is accessed, determining whether the capacitor is charged (representing a 1) or discharged (representing a 0).
*   **Refresh:** Because capacitors leak charge over time, DRAM cells must be periodically refreshed to maintain the stored data. This involves reading the data from a row and then writing it back, recharging the capacitors.
*   **Row Buffer:** A small, fast memory buffer that holds the data from the most recently accessed row. Accessing the same row repeatedly is very fast because the data is already in the row buffer.

**The Rowhammer Phenomenon:**

Rowhammer is a *hardware reliability* issue that stems from the physical density of DRAM cells. When a particular row in DRAM (the "aggressor" row) is accessed repeatedly and rapidly (hundreds of thousands of times within a refresh interval), it can cause electrical interference in physically adjacent rows (the "victim" rows). This interference can lead to a gradual leakage of charge from the capacitors in the victim rows, eventually causing *bit flips*.

*   **Mechanism:**
    1.  **Repeated Activation:** The attacker repeatedly activates and deactivates a specific "aggressor" row in DRAM (this is often called "hammering" the row). This involves sending repeated `ACTIVATE` commands to the memory controller for the same row.
    2.  **Charge Leakage:** The rapid switching of the wordline voltage in the aggressor row creates electromagnetic interference that affects the physically adjacent rows (victim rows). This interference causes a small amount of charge to leak from the capacitors in the victim rows.
    3.  **Bit Flip:** If enough charge leaks from a capacitor in a victim row before the next refresh cycle, the sense amplifier may misinterpret the remaining charge, resulting in a bit flip (changing a 0 to a 1, or vice versa).

## 2. Rowhammer Attack Scenarios

Rowhammer is not just a theoretical vulnerability; it has been exploited in practical attacks with serious security implications.

*   **Privilege Escalation:** One of the most significant consequences of Rowhammer is the ability to gain elevated privileges. By carefully crafting memory access patterns, an attacker can induce bit flips in specific kernel data structures, such as page table entries (PTEs). For example:
    *   **Flipping Page Table Entry (PTE) Bits:** An attacker can target a PTE that controls the permissions of a memory page. By flipping a bit in the PTE, the attacker can change a page from read-only to read-write, or grant themselves access to a page they shouldn't have access to. This can allow them to modify kernel code or data, effectively gaining kernel-level privileges.

*   **Data Corruption:** Rowhammer can be used to corrupt arbitrary data in memory. While less targeted than privilege escalation, this can still lead to system instability, crashes, or denial-of-service.

*   **Cross-VM Attacks (in Virtualized Environments):** In a virtualized environment, Rowhammer can potentially be used to attack other virtual machines running on the same physical host. This is because VMs often share the same physical DRAM. An attacker in one VM could hammer rows that cause bit flips in the memory of another VM, compromising its security.

*   **JavaScript-Based Attacks:**  It has been demonstrated that Rowhammer attacks can be launched from JavaScript running in a web browser. This is particularly concerning because it allows for remote exploitation without requiring the user to download or execute any malicious software. The JavaScript code can use techniques like carefully crafted array accesses to trigger Rowhammer.

## 3. Rowhammer Mitigations

Mitigating Rowhammer is a complex challenge, as it is fundamentally a hardware vulnerability. Several mitigation strategies have been proposed and implemented, each with its own trade-offs:

*   **Increased Refresh Rates:**  The simplest mitigation is to increase the refresh rate of DRAM.  This reduces the time window during which charge leakage can occur, making it harder to induce bit flips. However, increasing the refresh rate consumes more power and reduces performance.

*   **Error-Correcting Code (ECC) Memory:** ECC memory can detect and correct single-bit errors (and sometimes double-bit errors).  While ECC can mitigate some Rowhammer-induced bit flips, it is not a complete solution.  Aggressive Rowhammer attacks can cause multi-bit errors that overwhelm ECC capabilities.  ECC is also more expensive and typically used only in servers and high-end workstations.

*   **Target Row Refresh (TRR) / Pseudo-Target Row Refresh (pTRR):** This is a hardware-based mitigation implemented by DRAM manufacturers.  TRR attempts to detect rows that are being hammered and proactively refresh the adjacent rows.  This is done by monitoring row activation counts.  However, attackers have developed techniques to bypass TRR, leading to the development of pTRR and other more sophisticated variants.

*   **Probabilistic Adjacent Row Activation (PARA):** A hardware mitigation where, with a small probability, activating a row also refreshes its adjacent rows.

*   **Blockhammer:** A defense mechanism using performance counters, with significant overhead.

*   **Memory Isolation:**  In virtualized environments, techniques like memory ballooning (reclaiming unused memory from VMs) and memory deduplication (sharing identical memory pages between VMs) can increase the risk of Rowhammer attacks.  Stronger memory isolation mechanisms can help mitigate cross-VM attacks.

*   **Software-Based Detection:**  Some software-based detection mechanisms attempt to identify Rowhammer access patterns.  However, these are generally less effective than hardware-based mitigations and can introduce significant performance overhead.

*   **DRAM Manufacturing Improvements:**  DRAM manufacturers are continuously working to improve the resilience of DRAM cells to Rowhammer. This includes using different materials, improving the physical layout of the cells, and enhancing the sense amplifiers.  However, completely eliminating the Rowhammer vulnerability through manufacturing changes is difficult.
