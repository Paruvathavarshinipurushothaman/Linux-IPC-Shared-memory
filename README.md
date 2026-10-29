# Linux-IPC-Shared-memory
Ex06-Linux IPC-Shared-memory

# AIM:
To Write a C program that illustrates two processes communicating using shared memory.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Shared Memory

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that illustrates two processes communicating using shared memory.
```

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/wait.h>

#define TEXT_SZ 2048

struct shared_use_st {
    int written;
    char some_text[TEXT_SZ];
};

int main(void) {
    int shmid;
    void *shared_memory;
    struct shared_use_st *shared_stuff;

    // Create shared memory
    shmid = shmget((key_t)1234,
                   sizeof(struct shared_use_st),
                   0666 | IPC_CREAT);

    if (shmid == -1) {
        perror("shmget failed");
        exit(EXIT_FAILURE);
    }

    printf("Shared memory id = %d\n", shmid);

    // Attach shared memory
    shared_memory = shmat(shmid, NULL, 0);

    if (shared_memory == (void *)-1) {
        perror("shmat failed");
        shmctl(shmid, IPC_RMID, NULL);
        exit(EXIT_FAILURE);
    }

    printf("Memory attached at %p\n", shared_memory);

    shared_stuff = (struct shared_use_st *)shared_memory;
    shared_stuff->written = 0;

    // Create child process
    pid_t pid = fork();

    if (pid == -1) {
        perror("fork failed");
        shmdt(shared_memory);
        shmctl(shmid, IPC_RMID, NULL);
        exit(EXIT_FAILURE);
    }

    if (pid == 0) {
        // Child process (Consumer)
        while (1) {
            while (shared_stuff->written == 0) {
                sleep(1);
            }

            printf("Consumer received: %s",
                   shared_stuff->some_text);
            fflush(stdout);

            if (strncmp(shared_stuff->some_text, "end", 3) == 0) {
                break;
            }

            shared_stuff->written = 0;
        }

        shmdt(shared_memory);
        exit(EXIT_SUCCESS);

    } else {
        // Parent process (Producer)
        char buffer[TEXT_SZ];

        while (1) {
            printf("Enter Some Text: ");
            fflush(stdout);

            if (fgets(buffer, sizeof(buffer), stdin) == NULL) {
                strcpy(buffer, "end\n");
            }

            snprintf(shared_stuff->some_text,
                     TEXT_SZ, "%s", buffer);

            shared_stuff->written = 1;

            // Corrected printf statement
            printf("%s", shared_stuff->some_text);
            fflush(stdout);

            if (strncmp(buffer, "end", 3) == 0) {
                break;
            }

            while (shared_stuff->written == 1) {
                sleep(1);
            }
        }

        // Wait for child process
        wait(NULL);

        // Detach shared memory
        if (shmdt(shared_memory) == -1) {
            perror("shmdt failed");
        }

        // Remove shared memory
        if (shmctl(shmid, IPC_RMID, NULL) == -1) {
            perror("shmctl failed");
        }
    }

    return 0;
}
```
## OUTPUT
<img width="1460" height="667" alt="image" src="https://github.com/user-attachments/assets/1c466f03-de0d-42b8-a0d9-9a81aa28ecc9" />

# RESULT:
The program is executed successfully.
