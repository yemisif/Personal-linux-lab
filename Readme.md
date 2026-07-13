# Personal Linux Lab: Multi-User Staging Server

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)
![SSH](https://img.shields.io/badge/SSH-000000?style=for-the-badge&logo=ssh&logoColor=white)

### Overview

This project demonstrates foundational Linux administration skills required for Cloud and DevOps roles. I built a multi-user staging environment from scratch, complete with user permissions, a deployment script, and a secured file system. All managed remotely via SSH

**The Setup:**
- **Target Server:** Bare-metal Ubuntu PC (acts as the "Cloud Server")
- **Workstation:** Windows 11 PC (acts as the admin jump-box)
- **Connection:** PowerShell > SSH > Ubuntu Bash

 **Why This Matters:**
This project proves I can navigate filesystems, manage users, write Bash scripts, and secure sensitive data-skills that translate directly to managing GCP, AWS, Azure on VMs on Day 1

---

### Project Goals

1. Securely SSH into a remote Ubuntu server from Windows
2. Create a structured application directory tree
3. Implement user and group management (IAM simulation)
4. Restrict 'sudo' privileges (least privilege principle)
5. Write and execute a Bash deployment script
6. Harden file permissions with  'chmod 700'

---
### Architecture
┌─────────────────────────────────────────────────────────────┐
│ Windows 11 PC │
│ (Workstation / Admin) │
│ │
│ ┌───────────────────────────────────────────────────┐ │
│ │ PowerShell / SSH Client │ │
│ │ ssh yemi@192.168.0.194 │ │
│ └───────────────────┬───────────────────────────────┘ │
└──────────────────────┼────────────────────────────────────┘
│ SSH (Port 22)
▼
┌─────────────────────────────────────────────────────────────┐
│ Ubuntu 22.04 LTS │
│ (Target / Server) │
│ 192.168.0.194 │
│ │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ /opt/staging_apps/ │ │
│ │ ├── src/ # Source code folder │ │
│ │ ├── logs/ # Log files │ │
│ │ │ └── deploy.log # Deployment logs │ │
│ │ ├── config/ # Configuration files │ │
│ │ ├── secure_archive/ # 🔒 Only root can access │ │
│ │ └── deploy.sh # Deployment script │ │
│ └─────────────────────────────────────────────────────┘ │
│ │
│ 👥 Users: │
│ - yemi (Admin, full sudo access) │
│ - Dev_Tobi (Developer, restricted sudo) │
└─────────────────────────────────────────────────────────────┘

---

## Step-by-Step Walkthrough

### 1. Remote Connectivity (SSH)

The entire project was executed remotely from Windows to prove headless server management.

```powershell
# Windows PowerShell
PS C:\Users\User > ssh yemi@192.168.0.194
yemi@yemidevbox:~$
```
https://screenshots/ssh-connection.png

### 2. Filesystem Architecture
Built a logical directory tree to organize application components.

```bash
# Create the directory structure
yemi@yemidevbox:/opts$ sudo mkdir -p staging_apps/{src, logs, config, backup}

# verify the structure
yemi@yemidevbox:/opt$ ls -R staging_apps/
staging_apps/:
backup config logs src

staging_apps/backup:
staging_apps/config:
staging_apps/logs:
staging_apps/src:
```

https://screenshots/directory-structure.png

### 3. User & Group Management (IAM Simulation)

Created a developer group and restricted their sudo priviledges to only restarting services, mimicking enterprise "least privilege" policies.

```bash
# Create the developer group
yemi@yemidevbox:/opt$ sudo groupadd dev_team

# Create a user and add them to the group
yemi@yemidevbox:/opt$ id Dev_Tobi
uid=1001(Dev_Tobi) gid=1002(Dev_Tobi),102(Dev_Team)
```

#### Sudo Restriction
Created a custom `sudoers` file to restrict `dev_team` to only running `systemctl restart nginx`.

```bash
# Open the sudoers configuration
yemi@yemidevbox:opt$ sudo visudo -f /etc/sudoers.d/Dev_Team
#Added this line:
%Dev_Team ALL +(ALL) /usr/bin/systemctl restrt nginx
```

#### Testing the Restriction

```bash
# Switch to Dev_Tobi
yemi@yemidevbox:/opt$ sudo su -Dev_Tobi

# Attempt a disallowed command (fails)
Dev_Tobi@yemidevbox:~$ sudo apt update
Sorry, user Dev_Tobi is not allowed to execute '/usr/bin/apt update' as root.

# Attempt the allowed command (succeess-no Permission denied)
Dev_Tobi@yemidevbox:`$ sudo systemctl restart nginx
Failed to restart nginx.service:Unit nginx.service not found.
```

https://screenshots/user-restriction.png

### 4. Scripting with Vim
 
Wrote a Bash deployment script to automatically log deployment timestamps.

```bash
# Create the script with Vim
yemi@yemidevbox://opt/staging_apps$ sudo vim deploy.sh
```
#### deploy.sh:
```bash
#!/bin/bash
echo "Deployment ran by User: $USER at $(date)" >> /opt/staging_apps/logs/deploy.log
echo "Deployment compete! Check the logs folder."
echo "Log Location: /opt/staging_apps/logs/deploy.log"
```

```bash
Make the script executable
yemi@yemidevbox:/opt/staging_apps$ sudo chmod +x deploy.sh

# Run the script
yemi@yemidevbox:/opt/staging_apps$ ./deploy.sh
Deployment complete! check the logs folder.
Log location: /opt/staging_apps/logs/deploy.log

# View the logs
yemi@yemidevbox:/opt/staging_apps$ cat logs/deploy.log
Deployment ran by user: root at Thu Jul 9 11:54:15 AM UTC 2026
Deployment ran by user: root at Thu Jul 9 12:12:17 PM UTC 2026
Deployment ran by user: root at Thu Jul 9 12:42:39 PM UTC 2026
```

https://screenshots/script-logs.png

### 5. Permission Hardening
Secured sensitive folders with Strict permissions - Only root can access `secure_archive`

```bash
# Create a secure archive folder
yemi@yemidevbox:/opt/staging_apps$ sudo mkdir secure_archive

# Set permissions: only root can read/write/execute
yemi@yemidevbox:/opt/staging_apps$ sudo chmod 700 secure_archive

# verify permisiions
yemi@yemidevbox:/opt/staging_apps$ ls -la
total 28
drwxr-xr-x 6 root root 4096 Jull 9 12:19 .
drwxr-xr-x 4 root root 4096 Jul 9 10:24 ..
drwxr-xr-x 2 root root 4096 Jul 9 10:24 config
-rwxr-xr-x 1 root root  198 Jul 9 11:52 deploy.sh
drwxr-xr-x 2 root root 4096 Jul 11:54 logs
drwx------ 2 root root 4096 Jul 9 12:19 secure_archive
```


https://screenshots/permissions.png

## Final Dircectory Structure

```text
/opt/staging_apps/
├── src/              # Source code folder
├── logs/             # Log files
│   └── deploy.log    # Deployment logs (with timestamps!)
├── config/           # Configuration files
├── secure_archive/   # 🔒 SECURE: Only root can access (700)
└── deploy.sh         # Deployment script
```

## Testing & Validation

| Test | Expected Result | Status |
|------|-----------------|--------|
| SSH connection from Windows |  Successful login |  PASS |
| Directory creation (`mkdir -p`) |  Folders created |  PASS |
| User creation (`useradd`) |  `Dev_Tobi` created |  PASS |
| `sudo` restriction (disallowed command) |  Permission denied |  PASS |
| `sudo` restriction (allowed command) |  No "Permission denied" error |  PASS |
| Script execution (`./deploy.sh`) |  Log file created |  PASS |
| Permission hardening (`chmod 700`) |  Only root can access |  PASS |

## Commands Used

| **Command** |**Purpose** |
|-------------|------------|
|`ssh yemi@192.168.0.194` | Secure remote connection to Ubuntu server |
|`sudo mkdir -p staging_apps/{src,logs,config,backup}` | Create directory structure |
|`sudo groupadd dev_team` | Create developer group |
|`sudo useradd -m -G dev_team Dev_Tobi` | Create restricted user |
|`sudo passwd Dev_Tobi` | Set password for `dev_Tobi` |
|`sudo visudo -f /etc/sudoers.d/dev_team` | Restrict sudo privileges
|`sudo vim deploy.sh` | Create Bash script |
|`sudo chmod +x deploy.sh` | Make script executable |
|`sudo chmod 700 secure_archive` | Restrict folder access (only root) |


## Key Takeaways
1. **SSH is the backbone of remote administration.** understanding how to connect troubleshoot, and maintain SSH sessions is essential.
2.**Linux permissions are your first line of defense.** `chmod 700` ensures only root can access sensitive data-mimicking real-world security practices.
3. **"Least Privilege is best practice.** Restricting `sudo` to specific commands prevent developers from accidentally (or maliciously) breaking production environments.
4. **Automation is key** writing Bash scripts to handle repetitive tasks(like logging deployments) saves time and reduces human error.

 ---

## Screenshots
All screenshots are available in the `/screenshots` folder;

1. `ssh-connection.png` - PowerShell SSH connection to Ubuntu
2. `dirctory-struture.png` - initila directory tree
3. `user-restriction.png` - `sudo apt update` blocked for `Dev_Tobi`
4. `scripts-logs.png` -bash script execution and log output
5. `permissions.png` - `secure_archive` with `drwx------` permissions


## Conclusion 
This projects bridges the gap between "memorized commands" and practical administration." The skills demonstarted here; remnote access, user management, filesystem navigation, bash scripting, and permisiion hardening are directly transferable to mamaging cloud compute instances (GCP, AWS, Azure) in a production environment.

## Connect with me

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/yemisifolaranmi/)
[![Github](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/yemisif)
  
Built with ❤️ and a lot of sudo commands.


