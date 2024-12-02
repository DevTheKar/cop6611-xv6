# **Modifying xv6 for Virtual Memory and CPU Scheduling**

## **Overview**
This README details the modifications made to xv6 in two parts:

1. **Part A: Pointer Dereference**  
   Modification of xv6's virtual memory layout to leave the first three pages unmapped and handle null pointer dereferences effectively.
   
2. **Part C: CPU Scheduling**  
   Implementation of additional scheduling algorithms—First-Come, First-Serve (FCFS) and Priority-Based Scheduling (PBS)—and enhancements for process management and statistics reporting.

---

## **How to Run**
To compile and run xv6 with a specific scheduler:
```bash
make clean
make qemu-nox SCHEDULER=[FLAG]
```
Where `FLAG` can be one of the following:
- `RR` (default): Round Robin
- `FCFS`: First-Come, First-Serve
- `PBS`: Priority-Based Scheduling
- `MLFQ`: Multi-Level Feedback Queue

---

## **Test Files**
| **File**           | **Command**           | **Description**                                   |
|---------------------|-----------------------|---------------------------------------------------|
| `setPriority.c`     | `setPriority <priority> <pid>` | Change the priority of a process.                |
| `time.c`            | `time <process>`     | Measure the runtime and wait time of a process.  |
| `tester.c`          | `time tester`        | Run a benchmarking program for scheduling.       |
| `tester_ps.c`       | `tester_ps`          | Test process statistics (includes `getps`).      |
| `tester_pbs.c`      | `tester_pbs`         | Test Priority-Based Scheduling (PBS).            |
| `tester_mlfq.c`     | `tester_mlfq`        | Test Multi-Level Feedback Queue (MLFQ).          |

---

## **Part A: Pointer Dereference**
### **Objective**
To modify xv6's virtual memory layout such that the first three pages (0x0 to 0x2FFF) remain unmapped. This ensures that null pointer dereferences result in a trap and termination of the offending process.

### **Changes Made**
1. **Unmapped Pages:**
   - Updated the virtual memory layout to ensure user code starts at 0x3000, leaving the first three pages unmapped.
   - Modified `exec.c`, `syscall.c`, and `vm.c` to align with this new memory layout.
   
2. **Trap Handling:**
   - Added logic in `trap.c` to detect and terminate processes attempting to access the unmapped range.

3. **Makefile:**
   - Adjusted user program entry points to start at 0x3000.

### **Testing**
**Null Pointer Dereference Test**
- A test program dereferences a null pointer to verify that the process is terminated as expected.
- Run using:
  ```bash
  nulltest
  ```

---

## **Part C: CPU Scheduling**

### **Objective**
To implement and test additional CPU scheduling algorithms, including:
1. **First-Come, First-Serve (FCFS):** Non-preemptive scheduling based on process arrival time.
2. **Priority-Based Scheduling (PBS):** Scheduling processes based on priority values, with lower numbers indicating higher priority.

### **Modifications**

#### **1. FCFS Scheduler**
- Implemented in `scheduler.c`.
- Processes are selected based on the minimum `p->ctime` (creation time).
- Preemption is disabled for FCFS:
  ```c
  #ifndef FCFS
      yield();
  #endif
  ```
- Ensures non-preemptive behavior.

#### **2. PBS Scheduler**
- Processes are scheduled based on priority. In case of ties, processes are selected based on `p->ctime`.
- Added `setPriority` system call:
  ```c
  int sys_set_priority(void) {
      int pid, priority;
      if (argint(0, &pid) < 0 || argint(1, &priority) < 0) return -1;
      return set_priority(pid, priority);
  }
  ```

#### **3. Process Statistics**
- Enhanced `struct proc` in `proc.h` to include fields for:
  - `ctime`: Creation time.
  - `etime`: Exit time.
  - `rtime`: Runtime.
  - `iotime`: I/O wait time.
- Added `getps` system call for detailed process statistics.

#### **4. Testing Enhancements**
- Benchmarked scheduling algorithms with `time tester` and analyzed runtime, wait time, and sleep time.

### **Testing Results**

#### **Comparison Table**
| Scheduler | Wait Time | Runtime | Sleep Time | Total Time |
|-----------|-----------|---------|------------|------------|
| **RR**    | 3         | 17      | 2435       | 2455       |
| **FCFS**  | 5         | 1       | 4220       | 4226       |
| **PBS**   | 6         | 15      | 2422       | 2444       |
| **MLFQ**  | 5         | 1       | 2404       | 2410       |

- **Order of Efficiency:**  
  MLFQ < PBS ≈ RR < FCFS  
  - FCFS has the longest total time due to non-preemption.
  - MLFQ demonstrates the shortest total time by dynamically adjusting priorities and preventing starvation.

---

## **Summary of Changes**
1. **Core Files Modified:**
   - `proc.c`, `proc.h`, `trap.c`, `scheduler.c`, `syscall.c`, `sysproc.c`, `defs.h`, `usys.S`, `user.h`
   
2. **Test Files Added:**
   - `time.c`, `tester.c`, `tester_ps.c`, `tester_pbs.c`, `tester_mlfq.c`, `setPriority.c`

3. **Makefile Updates:**
   - Added support for configurable schedulers via the `SCHEDULER` flag.

4. **Documentation:**
   - README updated with detailed instructions and results.

---
