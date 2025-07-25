# 📘 Full Guide: Deploying a Static Website with Nginx on AWS (10 Pages)

## 📝 Overview

This comprehensive guide helps you deploy a static website using **AWS EC2 (Ubuntu)**, **Nginx**, and **Let's Encrypt**. You'll configure DNS, use `scp` to upload files, and validate HTTPS setup with `openssl`.

---

## 📌 Table of Contents

1. [Project Goals](#1-project-goals)  
2. [Prerequisites](#2-prerequisites)  
3. [Step-by-Step Instructions](#3-step-by-step-instructions)  
    - 3.1 [Buy a Domain](#31-buy-a-domain)  
    - 3.2 [Launch EC2 Instance](#32-launch-ec2-instance)  
    - 3.3 [SSH into Server](#33-ssh-into-server)  
    - 3.4 [Install and Configure Nginx](#34-install-and-configure-nginx)  
    - 3.5 [Upload Website Using SCP](#35-upload-website-using-scp)  
    - 3.6 [DNS: Set A Record](#36-dns-set-a-record)  
    - 3.7 [Verify Domain and IP](#37-verify-domain-and-ip)  
    - 3.8 [Install HTTPS Certificate](#38-install-https-certificate)  
    - 3.9 [Verify SSL Certificate](#39-verify-ssl-certificate)  
    - 3.10 [Enable Auto-Renew](#310-enable-auto-renew)  
4. [Folder Structure](#4-folder-structure)  
5. [Security Best Practices](#5-security-best-practices)  
6. [Advanced Configuration](#6-advanced-configuration)  
7. [Troubleshooting](#7-troubleshooting)  
8. [References](#8-references)

---

## 1. Project Goals

- Deploy a responsive HTML/CSS site on a cloud server
- Ensure HTTPS using free SSL (Let's Encrypt)
- Use your own domain name
- Learn DevOps-style deployment workflow

## 2. Prerequisites

- AWS Free Tier Account
- Domain Name (e.g., from Namecheap, Freenom)
- SSH key pair (.pem file)
- HTML/CSS files for website
- Terminal (Linux/Mac) or PuTTY (Windows)

---

## 3. Step-by-Step Instructions

### 3.1 Buy a Domain

You can purchase from:

- [Freenom](https://www.freenom.com) (Free domains)
- [Namecheap](https://www.namecheap.com)
- [Google Domains](https://domains.google)

Ensure access to manage DNS records.

### 3.2 Launch EC2 Instance

1. AWS Console > EC2 > Launch Instance
2. Select **Ubuntu Server 22.04 LTS**
3. t2.micro instance type (free tier)
4. Add security group:
    - SSH (22)
    - HTTP (80)
    - HTTPS (443)
5. Launch with key pair

### 3.3 SSH into Server

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

### 3.4 Install and Configure Nginx

```bash
sudo apt update && sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 3.5 Upload Website Using SCP

```bash
scp -i your-key.pem -r ./website ubuntu@<EC2_PUBLIC_IP>:/tmp
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
sudo mv /tmp/website/* /var/www/html/
```

### 3.6 DNS: Set A Record

Login to domain DNS settings and set:

| Type | Name | Value (Elastic IP) |
|------|------|---------------------|
| A    | @    | 13.55.xxx.xxx       |

### 3.7 Verify Domain and IP

```bash
dig yourdomain.com +short
```

### 3.8 Install HTTPS Certificate

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

### 3.9 Verify SSL Certificate

```bash
openssl s_client -connect yourdomain.com:443
```

Check for certificate chain and expiration.

### 3.10 Enable Auto-Renew

```bash
sudo systemctl list-timers | grep certbot
sudo certbot renew --dry-run
```

---

## 4. Folder Structure

```
/var/www/html/
├── index.html
├── style.css
└── images/
    └── logo.png
```

---

## 5. Security Best Practices

- Change default SSH port
- Use a firewall (UFW)
- Disable root login
- Set up fail2ban

```bash
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

---

## 6. Advanced Configuration

### Nginx Site Block (Optional)

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

To apply:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 7. Troubleshooting

| Problem                            | Solution                                                                 |
|------------------------------------|--------------------------------------------------------------------------|
| SSH permission denied              | Check key permissions (`chmod 400 key.pem`)                              |
| Domain not resolving               | Wait 10–30 minutes or fix DNS A record                                   |
| Nginx 403 or 404 error             | Check file permissions and locations in `/var/www/html/`                 |
| Certbot error: No matching domains | Ensure DNS points to EC2 and domain is active                            |
| HTTPS not working                  | Check if port 443 is allowed in security group                           |
| Certbot fails to renew             | Run `certbot renew --dry-run` and ensure renewal cron is active          |
| Site still using HTTP              | Force HTTPS in Nginx config or with Certbot redirection option           |
| `scp` connection refused           | EC2 may be down or firewall blocks port 22                               |
| Dig shows wrong IP                 | Wait for DNS propagation or correct the A record                         |
| SSL expired                        | Check cert renewal cron or rerun certbot                                 |

---

## 8. References

- [AWS EC2 Docs](https://docs.aws.amazon.com/ec2/)
- [Nginx Official Docs](https://nginx.org/en/docs/)
- [Certbot Guide](https://certbot.eff.org/)
- [Let's Encrypt](https://letsencrypt.org)
- [Linux SCP Guide](https://linuxize.com/post/how-to-use-scp-command/)
- [OpenSSL Toolkit](https://www.openssl.org/docs/)
- [UFW Firewall](https://help.ubuntu.com/community/UFW)

---



## 🎉 Done!

You now have a production-ready static site hosted on AWS EC2 with HTTPS.

---