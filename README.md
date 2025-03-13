Below is a comprehensive, beginner-friendly guide that walks you through creating and managing multiple developer accounts on your Ubuntu server. In this example, you’ll set up five developer accounts that can log in via SSH, work within your Nginx project directory (`/var/www/html/google.com`), and run PHP commands—all while being prevented from performing any major system operations.


### 🔐 **Ultimate Guide to Secure Developer Access on Ubuntu** 🚀  
**✔️ Linux User Management & Access Control for Developers**  
**✔️ Secure Multi-User Environment on Ubuntu**  
**✔️ Restricted Shell (RBash) & Permissions Setup for Web Developers**  
**✔️ User Isolation & Access Restriction in Linux Servers**  

📌 **What You'll Learn:**  
🔹 How to **create and manage developer accounts** on Ubuntu  
🔹 Secure **SSH access** with restricted shell (rbash)  
🔹 Set up **file permissions** for your web projects  
🔹 **Prevent unauthorized system modifications**  
🔹 **Allow only necessary commands (PHP, nano, ls, etc.)**  
🔹 **Restrict navigation** beyond project directories  

💡 **Perfect for:** DevOps engineers, system admins, and developers managing multi-user environments securely.  

🚦 **Let’s dive in and lock down access the right way!** 🔒



---

# 🚀 Ultimate Guide: Secure Developer Accounts on Ubuntu

In this guide, you'll learn how to:

- **Create and manage multiple developer accounts**  
- **Set up a dedicated `developers` group**  
- **Configure file permissions for your project directory**  
- **Restrict system access (including sudo and shell access)**  
- **Allow only PHP and a few other commands**  
- **Automatically land in the project directory on login**

Let's dive in!

---

## 1. 👥 Create a Developer Group and Add Users

### **A. Create the Developer Group**
Creating a group makes it easier to manage permissions for all developer accounts.
```bash
sudo groupadd developers
```

### **B. Create User Accounts and Add Them to the Group**
We’ll create five users (`dev1` through `dev5`) and add them to the `developers` group:
```bash
for user in dev1 dev2 dev3 dev4 dev5; do
    sudo adduser $user
    sudo usermod -aG developers $user
done
```
*Each user will be prompted to set a password and provide some basic information.*

---

## 2. 📂 Set Permissions for Your Project Directory

Developers need to view, edit, and modify files in `/var/www/html/google.com`. Ensure the permissions and ownership are set correctly.

### **A. Change the Owner and Group**
Assign the owner as `www-data` (Nginx/PHP-FPM user) and the group as `developers`:
```bash
sudo chown -R www-data:developers /var/www/html/google.com
```

### **B. Set Group Read/Write Permissions**
Allow the group to read, write, and execute:
```bash
sudo chmod -R 775 /var/www/html/google.com
```

### **C. Ensure New Files Inherit Group Permissions**
Set the “setgid” bit on directories so that new files inherit the group:
```bash
sudo find /var/www/html/google.com -type d -exec chmod 2775 {} +
```

---

## 3. 🔐 Restrict System Access

Even though your developers have user accounts, you must ensure they don’t have full root privileges or the ability to navigate and modify critical system files.

### **A. Prevent Developers from Using sudo**
Even if users exist on the system, they should not have full sudo access. Open the sudoers file:
```bash
sudo visudo
```
Then, add these two lines at the end:
```sudoers
%developers ALL=(ALL) NOPASSWD: /usr/bin/php
%developers ALL=(ALL) !ALL
```
> **Explanation:**  
> - **`%developers ALL=(ALL) NOPASSWD: /usr/bin/php`**  
>   ➔ Allows members of the `developers` group to run `/usr/bin/php` with sudo without needing a password.  
> - **`%developers ALL=(ALL) !ALL`**  
>   ➔ Explicitly denies all other sudo commands.  
>   
> **✅ Result:** Developers can run PHP commands using sudo but are blocked from executing any other sudo commands.

---

### **B. Restrict Users to the Project Directory Using a Restricted Shell (rbash)**
To keep developers from navigating and modifying other system files, change their shell to a restricted shell.

1. **Edit the `/etc/passwd` File:**
   ```bash
   sudo nano /etc/passwd
   ```
2. **Find the User Lines**  
   They look similar to:
   ```plaintext
   dev1:x:1001:1001::/home/dev1:/bin/bash
   ```
