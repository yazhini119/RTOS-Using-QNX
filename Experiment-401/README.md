# Experiment 401: Interrupt Handling with Counter in QNX

## Aim

To implement **hardware interrupt handling in QNX** and maintain a **count of the number of interrupts received**.

---

## Objective

* To understand **interrupt handling in QNX Neutrino RTOS**.
* To use **InterruptAttach()** and **InterruptWait()** functions.
* To maintain a **counter to track the number of interrupts received**.

---

## Problem Statement

Develop a QNX program that **attaches to a hardware interrupt (e.g., keyboard interrupt)** and waits for the interrupt to occur.
Each time the interrupt occurs, the program should **display a message and increment a counter showing how many times the interrupt has been received**.

---

# Algorithm

1. Start the program.
2. Declare a variable for the **interrupt number (IRQ)**.
3. Declare a variable to store the **interrupt attachment ID**.
4. Declare and initialize a **counter variable to 0**.
5. Display a message indicating the start of the interrupt example.
6. Attach to the interrupt using `InterruptAttach()`.
7. If interrupt attachment fails:

   * Display the error message.
   * Terminate the program.
8. Display a message indicating successful interrupt attachment.
9. Enter an infinite loop.
10. Wait for the interrupt using `InterruptWait()`.
11. When the interrupt occurs:

    * Print **"Interrupt received! Count is X"**.
12. Increment the counter value.
13. Continue waiting for the next interrupt.

---

# Program

```c id="xtm12h"
#include <stdio.h>
#include <stdlib.h>
#include <sys/neutrino.h>
#include <sys/syspage.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    int irq = 1;           // interrupt number (example: keyboard)
    int id;
    int count = 0;

    printf("Simple Interrupt \n");

    /* Attach interrupt */
    id = InterruptAttach(irq, NULL, NULL, 0, 0);
    if (id == -1)
    {
        perror("InterruptAttach");
        return EXIT_FAILURE;
    }

    printf("Attached to interrupt %d\n", irq);

    while (1)
    {
        /* Wait for interrupt */
        InterruptWait(0, NULL);

        printf("Interrupt received! , Count is %d\n", count);
        count++;
    }

    return 0;
}
```

---

# Expected Output

```text id="bl34ke"
Simple Interrupt 
Attached to interrupt 1
Interrupt received! , Count is 0
Interrupt received! , Count is 1
Interrupt received! , Count is 2
Interrupt received! , Count is 3
Interrupt received! , Count is 4
...
```

*(The counter increases each time the interrupt occurs.)*

---
# Output
<img width="1457" height="243" alt="image" src="https://github.com/user-attachments/assets/8590fe88-1865-4c0b-9752-e5766200a0a6" />
```

# Result

Thus, the **hardware interrupt handling mechanism in QNX** was successfully implemented using `InterruptAttach()` and `InterruptWait()`.
The program correctly detected interrupts and **maintained a counter showing the number of interrupts received**.
