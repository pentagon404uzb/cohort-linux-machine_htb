# Cohort HTB Challenge Walkthrough

## Overview
This repository contains a comprehensive walkthrough for the Cohort HTB (Hack The Box) machine challenge. The walkthrough documents the process of enumerating, exploiting, and gaining root access to the target machine.

## Challenge Details
- **Machine**: Cohort
- **Difficulty**: Medium (3.5/5 rating)
- **XP Reward**: 585

## Attack Vectors Covered
- **Web Enumeration** - Finding SSRF vulnerabilities
- **Bypass Techniques** - URL filtering bypass using `http://127.1`
- **Command Injection** - Exploiting SSRF for internal enumeration
- **WebSocket Exploitation** - Accessing internal Marimo notebook
- **Remote Code Execution (RCE)** - Achieving shell access
- **Privilege Escalation** - Using `linpeas.sh` and exploit binaries

## Walkthrough Steps

### 1. Port Enumeration
```
Open Ports: 22 (SSH), 80 (HTTP), 443 (HTTPS)
```

### 2. SSRF Discovery
- Found an interesting endpoint leading to SSRF vulnerability
- Tested common localhost payloads (`127.0.0.1`, `0.0.0.0`, `localhost`)
- Successfully bypassed with `http://127.1` (short form of 127.0.0.1)

### 3. Internal Service Enumeration
Accessed internal status endpoint:
```
http://127.1/status
```

Discovered internal services:
- **marketing**: cohort.htb
- **insights-api**: 127.0.0.1:5000
- **notebooks**: nb-1be3782a8afd3ad5.cohort.htb (internal Marimo notebook on 127.0.0.1:8888)

### 4. Alternative Bypass Techniques
If `127.1` is filtered:
- `http://127.0.0.1@127.1/status`
- `http://0x7f000001/status`
- `http://127.1:80/status`

### 5. WebSocket RCE Exploitation
```python
import ssl, threading, websocket

host = "nb-1be3782a8afd3ad5.cohort.htb"
ws_url = "wss://<your-target-ip>/terminal/ws"

pentagon404 = websocket.create_connection(
    ws_url,
    host=host,
    origin="https://" + host,
    sslopt={"cert_reqs": ssl.CERT_NONE},
    timeout=5
)
```

### 6. Privilege Escalation
1. Uploaded `linpeas.sh` using `curl`
2. Made executable with `chmod 4777 linpeas.sh`
3. Ran enumeration with `./linpeas.sh`
4. Used `Pack2TheRoot` exploit tool:
   - Downloaded exploit binary via `curl`
   - Made executable
   - Executed with `nohup` for persistence
   - Retrieved SUID bash shell

### 7. Root Access
```
/tmp/.suid_bash -p -c 'id; cat /root/root.txt'
```

## Tools Used
- **WebSocket** - For RCE exploitation
- **linpeas.sh** - Linux privilege escalation enumeration
- **Pack2TheRoot** - Exploit tool for privilege escalation
- **curl** - File transfer and HTTP requests

## Repository Structure
```
.
├── README.md
├── exploit/
│   └── websocket_rce.py
├── scripts/
│   ├── linpeas.sh
│   └── exploit.bin
└── walkthrough/
    └── Cohort HTB Challenge WriteUp.pdf
```

## Key Learnings
1. SSRF can be bypassed using alternative IP representations
2. Internal services can expose sensitive information
3. WebSocket connections can provide shell access
4. Proper privilege escalation enumeration is crucial
5. SUID binaries can be abused for root access

## Disclaimer
This walkthrough is for educational purposes only. Use this knowledge responsibly and only on systems you have permission to test.

## Author
[pentagonuzb404/cohort-linux-machine]

## License
This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments
- Hack The Box for creating the challenge
- The security community for developing enumeration and exploitation tools
