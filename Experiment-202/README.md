# Experiment 202: Condition Variables and State Machine using Threads

## Aim

To implement a **multi-state machine using POSIX threads and condition variables** in QNX, where multiple threads synchronize using a shared state variable.

---

## Objective

* To understand **thread synchronization using mutex and condition variables**.
* To implement a **state machine with four states (0, 1, 2, 3)**.
* To demonstrate **state transitions controlled by multiple threads**.

---

## Problem Statement

Design a multithreaded program where **four threads represent four states of a state machine**.
All threads share a **common state variable, mutex, and condition variable**.

The system should follow the transitions:

```
State 0 → State 1
State 1 → State 2 (if internal variable is even)
State 1 → State 3 (if internal variable is odd)
State 2 → State 0
State 3 → State 0
```

Each state is handled by a **separate thread**, and threads synchronize using **mutex locks and condition variables**.

---

# Algorithm

1. Start the program.
2. Declare a **global state variable** initialized to **0**.
3. Declare a **mutex and condition variable** for synchronization.
4. Initialize the mutex and condition variable.
5. Create **four threads**, each representing a state:

   * Thread for **State 0**
   * Thread for **State 1**
   * Thread for **State 2**
   * Thread for **State 3**
6. Each thread runs continuously in a loop.

### State 0 Thread

7. Lock the mutex.
8. Wait until the shared state becomes **0**.
9. Print **"transit 0 → 1"**.
10. Change the state to **1**.
11. Broadcast the condition variable to wake other threads.
12. Unlock the mutex.
13. Delay briefly.

### State 1 Thread

14. Lock the mutex.
15. Wait until the state becomes **1**.
16. Increment internal variable `i`.
17. If `i` is **odd**, set state to **3**.
18. If `i` is **even**, set state to **2**.
19. Print **"transit 1 → state"**.
20. Broadcast the condition variable.
21. Unlock the mutex.

### State 2 Thread

22. Lock the mutex.
23. Wait until the state becomes **2**.
24. Print **"transit 2 → 0"**.
25. Set the state to **0**.
26. Broadcast the condition variable.
27. Unlock the mutex.

### State 3 Thread

28. Lock the mutex.

29. Wait until the state becomes **3**.

30. Print **"transit 3 → 0"**.

31. Set the state to **0**.

32. Broadcast the condition variable.

33. Unlock the mutex.

34. Allow the threads to run for a fixed duration.

35. Exit the program.

---

# Program

```c
/*
 * condvar.c
 */

#include <stdio.h>
#include <unistd.h>
#include <sys/neutrino.h>
#include <pthread.h>
#include <sched.h>
#include <errno.h>
#include <string.h>
#include <stdlib.h>

volatile int state;

pthread_mutex_t mutex;
pthread_cond_t cond;

void *state_0(void *);
void *state_1(void *);
void *state_2(void *);
void *state_3(void *);

int main()
{
    int ret;

    ret = pthread_mutex_init(&mutex, NULL);
    if (ret != EOK) {
        fprintf(stderr,"pthread_mutex_init failed: %s\n", strerror(ret));
        exit(EXIT_FAILURE);
    }

    ret = pthread_cond_init(&cond, NULL);
    if (ret != EOK) {
        fprintf(stderr,"pthread_cond_init failed: %s\n", strerror(ret));
        exit(EXIT_FAILURE);
    }

    state = 0;

    pthread_create(NULL,NULL,state_1,NULL);
    pthread_create(NULL,NULL,state_0,NULL);
    pthread_create(NULL,NULL,state_2,NULL);
    pthread_create(NULL,NULL,state_3,NULL);

    sleep(20);
    printf("main, exiting\n");
    return 0;
}

void *state_0(void *arg)
{
    while(1)
    {
        pthread_mutex_lock(&mutex);
        while(state != 0)
            pthread_cond_wait(&cond,&mutex);

        printf("transit 0 -> 1\n");
        state = 1;

        pthread_cond_broadcast(&cond);
        pthread_mutex_unlock(&mutex);
        delay(100);
    }
}

void *state_1(void *arg)
{
    int i = 1;

    while(1)
    {
        pthread_mutex_lock(&mutex);
        while(state != 1)
            pthread_cond_wait(&cond,&mutex);

        if(++i & 0x1)
            state = 3;
        else
            state = 2;

        printf("transit 1 -> %d\n",state);

        pthread_cond_broadcast(&cond);
        pthread_mutex_unlock(&mutex);
    }
}

void *state_2(void *arg)
{
    while(1)
    {
        pthread_mutex_lock(&mutex);
        while(state != 2)
            pthread_cond_wait(&cond,&mutex);

        printf("transit 2 -> 0\n");
        state = 0;

        pthread_cond_broadcast(&cond);
        pthread_mutex_unlock(&mutex);
    }
}

void *state_3(void *arg)
{
    while(1)
    {
        pthread_mutex_lock(&mutex);
        while(state != 3)
            pthread_cond_wait(&cond,&mutex);

        printf("transit 3 -> 0\n");
        state = 0;

        pthread_cond_broadcast(&cond);
        pthread_mutex_unlock(&mutex);
    }
}
```

---

# Expected Output

```
transit 0 -> 1
transit 1 -> 2
transit 2 -> 0
transit 0 -> 1
transit 1 -> 3
transit 3 -> 0
transit 0 -> 1
transit 1 -> 2
transit 2 -> 0
transit 0 -> 1
transit 1 -> 3
transit 3 -> 0
...
main, exiting
```

*(The sequence alternates between state 2 and state 3 depending on whether the counter is even or odd.)*

---

# Output
<img width="1439" height="795" alt="image" src="https://github.com/user-attachments/assets/3e85ae2a-eff1-43db-a502-c49235644243" />

---

# Result

Thus, a **multi-state machine using POSIX threads, mutex, and condition variables** was successfully implemented and the **state transitions were executed correctly**.
