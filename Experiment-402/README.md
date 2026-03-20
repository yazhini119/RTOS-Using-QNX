# Experiment 402: Repeating Timer using Pulse Messages in QNX

## Aim

To implement a **repeating timer in QNX** that periodically wakes a thread by sending **pulse messages**.

---

## Objective

* To understand **timer creation in QNX** using `timer_create()`.
* To learn how **pulse events are generated using SIGEV_PULSE_INIT()`.
* To demonstrate **periodic task execution using a repeating timer**.

---

## Problem Statement

Develop a QNX program that creates a **repeating timer**.
When the timer expires, it sends a **pulse message to a channel**.
The program waits in a `MsgReceive()` loop and processes the pulse whenever the timer expires.

The timer should:

* Trigger the **first event after 5 seconds**.
* Then **repeat every 1.5 seconds**.

---

# Algorithm

1. Start the program.
2. Declare variables for:

   * Channel ID
   * Connection ID
   * Timer ID
   * Timer specification
   * Message structure for pulses.
3. Create a **channel using `ChannelCreate()`**.
4. Attach a **connection to the channel using `ConnectAttach()`**.
5. Initialize a **pulse event using `SIGEV_PULSE_INIT()`**.
6. Create a **timer using `timer_create()`**.
7. Configure the timer using `itimerspec`:

   * Set first expiry to **5 seconds**.
   * Set repeating interval to **1.5 seconds**.
8. Start the timer using `timer_settime()`.
9. Enter an infinite loop.
10. Wait for incoming messages using `MsgReceive()`.
11. If a pulse message is received:

    * Check the pulse code.
12. If the pulse code matches the timer event:

    * Print **"Timer expired – pulse received"**.
13. Continue waiting for the next timer pulse.

---

# Program

```c id="rtm0pf"
/*
 * reptimer.c
 */

#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <sys/neutrino.h>
#include <sys/dispatch.h>
#include <unistd.h>
#include <signal.h>
#include <time.h>
#include <string.h>

#define TIMER_PULSE_EVENT (_PULSE_CODE_MINAVAIL + 7)

typedef union
{
    struct _pulse pulse;
} message_t;

int main(int argc, char *argv[])
{
    rcvid_t rcvid;
    struct sigevent event;
    int chid, coid;
    message_t msg;
    timer_t timerid;
    struct itimerspec it;

    chid = ChannelCreate(_NTO_CHF_PRIVATE);
    if (chid == -1)
    {
        fprintf(stderr, "ChannelCreate failed: %s\n", strerror(errno));
        exit(EXIT_FAILURE);
    }

    coid = ConnectAttach(0, 0, chid, _NTO_SIDE_CHANNEL, 0);
    if (coid == -1)
    {
        fprintf(stderr, "ConnectAttach failed: %s\n", strerror(errno));
        exit(EXIT_FAILURE);
    }

    SIGEV_PULSE_INIT(&event, coid, 10, TIMER_PULSE_EVENT, 0);

    if (timer_create(CLOCK_MONOTONIC, &event, &timerid) == -1)
    {
        perror("timer_create");
        exit(EXIT_FAILURE);
    }

    it.it_value.tv_sec = 5;
    it.it_value.tv_nsec = 0;

    it.it_interval.tv_sec = 1;
    it.it_interval.tv_nsec = 500 * 1000 * 1000;

    if (timer_settime(timerid, 0, &it, NULL) == -1)
    {
        perror("timer_settime");
        exit(EXIT_FAILURE);
    }

    while (1)
    {
        rcvid = MsgReceive(chid, &msg, sizeof(msg), NULL);

        if (rcvid == -1)
        {
            fprintf(stderr, "MsgReceive failed: %s\n", strerror(errno));
            continue;
        }

        if (rcvid == 0)
        {
            switch (msg.pulse.code)
            {
                case TIMER_PULSE_EVENT:
                    printf("got our pulse, the timer must have expired\n");
                    break;

                default:
                    printf("unexpected pulse code: %d\n", msg.pulse.code);
                    break;
            }
        }
    }
}
```

---

# Expected Output

```text id="aq8s4n"
got our pulse, the timer must have expired
got our pulse, the timer must have expired
got our pulse, the timer must have expired
got our pulse, the timer must have expired
...
```

*(The message appears every 1.5 seconds after the initial 5-second delay.)*

---

# Output
<img width="1445" height="257" alt="image" src="https://github.com/user-attachments/assets/346da1b0-a74c-4aef-a0b7-891554f8cc58" />


---

# Result

Thus, a **repeating timer using pulse messages in QNX** was successfully implemented.
The timer periodically generated pulses that were received through the **MsgReceive() loop**, demonstrating **periodic task execution in QNX**.
