# BreachLab: Ghost - Level 12 → 13

## **Objective**

Utilize an asymmetric private key file to authenticate to a target server, bypassing standard password-based authentication.

## **Why this matters in the real world**

Key-based auth is how every production server on the planet is accessed. If you cannot use a private key you cannot do the job. Not one job. Zero.

## **Investigation & Solution**

1.  **Enumeration:** Started by listing directory contents. Identified two unusual files: `sshkey.private` and `sshkey.private.pub`.

    ```bash
    ls -la
    ```
    
2.  **Key Analysis:** Examined the contents of `sshkey.private`. The header (`-----BEGIN OPENSSH PRIVATE KEY-----`) and format confirmed it was a modern `ed25519` SSH private key. Crucially, I noted the file permissions in the previous `ls -la` output were already securely set to `-rw-------` (600).

    ```bash
    cat sshkey.private
    ```
    
3.  **Authentication & Lateral Movement:** Checked the `ssh` manual to confirm the correct syntax for supplying an identity file. Used the `-i` flag to pass the private key and successfully authenticated as the `ghost13` user.

    ```bash
    ssh --help
    ssh -i sshkey.private ghost13@204.168.229.209 -p 2222
    ```
    
4.  **Payload Extraction:** Once logged into the `ghost13` account, I listed the home directory, identified a `flag` file, and read its contents to capture the password.

    ```bash
    ls -la
    cat flag
    exit
    ```
    

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1167274.svg)](https://asciinema.org/a/1167274)

## **Command Breakdown**

|**Command**|**Flag/Option**|**Purpose**|
|----------------|-------------------------------|-----------------------------|
|`cat`|None|Concatenates and prints the raw contents of a file directly to standard output.|
|`ssh`|`--help`|Displays the short-form usage documentation and available flags for the SSH client.|
|`ssh`|`-i`|Selects a file from which the identity (private key) for public key authentication is read.|
|`ssh`|`-p`|Specifies the port to connect to on the remote host.|

## **Notes & Takeaways**

A critical detail to understand about SSH keys is strict permission enforcement. In this lab, the server generously provided the `sshkey.private` file with `-rw-------` (600) permissions.

In the real world, if you generate or copy a private key and its permissions are too open (e.g., `-rw-r--r--` or 644), the SSH client will actively refuse to use it and throw an "unprotected private key file" error. The SSH protocol considers a readable private key to be compromised by default. Knowing to run `chmod 600 <key_file>` is a mandatory troubleshooting step whenever key authentication fails. Furthermore, `ed25519` is the current cryptographic standard; recognizing its signature over older RSA formats is useful during key audits.

### **Understanding Asymmetric Keys: Public vs. Private**

SSH utilizes an asymmetric cryptographic key pair to handle authentication without transmitting passwords over the network. It is easiest to understand them as a **Lock** and a **Key**.

-   **`sshkey.private.pub` (The Lock):** This is your **Public Key**. It is completely safe to share with anyone. In a production environment, you take this public key and place it on the target server (specifically inside an `authorized_keys` file). It acts as a cryptographic padlock on that specific account.
    
-   **`sshkey.private` (The Key):** This is your **Private Key**. It is the ultimate secret. You keep this strictly on your local machine and never share it. When you attempt to log in, your SSH client uses this private key to mathematically prove to the server that you own the padlock. If the math checks out, you are granted access.
    

**The Golden Rule:** You can distribute your public key to a thousand servers, but if your private key is ever exposed or stolen, the attacker instantly gains access to every single one of those servers.
