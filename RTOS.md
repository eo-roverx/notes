## basic terms
*   **process:** a program that is currently running. this is handled at OS level
*   **job:** a part of a process that can be executed independently. this is handled at kernel level
*   **scheduling:** the process of assigning jobs to processes
*   **interrupt:** a signal that causes the current process to be suspended, and a new process to be executed. this is handled at hardware level
*   **time-sharing:** the process of sharing the CPU between multiple processes
*   **clock interrupt:** a signal that is generated by the system clock at regular intervals. this is handled at hardware level (RTOS)
*   **context switch:** the process of saving the current state of a process, and loading the state of a new process. this is handled at kernel level

## notes
*   a real-time system **must guarantee a response within a specified time limit** (pre-emtive)
*   an event-driven system is one where the system responds to events as they occur (non-pre-emptive)
*   most general-purpose OSes are partly pre-emptive and partly non-pre-emptive (such as in round-robin scheduling)
*   RTOS gives us deterministic behavior and predictable response times
*   RTOS is used in systems where guarantees that all processes will execute are critical (i.e., the system is responsive and no task will "hang")
*   RTOS can switch between tasks fairly quickly
*   round-robin scheduling is a pre-emptive scheduling algorithm. however, unlike a RT system, it does not guarantee a response within a specified time limit

## types of RTOS
*   **hard RTOS:** the system must respond within a specified time limit. if it doesn't, the system is considered to have failed
*   **soft RTOS:** the system should respond within a specified time limit. if it doesn't, the system is considered to have degraded performance

## RTOS "distros"
*   **FreeRTOS:** open source, small footprint, portable, pre-emptive, priority-based scheduler, supports multiple architectures. free to program as hard or soft
*   **Zephyr:** open source, large footprint, multiple tools and libraries. free to program as hard or soft.
*   **RTLinux:** open source, small footprint, POSIX-compliant, pre-emptive, priority-based scheduler, supports multiple architectures. hard RTOS