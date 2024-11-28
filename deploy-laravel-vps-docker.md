
# Deploy Laravel on VPS Using Docker

## **1. Prepare Your VPS**
1. Log in to your VPS via SSH.
2. Update your system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
3. Install Docker and Docker Compose:
   ```bash
   sudo apt install docker.io docker-compose -y
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

---

## **2. Install Laravel Project Locally (if not already set up)**
Ensure your Laravel project is ready to be containerized. If you already have your Laravel project, skip to the next step.

---

## **3. Create a Dockerfile**
In your Laravel project directory, create a `Dockerfile` to define the environment for the application.

Example `Dockerfile`:
```dockerfile
# Use the official PHP image with Apache
FROM php:8.1-apache

# Install necessary PHP extensions
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    curl \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Enable Apache mod_rewrite
RUN a2enmod rewrite

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set the working directory
WORKDIR /var/www/html

# Copy application files
COPY . .

# Install application dependencies
RUN composer install --no-dev --optimize-autoloader

# Set proper permissions for Laravel
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache

# Expose port 80
EXPOSE 80

# Start Apache
CMD ["apache2-foreground"]
```

---

## **4. Create a Docker Compose File**
In your Laravel project directory, create a `docker-compose.yml` file to orchestrate services (e.g., web server and database).

Example `docker-compose.yml`:
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: laravel_app
    ports:
      - "8080:80"
    volumes:
      - .:/var/www/html
    environment:
      - APP_ENV=production
      - APP_DEBUG=false
      - APP_KEY=${APP_KEY}
      - DB_HOST=db
      - DB_PORT=3306
      - DB_DATABASE=laravel
      - DB_USERNAME=root
      - DB_PASSWORD=root

  db:
    image: mysql:8.0
    container_name: laravel_db
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel
      MYSQL_USER: root
      MYSQL_PASSWORD: root
    volumes:
      - dbdata:/var/lib/mysql

volumes:
  dbdata:
```

---

## **5. Copy Your Laravel Project to VPS**
- Use SCP or Git to transfer your Laravel project to your VPS:
  ```bash
  scp -r /path/to/laravel-project user@vps_ip:/path/to/deploy/
  ```

---

## **6. Set Environment Variables**
Create a `.env` file in your Laravel project directory on the VPS (if not present). Ensure `APP_KEY` is set. If needed, generate it:
```bash
docker run --rm -v $(pwd):/app composer install --no-scripts --optimize-autoloader
php artisan key:generate
```

---

## **7. Build and Start the Containers**
On your VPS, navigate to your Laravel project directory and run:
```bash
docker-compose up -d --build
```

This command builds the Docker image and starts the containers.

---

## **8. Verify the Deployment**
- Access your Laravel application at `http://vps_ip:8080`.
- Check logs if needed:
  ```bash
  docker logs laravel_app
  docker logs laravel_db
  ```

---

## **9. Configure a Reverse Proxy (Optional)**
For production, you can set up Nginx as a reverse proxy to forward traffic from port 80 (or 443 for HTTPS) to your Laravel container.

---

## **10. Automate Deployment (Optional)**
Consider using tools like Git hooks, CI/CD pipelines, or Docker Swarm/Kubernetes for automated deployments.

---
