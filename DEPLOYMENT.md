# Docker Deployment Guide

This guide explains how to deploy the Django application using Docker on an Ubuntu server in both development and production environments.

## Prerequisites

- Ubuntu server (20.04 or later)
- Docker and Docker Compose installed
- Git (to clone the repository)

### Install Docker on Ubuntu

```bash
# Update package index
sudo apt update

# Install required packages
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker's GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Add your user to docker group (optional, to run docker without sudo)
sudo usermod -aG docker $USER

# Verify installation
docker --version
docker compose version
```

## Development Deployment

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd dockertest2
```

### 2. Set Up Environment Variables

```bash
# Copy the example environment file
cp .env.dev.example .env.dev

# Edit the environment file if needed
nano .env.dev
```

### 3. Build and Run

```bash
# Build and start the development container
docker compose -f docker-compose.dev.yml up --build

# Or run in detached mode
docker compose -f docker-compose.dev.yml up -d --build
```

### 4. Access the Application

The application will be available at:
- **URL**: http://localhost:8000

### 5. Run Migrations

```bash
# Run database migrations
docker compose -f docker-compose.dev.yml exec web python manage.py migrate

# Create a superuser (optional)
docker compose -f docker-compose.dev.yml exec web python manage.py createsuperuser
```

### 6. Development Commands

```bash
# View logs
docker compose -f docker-compose.dev.yml logs -f

# Stop containers
docker compose -f docker-compose.dev.yml down

# Stop and remove volumes
docker compose -f docker-compose.dev.yml down -v

# Access Django shell
docker compose -f docker-compose.dev.yml exec web python manage.py shell

# Run tests
docker compose -f docker-compose.dev.yml exec web python manage.py test
```

## Production Deployment

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd dockertest2
```

### 2. Set Up Environment Variables

```bash
# Copy the example environment file
cp .env.prod.example .env.prod

# Edit the production environment file
nano .env.prod
```

**Important**: Update the following variables in `.env.prod`:
- `SECRET_KEY` - Generate a new secret key
- `ALLOWED_HOSTS` - Add your domain name and server IP
- `DB_PASSWORD` - Set a strong database password
- `POSTGRES_PASSWORD` - Must match DB_PASSWORD

### 3. Generate a Secret Key

```bash
# Generate a new Django secret key
python3 -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
```

### 4. Build and Run

```bash
# Build and start production containers
docker compose -f docker-compose.prod.yml up -d --build

# View logs
docker compose -f docker-compose.prod.yml logs -f
```

### 5. Run Migrations and Collect Static Files

```bash
# Wait for database to be ready, then run migrations
docker compose -f docker-compose.prod.yml exec web python manage.py migrate

# Create a superuser
docker compose -f docker-compose.prod.yml exec web python manage.py createsuperuser

# Static files are collected automatically during build
# But you can collect them manually if needed:
# docker compose -f docker-compose.prod.yml exec web python manage.py collectstatic --noinput
```

### 6. Access the Application

The application will be available at:
- **HTTP**: http://your-server-ip or http://your-domain.com
- **Admin Panel**: http://your-server-ip/admin/

### 7. Production Commands

```bash
# View logs
docker compose -f docker-compose.prod.yml logs -f web
docker compose -f docker-compose.prod.yml logs -f nginx
docker compose -f docker-compose.prod.yml logs -f db

# Restart services
docker compose -f docker-compose.prod.yml restart web
docker compose -f docker-compose.prod.yml restart nginx

# Stop containers
docker compose -f docker-compose.prod.yml down

# Backup database
docker compose -f docker-compose.prod.yml exec db pg_dump -U helloworld_user helloworld_db > backup.sql

# Restore database
docker compose -f docker-compose.prod.yml exec -T db psql -U helloworld_user helloworld_db < backup.sql
```

## Architecture

### Development Setup
- **Django Development Server** (port 8000)
- **SQLite Database** (file-based, stored in volume)
- **Source Code**: Mounted as volume for live reload

### Production Setup
- **Nginx** (reverse proxy, port 80/443)
- **Gunicorn** (WSGI server, 3 workers)
- **PostgreSQL** (database server)
- **Static Files**: Served by Nginx using WhiteNoise

## File Structure

```
.
├── Dockerfile.dev              # Development Dockerfile
├── Dockerfile.prod             # Production Dockerfile
├── docker-compose.dev.yml      # Development compose file
├── docker-compose.prod.yml     # Production compose file
├── .dockerignore               # Files to exclude from Docker build
├── .env.dev.example            # Development environment template
├── .env.prod.example           # Production environment template
├── requirements.txt            # Base Python dependencies
├── requirements-dev.txt        # Development dependencies
├── requirements-prod.txt       # Production dependencies
├── nginx/
│   └── nginx.conf              # Nginx configuration
├── helloworld_project/
│   └── settings.py             # Django settings (environment-aware)
└── manage.py                   # Django management script
```

## Security Considerations

1. **Never commit `.env.prod` to version control**
2. Use strong passwords for `SECRET_KEY` and database credentials
3. Keep `DEBUG=False` in production
4. Update `ALLOWED_HOSTS` with your actual domain
5. Consider setting up SSL/TLS certificates for HTTPS
6. Regularly update dependencies for security patches

## SSL/HTTPS Setup (Optional)

For production, you should set up SSL certificates. Here's a basic guide using Let's Encrypt:

```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Obtain SSL certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal is set up by default, but you can test it:
sudo certbot renew --dry-run
```

Then update `nginx/nginx.conf` to include SSL configuration.

## Troubleshooting

### Container won't start
```bash
# Check logs
docker compose -f docker-compose.prod.yml logs

# Check container status
docker ps -a
```

### Database connection errors
```bash
# Check if database is healthy
docker compose -f docker-compose.prod.yml ps

# Restart database
docker compose -f docker-compose.prod.yml restart db
```

### Permission issues
```bash
# Fix ownership issues
docker compose -f docker-compose.prod.yml exec web chown -R django:django /app
```

### Clear everything and start fresh
```bash
# Stop and remove all containers, networks, and volumes
docker compose -f docker-compose.prod.yml down -v

# Rebuild from scratch
docker compose -f docker-compose.prod.yml up -d --build
```

## Monitoring

### Check container resource usage
```bash
docker stats
```

### View running containers
```bash
docker ps
```

### Access container shell
```bash
# Development
docker compose -f docker-compose.dev.yml exec web bash

# Production
docker compose -f docker-compose.prod.yml exec web sh
```

## Updates and Maintenance

### Update application code
```bash
# Pull latest code
git pull origin main

# Rebuild and restart
docker compose -f docker-compose.prod.yml up -d --build

# Run migrations
docker compose -f docker-compose.prod.yml exec web python manage.py migrate
```

### Update dependencies
```bash
# Edit requirements files
# Then rebuild containers
docker compose -f docker-compose.prod.yml up -d --build
```
