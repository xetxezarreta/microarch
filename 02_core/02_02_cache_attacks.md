# Cache Attacks: Exploiting Microarchitectural Timing for Information Leakage

This document provides a detailed examination of cache attacks, a significant class of side-channel vulnerabilities that exploit the timing characteristics of CPU caches to extract sensitive information. These attacks leverage the fact that memory access times are significantly faster when data is present in the cache (a "cache hit") compared to when it needs to be fetched from main memory (a "cache miss"). By carefully observing these timing differences, an attacker can infer information about a victim's memory access patterns, potentially revealing secret keys, passwords, or other confidential data.

## 1. Introduction: CPU Caches and Their Security Implications

Modern CPUs rely heavily on caches to bridge the performance gap between the processor and main memory (RAM).  Caches are small, fast memory units located close to the processor core.  They store copies of frequently accessed data from main memory, allowing the CPU to retrieve that data much more quickly than if it had to access RAM directly.  This significantly speeds up program execution.

**Why are caches a security problem?** Although caches are essential for performance, their *implementation details* create *microarchitectural side channels* that can be exploited by attackers.  The fundamental principle behind cache attacks is that an attacker can observe the *timing* of memory accesses to determine whether a particular memory location (a "cache line") is present in the cache or not. This seemingly innocuous information can be used to infer what data the victim process (or the operating system kernel) has accessed.

**Cache Terminology:**

*   **Cache Line:** The smallest unit of data that can be transferred between the cache and main memory.  Typical cache line sizes are 64 bytes or 128 bytes.
*   **Cache Set:** A group of cache lines within the cache.  Caches are typically organized as "set-associative," meaning that a given memory address can only be mapped to a specific set of cache lines.
*   **Associativity:**  Determines how many cache lines are in each set.  A fully associative cache can store any memory location in any cache line.  A direct-mapped cache has only one cache line per set.  A set-associative cache (the most common type) has a fixed number of cache lines per set (e.g., 8-way set-associative).
*   **Cache Hit:**  The requested data is found in the cache.  This is fast.
*   **Cache Miss:** The requested data is *not* found in the cache, requiring a slower access to main memory.
*   **Eviction:**  When a cache is full and a new cache line needs to be brought in, an existing cache line must be evicted (removed) to make room.  The eviction policy (e.g., Least Recently Used - LRU) determines which cache line is evicted.
* **Inclusion Property:** The relationship between different levels of caches, it could be:
    *   **Inclusive:** If the data exists in L_i, then it must also be present in L_{i+1} cache.
    *   **Exclusive:** If data exists in L_i, it can't be present in any other cache level.
    *   **Non-inclusive:** There are no restrictions on where data is stored in the cache hierarchy.
*   **Cache Coherence Protocol:** A protocol that ensures consistency of data across multiple caches (e.g., in a multi-core processor) or between caches and main memory. Examples include MESI, MOESI.

## 2. Common Cache Attack Techniques

Cache attacks typically involve two main phases:

