# BreachLab: Ghost - Level 13 → 14

## **Objective**

Establish a raw TCP connection to a local network service and transmit the current credentials via standard input to retrieve the next flag.

## **Why this matters in the real world**

Every production service talks to other services over TCP. Knowing how to hand-craft a request without a client library is the difference between a pentester who finds issues and one who only runs tools.

## **Investigation & Solution**

1.  **Syntax Verification:** The objective required sending the current password (`K3y_N0t_P4ss`) to a local service listening on port 30000. I initiated the interaction by checking the `nc` (Netcat) usage documentation to confirm the correct connection syntax.

    ```bash
    nc --help
    ```
    
2.  **Data Transmission & Extraction:** I used `echo` to print the current password and piped (`|`) that standard output directly into Netcat. After an initial syntax correction (removing the `-p` flag, which specifies a local source port rather than a destination port), the TCP connection was established, the string was sent, and the service returned the next flag.

    ```bash
    echo "K3y_N0t_P4ss" | nc localhost 30000
    exit
    ```
    

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1172320.svg)](https://asciinema.org/a/1172320)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`echo`|None|Prints the specified string to standard output.|
| &#124; (Pipe)|None|Takes the standard output of the command on the left and passes it as standard input to the command on the right.|
|`nc`|None|Netcat: a versatile utility for reading from and writing to network connections using TCP or UDP.|

## **Notes & Takeaways**

A common syntax trap with Netcat is the misuse of the `-p` flag. When acting as a client connecting to a remote service, the destination port is simply appended as a positional argument (e.g., `nc localhost 30000`). The `-p` flag is reserved for specifying the _source_ port you are binding to, typically used when setting up a listener (`nc -lvnp 4444`).

Piping `echo` directly into `nc` is a fundamental technique for interacting with raw sockets and custom services when standard web clients like `curl` or browser interfaces are unavailable.
