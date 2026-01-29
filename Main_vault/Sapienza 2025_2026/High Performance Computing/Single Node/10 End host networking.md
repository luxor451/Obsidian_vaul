**Tags:** #HPC #Networking #LinuxKernel #KernelBypass #DPDK #SmartNIC #eBPF #PCIe #Skbuff

---

## 1. Context: The End-Host

In the topology of a datacenter, the **End-Hosts** (servers) are the leaves of the tree where applications run and traffic is generated or received.



### Why focus on the End-Host?
Historically, networking innovation focused on routers and switches ($R_1 \dots R_k$). However, modifying the network fabric is difficult:
* **Operational Risk:** Changing a core switch affects thousands of users.
* **Cost:** New hardware is expensive and requires recabling.
* **Heterogeneity:** Different applications have different needs; deploying specialized hardware in the core for one specific app is inefficient.

**The End-Host Advantage:**
* **Agility:** It is easier to deploy "fancy new functionality" at the edge (the server).
* **Proximity:** You are closer to the application logic.
* **Testing:** It allows for isolated testing of new network stacks without disrupting the whole datacenter.

---

## 2. The Standard Linux Networking Stack

To understand optimization, we must first understand the default path of a packet, often referred to as the "Life of a Packet" in the Linux Kernel.



### The Journey (Rx Path)
1.  **Hardware (NIC):** The Network Interface Card receives the packet from the wire.
2.  **DMA (Direct Memory Access):** The NIC writes the packet data into a ring buffer in RAM (Rx Ring) via PCIe.
3.  **Interrupt:** The NIC raises a hardware interrupt to tell the CPU "I have data."
4.  **Driver & NAPI:**
    * The CPU pauses, acknowledges the interrupt.
    * To avoid interrupt storms (too many interrupts freezing the CPU), Linux uses **NAPI (New API)**. It switches to **Polling** mode, processing a batch of packets (quota) before re-enabling interrupts.
5.  **Memory Allocation (`sk_buff`):** The driver allocates a massive metadata structure called `sk_buff` to wrap the packet.
6.  **GRO (Generic Receive Offload):** The kernel tries to merge small TCP segments into one large segment to reduce processing overhead.
7.  **TC (Traffic Control) & Netfilter:** The packet passes through QoS queues and Firewall rules (`iptables`).
8.  **IP & Routing:** Validity checks, checksums, and routing lookup (is this for me?).
9.  **Transport (TCP/UDP):** The protocol handler processes the segment (congestion control, ack).
10. **Socket Buffer:** Data is queued in the socket's receive buffer.
11. **Context Switch:** The Application (User Space) calls `recv()`. The kernel copies the data from Kernel Space to User Space.

---

## 3. The Bottleneck: Why is Linux Slow?

The Linux kernel was designed for general-purpose computing, not high-performance packet processing.

### The Performance Cliff
As network speeds increased (10Gb $\to$ 40Gb $\to$ 100Gb), the CPU could not keep up, especially with small packets.
* **Throughput Graph:** At 64-byte packet sizes, the throughput crashes significantly compared to 1500-byte (MTU) packets.

### The Math of Packet Processing
For a **10 Gbps** link:
* **Packet Size:** 64 Bytes (minimum Ethernet frame).
* **Packet Rate:** $14.88$ Million Packets Per Second (Mpps).
* **Time Budget:** You have **67.2 nanoseconds** to process one packet.

