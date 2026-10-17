# BSides CTF - October 2026

The objective of this exercise is to document the steps and investigative techniques used to capture flags during the BSidesNYC CTF 2026.

This repository contains my personal write-ups for **Linux Forensics challenges** focused on investigating a compromised Linux server using a provided filesystem dump.

The challenges involved several digital forensics techniques, including:

* **Filesystem Analysis:** Extracting and examining a Linux filesystem archive to identify relevant files and directories.
* **Log Analysis:** Investigating Linux authentication and system logs to identify suspicious activity.
* **Software Installation Analysis:** Examining APT package management logs to identify software installed on the server.

Each write-up walks through my approach to solving the challenge, including how I interpreted the clues, examined forensic artifacts, executed Linux commands, and analyzed the evidence to identify the correct flag.

Disclaimer: These write-ups are intended strictly for educational purposes and document forensic investigations conducted within an authorized CTF environment.


## Linux Forensics Challenges
### CTF Challenge #1 - No SSH key file?

**CTF objective:** What was the IP address of the first SSH login?
When someone (legitimate employee, system administrator, or an attacker.) connects to the organization's Linux server using SSH, the operating system may record information about the connection, including the username, IP address, and login time. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="255" height="339" alt="image" src="https://github.com/user-attachments/assets/494a6f42-bff8-4182-ba53-430811486231" />

1) Within your Linux system, download the **filesystem.tar.gz** by clicking on the file within the challenge:

2) Verify the downloaded file by running:

  ```javascript
  ls ~/Downloads
  ```

3) You should see **filesystem.tar.gz** among the listed files.
4) Verify the downloaded file by running:

  ```javascript
  file ~/Downloads/filesystem.tar.gz
  ```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="784" height="204" alt="image" src="https://github.com/user-attachments/assets/bbd86dad-bbcc-472f-bb01-c92fb9d998a1" />

5) This confirms that the file is a **gzip-compressed archive**.

#### Create a folder for our investigation.
6) Before extracting the files, create a separate folder to keep the forensic evidence organized. Run:

  ```javascript
  mkdir ~/forensics
  ```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="759" height="222" alt="image" src="https://github.com/user-attachments/assets/c7f2c3a9-fc4c-4247-b0d9-618107f93010" />

#### Extract the archive.
7) Extract the filesystem into the forensics folder previously created. Run:

  ```javascript
  tar -xzf ~/Downloads/filesystem.tar.gz -C ~/forensics
  ```

Explanation:
•	tar — Works with archive files.
•	-x — Extracts files.
•	-z — Decompresses the gzip archive.
•	-f — Specifies the archive to extract.
•	-C ~/forensics — Extracts everything into the forensics directory.

8)	Archive was extracted successfully.

#### Verify the extracted files
9) Confirm that the filesystem was extracted into the forensics folder. Run:

  ```javascript
  ls ~/forensics
  ```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="928" height="186" alt="image" src="https://github.com/user-attachments/assets/b016218e-a955-4ec3-8565-e4cee68859b4" />

10) These directories (bin  boot  dev  etc  home  mnt  root  usr  var) represent different parts of the Linux server's filesystem

#### Ready to begin the actual forensic investigation!
11) Locate the Linux server's log files. On many Linux systems, authentication events are stored in /var/log/.
12) Check what logs are available in our extracted filesystem. Run:

  ```javascript
  ls -lah ~/forensics/var/log
  ```

**Understanding the command:**
•	ls — Lists files and directories.
•	-l — Displays detailed information about each file.
•	-a — Includes hidden files.
•	-h — Displays file sizes in a readable format.
•	~/forensics/var/log — The location of the server's extracted log files.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="869" height="361" alt="image" src="https://github.com/user-attachments/assets/005fc485-bd9e-4c45-ad81-e5059bd04ff9" />

#### Search for Dropbear in the system logs
13) One particularly interesting file is auth.log, which commonly records SSH authentication attempts on systems. Run:

  ```javascript
  grep -i "dropbear" ~/forensics/var/log/syslog* | head -20
  ```