1.  **Manipulating the Cache State:** The attacker modifies the cache state in a controlled way, often by forcing specific cache lines to be evicted or loaded.
2.  **Observing Timing Differences:** The attacker measures the time it takes to access memory, inferring information about the cache state (and, indirectly, about the victim's memory accesses).

Here are some of the most common cache attack techniques:

*   **Flush+Reload:**
    *   **Mechanism:** This is a classic and highly precise cache timing attack.
        1.  **Flush:** The attacker flushes a specific cache line from the cache.  This can be done using a dedicated instruction like `clflush` (on x86) or by accessing other memory locations that map to the same cache set, forcing the target line to be evicted (if the cache is not fully associative). The key requirement is that the attacker and victim *share* the memory being monitored (e.g., a shared library).
        2.  **Wait:** The attacker waits for a period of time, during which the victim may or may not access the monitored cache line.
        3.  **Reload:** The attacker reloads the cache line (by accessing the corresponding memory address) and measures the time it takes.
        4.  **Analysis:**
            *   **Fast Reload (Cache Hit):**  If the reload is fast, it indicates that the victim accessed the cache line during the "wait" period, bringing it back into the cache.
            *   **Slow Reload (Cache Miss):** If the reload is slow, it indicates that the victim did *not* access the cache line, and it remained evicted.
    *   **Advantages:** Very precise; can detect individual memory accesses.
    *   **Disadvantages:** Requires a mechanism to flush the cache line (e.g., `clflush` instruction, or shared memory with the victim to induce eviction).

*   **Prime+Probe:**
    *   **Mechanism:** This attack does *not* require explicit flushing of cache lines.
        1.  **Prime:** The attacker fills the entire cache (or a specific cache set) with their own data. This evicts any data that was previously in the cache, including any data belonging to the victim.
        2.  **Wait:** The attacker waits for a period of time, during which the victim may execute and access memory.
        3.  **Probe:** The attacker accesses their own data (the data they used to "prime" the cache) and measures the access time for each cache line.
        4.  **Analysis:**
            *   **Fast Access (Cache Hit):**  If the access to a particular cache line is fast, it indicates that the attacker's data is still in the cache, and the victim did *not* access a memory location that maps to that cache set.
            *   **Slow Access (Cache Miss):** If the access to a particular cache line is slow, it indicates that the attacker's data was *evicted* from the cache. This means the victim *did* access a memory location that maps to the same cache set, causing the eviction.
    *   **Advantages:** Does not require `clflush` or shared memory; can be used to attack across different processes or security domains.
    *   **Disadvantages:** Less precise than Flush+Reload; can be affected by noise from other processes.

*   **Evict+Time/Reload:**
    *   **Mechanism:**  Very similar to Flush+Reload, but instead of using a dedicated flush instruction, it relies on *eviction*.
        1. **Evict:** The attacker accesses a set of memory locations that map to the *same* cache set as the target cache line, ensuring that the target line is evicted. This requires knowledge of the cache geometry (cache size, associativity, mapping function).
        2. **Time/Reload:** The attacker either measures the time to execute a code snippet that *may or may not* contain an access to the target line (Evict+Time) or explicitly reloads the target address and times that (Evict+Reload).
        3. **Analysis:** If the access time is slow, the victim did not use the cache line; fast access suggests the victim loaded it.
    * **Advantages:** It does not need a flush instruction.
    * **Disadvantages:** It requires knowledge about cache organization.

*   **Cache Coherency Attacks:**
    *   **Mechanism:** These attacks exploit weaknesses or implementation flaws in the cache coherence protocols used in multi-core processors.  Cache coherence protocols (like MESI or MOESI) ensure that all cores have a consistent view of shared memory.  However, subtle timing variations or race conditions in these protocols can be exploited to leak information.
        * **Example:** An attacker might observe the timing of coherence messages (e.g., invalidations, write-backs) to infer information about another core's memory access patterns.
    *   **Advantages:** Can be very powerful, potentially allowing attacks across cores.
    *   **Disadvantages:** Highly complex; require deep understanding of the specific processor's cache coherence protocol.  Often require privileged access (e.g., to modify page table entries or manipulate cache coherence settings).

## 3. Mitigations

Mitigating cache attacks is a challenging problem, as it requires balancing security and performance. Several approaches exist, each with trade-offs:

*   **Constant-Time Programming:**  Write code that executes in a time that is *independent* of secret data.  This is the most robust defense, but it can be very difficult to achieve in practice, especially for complex algorithms.  This involves:
    *   Avoiding data-dependent branches (e.g., `if` statements that depend on secret values).
    *   Avoiding data-dependent memory accesses (e.g., array indices that depend on secret values).
    *   Using constant-time cryptographic implementations.

*   **Cache Partitioning/Isolation:** Divide the cache into separate partitions, assigning different partitions to different security domains (e.g., different processes, user vs. kernel). This prevents attackers from evicting or observing cache lines belonging to other domains.  This can be done in hardware or software.

*   **Randomization:** Introduce randomness into the cache mapping function or eviction policy. This makes it more difficult for the attacker to predict which memory locations map to which cache sets, hindering attacks like Prime+Probe and Evict+Reload.

*   **Hardware Countermeasures:**  Processor manufacturers are implementing hardware changes to mitigate cache attacks.  These include:
    *   **Cache Flushing Instructions with Finer Granularity:**  Allowing more precise control over cache flushing.
    *   **Reduced Timing Precision:** Making it harder to measure subtle timing differences.
    *   **Improved Cache Coherence Protocols:**  Addressing vulnerabilities in coherence protocols.

*   **Detection:** Monitor system performance for patterns that are indicative of cache attacks. This is a reactive approach, rather than a preventative one.

* **Disabling Caching (Extreme):** In highly sensitive environments, caching might be disabled entirely for critical operations, but this comes at a severe performance penalty.
