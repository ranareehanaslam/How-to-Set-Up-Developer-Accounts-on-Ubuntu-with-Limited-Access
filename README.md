Below is a complete,step-by-step instructions to add and manage multiple users (developers) on your Ubuntu server. In this example, we’ll set up five developer accounts that can log in via SSH, work in your Nginx project folder (`/var/www/html/google.com`), run PHP commands, but not perform any major system operations.

---

# 🚀 How to Set Up Developer Accounts on Ubuntu with Limited Access

In this guide, we’ll show you how to:

- **Create and manage multiple users**  
- **Set up a dedicated developers group**  
- **Restrict sudo privileges to only allow PHP commands**  
- **Force developers to land directly in your project directory on login**  
- **Optionally restrict users further with a chroot jail**

Let’s dive in!

---

## 1. 👥 Create a Developer Group and Add Users

**A. Create the Developer Group**

First, create a group called `developers` to easily manage permissions for all your developer accounts.

```bash
sudo groupadd developers
```

**B. Create User Accounts and Add Them to the Group**

Create five users (e.g., `dev1` through `dev5`) and add them to the `developers` group:

```bash
for user in dev1 dev2 dev3 dev4 dev5; do
    sudo adduser $user
    sudo usermod -aG developers $user
done
```

*Each user will be prompted to set a password and basic information during the `adduser` process.*

---

## 2. 📂 Set Permissions for Your Project Directory

To let developers view and modify files in your project directory, adjust the directory’s ownership and permissions.

1. **Change the Owner and Group:**  
   Assign the owner as `www-data` (commonly used by Nginx/PHP-FPM) and the group as `developers`.

   ```bash
   sudo chown -R www-data:developers /var/www/html/google.com
   ```

2. **Set Group Read/Write Permissions:**  
   Allow the group to read, write, and execute files (required for directory traversal):

   ```bash
   sudo chmod -R 775 /var/www/html/google.com
   ```

3. **Ensure New Files Inherit Group Permissions:**  
   This command sets the “setgid” bit on directories so that new files inherit the group.

   ```bash
   sudo find /var/www/html/google.com -type d -exec chmod 2775 {} +
   ```

---

## 3. 🔐 Restrict sudo Access to PHP Only

We want your developers to run PHP commands (e.g., `php artisan`) with elevated privileges—but nothing else.

1. **Open the sudoers File Safely:**

   ```bash
   sudo visudo
   ```

2. **Add These Two Lines at the End:**

   ```bash
   %developers ALL=(ALL) NOPASSWD: /usr/bin/php
   %developers ALL=(ALL) !ALL
   ```

   **Explanation:**  
   - **`%developers ALL=(ALL) NOPASSWD: /usr/bin/php`**  
     ➔ Members of the developers group can run `/usr/bin/php` via `sudo` without a password.  
     
   - **`%developers ALL=(ALL) !ALL`**  
     ➔ Denies all other commands through `sudo` for developers.  

   *Together, these rules allow developers to run PHP commands with sudo while blocking other system-wide operations.*

---

## 4. 🌐 Make Developers Land in the Project Directory on Login

There are two common approaches:

### **Option A: Change Each User’s Home Directory**

Change the home directory of each developer so that they directly land in your project folder:

```bash
for user in dev1 dev2 dev3 dev4 dev5; do
    sudo usermod -d /var/www/html/google.com -s /bin/bash $user
done
```

> **Note:** This permanently sets the user’s home directory to your project folder.

### **Option B: Automatically Change Directory on Login**

If you prefer not to change the home directory permanently, add an automatic change in the global profile.

1. **Edit `/etc/profile`:**

   ```bash
   sudo nano /etc/profile
   ```

2. **Add at the End of the File:**

   ```bash
   if groups $USER | grep -q "\bdevelopers\b"; then
       cd /var/www/html/google.com
   fi
   ```

*This snippet checks if the logged-in user belongs to the `developers` group and then changes the directory accordingly.*

---

## 5. 🔒 (Optional) Further Restrict Developers with a chroot Jail

For even tighter control, you can confine developers to the project directory using a chroot jail.

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

> **Warning:** Using a chroot jail will restrict developers to SFTP access only. They won’t have full shell access. Use this option if you want to limit file transfers and prevent shell access entirely.

---

## 6. 🛡️ Restrict SSH Access to Developer Accounts (Optional)

To further secure your server, limit SSH logins to only the developer accounts.

1. **Edit SSH Config:**

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Add an AllowUsers Line:**

   ```bash
   AllowUsers dev1 dev2 dev3 dev4 dev5
   ```

3. **Restart SSH:**

   ```bash
   sudo systemctl restart ssh
   ```

---

## 7. 🔍 Monitor User Activity (Optional)

Keeping an eye on file changes and user activity can be important:

1. **Install auditd:**

   ```bash
   sudo apt install auditd
   sudo systemctl enable --now auditd
   ```

2. **Set Up a Watch on Your Project Directory:**

   ```bash
   sudo auditctl -w /var/www/html/google.com -p war -k dev_changes
   ```

3. **View Audit Logs:**

   ```bash
   sudo ausearch -k dev_changes
   ```

---

# ✅ Final Thoughts

By following these steps, you have set up a secure environment where:

- **Developers can:**  
  - Log in via SSH  
  - Work within the project directory (`/var/www/html/google.com`)  
  - Run PHP commands using `sudo` (without a password)

- **Developers cannot:**  
  - Execute any other sudo commands that might harm the system  
  - Easily navigate to sensitive parts of the server (especially if using a chroot jail)

This setup strikes a balance between **productivity** and **security**, making it ideal for collaborative web development projects on an Ubuntu server.

Happy coding! 🚀

Feel free to ask if you have any more questions or need further clarification!