**The Cache Reality:**
* A Last Level Cache ([[6 Top-down Microarchitecture Analysis#Drilling into Back-End Bound (Most Common in HPC)|L3]]) miss takes **~15-20 ns**.
* This means if you have **3 cache misses** per packet, you have already exceeded your time budget.
* *Result:* Packet loss and reduced throughput.

### Sources of Overhead
1.  **Dynamic Memory Allocation:** Allocating and freeing `sk_buff` for every packet is expensive. The metadata (256+ bytes) is often larger than the packet itself (64 bytes).
2.  **Per-Packet Costs:** Traversing the huge Linux stack (Netfilter, TC, Routing) for every single packet burns CPU cycles.
3.  **Data Copying:** Copying data from Kernel to User space wastes memory bandwidth.
4.  **Context Switches:** Moving between User Mode and Kernel Mode flushes TLBs and pollutes caches.

---

## 4. Solution 1: Kernel Bypass (DPDK)

If the Kernel is the problem, the solution is to remove it from the critical path. This is called **Kernel Bypass**.



### How it works
1.  **Direct Hardware Access:** The Userspace Application maps the NIC's registers and memory directly into its own address space (using PCIe features like VFIO).
2.  **Polling:** Instead of waiting for Interrupts (which are slow), the application spins a core 100% of the time checking the Rx Ring.
    * *Pros:* Lowest possible latency.
    * *Cons:* Burns 100% CPU of one core, even if no packets arrive.
3.  **Zero Copy:** Data is read directly from the DMA ring without copying it to kernel buffers.

### Examples
* **Netmap:** An academic framework for efficient packet I/O.
* **DPDK (Data Plane Development Kit):** The industry standard (Intel). It provides libraries for memory management, queue management, and driver access in userspace.

### Google Snap (Example)
Google uses a userspace networking system called **Snap**.
* Instead of letting the application handle raw packets (which is hard), Snap acts as a "Userspace Microkernel."
* It runs as a library linked to the application.
* It implements a custom TCP/IP stack in userspace.
* It handles scheduling and transport, allowing Google to upgrade networking logic without rebooting the host machine.

---

## 5. The New Bottleneck: PCIe and Memory

Even with Kernel Bypass, we hit hardware limits.

### PCIe Bandwidth
* **[[11 MultiGPU#2. Scale-Up Networks (Intra-Node)|PCIe]] Gen3 x16:** Theoretical max $\approx 126$ Gbps. Practical max for networking $\approx 100$ Gbps.
* **Overhead:** PCIe packet headers and protocol overhead eat into the bandwidth.
* **DDIO (Data Direct I/O):** Intel technology allows the NIC to write data directly into the CPU's **L3 Cache**, bypassing the slow DRAM.
    * *Limit:* The L3 cache is small (e.g., 30-50MB). If the packet rate is too high, we trash the cache (evicting useful application data), leading to **LLC Thrashing**.

---

## 6. Solution 2: Programmable NICs (SmartNICs)

If the CPU is struggling to handle the packet rate, we can offload processing to the Network Interface Card itself.



### Types of Offload
1.  **Fixed Function (ASIC):** Standard NICs can do Checksum offload, Segmentation offload (TSO/LRO). Fast but not flexible.
2.  **FPGA-based:** Highly flexible and fast, but very difficult to program (Verilog/VHDL).
3.  **SoC-based (System on Chip):** A NIC with a mini-CPU (e.g., ARM cores) on board. Easy to program, but slower and power-hungry.
4.  **NPU-based (Network Processing Unit):** Specialized many-core architectures optimized for packet processing.

### Example: Netronome (NFP)
The slides highlight the **Netronome NFP (Network Flow Processor)**.
* **Architecture:** It has many small, specialized cores designed for packet manipulation.
* **Offload Capability:** It can run **eBPF** or **P4** programs directly on the NIC.
* **Use Case - DDoS Mitigation:**
    * *Without SmartNIC:* The attack packets travel over PCIe $\to$ CPU $\to$ Dropped by Software. (Wastes PCIe and CPU).
    * *With SmartNIC:* The NIC matches the attack signature and drops the packet *on the card*. The packet never crosses the PCIe bus.

---

## 7. Solution 3: eBPF (Extended Berkeley Packet Filter)

Kernel Bypass (DPDK) is fast, but it has a major drawback: you lose the Linux ecosystem. Tools like `tcpdump`, `iptables`, and standard file descriptors don't work.

**eBPF** allows us to keep the Linux kernel but make it programmable and faster.



### How eBPF Works
1.  **Write Code:** The developer writes a small C program.
2.  **Compile:** Uses Clang/LLVM to compile it into eBPF Bytecode.
3.  **Verify:** The Kernel **Verifier** checks the code. It ensures:
    * No infinite loops (guaranteed termination).
    * No invalid memory access (safety).
4.  **JIT Compile:** The kernel Just-In-Time compiles the bytecode to native CPU assembly.
5.  **Hook:** The program is attached to a specific hook point in the kernel.

### XDP (eXpress Data Path)
XDP is a specific eBPF hook that sits at the very bottom of the networking stack, **inside the device driver**, before the `sk_buff` is even allocated.
* **Action:** The eBPF program can look at the raw packet and decide:
    * `XDP_DROP`: Drop it immediately (DDoS defense).
    * `XDP_TX`: Bounce it back out the same NIC (Load Balancing).
    * `XDP_PASS`: Pass it up to the normal Linux stack.

---

## 8. Specific Optimization Example: Struct Reorganization

The slides present a concrete example of optimization submitted to the Linux Kernel mailing list (Google, Patch v8).

### The Problem: Cache Line Consumption
In the Linux networking core, large C structures (like `struct sock` or `struct tcp_sock`) hold the state of a connection.
* **Old Layout:** Variables were organized logically (e.g., all timer variables together, all flags together) or chronologically (added as features were developed).
* **Cache Impact:** When the CPU processes a packet (the "Fast Path"), it needs to read specific fields (e.g., sequence number, window size). If these fields are scattered across the struct, the CPU must load **multiple cache lines** (64 bytes each).
* Even if you only need 1 byte from a cache line, you pay the cost of loading the whole 64 bytes.

### The Solution: "Analyze and Reorganize"
Google engineers analyzed exactly which variables are accessed during the data transfer phase.
* **Reorganization:** They moved these "hot" variables to sit next to each other in memory.
* **Goal:** Fit all necessary data for the fast path into **one or two cache lines**.

### The Result
* **Hardware:** AMD Platform with 100Gb NIC and 256MB L3 Cache.
* **Performance:** ~40% improvement in TCP throughput for specific scenarios, simply by rearranging the layout of a C struct to be cache-friendly.

---

## 9. RDMA (Remote Direct Memory Access)

While DPDK solves the packet processing bottleneck on a single host, we still face the problem of moving data *between* hosts efficiently. Standard TCP/IP processing consumes significant CPU cycles on both the sender and receiver.

**RDMA** allows one computer to directly access the memory of another computer without involving either one's Operating System or CPU.

### How RDMA Works

1.  **Zero Copy:** Data is transferred directly from the application memory of the Sender to the application memory of the Receiver.
2.  **Kernel Bypass:** The OS kernel is not involved in the data path.
3.  **CPU Offload:** The entire transport protocol (segmentation, reassembly, ACKs, congestion control) is handled by the **RDMA NIC (RNIC)** hardware, not the CPU.

### Performance Impact
* **Latency:** Extremely low (1-2 microseconds vs 10-20 microseconds for TCP).
* **Throughput:** Can saturate 100Gbps+ links with minimal CPU usage.
* **CPU Usage:** Near zero. The CPU just initiates the transfer and can go do other work (e.g., training an AI model).

### RDMA Transports
RDMA requires a lossless network fabric to work efficiently.
* **InfiniBand:** A specialized network architecture designed for RDMA. Very fast, lossless by design, but expensive and requires custom switches/cabling.
* **RoCE (RDMA over Converged Ethernet):** Runs RDMA over standard Ethernet.
    * **RoCEv1:** Layer 2 only (non-routable).
    * **RoCEv2:** Runs over UDP/IP (routable). Requires **PFC (Priority Flow Control)** in switches to ensure a lossless Ethernet network.
* **iWARP:** Runs RDMA over standard TCP/IP. Routable and works on lossy networks, but requires more complex NIC hardware to handle TCP offload.

### RDMA Operations (Verbs)
1.  **SEND/RECEIVE:** Two-sided operation. The receiver must post a buffer to receive data. Similar to `send()/recv()`.
2.  **WRITE:** One-sided. The sender writes directly to a specific address in the receiver's memory. The receiver CPU doesn't even know it happened.
3.  **READ:** One-sided. The sender reads data from the receiver's memory.

---

## 10. Advanced eBPF: Programmability in the Kernel

We revisited eBPF in the context of creating a fully programmable data plane within the Linux kernel.

### The Evolution of BPF
* **cBPF (Classic BPF):** Used by `tcpdump`. Two 32-bit registers. Only for packet filtering.
* **eBPF (Extended BPF):** * Ten 64-bit registers (mapping 1:1 to hardware registers).
    * JIT compilation for speed.
    * Can call helper functions in the kernel.
    * **Maps:** Key-Value stores that allow eBPF programs to share data with userspace or between different eBPF calls (e.g., counting packets, storing routing tables).



### Key eBPF Hook Points
1.  **XDP (eXpress Data Path):** Runs in the driver. Fastest. Good for dropping/redirecting (DDoS, Load Balancing).
2.  **TC (Traffic Control):** Runs after the `sk_buff` is allocated but before the IP stack. Good for packet mangling and shaping.
3.  **Socket Filter:** Runs right before the packet is given to the application. Good for observability.
4.  **Kprobes/Tracepoints:** Can attach to *any* kernel function for tracing and debugging.

### Case Study: Katran (Meta/Facebook)
**Katran** is a Layer 4 Load Balancer built with XDP/eBPF.
* **Problem:** Traditional hardware load balancers were inflexible and expensive. Software LBs (IPVS) were too slow.
* **Solution:** * Katran runs at the XDP hook.
    * It encapsulates packets (Ip-in-Ip) to send them to backend servers.
    * It uses eBPF Maps to store the state of millions of connections.
* **Performance:** It can process millions of packets per second on a single CPU core, linearly scaling with the number of cores.

### Case Study: Cilium
**Cilium** uses eBPF to provide networking, security, and observability for Kubernetes containers.
* Instead of using `iptables` (which becomes slow with thousands of rules), Cilium compiles security policies directly into efficient eBPF hashmaps.
* It creates a "sidecar-less" service mesh, handling logic in the kernel rather than injecting proxies into every pod.

---

## 11. SmartNICs vs. eBPF
* **SmartNICs** (like Netronome or NVIDIA BlueField) are great for offloading the *entire* CPU workload, but they are expensive, proprietary, and harder to program.
* **eBPF** runs on the host CPU but is standard, open-source, and works on any hardware.
* **Future:** "eBPF offload" allows running eBPF bytecode directly on the SmartNIC, giving the best of both worlds.

## 12. Fun Facts & Ecosystem
* **Windows Support:** Microsoft has ported eBPF to Windows, allowing cross-platform high-performance networking tools.
* **Android:** Google uses eBPF in Android for network accounting (Data Saver features) and security.
* **The Foundation:** The eBPF ecosystem is now managed by the **eBPF Foundation** (under the Linux Foundation), with members like Google, Meta, Netflix, Microsoft, and Isovalent.