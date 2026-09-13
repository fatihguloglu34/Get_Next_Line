*This project has been created as part of the 42 curriculum by fguloglu.*

# Get Next Line (GNL)

## Description
**Get Next Line** is a fundamental C programming project in the 42 curriculum. The objective is to write a function that reads and returns a single line from a given file descriptor (`fd`) upon each call. 

Repeated calls to `get_next_line()` allow reading a text file or standard input line-by-line until reaching the end of the file (EOF). The function correctly preserves state across calls using a **static variable**, handles varying buffer sizes dynamically, and manages memory to prevent memory leaks.

### Features
- **Line-by-line Reading:** Reads from a file descriptor line-by-line using a customizable buffer size (`-D BUFFER_SIZE=n`).
- **State Preservation:** Retains unread remaining buffer content across function calls using static memory allocations.
- **Proper Line Formatting:** Includes the newline (`\n`) character in the output line (unless EOF is reached without a trailing newline).
- **Bonus Feature (Multiple FDs):** Manages multiple active file descriptors simultaneously using a single static pointer array (`OPEN_MAX`), allowing reading from `fd 3`, `fd 4`, `fd 5`, etc., in an interleaved order without losing state or mixing data streams.
- **Bonus Feature (Single Static Variable):** Implements the full logic using only **one single static variable**.

---

## Instructions

### Compilation
The subject specifies that no Makefile is required for Get Next Line. Compile the project files directly using `cc` along with the standard flags (`-Wall -Wextra -Werror`) and the required `-D BUFFER_SIZE` macro.

**1. Mandatory Part Compilation:**
```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 main.c get_next_line.c get_next_line_utils.c -o gnl
```

**2. Bonus Part Compilation:**
```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 main.c get_next_line_bonus.c get_next_line_utils_bonus.c -o gnl_bonus
```

### Usage Example
You can test both mandatory and bonus functionality using a simple `main.c`:

```c
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include "get_next_line_bonus.h"

int main(void)
{
    int   fd1;
    int   fd2;
    char  *line;

    fd1 = open("file1.txt", O_RDONLY);
    fd2 = open("file2.txt", O_RDONLY);
    if (fd1 < 0 || fd2 < 0)
        return (1);

    // Reading interleavedly from two different file descriptors (Bonus)
    line = get_next_line(fd1);
    printf("FD1 Line 1: %s", line);
    free(line);

    line = get_next_line(fd2);
    printf("FD2 Line 1: %s", line);
    free(line);

    line = get_next_line(fd1);
    printf("FD1 Line 2: %s", line);
    free(line);

    close(fd1);
    close(fd2);
    return (0);
}
```

---

## Algorithm Explanation & Justification

The core algorithm relies on incremental dynamic buffer management combined with static persistence across function executions.

### Core Execution Flow
1. **Chunk Reading & Accumulation:**
   The function calls `read()` to pull `BUFFER_SIZE` bytes at a time into a temporal buffer. This buffer is repeatedly appended to the static string variable (`left_str`) until a newline (`\n`) is encountered or `read()` returns `0` (EOF).
2. **Line Extraction:**
   Once a newline (`\n`) is identified or EOF is reached, the algorithm scans `left_str` up to the delimiter, allocates exact memory, and builds the line string to return to the caller.
3. **Leftover Truncation:**
   The extracted line is removed from `left_str`, leaving only the unread trailing characters stored for the next function invocation. Memory is freed and reset to `NULL` upon encountering errors or EOF.

### Bonus Algorithm: Managing Multiple FDs with 1 Static Variable
To fulfill the bonus requirement of handling multiple file descriptors simultaneously with a **single static variable**, `left_str` is declared as an array of char pointers indexed directly by the file descriptor value:

```c
static char *left_str[OPEN_MAX];
```

**Justification for this Approach:**
- **O(1) Direct Mapping:** Utilizing `left_str[fd]` allows direct, constant-time indexing for each individual file stream without needing dynamic search structures.
- **State Isolation:** Each file descriptor maintains its own independent reading buffer and EOF state, enabling interleaved calls between multiple files.
- **Resource Efficiency:** Memory is allocated dynamically per `fd` only when actively read, and freed immediately when EOF or an error occurs.

---

## Resources

### References & Documentation
- **C Static Variables:** [GeeksforGeeks - Static Variables in C](https://www.geeksforgeeks.org/static-variables-in-c/)
- **File Descriptors & System Calls:** `man 2 read`, `man 2 open`
- **Linux File Systems:** [IBM Developer Documentation on File I/O System Calls](https://www.ibm.com/docs/en/aix)

### AI Usage Statement
In compliance with the 42 AI Guidelines:
- **Tasks Performed with AI:** AI assistance was used exclusively for structuring, translating, and formatting the documentation (`README.md`) according to the updated project PDF standards, as well as refining technical prose.
- **Exclusions:** No algorithm logic, C source code (`get_next_line.c`, `get_next_line_bonus.c`), or memory management code was generated by AI. All implementation details were designed and coded independently.

This README was generated with the assistance of AI.