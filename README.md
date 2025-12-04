# Nutthawut Jaroenchokwittaya
# 🛡️ Linux Permissions Lab

This is a small lab I built to practice Linux file permissions and basic security concepts:

- 🔐 `chmod`, `chown`, `chgrp`
- 🎭 SUID, SGID, sticky bit
- 👥 Shared group directories
- 🧱 Least-privilege access control (role-based access)

---

## 📂 Project Structure

- `proj-dir/`  
  📁 Simple files for testing basic permissions (`chmod`, `chown`, `chgrp`).

- `shared-dir/`  
  🤝 Example of a shared project folder with the **SGID bit** set on the directory.  
  New files created inside inherit the directory's group, so all members of the group can collaborate.

- `public-dir/`  
  🗑️ Example “public” directory used to demonstrate the **sticky bit** (`chmod +t`).  
  Even if everyone can write to the directory, users cannot delete each other’s files.

- `system/`
  - `config/` – 🛠️ config files meant only for **"admins"**  
  - `logs/` – 📊 log files meant for **"analysts"**  
  - `docs/` – 📚 docs meant for **"interns"**  
  This folder is used to simulate a simple **role-based / least-privilege** design using groups and directory permissions.

- `whoami_suid.c`  
  🧪 Small C program used to demonstrate **real UID vs effective UID/GID** when experimenting with SUID/SGID.

---

## 🧠 Example Concepts

### 1️⃣ Shared group directory with SGID
```bash
sudo chown Tik:Dev shared-dir
sudo chmod 2770 shared-dir     # rwxrws---
```
### 2️⃣ Public directory with sticky bit
```bash
mkdir public-dir
chmod 1777 public-dir          # rwxrwxrwt
```

### 3️⃣ Simple least-privilege layout
system/
├─ config/   # admins only
├─ logs/     # analysts
└─ docs/     # interns
###🧰 Full Lab Commands (Step by Step)
This section shows the main commands I used to build and test the lab.
##0️⃣ Setup lab directory
```bash
mkdir -p ~/permissions-lab
cd ~/permissions-lab
```
### 1️⃣ Create Users and Groups for Testing
Create an extra test user
```bash
sudo useradd -m Ryu
```
Create a shared group
```bash
sudo groupadd Dev
```
Add users to Dev group
```bash
sudo usermod -aG Dev Tik
sudo usermod -aG Dev Ryu
```
### 2️⃣ proj-dir — Basic chmod / chown / chgrp Practice
```bash
mkdir proj-dir
echo "secret for alice" > proj-dir/alice.txt
echo "notes for group"  > proj-dir/shared.txt
```
Assign owner & group
```bash
sudo chown Tik:Dev proj-dir
sudo chown Tik:Dev proj-dir/*
```
Adjust permissions
```bash
sudo chmod 640 proj-dir/alice.txt   # owner rw, group r, others -
sudo chmod 664 proj-dir/shared.txt  # owner rw, group rw, others r

ls -ld proj-dir
ls -l proj-dir
```
Test access:
```bash
sudo -u Tik cat proj-dir/alice.txt     # allowed
sudo -u Ryu cat proj-dir/alice.txt     # allowed/denied depending on mode
```
###3️⃣ shared-dir — SGID Shared Group Folder
```bash
mkdir shared-dir
```
Owner = Tik, Group = Dev
```bash
sudo chown Tik:Dev shared-dir
```
Enable SGID on directory (+ block access for others)
```bash
sudo chmod 2770 shared-dir     # rwxrws---
```
Create files
```bash
sudo -u Tik bash -c 'cd ~/permissions-lab/shared-dir && echo "made by Tik"  > tik-file.txt'
sudo -u Ryu bash -c 'cd ~/permissions-lab/shared-dir && echo "made by Ryu" > ryu-file.txt'

ls -ld shared-dir
ls -l shared-dir
```
### 4️⃣ public-dir — Sticky Bit like /tmp
```bash
mkdir public-dir
chmod 777 public-dir             # world-writable
```
Create files:
```bash
sudo -u Tik bash -c 'cd ~/permissions-lab/public-dir && echo "from Tik"  > tik.txt'
sudo -u Ryu bash -c 'cd ~/permissions-lab/public-dir && echo "from Ryu" > ryu.txt'
```
Without sticky bit, users can delete each other’s files:
```bash
sudo -u Ryu bash -c 'cd ~/permissions-lab/public-dir && rm tik.txt'
```
Recreate file and enable sticky bit:
```bash
sudo -u Tik bash -c 'cd ~/permissions-lab/public-dir && echo "from Tik" > tik.txt'

sudo chmod +t public-dir
ls -ld public-dir     # look for 't'
```
Now deletion fails:
```bash
sudo -u Ryu bash -c 'cd ~/permissions-lab/public-dir && rm tik.txt'
```
 → rm: cannot remove 'tik.txt': Operation not permitted
###5️⃣ system/ — Least-Privilege Role-Based Layout

Create structure:
```bash
mkdir -p system/config system/logs system/docs

```
Create role groups:
```bash
sudo groupadd admins
sudo groupadd analysts
sudo groupadd interns

```
Assign users:
```bash
sudo usermod -aG admins   Tik
sudo usermod -aG analysts Ryu
sudo useradd -m Ivan
sudo usermod -aG interns  Ivan

```
Set ownership:
```bash
sudo chown root:admins   system/config
sudo chown root:analysts system/logs
sudo chown root:interns  system/docs

```
Set Permissions:
```bash
sudo chmod 770 system/config   # admins only
sudo chmod 750 system/logs     # analysts
sudo chmod 750 system/docs     # interns

```
Add example files:
```bash
echo "very secret config"  | sudo tee system/config/main.conf > /dev/null
echo "log entry 1"         | sudo tee system/logs/app.log     > /dev/null
echo "guide for interns"   | sudo tee system/docs/guide.txt   > /dev/null

```
Test access:
```bash
# Admin
sudo -u Tik cat system/config/main.conf   # allowed
sudo -u Tik cat system/logs/app.log       # allowed
sudo -u Tik cat system/docs/guide.txt     # allowed

# Analyst
sudo -u Ryu cat system/logs/app.log       # allowed
sudo -u Ryu cat system/config/main.conf   # denied
sudo -u Ryu cat system/docs/guide.txt     # denied

# Intern
sudo -u Ivan cat system/docs/guide.txt    # allowed
sudo -u Ivan cat system/logs/app.log      # denied
sudo -u Ivan cat system/config/main.conf  # denied
```

###6️⃣ whoami_suid.c — SUID Demonstration Program

Source code:
```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("Real UID: %d\n", getuid());
    printf("Effective UID: %d\n", geteuid());
    printf("Real GID: %d\n", getgid());
    printf("Effective GID: %d\n", getegid());
    return 0;
}
```

Compile:
```bash
gcc whoami_suid.c -o whoami_suid

```
Enable SUID:
```bash
sudo chown root:root whoami_suid
sudo chmod 4755 whoami_suid     # -rwsr-xr-x

```
Run:
```bash
./whoami_suid
sudo -u Ryu ./whoami_suid

```
###You will see different Real UID but the same Effective UID (root) due to SUID.
