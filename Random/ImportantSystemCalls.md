
# Important System Calls for Backend Engineers

Backend engineers rely on system calls to interact with the operating system kernel, enabling them to build robust and efficient applications. These calls are fundamental for managing resources, handling data, and communicating over networks. Here's a breakdown of important system calls categorized by their function:

### File I/O

A cornerstone of backend development is file manipulation. The following system calls are essential for these operations:

*   **`open()`**: This call is used to open a file, returning a file descriptor that is used by other file-related system calls.
    *   **Input**: `pathname` (path to the file), `flags` (access modes like `O_RDONLY`, `O_WRONLY`, `O_RDWR`, and options like `O_CREAT`, `O_TRUNC`), `mode` (permissions for a new file).
    *   **Output**: A file descriptor (a small, non-negative integer) on success, or -1 on error.
*   **`read()`**: This call reads data from an open file into a buffer.
    *   **Input**: `fd` (file descriptor), `buf` (pointer to a buffer), `count` (maximum number of bytes to read).
    *   **Output**: The number of bytes read on success, 0 at the end of the file, or -1 on error.
*   **`write()`**: This call writes data from a buffer to an open file.
    *   **Input**: `fd` (file descriptor), `buf` (pointer to a buffer), `count` (number of bytes to write).
    *   **Output**: The number of bytes written on success, or -1 on error.
*   **`close()`**: This call closes an open file descriptor, releasing it.
    *   **Input**: `fd` (file descriptor).
    *   **Output**: 0 on success, or -1 on error.

### Process Management

Backend systems often need to manage multiple processes to handle concurrent requests. Key system calls for process management include:

*   **`fork()`**: This call creates a new process by duplicating the calling process.
    *   **Input**: None.
    *   **Output**: Returns the child's PID in the parent process, 0 in the child process, or -1 on error.
*   **`exec()`**: This call replaces the current process image with a new process image.
    *   **Input**: `path` (path to the new executable), `argv` (arguments for the new program), `envp` (environment variables).
    *   **Output**: Does not return on success. Returns -1 on error.
*   **`wait()`**: This call is used to wait for a child process to change state.
    *   **Input**: `stat_loc` (pointer to store the child's exit status).
    *   **Output**: The PID of the terminated child on success, or -1 on error.
*   **`exit()`**: This call terminates the calling process.
    *   **Input**: `status` (the exit status of the process).
    *   **Output**: Does not return.
*   **`kill()`**: This call sends a signal to a process or a group of processes.
    *   **Input**: `pid` (the process ID or process group ID), `sig` (the signal to send).
    *   **Output**: 0 on success, or -1 on error.

### Networking

Networking is critical for backend services to communicate with clients and other services. The Berkeley Sockets API is the standard for network communication, and its key system calls include:

*   **`socket()`**: This call creates an endpoint for communication and returns a file descriptor.
*   **`bind()`**: This call assigns a name to a socket.
*   **`listen()`**: This call marks a socket as a passive socket that will be used to accept incoming connection requests.
*   **`accept()`**: This call extracts the first connection request on the queue of pending connections for a listening socket and creates a new connected socket.
*   **`connect()`**: This call establishes a connection to a specified socket.
*   **`send()`** and **`recv()`**: These calls are used to send and receive data over a socket.

### Memory Management

Efficient memory management is crucial for application performance. Important system calls in this category are:

*   **`mmap()`**: This call maps files or devices into memory.
*   **`brk()`**: This call is used to change the data segment size.

### Inter-Process Communication (IPC)

When multiple processes need to communicate with each other on the same machine, the following system calls are useful:

*   **`pipe()`**: This call creates a unidirectional data channel that can be used for inter-process communication.
*   **`shmget()`**: This call allocates a shared memory segment.
