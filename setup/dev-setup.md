# Install Odoo Docker with Nginx

## Remove Conflicting Packages
Before proceeding with the clean installation of Docker, it is recommended to remove any conflicting packages:
```bash
sudo apt-get remove docker docker-engine docker.io docker-doc docker-compose podman-docker containerd runc
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```
Also, the images, containers, volumes, custom configuration files, etc., are not removed automatically. To remove these:
```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

## Installation

### Update the Package Index
```bash
sudo apt-get update
```

### Install Necessary Packages
Install necessary packages to enable access to the Docker repositories over HTTPS:
```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg software-properties-common
```

### Add Docker’s Official GPG Key
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

### Add the Docker Repository
```bash
echo \  
"deb [arch="$(dpkg --print-architecture)" \  
signed-by=/etc/apt/keyrings/docker.gpg] \  
https://download.docker.com/linux/ubuntu \  
"$(. /etc/os-release && echo "$UBUNTU_CODENAME")" stable" | \  
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Ensure the Source is from the Docker Repo
```bash
apt-cache policy docker-ce
```

### Update the Local Package Index
```bash
sudo apt-get update
```

### Install Docker
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-compose
```

### Check if Docker is Running
```bash
sudo systemctl status docker
```

## Install Odoo in Docker
### Create Necessary Directories
```bash
mkdir -p ~/docker/odoo
cd ~/docker/odoo
mkdir config
nano odoo.conf
```

### Odoo Configuration File (`odoo.conf`)
```ini
[options]
addons_path = /var/lib/odoo,/mnt/extra-addons/bookds
default_productivity_apps = True
limit_memory_soft = 2147483648 
limit_memory_hard = 4294967296
limit_time_cpu = 600
limit_time_real = 1200
proxy_mode = True
```

### Clone Custom Addons Repository
```bash
mkdir custom-addons
git clone "custom code repo"
```

### Create a Docker Compose File (`docker-compose.yml`)
```yaml
version: '3.1'
services:
  web:
    image: odoo:18.0
    depends_on:
      - db
    ports:
      - "8069:8069"
    volumes:
      - odoo-web-data:/var/lib/odoo
      - ./config:/etc/odoo
      - ./addons:/mnt/extra-addons
    environment:
      - PASSWORD_FILE=/run/secrets/postgresql_password
    secrets:
      - postgresql_password
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_PASSWORD_FILE=/run/secrets/postgresql_password
      - POSTGRES_USER=odoo
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - odoo-db-data:/var/lib/postgresql/data/pgdata
    secrets:
      - postgresql_password
volumes:
  odoo-web-data:
  odoo-db-data:

secrets:
  postgresql_password:
    file: odoo_pg_pass
```

### Create PostgreSQL Password File (`odoo_pg_pass`)
```bash
nano odoo_pg_pass
```
Add the following content:
```bash
add the password in odoo_pg_pass
```

### Start Odoo with Docker Compose
```bash
docker-compose up -d
```

## Install and Configure Nginx

### Create Nginx Configuration File (`/etc/nginx/sites-available/nginx.conf`)
```nginx
server {
    listen 80;
    server_name "$domainname;

    location / {
        proxy_pass http://127.0.0.0:8069/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Enable Nginx Configuration and Restart
```bash
sudo ln -s /etc/nginx/sites-available/nginx.conf /etc/nginx/sites-enabled/
sudo service nginx restart
```
