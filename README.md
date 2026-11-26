# Pickle Rick — TryHackMe Walkthrough

A complete penetration testing report and walkthrough for the **Pickle Rick** vulnerable machine on TryHackMe.

---

## 1. Target Information

- **Room Name:** Pickle Rick
- **Room URL:** https://tryhackme.com/room/picklerick
- **Objective:** Exploit the web application and escalate privileges to retrieve three hidden ingredient flags.

---

## 2. Methodology

This assessment followed a standard four-phase penetration testing methodology:

1. **Reconnaissance & Enumeration:** Identifying open ports, services, hidden web directories, and sensitive information leaked in page source code and `robots.txt`.
2. **Initial Access (Exploitation):** Authenticating into the administrative portal using discovered credentials to gain remote command execution.
3. **Post-Exploitation & Filter Bypass:** Navigating the filesystem and bypassing command blacklists (`cat` restriction) using alternative Linux file-reading utilities.
4. **Privilege Escalation:** Auditing `sudo` misconfigurations (`NOPASSWD: ALL`) to elevate privileges from `www-data` to `root` and capture the final flag.

---

## 3. Step-by-Step Walkthrough & Evidence

### Phase 1: Reconnaissance & Enumeration

1. **Port Scanning with Nmap**
   
   Run a service and default script scan against the target to identify exposed services:
   ```bash
   nmap -sC -sV 10.130.165.80
   ```
   **Finding:** Port 22 (SSH) and Port 80 (Apache HTTP) are open.
   
   <img width="1009" height="553" alt="Screenshot 2026-11-26 145409" src="https://github.com/user-attachments/assets/1c6f97f8-fed6-47ad-b57a-962f7a5827d5" />

2. **Web Application & Source Code Inspection**
   
    Open `http:/10.130.165.80` in a browser and inspect the page source.
   **Finding:** An HTML comment reveals the username: `R1ckRul3s`

   <img width="494" height="159" alt="Screenshot 2026-11-26 145800" src="https://github.com/user-attachments/assets/355c2012-fc71-4cdd-9d11-3198a1d38a8d" />

4. **Directory Enumeration**

   Use Gobuster to discover hidden pages on the web server:
   ```bash
   gobuster dir -u http://10.130.165.80 -w /usr/share/wordlists/dirb/common.txt
   ```
    **Finding:** Discovered `robots.txt`, `/login.php` and `/portal.php`.
 
     <img width="709" height="634" alt="Screenshot 2026-11-26 145948" src="https://github.com/user-attachments/assets/e5dbd6af-5554-41a7-bc53-832aa2c6bf4d" />

5. **Checking robots.txt**
   
   Navigate to `http://10.130.165.80/robots.txt`.
   
   **Finding:** Contains the string `Wubbalubbadubdub`, which serves as the password.

   <img width="785" height="121" alt="Screenshot 2026-11-26 150058" src="https://github.com/user-attachments/assets/79e1b875-604c-4c3e-b650-52d4b627c2a8" />


### Phase 2: Initial Access

1. Navigate to `http://10.130.165.80/login.php`.
2. Log in using the enumerated credentials:
   - **Username:** `R1ckRul3s`
   - **Password:** `Wubbalubbadubdub`
3. After logging in, the browser redirects to `/portal.php`, exposing a web-based Command Panel running as the `www-data` user using the command:
   ```bash
   whoami
   ```

   <img width="931" height="80" alt="Screenshot 2026-11-26 155026" src="https://github.com/user-attachments/assets/eef7e5d4-258d-42dc-94b4-0a3513ebdef6" />

### Phase 3: Retrieving Ingredient 1 & Ingredient 2

1. **First Ingredient**
   - List the files in the current working directory (/var/www/html):
   
   ```bash
   ls -la
   ```
   **Finding:** `Sup3rS3cretPickl3Ingred.txt` and `clue.txt` files.

   <img width="1124" height="608" alt="Screenshot 2026-11-26 150329" src="https://github.com/user-attachments/assets/5eeac40d-1f38-4bab-b370-d98fcc5c638a" />

    - Standard command such as `cat` are blocked by the application's command filter. Use `less` to bypass this restriction:
    
    ```bash
    less Sup3rS3cretPickl3Ingred.txt
    ```
    <img width="1127" height="108" alt="Screenshot 2026-11-26 150442" src="https://github.com/user-attachments/assets/e3aeb2fa-72db-463d-9169-489e7f959e2e" />

2. **Second Ingredient**
   - Read the contents of `clue.txt` for second ingredient's hint:
     
     ```bash
     less clue.txt
     ```

    <img width="1732" height="89" alt="image" src="https://github.com/user-attachments/assets/cd0a592f-03c7-4130-bfb4-3171e717e6d2" />

   - Enumerate user home directories and read the file inside `/home/rick`(use double quotes to wrap the filename to handle the space):
   
     ```bash
     ls -la /home/rick
     less "/home/rick/second ingredient"
     ```
   
     <img width="1101" height="96" alt="Screenshot 2026-11-26 150729" src="https://github.com/user-attachments/assets/6bf8ea93-1fd0-4d8c-b59c-b4d9bb4fd891" />


### Phase 4: Privilege Escalation & Final Ingredient

1. **Checking sudo Permissions**
   Check what privileges the `www-data` user holds:
   
   ```bash
   sudo -l
   ```
   **Finding:** `(ALL) NOPASSWD: ALL` means the `www-data` user can execute any commands as root without a password.

   <img width="1091" height="194" alt="Screenshot 2026-11-26 152629" src="https://github.com/user-attachments/assets/a777bae9-07ed-4a05-b20e-501367ea42b7" />

3. **Third Ingredient**
   List and read the final flag inside `/root` using sudo:
   
   ```bash
   sudo ls -la /root
   sudo less /root/3rd.txt
   ```
   <img width="1107" height="85" alt="Screenshot 2026-11-26 150832" src="https://github.com/user-attachments/assets/e4b584f8-a04f-40d5-b808-faf77dc8c4a0" />
