# Part A: Pointer Dereference - xv6 Virtual Memory Layout

## **Purpose**
The goal of **Part A** is to modify the virtual memory (VM) layout of xv6 so that the first three pages of the address space (0x0 to 0x2FFF) remain unmapped. This prevents null pointer dereferences from accessing arbitrary memory. Any process attempting to access these unmapped pages will be trapped and terminated. The updated layout ensures user code starts at 0x3000.

---

## **Changes Made**
Below is a summary of the changes made to implement the required functionality:

### **1. `exec.c`**
- **File:** `exec.c`
- **Line:** 44
- **Modification:**
  - Changed `sz` initialization to start at `PGSIZE`.
- **Code:**
  ```c
  sz = PGSIZE;
  ```
- **Reason:** Ensures the code segment starts after the first unmapped page.

---

### **2. `syscall.c`**
- **File:** `syscall.c`
- **Line:** 69 (inside `argptr()` function)
- **Modification:**
  - Added validation to reject null or invalid pointers passed as syscall arguments.
- **Code:**
  ```c
  if(size < 0 || (uint)i >= curproc->sz || (uint)i + size > curproc->sz || i == 0)
      return -1;
  ```
- **Reason:** Protects against null pointer dereferences and memory access violations in syscalls.

---

### **3. `vm.c`**
- **File:** `vm.c`
- **Line:** 330 (inside `copyuvm()` function)
- **Modification:**
  - Updated loop to skip the unmapped pages (0x0 to 0x2FFF) when copying memory.
- **Code:**
  ```c
  for(i = PGSIZE; i < sz; i += PGSIZE){
  ```
- **Reason:** Avoids copying unmapped memory when creating a new process.

---

### **4. `Makefile`**
- **File:** `Makefile`
- **Lines:** 149, 156
- **Modification:**
  - Updated user program entry points to 0x3000.
- **Code:**
  ```make
  ld -N -e 0x3000 -Ttext 0x3000 -o initcode.out initcode.o
  ld -N -e 0x3000 -Ttext 0x3000 -o _init $(ULIB) $(USRC) $(UOBJ)
  ```
- **Reason:** Aligns user programs with the new VM layout.

---

### **5. `usertests.c`**
- **File:** `usertests.c`
- **Line:** 1563 (inside `validatetest()` function)
- **Modification:**
  - Changed the starting address of validation tests to `0x3000`.
- **Code:**
  ```c
  p = 0x3000;
  ```
- **Reason:** Ensures tests align with the new VM layout.

---

### **6. `trap.c`**
- **File:** `trap.c`
- **Modification:**
  - Added handling for null pointer dereferences and accesses to the unmapped range in the `T_PGFLT` (page fault) case.
- **Code:**
  ```c
  case T_PGFLT: {
      uint fault_addr = rcr2();
      if (fault_addr < 0x3000) {
          cprintf("pid %d %s: null pointer or unmapped access at 0x%x\n",
                  myproc()->pid, myproc()->name, fault_addr);
          myproc()->killed = 1;
      }
      break;
  }
  ```
- **Reason:** Traps invalid memory accesses and terminates the offending process.

---

## **Testing**
### **1. Null Pointer Dereference Test**
**Test Program:**
```c
#include "syscall.h"
#include "types.h"
#include "user.h"

#define NULL 0
#define stdout 1
int main()
{
    printf(stdout, "This is a test for NULL pointer deference \n");
    int *p = NULL;
    printf(1, "*p: %d \n",*p);
    exit();
}
```
- **Expected Behavior:** The kernel logs the null pointer dereference and terminates the process.
- **Actual Behavior:**
![alt text](image.png)
---

## **Challenges and Solutions**
1. **Challenge:** Understanding xv6's memory allocation flow, especially in `exec.c` and `userinit()`.
   - **Solution:** Closely reviewed the code to identify where the memory layout is initialized and adjusted `sz`.

2. **Challenge:** Handling null pointer dereferences in `trap.c`.
   - **Solution:** Used `rcr2()` to detect the faulting address and added specific handling for addresses in the range 0x0 to 0x2FFF.

3. **Challenge:** Aligning user programs with the new layout.
   - **Solution:** Updated the `Makefile` to set the entry point and text segment address to 0x3000.

---

## **How to Run**
1. **Build the Project:**
   ```bash
   make clean
   make
   ```
2. **Run xv6 in QEMU:**
   ```bash
   make qemu
   ```
3. **Test Null Pointer Dereference:**
   - Compile and run the test program:
     ```bash
     nulltest
     ```
---


# Part B: Pointer Dereference - xv6 Virtual Memory Layout