**Understanding the command:**
•	grep — Searches through the log files.
•	-i — Ignores uppercase and lowercase differences.
•	"dropbear" — Searches for activity involving the Dropbear SSH server.
•	syslog* — Examines both the current and older system logs.
•	head -20 — Shows the first 20 matches.

<details>
  <summary> Why are we doing this? </summary>
              SSH authentication activity can sometimes appear in system logs rather than auth.log, depending on the server's configuration.
</details>

<details>
  <summary> What is Dropbear? </summary>
              Dropbear is an SSH server and client software commonly used on lightweight Linux systems, such as embedded devices, routers, and systems with limited resources.
</details>
    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="1365" height="419" alt="image" src="https://github.com/user-attachments/assets/9fb658b3-a232-4ab7-9401-f97e1b467e30" />

14) The first successful SSH login has been identified! 192.168.0.207 is the IP address associated with the earliest successful SSH authentication shown in the logs.

**Flag:** 192.168.0.207

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="210" height="353" alt="image" src="https://github.com/user-attachments/assets/1ae993ae-36f3-4a07-bba0-617a8def664d" />

### CTF Challenge #2 - Who does Number 2 work for?

**Objective:** Find the IP address of the second successful SSH login.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="243" height="234" alt="image" src="https://github.com/user-attachments/assets/64450bd2-f88d-4cba-aba5-14b772d503d8" />

#### Review the SSH login records
1) Search for the second **Child Connection**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="1365" height="419" alt="image" src="https://github.com/user-attachments/assets/f8c6ae55-0920-4f5e-af19-e433cf56f287" />

2) The first successful SSH login has been identified! 100.94.78.57 is the IP address associated with the second successful SSH authentication shown in the logs.

**Flag:** 100.94.78.57


### CTF Challenge #3 - What are you chasing?

**Objective:** Identify what software was installed on the Linux server. In digital forensics, investigating installed software can help us understand what changes were made to a system and whether any suspicious tools were introduced.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="271" height="260" alt="image" src="https://github.com/user-attachments/assets/a7da12d2-c01b-427e-8ff4-2f8b573a1fbd" />


#### Examine the software installation history
1)	Since the server uses a Debian-based Linux system, we can look at logs generated by APT (Advanced Package Tool), which manages software installation. Run:

  ```javascript
  ls -lah ~/forensics/var/log/apt/
  ```

**Understanding the command:**
•	ls — Lists files and folders.
•	-l — Displays detailed information.
•	-a — Includes hidden files.
•	-h — Displays file sizes in a readable format.
•	~/forensics/var/log/apt/ — Looks for APT logs in the extracted server filesystem.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="598" height="199" alt="image" src="https://github.com/user-attachments/assets/36787b82-c57f-4f98-93ec-a304f77c47ae" />

<details>
  <summary> Why are we looking at APT logs? </summary>
              APT logs can reveal software installation events, including package names and dates. If these logs exist, they could help us identify which software was installed.
</details>

2) APT installation logs are available in the extracted filesystem. Interesting files are:

•	history.log	--- Records software installation, removal, and upgrade activity
•	history.log.1.gz	--- Older, compressed installation history
•	term.log	---	Records terminal output from APT operations
•	eipp.log.xz	---	Package installation planning information

#### Examine the software installation history
3)	Run:

  ```javascript
  cat ~/forensics/var/log/apt/history.log
  ```

**Understanding the command:**
•	cat — Displays the contents of a file.
•	~/forensics/var/log/apt/history.log — Points to the software installation history from the CTF server.

4) These records can reveal which software was installed and when.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="1356" height="160" alt="image" src="https://github.com/user-attachments/assets/a47508ca-f127-4ffa-8e18-a4839c6b7ec9" />

5) Software that was installed found! 	tailscale — The software being installed.

**Flag:** tailgate

<details>
  <summary> What is Tailscale? </summary>
              Tailscale is networking software that allows devices to communicate securely over a private network, even when they are in different physical locations.
</details>
