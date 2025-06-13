# 🛡️ Cowrie Honeypot + Fail2Ban + IP Geolocation Visualizer 🌍  

This project deploys a Cowrie honeypot to simulate vulnerable SSH/Telnet servers, integrated with Fail2Ban for real-time IP banning and a Python-based module to visualize attacker IPs on a world map using geolocation services.

---

## 📘 Table of Contents
1. 🔍 Project Overview
2. 🚧 System Requirements
3. 🧑‍💻 Create Dedicated `cowrie` User
4. ⚙️ Step-by-Step Installation
5. 🔐 Configuring Fail2Ban
6. 🌍 IP Geolocation & Mapping
7. 🧪 Testing the Setup
8. 📊 Sample Output
9. 🧠 Why This Matters
10. 👨‍💻 Author

---

## 🔍 Project Overview

Cowrie is a **medium-to-high interaction honeypot** designed to log brute force attacks and shell interaction performed by attackers. This project extends it by:

- Setting up Cowrie to run as a background daemon on port `2222` (instead of standard `22`)
- Enabling **Fail2Ban** to detect repeated failed logins from logs and auto-ban IPs using `iptables`
- Extracting **attacker IP addresses** from Cowrie logs
- Using **geolocation APIs** and `folium` to **map** attacker origins globally

---

## 🚧 System Requirements

| Component        | Required Version         |
|------------------|--------------------------|
| OS               | Kali Linux / Debian      |
| Python           | 3.8+                     |
| Cowrie Honeypot  | Latest GitHub version    |
| Tools            | pip, venv, Fail2Ban      |
| Python Libraries | `requests`, `folium`, `pandas`, `geoip2`, `matplotlib` |

---

## 🧑‍💻 Create Dedicated `cowrie` User

For security best practices, Cowrie should **not** be run as root. Let’s create a dedicated user:

```bash
sudo adduser --disabled-password --gecos "" cowrie
````

📌 **Explanation:**

* `--disabled-password` prevents direct login
* `cowrie` user will own the honeypot directory and run the service safely

Now give this user permission to manage its own environment:

```bash
sudo chown -R cowrie:cowrie /home/cowrie
```

Switch to the user:

```bash
sudo su - cowrie
```

---

## ⚙️ Step-by-Step Installation

### 🐍 Step 1: Install Dependencies

```bash
sudo apt update
sudo apt install git python3-venv python3-pip libssl-dev libffi-dev build-essential fail2ban -y
```

📌 These packages support Cowrie and Fail2Ban. They install compilers, crypto libs, virtualenv, and pip.

---

### 📂 Step 2: Clone Cowrie and Set It Up (As `cowrie` user)

```bash
cd ~
git clone https://github.com/cowrie/cowrie.git
cd cowrie
python3 -m venv cowrie-env
source cowrie-env/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
cp etc/cowrie.cfg.dist etc/cowrie.cfg
```

---

### 🚀 Step 3: Run Cowrie Honeypot

```bash
bin/cowrie start
```

To view logs live:

```bash
tail -f var/log/cowrie/cowrie.log
```

To ensure Cowrie starts at boot (optional):

```bash
sudo cp etc/systemd/cowrie.service /etc/systemd/system/
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable cowrie
sudo systemctl start cowrie
```

---

## 🔐 Configuring Fail2Ban

### 📝 Step 4: Create Filter File

```bash
sudo nano /etc/fail2ban/filter.d/cowrie.conf
```

Add:

```ini
[Definition]
failregex = .*HoneyPotSSHTransport.*authentication failed.*
ignoreregex =
```

---

### 🧱 Step 5: Jail Configuration

```bash
sudo nano /etc/fail2ban/jail.local
```

```ini
[cowrie-ssh]
enabled = true
filter = cowrie
port = 2222
logpath = /home/cowrie/cowrie/var/log/cowrie/cowrie.log
maxretry = 3
bantime = 1h
findtime = 10m
backend = auto
```

Restart Fail2Ban and check:

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status cowrie-ssh
```

---

## 🌍 IP Geolocation & Mapping

### 📦 Step 6: Install Python Libraries

```bash
pip install requests folium pandas geoip2 matplotlib
```

---

### 🧾 Step 7: Script to Generate Map

Create `geolocate_attackers.py`:

```python
import json, folium, requests

log_file = '/home/cowrie/cowrie/var/log/cowrie/cowrie.json'
ip_map = folium.Map(location=[20, 0], zoom_start=2)
ips = set()

with open(log_file) as f:
    for line in f:
        try:
            data = json.loads(line)
            if 'src_ip' in data:
                ips.add(data['src_ip'])
        except:
            continue

for ip in ips:
    try:
        res = requests.get(f"http://ip-api.com/json/{ip}").json()
        if res['status'] == 'success':
            lat, lon = res['lat'], res['lon']
            folium.Marker([lat, lon], popup=f"{ip}").add_to(ip_map)
    except:
        continue

ip_map.save("attack_map.html")
```

Run it:

```bash
python3 geolocate_attackers.py
xdg-open attack_map.html
```

---

## 🧪 Testing the Setup

1. Simulate brute-force attack:

   ```bash
   ssh root@<your_ip> -p 2222
   ```
2. After 3 failed attempts, Fail2Ban will ban the IP.
3. Verify bans:

   ```bash
   sudo fail2ban-client status cowrie-ssh
   ```
4. Visualize the attacks using:

   ```bash
   python3 geolocate_attackers.py
   ```

---

## 📊 Sample Output

* `cowrie.log`: SSH interaction logs
* `cowrie.json`: JSON sessions
* `fail2ban.log`: Ban actions
* `attack_map.html`: Map of attacker IPs globally 🌍

---

## 🧠 Why This Matters

* **Real-time Threat Detection**
* **Automatic Active Response**
* **Geopolitical Insight**
* **Educational Tool for Cybersecurity**

---

## 👨‍💻 Author

**Darsh Chatrani**  
🔗 [LinkedIn](https://linkedin.com/in/darshchatrani)  
📞 Contact: +91 97899 57123

---
