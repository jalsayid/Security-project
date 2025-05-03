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


#### Search 2: Successful Login Detection (Victim Side)

![Successful Logins](https://github.com/jalsayid/Security-project/blob/eb4658578cd19cdc0129b608324c9d82e7598956/Phase2/screenshots/11.png)




## Conclusion


This project successfully demonstrated:
1. How to set up a controlled environment for ethical hacking experiments
2. The mechanics of an SSH brute force attack
3. Effective log collection and centralization strategies
4. Advanced SIEM visualization and analysis techniques
5. The importance of comprehensive security monitoring from multiple perspectives

The integration of logs from both the attacker and victim perspectives provides a holistic view of security events that would be valuable in real-world security operations centers.
