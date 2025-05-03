# Phase 3: Defensive Strategy Proposal – Securing SSH with Fail2Ban

## 📌 Objective

This phase focuses on protecting the SSH service from brute-force attacks by implementing a defense mechanism. Moreover, the effectiveness of the chosen defense will be tested and validated by repeating the attack after the mechanism is implemented.

---

## 🛡️ Selected Defense Mechanism: Fail2Ban

**Tool Used:** Fail2Ban

Fail2Ban is a security tool that monitors log files for suspicious activity. It helps prevent automated attacks, like SSH brute-force login attempts, by banning IP addresses after a specified number of failed login attempts. Once banned, the attacker’s IP is blocked from further access for a defined period of time.

---

## ⚙️ Implementation Steps

### 1. Install Fail2Ban

Before installing Fail2Ban, update the system to ensure all dependencies are up-to-date.

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

Then, install Fail2Ban:

```bash
sudo apt install fail2ban
```

📷 **Screenshot:** Fail2Ban Installation Screenshot

![Fail2Ban Installation Screenshot](https://github.com/jalsayid/Security-project/blob/a8eee4dd43cdfb4b6324600158eaad5f88705391/Phase3/screenshots/1-%20install%20Fail2ban.png)


---

### 2. Configure SSH Jail

To avoid having configuration changes overwritten during future updates, copy the default configuration file to a local file:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

📷 **Screenshot:** Copying the configuration file

![Copying the configuration file](https://github.com/jalsayid/Security-project/blob/a8eee4dd43cdfb4b6324600158eaad5f88705391/Phase3/screenshots/2-%20create%20a%20local%20copy%20of%20the%20configuration%20file.png)


Open the copied file to edit:

```bash
sudo nano /etc/fail2ban/jail.local
```

📷 **Screenshot:** Opening the configuration file

![Opening the configuration file](https://github.com/jalsayid/Security-project/blob/a8eee4dd43cdfb4b6324600158eaad5f88705391/Phase3/screenshots/3-%20Opening%20the%20configuration%20file.png)


Scroll to the `[sshd]` section and update the settings as follows:

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
```

📷 **Screenshot:** SSH jail configuration

![SSH jail configuration](https://github.com/jalsayid/Security-project/blob/a8eee4dd43cdfb4b6324600158eaad5f88705391/Phase3/screenshots/4-%20SSH%20jail%20configuration.png)


### Explanation of Key Settings:
- `enabled`: Activates SSH protection
- `filter`: Uses the sshd filter to detect failed attempts
- `port`: Monitors the SSH port
- `logpath`: Path to the SSH authentication log file
- `maxretry`: Number of failed login attempts before banning the IP
- `bantime`: Duration of the ban in seconds (600s = 10 minutes)

---

### 3. Restart and Enable Fail2Ban

Restart the service to apply the configuration changes:

```bash
sudo service fail2ban restart
```

📷 **Screenshot:** Service restarted successfully

![Service restarted successfully](https://github.com/jalsayid/Security-project/blob/10d0b3024f0da7590070cb69067245178786bce5/Phase3/screenshots/5-%20Service%20restarted%20successfully.png)


---

### 4. Verify Fail2Ban Status (Before Attack)

To check if the Fail2Ban jail for SSH is active and monitoring:

```bash
sudo fail2ban-client status sshd
```

📷 **Screenshot:** SSH jail status showing active monitoring

![SSH jail status showing active monitoring](https://github.com/jalsayid/Security-project/blob/bb7a2de3cb2df65feeef8bff98849766469cc07a/Phase3/screenshots/6-%20Verify%20Fail2Ban%20Status%20before%20the%20attack.png)


---

## 🔎 Testing & Validation

### 🚫 Before Defense – No Protection Enabled

An SSH brute-force attack was executed before implementing Fail2Ban. The system did not block the attacker, and repeated failed login attempts were allowed without limitation.

📷 **Screenshot:** successful brute-force attempts (Before implementing the defense)

![successful brute-force attempts](https://github.com/jalsayid/Security-project/blob/b978ffc84ec3268c87ae3c92f3e245b3f5744643/Phase3/screenshots/7-%20successful%20brute-force%20attempts%20(before%20defense).png)


---

### ✅ After Defense – Fail2Ban Enabled

After configuring Fail2Ban, the same brute-force attack was attempted. This time, the attacker’s IP was automatically banned after failed login attempts.

📷 **Screenshot:** Failed login attempts  

![Failed login attempts](https://github.com/jalsayid/Security-project/blob/787d80aac202830d445e59588daca8667d176aec/Phase3/screenshots/8%20-%20Failed%20login%20attempts%20(After%20defense).png)

📷 **Screenshot:** IP address successfully banned and blocked

![IP address successfully banned and blocked](https://github.com/jalsayid/Security-project/blob/a1d24e013f1aa14396684499ca47376491c6efad/Phase3/screenshots/9-%20SSH%20jail%20status%20showing%20IP%20address%20successfully%20banned%20and%20blocked.png)

