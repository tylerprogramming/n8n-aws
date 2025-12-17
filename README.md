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

## 6. Run n8n

```bash
docker compose pull
docker compose up -d
```

---

## 7. Troubleshooting

If something fails, try restarting Docker:

```bash
sudo systemctl restart docker
docker compose down
docker compose up -d
```

---

## 8. Update Security Group

Add the following inbound rule to your EC2 Security Group:

| Type       | Port     | Source                                 |
| ---------- | -------- | -------------------------------------- |
| Custom TCP | **5678** | **Your IP** (or `0.0.0.0/0` temporarily) |

---

## 9. Access n8n

Open in your browser:

```
http://YOUR_PUBLIC_IP:5678
```

Replace `YOUR_PUBLIC_IP` with your EC2 instance's public IP address.
