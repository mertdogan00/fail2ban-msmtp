
# 🛡️ Fail2ban + msmtp E-mail Integration for Linux Servers

Protect your Linux server with **Fail2ban real-time banning** and receive instant email alerts via **msmtp external SMTP integration.**  
This is a focused guide for email notification setup. 🔒

For a complete Linux server manual setup, see 👉 https://github.com/mertdogan00/server-manual-setup


---

## 📚 Table of Contents
- [Project Overview 🗂️](#-project-overview)
- [Installation 🛠️](#-installation)
- [msmtp Configuration ✉️](#-msmtp-configuration)
- [Fail2ban Mail Setup ⚙️](#-fail2ban-mail-setup)
- [Email Notification Test 🚀](#-email-notification-test)
- [Contributing 🤝](#-contributing)
- [License 📜](#-license)

---

## 🗂 Project Overview
This repository focuses on configuring **Fail2ban** to work with **msmtp** for sending real-time email notifications.

- Supports external SMTP servers like Gmail, Yandex, or custom mail servers.
- Fully compatible with **UFW (default firewall)** or **firewalld.**

🔗 Related Project: [Complete Linux Server Setup](https://github.com/mertdogan00/server-manual-setup)

---

## 🛠 Installation
Install msmtp and Fail2ban:

```
sudo apt update
sudo apt install msmtp msmtp-mta fail2ban -y
```

---

## ✉ msmtp Configuration
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



### ✅ Important Notes:
- The `from` and `user` addresses must usually be the same for most SMTP servers.
- The password should be an application-specific password if required.

Test your email:
```
echo -e "Subject: Fail2ban Test Mail\n\nThis is a test email from msmtp." | msmtp yourmail@domain.com
```

---

## ⚙ Fail2ban Mail Setup
Open Fail2ban configuration file:
```
sudo nano /etc/fail2ban/jail.local
```
```
Example configuration:
[DEFAULT]

# 📧 Recipient Email
destemail = yourmail@domain.com

# 📤 Sender Email (should match msmtp 'from')
sender = yourmail@domain.com

# 📦 Mail Transfer Agent
mta = sendmail

# 🛑 Action: Ban + Detailed Email (Log + Whois Info)
action = %(action_mwl)s

# ⏱️ Ban Duration (seconds)
bantime = 600

# ⌛ Find Time Window (seconds)
findtime = 300

# ❌ Maximum Retry Before Ban
maxretry = 5

```

Restart Fail2ban:

```
sudo systemctl restart fail2ban
```
---

## 🚀 Email Notification Test
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

---

## 🤝 Contributing
Pull requests are welcome to improve this guide or add new use cases.  
Let’s build a secure and well-documented system together! 🚀

---

## 📜 License 
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.
