# 🔹 Remote Linux Server Setup with SSH

This guide explains how to set up a remote Linux server, configure SSH access using key-based authentication, and enhance security using Fail2Ban.

---

## **📌 Prerequisites**
- A remote Linux server (e.g., **Ubuntu 20.04+**, **Amazon Linux**, etc.).
- OpenSSH server installed on the remote machine.
- A local machine with **SSH client** installed.

---

## **1️⃣ Create a Remote Server**
Use your **VPS provider** (AWS, DigitalOcean, Linode, etc.) to create a new Linux instance.

1. Log into your **VPS provider's control panel**.
2. Create a **new server** (Ubuntu, Debian, or CentOS).
3. Note down the **public IP address** of the server.

---

## **2️⃣ Generate SSH Key Pair Locally**
On your local machine, generate an **SSH key pair**:  
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

```bash
#### NOTE: 
1️⃣ Default Naming
If you just run:
ssh-keygen -t rsa -b 4096

    It prompts you to enter a filename.
    If you press Enter, it defaults to:
        Private key: ~/.ssh/id_rsa
        Public key: ~/.ssh/id_rsa.pub

2️⃣ Custom Naming

You can specify a filename with the -f flag:

ssh-keygen -t rsa -b 4096 -f ~/.ssh/custom_key (yes you can ssh-keygen -t rsa -b 4096 -C "your_email.com" -f ~/.ssh/custom_key)

This creates:

    Private key: ~/.ssh/custom_key
    Public key: ~/.ssh/custom_key.pub

3️⃣ Naming Based on Email

If you use -C "your_email@example.com" like this:

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

    It does NOT change the filename!
    The email is just added as a comment inside the public key.

4️⃣ Multiple Keys

If you generate multiple keys, you must use unique filenames:

ssh-keygen -t rsa -b 4096 -f ~/.ssh/aws_key
ssh-keygen -t rsa -b 4096 -f ~/.ssh/github_key

This avoids overwriting existing keys.
```

- -t rsa → Specifies the RSA algorithm.
- -b 4096 → Generates a strong 4096-bit key.
- -C "your_email@example.com" → Optional comment to identify the key.

~/.ssh/id_rsa      # Private key (KEEP THIS SECRET!)
~/.ssh/id_rsa.pub  # Public key (Safe to share)


## **3️⃣ Add the SSH Key to the Server
A. Using ssh-copy-id (Recommended)

If password authentication is enabled, you can transfer the key with:
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub user@server-ip
```
    Replace user with your server username (e.g., ubuntu, root).
    Replace server-ip with your server’s IP address.

B. Manually Adding the Key

If ssh-copy-id doesn’t work:

ssh user@server-ip
mkdir -p ~/.ssh
echo "your-public-key-content" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh


Replace "your-public-key-content" with the output of:

    cat ~/.ssh/id_rsa.pub

## **4️⃣ Configure SSH Access on Local Machine

To simplify SSH connections, edit the SSH config file on your local machine:

nano ~/.ssh/config

Add the following:

```bash
Host myserver
    HostName server-ip
    User user
    IdentityFile ~/.ssh/id_rsa
```
Now, you can connect with:
```bash
ssh myserver
```
instead of typing the full ssh user@server-ip -i ~/.ssh/id_rsa.

If you're using AWS, your ~/.ssh/config should be set up like this:

Host my-ec2
    HostName <your-ec2-public-ip>
    User ubuntu
    IdentityFile ~/.ssh/my-aws-key

## **5️⃣ Secure SSH Access on the Server

Edit the SSH daemon configuration:

sudo nano /etc/ssh/sshd_config

Ensure the following settings are applied:

PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no  # Disable password-based login
PermitRootLogin no  # Prevent root login

Restart SSH service:

sudo systemctl restart ssh

## **6️⃣ (Optional) Install Fail2Ban for Security

To prevent brute-force attacks, install Fail2Ban:

sudo apt update && sudo apt install fail2ban -y

Copy the default configuration:

sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

Edit the config:

sudo nano /etc/fail2ban/jail.local

Find the [sshd] section and ensure it looks like this:

[sshd]
enabled = true
maxretry = 5
bantime = 10m
findtime = 10m

Restart Fail2Ban:

sudo systemctl restart fail2ban
sudo systemctl enable fail2ban

Check status:

sudo fail2ban-client status sshd

✅ Summary

    🎯 Created an SSH key pair locally.
    🎯 Added the key to the remote server.
    🎯 Configured SSH access using an alias.
    🎯 Hardened SSH security settings.
    🎯 (Optional) Secured the server using Fail2Ban.


- The source of the project is:
https://roadmap.sh/projects/ssh-remote-server-setup

## 🔹 Author
Zoey 🚀
