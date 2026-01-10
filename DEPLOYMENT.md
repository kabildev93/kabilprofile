# GitHub CI/CD Setup for AWS EC2

This repository is configured with GitHub Actions for automatic deployment to AWS EC2.

## 📋 Prerequisites

1. **AWS EC2 Instance** running and accessible
2. **SSH Access** to your EC2 instance
3. **Web Server** installed on EC2 (Nginx, Apache, or other)

## 🔑 Required GitHub Secrets

Configure these secrets in your GitHub repository:

**Settings → Secrets and variables → Actions → New repository secret**

| Secret Name | Description | Example |
|------------|-------------|---------|
| `EC2_SSH_PRIVATE_KEY` | Your EC2 instance's SSH private key (PEM file content) | Contents of `your-key.pem` |
| `EC2_HOST` | Public IP or hostname of your EC2 instance | `ec2-xxx-xxx-xxx-xxx.compute.amazonaws.com` or `123.45.67.89` |
| `EC2_USER` | SSH username for your EC2 instance | `ubuntu`, `ec2-user`, or `admin` |
| `DEPLOY_PATH` | Path where files should be deployed on EC2 | `/var/www/html` or `/home/ubuntu/kabilprofile` |

## 🚀 Setting Up GitHub Secrets

### 1. EC2_SSH_PRIVATE_KEY

```bash
# On your local machine, display your private key
cat ~/.ssh/your-ec2-key.pem

# Copy the ENTIRE content including:
# -----BEGIN RSA PRIVATE KEY-----
# ...
# -----END RSA PRIVATE KEY-----
```

Paste this entire content into GitHub Secret `EC2_SSH_PRIVATE_KEY`.

### 2. EC2_HOST

Find your EC2 public IP or DNS:
- Go to AWS EC2 Console
- Select your instance
- Copy the **Public IPv4 address** or **Public IPv4 DNS**

### 3. EC2_USER

Common usernames by AMI:
- Amazon Linux: `ec2-user`
- Ubuntu: `ubuntu`
- Debian: `admin`
- Red Hat: `ec2-user`

### 4. DEPLOY_PATH

Choose where to deploy:
- For Nginx: `/var/www/html`
- For Apache: `/var/www/html`
- Custom location: `/home/ubuntu/kabilprofile`

## ⚙️ EC2 Instance Setup

### 1. Install Web Server (Nginx example)

```bash
# Connect to your EC2 instance
ssh -i your-key.pem ubuntu@your-ec2-ip

# Update system
sudo apt update && sudo apt upgrade -y

# Install Nginx
sudo apt install nginx -y

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 2. Configure Deployment Directory

```bash
# Create deployment directory
sudo mkdir -p /var/www/html

# Set permissions
sudo chown -R $USER:$USER /var/www/html
chmod -R 755 /var/www/html
```

### 3. Configure Security Group

In AWS Console:
1. Go to **EC2 → Security Groups**
2. Select your instance's security group
3. Add **Inbound Rules**:
   - SSH (Port 22) - Your IP
   - HTTP (Port 80) - Anywhere (0.0.0.0/0)
   - HTTPS (Port 443) - Anywhere (0.0.0.0/0)

### 4. Add GitHub Actions SSH Key (Alternative Method)

If you want to create a new SSH key specifically for GitHub Actions:

```bash
# On your local machine
ssh-keygen -t rsa -b 4096 -C "github-actions" -f ~/.ssh/github-actions-key

# Copy public key to EC2
ssh-copy-id -i ~/.ssh/github-actions-key.pub ubuntu@your-ec2-ip

# Copy private key content for GitHub Secret
cat ~/.ssh/github-actions-key
```

## 🔄 How It Works

1. **Push to main branch** triggers the deployment workflow
2. GitHub Actions connects to your EC2 instance via SSH
3. Files are synced using `rsync` (excluding .git, node_modules, etc.)
4. Web server can be restarted (optional, configure in workflow)
5. Deployment status is reported

## 📝 Nginx Configuration

Create a simple Nginx config for your site:

```bash
sudo nano /etc/nginx/sites-available/kabilprofile
```

Add this configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/kabilprofile /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 🎯 Testing the Deployment

1. Make a change to `index.html`
2. Commit and push to the `main` branch:
   ```bash
   git add .
   git commit -m "Test deployment"
   git push origin main
   ```
3. Go to **GitHub → Actions** tab to watch the deployment
4. Visit your EC2 instance's public IP to see the changes

## 🔧 Customizing the Workflow

Edit `.github/workflows/deploy.yml` to:
- Change deployment triggers
- Add build steps (npm install, build, etc.)
- Configure server restart commands
- Add notifications (Slack, Discord, email)

## 🛠️ Troubleshooting

### Permission Denied (publickey)
- Check that `EC2_SSH_PRIVATE_KEY` is correct
- Verify `EC2_USER` matches your AMI
- Ensure SSH port 22 is open in Security Group

### rsync: connection unexpectedly closed
- Check EC2 instance is running
- Verify `EC2_HOST` is correct
- Test SSH connection manually

### Files not updating
- Check `DEPLOY_PATH` permissions
- Verify Nginx/Apache is serving from correct directory
- Clear browser cache

## 📚 Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [Nginx Documentation](https://nginx.org/en/docs/)

## 🔒 Security Best Practices

1. Use SSH keys instead of passwords
2. Restrict SSH access to specific IPs when possible
3. Keep your private keys secure and never commit them
4. Regularly update your EC2 instance
5. Use IAM roles for additional AWS services
6. Consider using AWS Systems Manager Session Manager

---

**Need Help?** Check the Actions logs in your GitHub repository for detailed error messages.
