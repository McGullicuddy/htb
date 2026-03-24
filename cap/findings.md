# Cap Findings

## Exploit Chain
1. Enumerated the web dashboard on `http://10.129.15.193/` and identified the capture workflow under `/capture`.
2. Observed that `/capture` redirected to `/data/1`, indicating captures were stored under numeric identifiers.
3. Confirmed an IDOR by requesting `/data/0`, which exposed an older capture owned by another workflow context.
4. Downloaded `/download/0` and reviewed the PCAP locally.
5. Recovered cleartext FTP credentials from the capture:
   - `nathan:Buck3tH4TF0RM3!`
6. Reused the same credentials over SSH to obtain a shell as `nathan`.
7. Enumerated Linux capabilities and found `/usr/bin/python3.8` with `cap_setuid+eip`.
8. Used Python to set UID `0` and read the root flag.

## Evidence
- Target IP: `10.129.15.193`
- Historical capture artifact: `cap/artifacts/0.pcap`
- User flag: `debcef8fb8c93dbb7ad2109700c408fe`
- Root flag: `2422ffc556f6876fbc0384eb163eae41`

## Notes
- The initial foothold came from broken access control on capture IDs.
- The privilege-escalation path did not require kernel exploitation, sudo abuse, or password cracking.
