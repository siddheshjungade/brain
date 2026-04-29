# Operating System Roadmap (3+ Years Experience)

This roadmap is designed for experienced Software Engineers aiming for Senior or Mid-level roles at Top-Tier Tech companies. At this level, the focus is on **architectural trade-offs, system performance, and distributed environments.**

---

## 🛠 Phase 1: Core Mechanics (The Foundations)
Understanding the bridge between hardware and software.
* **The Kernel**: Understand the difference between Monolithic (Linux) and Microkernels (macOS/QNX).
* **User Mode vs. Kernel Mode:** The "Ring" protection model. Why syscalls are expensive and how the CPU switches privileges.
* **System Calls ($syscall$):** The lifecycle of a request (e.g., `read()`, `write()`) from the application to the Kernel.
* **Interrupt Handling:** Hardware vs. Software interrupts. How the CPU pauses execution for I/O events.
* **Monolithic vs. Microkernels:** Understanding why Linux (Monolithic) is preferred for performance vs. Microkernels for security.

---

## 🚀 Phase 2: Process & Concurrency (The Execution Engine)
Focus on how modern OSs manage thousands of concurrent tasks.

* **Process vs. Thread:** Memory layouts, shared heap vs. private stacks, and the cost of creation.
* **The Scheduler:** Beyond Round Robin. Study the **Completely Fair Scheduler (CFS)** used in Linux.
* **Context Switching:** The cost of saving/loading CPU states and **TLB Flushes** (Cache pollution).
* **Inter-Process Communication (IPC):** Sockets, Pipes, Shared Memory (fastest), and Message Queues.

---

## 🔒 Phase 3: The Deadlock & Synchronization Vault
A high-priority topic for interviewers to test your reliability logic.

* **The 4 Necessary Conditions (Coffman Conditions):**
    1. **Mutual Exclusion:** Single access to a resource.
    2. **Hold & Wait:** Keeping a resource while requesting another.
    3. **No Preemption:** Resources cannot be taken away.
    4. **Circular Wait:** A closed dependency loop ($P1 \rightarrow P2 \rightarrow P1$).
* **Strategies:** * **Prevention:** Breaking one of the 4 conditions.
    * **Avoidance:** **Banker’s Algorithm** and "Safe States."
    * **Detection & Recovery:** Identifying cycles and killing/preempting processes.
* **Primitives:** Mutex vs. Semaphores vs. **Spinlocks** (when to spin vs. when to sleep).
* **Livelock & Starvation:** Solving starvation using **Aging**.

---

## 🧠 Phase 4: Memory Architecture (The Performance Pillar)
How the OS manages the most constrained resource.

* **Virtual Memory:** Abstracting physical RAM into a large, contiguous address space.
* **Paging & Segmentation:** Mapping virtual addresses to physical pages.
* **Page Faults & Thrashing:** What happens when the "Working Set" exceeds RAM and the disk starts swapping.
* **TLB (Translation Lookaside Buffer):** The hardware cache for page table entries.

---

## 📁 Phase 5: Storage & File Systems
* **Inodes & File Descriptors:** How Linux tracks files. "Everything is a file" philosophy.
* **Disk Scheduling:** Minimizing "Seek Time" through efficient I/O ordering.
* **RAID:** Understanding RAID 0, 1, 5, and 10 for high-availability systems.

---

## ☁️ Phase 6: Senior Systems & Cloud (The FANG Edge)
Applying OS concepts to modern infrastructure.

* **Containerization Internals:** * **Namespaces:** Process/Network isolation.
    * **Cgroups:** Resource limiting (CPU/RAM).
* **Distributed Deadlocks:** Detecting cycles across network-separated microservices or databases.
* **Observability:** Using tools like `strace`, `htop`, `lsof`, and `pstack` to debug system-level issues.

---

## 📅 4-Week Sprint Plan

| Week | Focus Area | Key Goal |
| :--- | :--- | :--- |
| **Week 1** | Processes & Scheduling | Implement a Thread Pool; explain Context Switching. |
| **Week 2** | Memory & I/O | Deep dive into Paging and Cache mapping. |
| **Week 3** | Concurrency & Deadlock | Solve "Dining Philosophers" and "Producer-Consumer". |
| **Week 4** | Systems & Containers | Link OS to Docker, K8s, and Cloud scalability. |

---

## 📚 Top Resources
1. **OSTEP (Operating Systems: Three Easy Pieces):** Best for conceptual depth.
2. **Linux Kernel Docs:** Best for understanding `cgroups` and `namespaces`.
3. **LeetCode (Concurrency Section):** Best for practicing synchronization logic.