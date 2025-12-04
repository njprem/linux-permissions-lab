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
