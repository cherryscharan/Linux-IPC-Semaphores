# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores
# Developed by: CHARAN KUMAR S
# Register number: 212223220015

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.
```
/*
 * sem.c  - demonstrates a basic producer-consumer
 *                            implementation.              */
#include <stdio.h>       /* standard I/O routines.              */
#include <stdlib.h>      /* rand() and srand() functions        */
#include <unistd.h>      /* fork(), etc.                        */
#include <time.h>        /* nanosleep(), etc.                   */
#include <sys/types.h>   /* various type definitions.           */
#include <sys/ipc.h>     /* general SysV IPC structures         */
#include <sys/sem.h>     /* semaphore functions and structs.    */
#define NUM_LOOPS   20   /* number of loops to perform. */

#if defined(__GNU_LIBRARY__) && !defined(_SEM_SEMUN_UNDEFINED)
#else
union semun {
    int val;                    /* value for SETVAL */
    struct semid_ds *buf;       /* buffer for IPC_STAT, IPC_SET */
    unsigned short int *array;  /* array for GETALL, SETALL */
    struct seminfo *__buf;      /* buffer for IPC_INFO */
};
#endif

int main(int argc, char* argv[]) {
    int sem_set_id;              /* ID of the semaphore set.       */
    union semun sem_val;         /* semaphore value, for semctl(). */
    int child_pid;               /* PID of our child process.      */
    int i;                       /* counter for loop operation.    */
    struct sembuf sem_op;        /* structure for semaphore ops.   */
    int rc;                      /* return value of system calls.  */

    /* Create a private semaphore set with one semaphore in it. */
    sem_set_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_set_id == -1) {
        perror("main: semget");
        exit(1);
    }
    printf("semaphore set created, semaphore set id '%d'.\n", sem_set_id);

    /* Initialize the semaphore to '0'. */
    sem_val.val = 0;
    rc = semctl(sem_set_id, 0, SETVAL, sem_val);

    /* Fork-off a child process. */
    child_pid = fork();
    switch (child_pid) {
        case -1:  /* fork() failed */
            perror("fork");
            exit(1);
        case 0:   /* Child process here */
            for (i = 0; i < NUM_LOOPS; i++) {
                /* Block on the semaphore. */
                sem_op.sem_num = 0;
                sem_op.sem_op = -1;  /* Wait operation */
                sem_op.sem_flg = 0;
                semop(sem_set_id, &sem_op, 1);
                printf("consumer: '%d'\n", i);
                fflush(stdout);
            }
            break;
        default:  /* Parent process here */
            for (i = 0; i < NUM_LOOPS; i++) {
                printf("producer: '%d'\n", i);
                fflush(stdout);
                
                /* Signal the semaphore. */
                sem_op.sem_num = 0;
                sem_op.sem_op = 1;  /* Signal operation */
                sem_op.sem_flg = 0;
                semop(sem_set_id, &sem_op, 1);

                /* Introduce a small delay to control output order */
                if (i < 7) { // Produce the first 7 items quickly
                    usleep(100); // sleep for 100 microseconds
                } else {
                    usleep(300); // slow down after 6
                }
            }
            /* Clean up the semaphore set */
            semctl(sem_set_id, 0, IPC_RMID, sem_val);
            break;
    }
    return 0;
}
```




## OUTPUT
$ ./sem.o 


![image](https://github.com/user-attachments/assets/035f2a6d-9afa-449e-8528-4a2ebb9db383)

$ ipcs


![image](https://github.com/user-attachments/assets/94898627-121c-422f-ae91-7619a4eddca7)



# RESULT:
The program is executed successfully.




