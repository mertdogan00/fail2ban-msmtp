# 🛡️ Fail2ban + Msmtp E-mail Integration for Linux Servers

Protect your Linux server with **Fail2ban real-time banning** and receive instant email alerts via **msmtp external SMTP integration.**  
This is a focused guide for email notification setup and includes advanced Nginx filters for request limiting and blocking. 🔒

For a complete Linux server manual setup, see 👉 https://github.com/mertdogan00/server-manual-setup


# 📚 Table of Contents
- [Project Overview 🗂️](#-project-overview)
- [Installation 🛠️](#-installation)
- [msmtp Configuration ✉️](#-msmtp-configuration)
- [Fail2ban Mail Setup ⚙️](#-fail2ban-mail-setup)
- [Fail2ban Nginx Configuration 🔧](#-fail2ban-nginx-configuration)
- [Firewall Setup 🔥](#-firewall-setup)
- [Email Notification Test 🚀](#-email-notification-test)
- [Contributing 🤝](#-contributing)
- [License 📜](#-license)

# 🗂 Project Overview
This repository focuses on configuring **Fail2ban** to work with **msmtp** for sending real-time email notifications and securing Nginx with advanced banning rules.

- Supports external SMTP servers like Gmail, Yandex, or custom mail servers.
- Protects Nginx against request floods and 444 status triggers.
- Fully compatible with **UFW (default firewall)** or **firewalld.**

🔗 Related Project: [Complete Linux Server Setup](https://github.com/mertdogan00/server-manual-setup)

# 🛠 Installation
Install msmtp and Fail2ban:
sudo apt update
sudo apt install msmtp msmtp-mta fail2ban -y

# ✉ msmtp Configuration
Open msmtp configuration file:
```
sudo nano /etc/msmtprc
```
Example configuration:
```
defaults
auth on
tls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt

account default
host smtp.yourmailserver.com
port 587
from yourmail@domain.com
user yourmail@domain.com
password your_smtp_password
logfile /var/log/msmtp.log
```

✅ Important Notes:
- The 'from' and 'user' addresses must usually be the same for most SMTP servers.
- The password should be an application-specific password if required.

Test your email:
```
echo -e "Subject: Fail2ban Test Mail\n\nThis is a test email from msmtp." | msmtp yourmail@domain.com
```

# ⚙ Fail2ban Mail Setup
Open Fail2ban configuration file:
sudo nano /etc/fail2ban/jail.local

Example configuration:
```
[DEFAULT]
destemail = yourmail@domain.com
sender = yourmail@domain.com
mta = sendmail
action = %(action_mwl)s
bantime = 600
findtime = 300
maxretry = 5
```

Restart Fail2ban:
```
sudo systemctl restart fail2ban
```

# 🔧 Fail2ban Nginx Configuration
Create Nginx fail2ban jail file:
```
sudo nano /etc/fail2ban/jail.d/nginx.conf
```

Example nginx.conf:
```
[nginx-limit-req]
enabled = true
filter = nginx-limit-req
port = http,https
logpath = /var/log/nginx/error.log /var/log/nginx/cp-main_access_log /var/log/nginx/cp-main_error_log
maxretry = 10
bantime = 900
findtime = 60

[nginx-block-444]
enabled = true
filter = nginx-block-444
port = http,https
logpath = /var/log/nginx/access.log /var/log/nginx/error.log
findtime = 60
bantime = 3600
maxretry = 5
```

Create nginx-block-444 filter file:
```
sudo nano /etc/fail2ban/filter.d/nginx-block-444.conf
```

Example nginx-block-444.conf:
```
[Definition]
failregex = ^<HOST> - - \[.*\] "(GET|POST|HEAD).*HTTP/.*" 444
ignoreregex =
```

This filter bans IPs that receive Nginx 444 status codes.

# 🔥 Firewall Setup
UFW is the **default firewall** in most Linux distributions, but firewalld can also be used.

## ✅ UFW - Default Firewall Example
```
sudo ufw allow 22/tcp  # SSH access
sudo ufw allow 80/tcp  # HTTP access
sudo ufw allow 443/tcp # HTTPS access
sudo ufw enable        # Enable UFW firewall
sudo ufw status        # Check status
```

Each rule should be adjusted based on your server’s service ports.

## 🔥 firewalld Example
```
sudo firewall-cmd --permanent --add-port=22/tcp  # SSH access
sudo firewall-cmd --permanent --add-port=80/tcp  # HTTP access
sudo firewall-cmd --permanent --add-port=443/tcp # HTTPS access
sudo firewall-cmd --reload                       # Apply changes
sudo firewall-cmd --list-all                     # Verify rules
```

You can use either UFW (default) or firewalld depending on your preference.

# 🚀 Email Notification Test
Simulate a ban to trigger an email:
```
sudo fail2ban-client set nginx-limit-req banip 127.0.0.2
```

Check msmtp logs:
```
tail -f /var/log/msmtp.log
```

Unban the test IP:
```
sudo fail2ban-client set nginx-limit-req unbanip 127.0.0.2
```

# 🤝 Contributing
Pull requests are welcome to improve this guide, add new filters, or extend firewall configurations.  
Let’s build a more secure Linux environment together! 🚀

# 📜 License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.
