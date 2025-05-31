# GitLab Deployment Guide (Rocky Linux 9)

📅 Last Updated: 2025-05-31  
👤 Maintainer: xianzhan-yang  
🖥️ OS: Rocky Linux 9  
🔧 GitLab Version: Latest CE Omnibus  
🌐 Access: Web UI + SSH

## 📦 Overview

- GitLab is a web-based DevOps lifecycle tool providing Git repository management, CI/CD, and more.
- This guide uses GitLab's **Omnibus package** for an all-in-one install.
- Tested on Rocky Linux 9 with SELinux enabled and firewalld active.

## 🔧 Installation

### 1. Install required dependencies

```bash
dnf install -y curl policycoreutils-python-utils openssh-server firewalld
systemctl enable --now firewalld
systemctl enable --now sshd
```

### 2. Add GitLab package repository

```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | bash
```

### 3. Install GitLab CE

```bash
EXTERNAL_URL="http://gitlab.example.com"
dnf install -y gitlab-ce
```
Replace gitlab.example.com with your actual domain or server IP (temporarily, you can use http://<ip>).

## 🛠️ Configuration

**Reconfigure GitLab**

After installation, run:
```bash
gitlab-ctl reconfigure
```
This sets up Nginx, PostgreSQL, Redis, etc. (included with Omnibus).

**Configuration File**

- Main config: /etc/gitlab/gitlab.rb
- SSL, ports, and custom settings should be added there.
- After any change, run gitlab-ctl reconfigure.

## 🔐 Security Setup

### 1. Open necessary firewall ports

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

### 2. SELinux configuration

```bash
setsebool -P httpd_can_network_connect 1
```
If using a custom port:
```bash
semanage port -a -t http_port_t -p tcp <custom-port>
```

## 🔄 Service Management

```bash
# Start GitLab (should already be running post-install)
gitlab-ctl start

# Check status
gitlab-ctl status

# Restart all components
gitlab-ctl restart

# Tail logs
gitlab-ctl tail
```

## 🧪 Access & First Login

**Web UI**

Visit: http://<your-ip-or-domain>

**Default login**

- Username: root
- Password: Set on first login (will prompt)
You can also reset with:
```bash
gitlab-rake "gitlab:password:reset"
```

## 🚀 Post-Install Recommendations

1. Set up hostname/DNS
   - Configure your domain to point to the server
2. Set up SSL
   - Recommended: Let’s Encrypt (letsencrypt['enable'] = true in gitlab.rb)
3. Email delivery
   - Configure SMTP to enable password reset emails
4. User & group structure
   - Create groups/projects, set up access roles
5. Backups
   - Set up cron for scheduled backups: /etc/gitlab/gitlab.rb → gitlab_rails['backup_path']

## 🔁 Maintenance

**Update GitLab**

```bash
dnf update -y gitlab-ce
gitlab-ctl reconfigure
```

**Backup**

```bash
gitlab-backup create
```
Backups are stored in /var/opt/gitlab/backups

**Restore**

```bash
gitlab-backup restore BACKUP=timestamp
```

**Log locations**

- /var/log/gitlab/
- Use gitlab-ctl tail to view live logs

## 🧯 Troubleshooting

**Reconfigure fails**

- Check /var/log/gitlab/reconfigure/ logs
- Ensure hostname resolves and ports are open

**GitLab not accessible**

- Confirm gitlab-ctl status
- Check SELinux logs: ausearch -m AVC,USER_AVC -ts recent
- Check ports: ss -tulnp

## 📚 References

- GitLab Official Docs (https://docs.gitlab.com/omnibus/)
- GitLab CE RPM Guide (https://about.gitlab.com/install/)
- SELinux Guide (https://wiki.centos.org/HowTos/SELinux)
