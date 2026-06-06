# BreachLab: Ghost - Level 6 → Level 7

## **Objective**

Extract credentials stored in memory as environment variables and decode the payload to retrieve the flag.

## **Why this matters in the real world**

Credential extraction. Environment variables are how secrets leak into process lists, crash logs, and CI pipelines every single day. *Obfuscating them with basic encoding does not secure them.*

## **Investigation & Solution**

1.  **Initial Enumeration:** Started by checking the current directory with `ls -la`. No obvious files or hidden artifacts were present to indicate stored credentials.
    
    ```bash
    ls -la
    ```

2.  **Environment Interrogation:** The briefing noted that KAEL stopped writing secrets to disk and assumed "the shell would forget them." This immediately points to memory or shell configuration. Dumped the current environment variables to inspect the session state. After reviewing the environment variables loaded into the current shell session, a suspicious variable named `API_DIGEST` was identified containing a string that appeared to be Base64 encoded.
    
    ```bash
    env

    echo $API_DIGEST
    ```
    
3.  **Decoding & Extraction:** To reveal the plaintext credential, the variable's contents were printed and piped directly into the base64 utility.
    
    ```bash
    echo $API_DIGEST | base64 -d
    exit
    ```
    
> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1135068.svg)](https://asciinema.org/a/1135068)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|ls|-la|Lists all files and directories, including hidden ones, in a detailed long format.|
|`echo`|None|Prints the value of a specified shell variable to standard output.|
|`env`|None|Prints a list of the current environment variables exported to the shell session.|
|`base64`|`-d`|Decodes the standard input from Base64 format back to plaintext.|
|`exit`|None|Closes the current shell session.|

## **Notes & Takeaways**

Relying on environment variables to store secrets is a massive operational failure, yet it happens constantly in cloud environments and CI/CD pipelines. The target in this scenario attempted to hide the credential by Base64 encoding it within the `$API_DIGEST` variable. Encoding is not encryption; it is merely obfuscation. Recognizing the Base64 signature (alphanumeric characters often ending in `=`) and decoding it on the fly is a fundamental analysis skill.
