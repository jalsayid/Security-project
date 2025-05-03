# Phase 2: SIEM Dashboard Analysis

This project demonstrates a controlled SSH brute force attack and subsequent analysis using Splunk SIEM and comprehensive log analysis through Splunk's SIEM capabilities.

### 1. Environment Setup

In this phase, we set up two virtual machines (VMs) using VirtualBox:
- Kali Linux (Attacker)
- Metasploitable3 (Victim)

Network Configuration for Both VMs:
- Adapter 1: NAT Network (to allow internet access)
- Adapter 2: Host-Only Adapter (for communication between VMs)

### 2. Verify VM Communication

![Network Verification](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/1.png)

Check the IP addresses of both machines to ensure they are connected:
- On Kali Linux, run:

```bash
ifconfig
```

output: 192.168.56.107 (Kali IP)

- On Metasploitable3, run:

```bash
ifconfig
```

If the IP is not in the 192.168.x.x range, reset the DHCP lease:

```bash
sudo dhclient -r eth1
sudo dhclient eth1
ifconfig
```

output: 192.168.56.106 (Metasploitable3 IP)

To test connectivity, ping from each machine:

```bash
ping 192.168.56.104  # From Metasploitable3
ping 192.168.56.105  # From Kali Linux
```
![test connectivity](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/2.png)
### 3. Discover Open Ports on the Victim

Run the following command on Kali to scan for open ports:

```bash
sudo nmap -sS -sV 192.168.56.105
```
![Port Scanning](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/3.png)



This reveals that port 22 (SSH) is open, indicating that SSH login is available for exploitation.


## SIEM Log Analysis with Splunk

This phase covers the setup and configuration of Splunk for comprehensive log collection and analysis.

![Splunk Overview](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/4.png)

### 1. Install Splunk Enterprise on Kali

Download and install Splunk:

```bash
wget -O splunk-9.3.2.deb https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb
sudo dpkg -i splunk-9.3.2.deb
sudo apt --fix-broken install
sudo /opt/splunk/bin/splunk start --accept-license
```

Open Splunk's web interface at http://localhost:8000.

### 2. Install Splunk Universal Forwarder on Metasploitable3

Download and install the Splunk Universal Forwarder:

```bash
wget -O splunkforwarder-9.4.1.deb "https://download.splunk.com/products/universalforwarder/releases/9.4.1/linux/splunkforwarder-9.4.1-e3bdab203ac8-linux-amd64.deb"
sudo dpkg -i splunkforwarder-9.4.1.deb
sudo apt --fix-broken install
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

### 3. Enable Receiving Port 9997 in Splunk

![Receiving Configuration](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/5.png)


In Splunk, navigate to Settings > Forwarding and Receiving, then click Add new receiving port and set the port to 9997.

### 4. Configure the Forwarder to Send Logs to Splunk

![Forwarder Configuration](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/6.png)

On Metasploitable3, add the Splunk server (Kali IP) as the forward destination:

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.56.104:9997
sudo /opt/splunkforwarder/bin/splunk restart
```

To verify the connection:

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

### 5. Monitor SSH Log Files

![Log Monitoring](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/7.png)


Add /var/log/auth.log to Splunk's monitoring list to track SSH login events:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
```

### 6. Run the Brute Force Attack

![Attack Execution](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/8.png)

From Kali, execute the brute force script:

```bash
cd ssh-brute
./script.sh
```

The Hydra output will indicate the success of the attack.

### 7. Analyze Logs in Splunk

![Splunk Analysis Overview](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/9.png)

After the attack, use Splunk's search and visualization tools to analyze the attack. Below are the key queries and visualizations:

#### Search 1: Failed SSH Logins (Victim Side)

![Failed Logins](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/10.png)

Use the following query to identify failed login attempts:

```
index=* sourcetype=syslog "Failed password" | rex "user (?<user>\w+).*from (?<ip>\d+\.\d+\.\d+\.\d+)" | stats count by ip, user
```

This will generate a bar chart showing the number of failed attempts per user and IP.

#### Search 2: Successful Login Detection (Victim Side)

![Successful Logins](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/11.png)

To identify when the attacker successfully logged in, use:

```
index=* sourcetype=syslog "Accepted password" | rex "for (?<user>\w+) from (?<ip>\d+\.\d+\.\d+\.\d+)" | table _time, user, ip
```

This query will show the exact timestamp and the source IP when the valid credentials were accepted.

#### Search 3: Attacker's Log Analysis (Kali Side)

![Attacker Logs](images/attacker_logs.png)
*Insert attacker log analysis screenshot here*

Hydra logs from Kali can be analyzed with:

```
index=* sourcetype="attacker_log"
| rex "host:\s(?<ip>\d+\.\d+\.\d+\.\d+).*login:\s(?<user>\w+).*password:\s(?<pass>\w+)"
| table ip, user, pass
```

This will display the extracted credentials used by the attacker.

## Phase 3: SIEM Dashboard Analysis

### SIEM Setup: Integration of Logs

![SIEM Integration](images/siem_integration.png)

In this phase, we integrated logs from both victim and attacker environments into our Splunk SIEM. This provides a comprehensive view of the attack from both perspectives and enables more robust analysis. The integration involved:

- Configuring the Splunk Universal Forwarder on the victim machine
- Setting up custom log monitoring for attack tools on the attacker machine
- Establishing a centralized dashboard for all security events

### Log Visualization and Analysis

![Attack Visualization](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/9.png)
*Insert screenshot of attack visualization and analysis dashboard*

Our visualization approach focused on:
- Timeline analysis of the attack progression
- Identification of attack patterns and brute force signature
- Real-time monitoring capabilities for rapid detection
- Correlation between source IPs and targeted accounts


## Log Analysis Summary

![Summary Dashboard](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/9.png)
*Insert screenshot of summary dashboard*

- **Failed Login Attempts**: Over 65 failed attempts were recorded before a successful login was made.
- **Successful Login**: The successful login occurred with the credentials vagrant:vagrant.
- **Source IP**: All failed and successful login attempts originated from Kali's IP (192.168.56.104).
- **Hydra Log Analysis**: The credentials used in the attack were captured in both Hydra logs and SSH logs, providing complete visibility of the attack.
- **Attack Duration**: The entire brute force process took approximately [X] minutes from start to finish.
- **Detection Time**: Our SIEM alerts were triggered within [Y] seconds of the attack initiation.

## Final Visualizations

Using Splunk, we created the following visualizations to monitor and analyze the attack:
- Bar Chart: Failed login attempts by username and IP
- Timechart: Number of failed attempts over time
- Table: Extracted credentials from Hydra
- Bar Chart: Timestamp of successful logins
- Heatmap: Intensity of login attempts over time
- Pie Chart: Distribution of targeted accounts
- Gauge: Success rate of the brute force attack

This demonstrates how Splunk can be used to monitor, analyze, and visualize a brute-force SSH attack in real-time.

## Conclusion


This project successfully demonstrated:
1. How to set up a controlled environment for ethical hacking experiments
2. The mechanics of an SSH brute force attack
3. Effective log collection and centralization strategies
4. Advanced SIEM visualization and analysis techniques
5. The importance of comprehensive security monitoring from multiple perspectives

The integration of logs from both the attacker and victim perspectives provides a holistic view of security events that would be valuable in real-world security operations centers.
