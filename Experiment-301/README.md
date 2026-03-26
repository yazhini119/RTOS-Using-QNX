# Experiment 301: Message Passing between Client and Server in QNX

## Aim

To implement **inter-process communication using message passing in QNX**, where a client sends a string to a server and receives a checksum as a reply.

---

## Objective

* To understand **QNX message passing mechanism**.
* To implement **client-server communication using ChannelCreate(), MsgSend(), MsgReceive(), and MsgReply()**.
* To calculate and return a **checksum for a string sent by the client**.

---

## Problem Statement

Design a **QNX server and client program** where:

* The **server creates a communication channel** and waits for messages from clients.
* The **client connects to the server using the server's PID and Channel ID**.
* The client sends a **string message** to the server.
* The server calculates a **checksum of the string**.
* The server replies with the **calculated checksum**.
* The client receives and prints the checksum.

---

# Algorithm
* Ensure that the header file: `msg_def.h` is created/copied for both the projects (server and client).

## Server Algorithm

1. Start the server program.
2. Create a communication channel using `ChannelCreate()`.
3. Obtain the process ID using `getpid()`.
4. Display the server **PID and Channel ID**.
5. Enter an infinite loop to receive messages from clients.
6. Wait for a message using `MsgReceive()`.
7. Check the received message type.
8. If the message type is **checksum request**:

   * Extract the string from the message.
   * Call the checksum calculation function.
9. Compute the checksum by summing ASCII values of characters in the string.
10. Send the checksum back to the client using `MsgReply()`.
11. If the message type is unknown, return an error using `MsgError()`.
12. Continue waiting for the next message.

---

## Client Algorithm

1. Start the client program.
2. Verify that command line arguments are provided.
3. Read the following inputs from the arguments:

   * Server PID
   * Server Channel ID
   * String to send.
4. Establish a connection with the server using `ConnectAttach()`.
5. Create a message structure containing the message type and string.
6. Send the message to the server using `MsgSend()`.
7. Wait for the server reply.
8. Receive the checksum value from the server.
9. Display the received checksum.
10. Terminate the client program.

---

# Program

## Server Program (server.c)

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <sys/neutrino.h>
#include <process.h>
#include "msg_def.h"

int calculate_checksum(char *text);

int main(void)
{
    int chid;
    int pid;
    rcvid_t rcvid;
    cksum_msg_t msg;
    int status;
    int checksum;

    chid = ChannelCreate(0);
    if (chid == -1)
    {
        perror("ChannelCreate()");
        exit(EXIT_FAILURE);
    }

    pid = getpid();
    printf("Server's pid: %d, chid: %d\n", pid, chid);

    while (1)
    {
        rcvid = MsgReceive(chid, &msg, sizeof(msg), NULL);
        if (rcvid == -1)
        {
            perror("MsgReceive");
            exit(EXIT_FAILURE);
        }

        if (msg.msg_type == CKSUM_MSG_TYPE)
        {
            printf("Got a checksum message\n");

            checksum = calculate_checksum(msg.string_to_cksum);

            status = MsgReply(rcvid, EOK, &checksum, sizeof(checksum));
            if (status == -1)
                perror("MsgReply");
        }
        else
        {
            printf("Got an unknown message type %d\n", msg.msg_type);

            if (MsgError(rcvid, ENOSYS) == -1)
                perror("MsgError");
        }
    }

    return 0;
}

int calculate_checksum(char *text)
{
    char *c;
    int cksum = 0;

    for (c = text; *c != '\0'; c++)
        cksum += *c;

    return cksum;
}
```

---

## Client Program (client.c)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/neutrino.h>
#include "msg_def.h"

int main(int argc, char* argv[])
{
    int coid;
    cksum_msg_t msg;
    int incoming_checksum;
    int status;
    int server_pid;
    int server_chid;

    if (argc != 4)
    {
        printf("ERROR: This program must be started with commandline arguments\n");
        printf("Example:\n");
        printf("client 482834 1 abcdefghi\n");
        exit(EXIT_FAILURE);
    }

    server_pid = atoi(argv[1]);
    server_chid = atoi(argv[2]);

    printf("attempting to establish connection with server pid: %d, chid %d\n",
           server_pid, server_chid);

    coid = ConnectAttach(0, server_pid, server_chid, _NTO_SIDE_CHANNEL, 0);

    if (coid == -1)
    {
        perror("ConnectAttach");
        exit(EXIT_FAILURE);
    }

    msg.msg_type = CKSUM_MSG_TYPE;
    strlcpy(msg.string_to_cksum, argv[3], sizeof(msg.string_to_cksum));

    printf("Sending string: %s\n", msg.string_to_cksum);

    status = MsgSend(coid, &msg, sizeof(msg),
                     &incoming_checksum, sizeof(incoming_checksum));

    if (status == -1)
    {
        perror("MsgSend");
        exit(EXIT_FAILURE);
    }

    printf("received checksum=%d from server\n", incoming_checksum);
    printf("MsgSend return status: %d\n", status);

    return EXIT_SUCCESS;
}
```
---
## Header file (msg_def.h)

```c

#ifndef MSG_DEF_H
#define MSG_DEF_H

#define CKSUM_MSG_TYPE  0x01
#define MAX_STRING_SIZE 256

typedef struct {
    int msg_type;
    char string_to_cksum[MAX_STRING_SIZE];
} cksum_msg_t;

#endif
```
# Expected Output

### Server Side

```
Server's pid: 12345, chid: 1
Got a checksum message
```

### Client Side

```
attempting to establish connection with server pid: 12345, chid 1
Sending string: abcdefghi
received checksum=909 from server
MsgSend return status: 0
```

*(The checksum value depends on the ASCII sum of characters in the string.)*

---

# Output
## Server Side
<img width="1004" height="314" alt="Screenshot 2026-03-26 085934" src="https://github.com/user-attachments/assets/40c450fe-1c94-4c7f-950b-cb75cdf0e7b7" />

## Client Side
<img width="1034" height="309" alt="image" src="https://github.com/user-attachments/assets/4c71bc16-a985-4085-971c-051bce777225" />

---

# Result

Thus, the **client-server communication using QNX message passing** was successfully implemented.
The client sent a string to the server, and the server calculated and returned the **checksum correctly**.
