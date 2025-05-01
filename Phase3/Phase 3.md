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

📷 **Image 1:** Fail2Ban Installation Screenshot

---

### 2. Configure SSH Jail

To avoid having configuration changes overwritten during future updates, copy the default configuration file to a local file:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

📷 **Image 2:** Copying the configuration file

Open the copied file to edit:

```bash
sudo nano /etc/fail2ban/jail.local
```

📷 **Image 3:** Opening the configuration file

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

📷 **Image 4:** SSH jail configuration

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

📷 **Image 4 (continued):** Service restarted successfully

---

### 4. Verify Fail2Ban Status (Before Attack)

To check if the Fail2Ban jail for SSH is active and monitoring:

```bash
sudo fail2ban-client status sshd
```

📷 **Image 5:** SSH jail status showing active monitoring

---

## 🔎 Testing & Validation

### 🚫 Before Defense – No Protection Enabled

An SSH brute-force attack was executed before implementing Fail2Ban. The system did not block the attacker, and repeated failed login attempts were allowed without limitation.

📷 **Image 6:** Screenshot showing successful brute-force attempts

---

### ✅ After Defense – Fail2Ban Enabled

After configuring Fail2Ban, the same brute-force attack was attempted. This time, the attacker’s IP was automatically banned after failed login attempts.

📷 **Image 7:** Failed login attempts  
📷 **Image 8:** IP address successfully banned and blocked
