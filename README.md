# n8n on AWS

A simple setup guide for running n8n on an AWS EC2 instance.

---

## 1. Update System & Install Git

```bash
sudo dnf update -y
sudo dnf install -y git
```

---

## 2. Clone Repository

```bash
git clone https://github.com/tylerprogramming/n8n-aws.git
cd n8n-aws
```

---

## 3. Install Docker

```bash
sudo dnf install -y docker
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user
```

> **Note:** Log out and back in for group changes to take effect.

---

## 4. Install Docker Compose

```bash
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
```

---

## 5. Setup Swap (Recommended for t2.micro)

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## 6. Configure Your Public IP

Before running n8n, you need to add your EC2 public IP to the configuration.

### Find Your Public IP

In the AWS Console, go to **EC2 → Instances → Your Instance** and copy the **Public IPv4 address**.

### Edit the docker-compose.yml

Open the file with nano:

```bash
nano docker-compose.yml
```

Add these lines under the `environment:` section (replace `YOUR_PUBLIC_IP` with your actual IP):

```yaml
      - N8N_HOST=YOUR_PUBLIC_IP
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://YOUR_PUBLIC_IP:5678/
      - N8N_EDITOR_BASE_URL=http://YOUR_PUBLIC_IP:5678
```

### Example (if your IP is 3.149.239.36):

Your environment section should look like this:

```yaml
    environment:
      - GENERIC_TIMEZONE=America/New_York
      - TZ=America/New_York
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_RUNNERS_ENABLED=true
      - N8N_SECURE_COOKIE=false
      - N8N_HOST=18.216.241.245
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://18.216.241.245:5678/
      - N8N_EDITOR_BASE_URL=http://18.216.241.245:5678
```

### Save and Exit

1. Press `Ctrl + O` to save
2. Press `Enter` to confirm
3. Press `Ctrl + X` to exit

---

## 7. Run n8n

```bash
docker compose pull
docker compose up -d
```

---

## 8. Troubleshooting

If something fails, try restarting Docker:

```bash
sudo systemctl restart docker
docker compose down
docker compose up -d
```

---

## 9. Update Security Group

Add the following inbound rule to your EC2 Security Group:

| Type       | Port     | Source      |
| ---------- | -------- | ----------- |
| Custom TCP | **5678** | **My IP**   |

In the AWS console, pick **My IP** in the Source dropdown; it fills in your address as a `/32`. Do not use `0.0.0.0/0`. This setup serves n8n over plain HTTP (`N8N_PROTOCOL=http`, `N8N_SECURE_COOKIE=false`), so anything you type, including your n8n password, crosses the internet unencrypted, and anyone on the internet can reach the editor and its login page.

If your home IP changes, edit the rule and pick **My IP** again.

---

## 10. Access n8n and create the owner account

Open in your browser:

```
http://YOUR_PUBLIC_IP:5678
```

Replace `YOUR_PUBLIC_IP` with your EC2 instance's public IP address.

On first open, n8n shows a signup screen to create the **owner account**. Do this right away: until an owner exists, whoever reaches port 5678 first can create it and take over the instance. Use a strong password (n8n requires at least 8 characters with a number and a capital letter).

n8n 1.0 and later always require this login. The old `N8N_BASIC_AUTH_*` variables were removed in 1.0 and do nothing ([n8n user management docs](https://docs.n8n.io/deploy/host-n8n/configure-n8n/user-management), [v1.0 migration guide](https://docs.n8n.io/changelog/v10-migration-guide)).
