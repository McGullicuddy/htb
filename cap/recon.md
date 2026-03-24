# Cap Recon

## Current State
- Target: `10.129.15.193`
- Open ports:
  - `21/tcp` FTP (`vsftpd 3.0.3`)
  - `22/tcp` SSH (`OpenSSH 8.2p1 Ubuntu 4ubuntu0.2`)
  - `80/tcp` HTTP (`gunicorn`, title `Security Dashboard`)
- Confirmed web username shown in UI: `Nathan`

## Assessment
- The web service on `80/tcp` is the primary initial attack surface.
- `/capture` creates a new capture and redirects to `/data/<id>`.
- `/data/<id>` exposes historical capture records without access control.
- `/download/<id>` allows direct retrieval of the associated PCAP file.
- Historical capture `0` is accessible and contains cleartext credentials.

## Confirmed Findings
- `GET /capture` returned `302` to `/data/1` after roughly 5 seconds.
- `GET /data/0` returned a valid capture summary with `72` packets and a `Download` button for `/download/0`.
- Downloaded artifact: `cap/artifacts/0.pcap`
- Local PCAP review exposed FTP credentials:
  - Username: `nathan`
  - Password: `Buck3tH4TF0RM3!`
- The same credentials may be reusable for SSH on `22/tcp`.
- SSH login as `nathan` succeeded with the recovered password.
- Local capability enumeration showed:
  - `/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip`
- Privilege escalation via Python `os.setuid(0)` succeeded.
- User flag: `debcef8fb8c93dbb7ad2109700c408fe`
- Root flag: `2422ffc556f6876fbc0384eb163eae41`

## Next Proposed Action
- Clean up notes and, if needed, capture a concise exploit chain summary for future reference.