3. **Change the Shell from `/bin/bash` to `/bin/rbash`:**
   ```plaintext
   dev1:x:1001:1001::/home/dev1:/bin/rbash
   ```
   Do this for each developer account.

> **✅ Result:** This locks developers to their directory so they cannot easily navigate to other parts of the file system.

---

### **C. Allow PHP & Other Essential Commands**
Since `rbash` restricts nearly every command by default, you need to explicitly allow a few commands that your developers will use (like PHP, text editors, and directory listing).

1. **Create a Personal `bin` Directory for a Developer (e.g., dev1):**
   ```bash
   mkdir /home/dev1/bin
   ```
2. **Create Symbolic Links to Allowed Commands:**
   ```bash
   ln -s /usr/bin/php /home/dev1/bin/php
   ln -s /usr/bin/nano /home/dev1/bin/nano
   ln -s /usr/bin/vi /home/dev1/bin/vi
   ln -s /bin/ls /home/dev1/bin/ls
   ```
3. **Update the Developer’s PATH Variable:**
   Append the following line to `/home/dev1/.bashrc`:
   ```bash
   echo 'export PATH=$HOME/bin' >> /home/dev1/.bashrc
   ```
4. **Repeat for Each Developer:**  
   You can repeat these steps (or script them) for each user so they all have access to the allowed commands.

> **✅ Result:** Developers can now run `php`, `nano`, `vi`, and `ls` while being restricted from executing other system commands.

---

## 4. 🌐 Make Developers Land in the Project Directory on Login

Ensure that developers automatically start in `/var/www/html/google.com` when they log in.

### **Option A: Change Each User’s Home Directory**
This permanently sets their home directory to your project folder:
```bash
for user in dev1 dev2 dev3 dev4 dev5; do
    sudo usermod -d /var/www/html/google.com -s /bin/bash $user
done
```
> **Note:** This option overrides the default `/home/username` directory.

### **Option B: Automatically Change Directory via Global Profile**
If you prefer not to change the home directory permanently, add the following snippet to the global profile:

1. **Edit `/etc/profile`:**
   ```bash
   sudo nano /etc/profile
   ```
2. **Add the Following at the End:**
   ```bash
   if groups $USER | grep -q "\bdevelopers\b"; then
       cd /var/www/html/google.com
   fi
   ```
> **✅ Result:** When a developer logs in, the system will automatically change the directory to `/var/www/html/google.com`.

---

## 5. 🔒 (Optional) Further Restrict with a chroot Jail

For an even more locked-down environment, you can use a chroot jail to confine developers to the project directory.

1. **Edit the SSH Configuration:**
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
2. **Add the Following at the End:**
   ```bash
   Match Group developers
       ChrootDirectory /var/www/html/google.com
       X11Forwarding no
       AllowTcpForwarding no
       ForceCommand internal-sftp
   ```
3. **Restart SSH:**
   ```bash
   sudo systemctl restart ssh
   ```
> **⚠️ Warning:** Using a chroot jail in this manner will restrict developers to SFTP access only (no shell access). Use this option only if you want to prevent shell logins entirely.

---

## 6. 🔍 Monitor User Activity (Optional)

To keep track of what changes are being made within your project directory, consider setting up auditd.

1. **Install and Enable auditd:**
   ```bash
   sudo apt install auditd
   sudo systemctl enable --now auditd
   ```
2. **Set Up a Watch on the Project Directory:**
   ```bash
   sudo auditctl -w /var/www/html/google.com -p war -k dev_changes
   ```
3. **View Audit Logs:**
   ```bash
   sudo ausearch -k dev_changes
   ```

---

# ✅ Final Thoughts

By following these detailed steps, you now have a secure, multi-user environment on your Ubuntu server where developers:

- Can log in via SSH and automatically start in the project directory.
- Can view and modify files in `/var/www/html/google.com`.
- Are allowed to run PHP commands (and a few other necessary commands) with sudo privileges.
- Are restricted from executing other system commands or modifying critical system files via a restricted shell (rbash).
- Optionally, can be further confined using a chroot jail and monitored for activity.

This balanced setup boosts collaboration while ensuring system security and integrity.

Happy coding! 🚀

Feel free to ask if you have any more questions or need further clarification!
