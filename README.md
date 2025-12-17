- sudo dnf update -y
- sudo dnf install -y git

- git clone https://github.com/tylerprogramming/n8n-aws.git

- sudo dnf install -y docker
- sudo systemctl enable docker
- sudo systemctl start docker
- sudo usermod -aG docker ec2-user

- sudo mkdir -p /usr/local/lib/docker/cli-plugins
- sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 \
  -o /usr/local/lib/docker/cli-plugins/docker-compose
- sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

- sudo fallocate -l 2G /swapfile
- sudo chmod 600 /swapfile
- sudo mkswap /swapfile
- sudo swapon /swapfile
- echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

- docker compose pull
- docker compose up -d

# IF something fails
- sudo systemctl restart docker
- docker compose down
- docker compose up -d

# Update Security Group
| Type       | Port     | Source                                 |
| ---------- | -------- | -------------------------------------- |
| Custom TCP | **5678** | **Your IP** (or 0.0.0.0/0 temporarily) |

http://YOUR_PUBLIC_IP:5678
