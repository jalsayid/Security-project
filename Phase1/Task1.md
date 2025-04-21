# Phase 1: Setup and Compromise the SSH Service

## Task 1.1: Compromise the SSH Service Using Metasploit

---

## 🎯 Objective
The goal of this task is to **gain unauthorized SSH access** to the victim machine (Metasploitable3) by performing a **brute-force attack** using **Metasploit**. We'll use the Metasploit module `auxiliary/scanner/ssh/ssh_login` to discover valid login credentials.

---

## 🖥️ Environment Setup

- **Attacker Machine**: Kali Linux 
- **Victim Machine**: Metasploitable3 
- **Target Service**: SSH
- **Target IP Address**: `192.168.56.102`
- **SSH Port**: 22 (default)

---

## 🔹 Step 1: Identify Target IP Address

On the victim machine (Metasploitable3), run the following command to get the IP address:
```bash
ifconfig
```


📸 Screenshot:

![Victim IP Address](https://github.com/jalsayid/Security-project/blob/dc3eb3a50f16f591446ceccfcf2ff306de2c5ca3/Phase1/screenshots/ip%20address%20of%20metasploitable%203%20(1).png)

---

## 🔹 Step 2: Test Connectivity

On the attacker machine (Kali Linux), we test if the victim machine is reachable over the network. This is done using the `ping` command to ensure that packets can be sent and received from the target IP.

```bash
ping 192.168.56.102
```


📸 Screenshot:

![test connectivity](https://github.com/jalsayid/Security-project/blob/983fe310f387ed02dc4abba13caba27e241a72da/Phase1/screenshots/testing%20VM%20connectivity%20from%20attacker%20(kali).png)

---

## 🔹 Step 3: Create Username and Password Files

Before running the brute-force attack, we need to prepare two text files:
- One containing possible usernames.
- One containing possible passwords.

These files will be used by Metasploit's `ssh_login` module.

Example content for `username` file:
```bash
msfadmin
admin
root
user
test
postgres
ubuntu
kali
vagrant
guest
oracle
administrator
ftp
support
webadmin
sysadmin
backup
mysql
pi
john
```

Example content for `password` file:
```bash
msfadmin
admin
123456
password
12345
123456789
12345678
1234
1234567
1234567890
admin123
root
toor
kali
guest
letmein
qwerty
abc123
111111
123123
vagrant
welcome
iloveyou
password1
123qwe
1q2w3e4r
000000
passw0rd
dragon
monkey
loveme
qwertyuiop
```


📸 Screenshot:

![test connectivity](https://github.com/jalsayid/Security-project/blob/fc2e81dd5e1a870890534e157783733ea46e9bea/Phase1/screenshots/files.png)

---

## 🔹 Step 4: Perform a Basic Nmap Scan

To check which ports are open on the victim machine, we start with a basic Nmap scan from the attacker machine (Kali Linux). This helps us discover services like SSH running on the target.

```bash
nmap 192.168.56.102
```

📸 Screenshot:

![test connectivity](https://github.com/jalsayid/Security-project/blob/83631ae4769ad8e04f964d0a4d6c37bc944ce80a/Phase1/screenshots/nmap.png)

---

## 🔹 Step 5: Perform a Detailed Nmap Scan with `-A`

After confirming that port 22 (SSH) is open using a basic scan, we run a detailed Nmap scan to gather more information about the services and operating system running on the victim machine.

This helps confirm the SSH version and identify the system fingerprint.

```bash
nmap -A 192.168.56.102
```

📸 Screenshot:

![test connectivity](https://github.com/jalsayid/Security-project/blob/83631ae4769ad8e04f964d0a4d6c37bc944ce80a/Phase1/screenshots/nmap%20-a.png
)






