# Author: Yahaya Meddy

# Description
We intercepted a suspicious file from a system, but instead of the password itself, it only contains its SHA-1 hash. Using OSINT techniques, you are provided with personal details about the target. Your task is to leverage this information to generate a custom password list and recover the original password by matching its hash.

Download the following files:

userinfo: Contains the personal details.

hash: Contains the SHA-1 hash of the password.

check_password: Script to test passwords against the hash.

# Hints
1. CUPP is a Python tool for generating custom wordlists from personal data.

# Steps
1. Download the provided files.
2. Check the function of check_password.py, it needs 2 files as arguments to run the script which are hash.txt and passwords.txt which is the next step that I have to do to create a wordlist from the userinfo.txt

```bash
#!/usr/bin/env python3
import hashlib

HASH_FILE = "hash.txt"
WORDLIST_FILE = "passwords.txt" # wordlist that was generated using CUPP

def load_hash():
    with open(HASH_FILE, "r") as f:
        return f.read().strip()

def crack_password(target_hash):
    with open(WORDLIST_FILE, "r", encoding="utf-8", errors="ignore") as f:
        for password in f:
            password = password.strip()
            if hashlib.sha1(password.encode()).hexdigest() == target_hash:
                return password
    return None

if __name__ == "__main__":
    target_hash = load_hash()
    result = crack_password(target_hash)
    if result:
        print(f"Password found: picoCTF{{{result}}}")
    else:
        print("No match found.")
```

3. Clone the CUPP repo and run the script `python3 cupp.py -i`, the -i flag is used for interactive page to fill in the info.
4. Insert the user's info with no other special options like adding a random numbers etc.
5. Then rename the output file to match the check_password.py script.
6. Run `python3 check_password.py hash.txt passwords.txt` and Voila.

Answer:  picoCTF{Aj_15901990}
