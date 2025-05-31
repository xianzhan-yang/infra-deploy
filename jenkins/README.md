# Jenkins Deployment Guide (Rocky Linux 9)

📅 Last Updated: 2025-05-31  
👤 Maintainer: xianzhan-yang  
🖥️ OS: Rocky Linux 9  
🔧 Jenkins: Latest LTS version  
☕ Java: OpenJDK 17

## 📦 Overview

- Jenkins is an open-source automation server used for CI/CD.
- This guide uses the official Jenkins YUM repository and assumes a fresh Rocky Linux 9 system with SELinux enabled.
- Internet access is required.

## 🔧 Installation

### 1. Install Java (OpenJDK 17)

```bash
dnf install -y java-17-openjdk
```

### 2. Add Jenkins repository and import the GPG key

```bash
dnf install -y wget

wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

### 3. Install Jenkins

```bash
dnf upgrade -y
dnf install -y jenkins git
```

## 🛠️ Configuration 

**Key Files and Directories**

- Jenkins home: /var/lib/jenkins
- Configuration: /etc/sysconfig/jenkins
- Logs: /var/log/jenkins/jenkins.log

**Change Jenkins Port (optional)**

Edit /etc/sysconfig/jenkins and change:
```bash
JENKINS_PORT="8081"
```

## 🔐 Security Configuration

### 1. Open firewall port

```bash
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```

### 2. SELinux Configuration (if changing port)

Install tools if not available:
```bash
dnf install -y policycoreutils-python-utils
```
Then, allow custom port (if not using default 8080):
```bash
semanage port -a -t http_port_t -p tcp 8081
```

### 3. Enable network access for plugins

```bash
setsebool -P httpd_can_network_connect 1
```

## 🔄 Service Management

**Enable and start Jenkins**

```bash
systemctl enable jenkins
systemctl start jenkins
```

**Check service status and logs**

```bash
systemctl status jenkins
journalctl -u jenkins -f
```

## 🧪 First Login Verification

**Retrieve initial admin password**

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Access Jenkins in browser**

Go to: http://<your-server-ip>:8080

## 🚀 Post-Login Setup Recommendations

1. Install suggested plugins
2. Create an admin user
3. Set system URL (Manage Jenkins → Configure System)
4. Recommended plugins:
   - Git
   - Pipeline
   - Blue Ocean (pipeline visualization)
   - Role-based Authorization Strategy

## 🔁 Maintenance & Operations

**Upgrade Jenkins**

```bash
dnf update -y jenkins
systemctl restart jenkins
```

**Backup Important Data**

- Jenkins home: /var/lib/jenkins/
- Plugins: /var/lib/jenkins/plugins/

**Restart Jenkins after config change**

```bash
systemctl restart jenkins
```

## 🧯 Troubleshooting

**Web UI Not Accessible**

- Check if service is running
- Ensure port is open in firewall
- Confirm SELinux isn’t blocking the port

**View logs**

```bash
journalctl -xeu jenkins
tail -f /var/log/jenkins/jenkins.log
```

**Plugin installation fails**

- Check network/DNS
- Use system proxy if needed
- Review "Manage Jenkins → Plugin Manager" logs

## 📚 References

- Jenkins Official Installation Docs (https://www.jenkins.io/doc/book/installing/linux/)
- Rocky Linux Documentation (https://docs.rockylinux.org/)
- SELinux Management Tools (https://wiki.centos.org/HowTos/SELinux)
