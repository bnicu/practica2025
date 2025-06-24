# Day 1: Git Setup, Docker Configuration & Laravel+Livewire Installation

## Duration: 1-2 hours

### Objectives
- Learn basic Git operations
- Set up GitHub account and repository
- Configure Docker environment for Laravel
- Install clean Laravel with Livewire

---

## Part 1: Git Tutorial & GitHub Setup (30 minutes)

### 1. Create GitHub Account
1. Go to [github.com](https://github.com)
2. Click "Sign up" and create your account
3. Verify your email address

### 2. Basic Git Commands
```bash
# Configure Git (replace with your info)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Check configuration
git config --list

# Initialize a repository
git init

# Add files to staging
git add .
git add filename.txt

# Commit changes
git commit -m "Your commit message"

# Check status
git status

# View commit history
git log --oneline
```

### 3. Connect to GitHub
```bash
# Add remote repository
git remote add origin https://github.com/yourusername/your-repo-name.git

# Push to GitHub
git push -u origin main

# Clone a repository
git clone https://github.com/username/repository.git

# Pull latest changes
git pull origin main
```

---

## Part 2: Docker Configuration (30 minutes)

### Prerequisites
- Docker installed on your system
- Docker Compose installed

### Docker Configuration Files

Create the following files in your project root:

#### docker-compose.yml
```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: laravel-blog-app
    restart: unless-stopped
    working_dir: /var/www/html
    volumes:
      - ./:/var/www/html
      - ./docker/php/local.ini:/usr/local/etc/php/conf.d/local.ini
    networks:
      - laravel-blog
    depends_on:
      - db

  webserver:
    image: nginx:alpine
    container_name: laravel-blog-webserver
    restart: unless-stopped
    ports:
      - "8000:80"
    volumes:
      - ./:/var/www/html
      - ./docker/nginx/conf.d/:/etc/nginx/conf.d/
    networks:
      - laravel-blog
    depends_on:
      - app

  db:
    image: mysql:8.0
    container_name: laravel-blog-db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: laravel_blog
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_PASSWORD: user_password
      MYSQL_USER: laravel_user
    volumes:
      - dbdata:/var/lib/mysql
    ports:
      - "3306:3306"
    networks:
      - laravel-blog

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: laravel-blog-phpmyadmin
    restart: unless-stopped
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_ARBITRARY: 1
    ports:
      - "8080:80"
    networks:
      - laravel-blog
    depends_on:
      - db

networks:
  laravel-blog:
    driver: bridge

volumes:
  dbdata:
    driver: local
```

#### Dockerfile
```dockerfile
FROM php:8.3-fpm

# Set working directory
WORKDIR /var/www/html

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libzip-dev \
    zip \
    unzip

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd zip

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Create system user to run Composer and Artisan Commands
RUN useradd -G www-data,root -u 1000 -d /home/laravel laravel
RUN mkdir -p /home/laravel/.composer && \
    chown -R laravel:laravel /home/laravel

# Set user
USER laravel

# Copy existing application directory contents
COPY --chown=laravel:laravel . /var/www/html

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
```

### Docker Support Files

Create directory structure:
```
docker/
  php/
     local.ini
  nginx/
     conf.d/
        app.conf
```

#### docker/php/local.ini
```ini
upload_max_filesize=40M
post_max_size=40M
memory_limit=512M
max_execution_time=300
```

#### docker/nginx/conf.d/app.conf
```nginx
server {
    listen 80;
    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/html/public;

    client_max_body_size 20M;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }

    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }
}
```

### Starting the Environment
```bash
# Build and start containers
docker compose up -d --build

# Check running containers
docker compose ps

# Access application container
docker compose exec app bash

# Stop containers
docker compose down

# Stop and remove volumes
docker compose down -v
```

---

## Part 3: Laravel Installation (45 minutes)

### 1. Create Laravel Project
```bash
# Inside the app container
composer create-project laravel/laravel . "^12.0"

# Or if running locally
composer create-project laravel/laravel laravel-blog "^12.0"
```

### 2. Environment Configuration
```bash
# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate
```

#### Update .env file:
```env
APP_NAME="Laravel Blog Tutorial"
APP_ENV=local
APP_KEY=base64:generated_key_here
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel_blog
DB_USERNAME=laravel_user
DB_PASSWORD=user_password
```

### 3. Install Livewire
```bash
# Install Livewire
composer require livewire/livewire

# Publish Livewire assets (optional)
php artisan livewire:publish --config
```

### 4. Basic Livewire Setup

#### Create a basic Livewire component
```bash
php artisan make:livewire HelloWorld
```

This creates:
- `app/Livewire/HelloWorld.php`
- `resources/views/livewire/hello-world.blade.php`

#### Update the welcome view
**resources/views/welcome.blade.php**
```php
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Laravel Blog Tutorial</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
    @livewireStyles
</head>
<body>
    <div class="container mx-auto px-4 py-8">
        <h1 class="text-3xl font-bold text-center mb-8">Laravel Blog Tutorial - Day 1</h1>
        @livewire('hello-world')
    </div>
    @livewireScripts
</body>
</html>
```

### 5. Test Installation
```bash
# Run migrations
php artisan migrate

# Start development server (if not using Docker)
php artisan serve

# If using Docker, visit http://localhost:8000
```

---

## Part 4: Initial Git Commit (15 minutes)

### Create GitHub Repository
1. Go to GitHub and create a new repository named "laravel-blog-tutorial"
2. Don't initialize with README (we'll push existing code)

### Initial Commit
```bash
# Initialize git (if not already done)
git init

# Add all files
git add .

# Create .gitignore (Laravel should have one already)
# Make sure it includes:
# /vendor
# /node_modules
# .env
# /storage/*.log

# Commit
git commit -m "Day 1: Initial Laravel + Livewire setup with Docker configuration"

# Add remote and push
git remote add origin https://github.com/yourusername/laravel-blog-tutorial.git
git branch -M main
git push -u origin main
```

---

## Verification Checklist

- [ ] GitHub account created and repository set up
- [ ] Docker containers running (app, webserver, db, phpmyadmin)
- [ ] Laravel application accessible at http://localhost:8000
- [ ] PHPMyAdmin accessible at http://localhost:8080
- [ ] Livewire component displaying on homepage
- [ ] Database connection working
- [ ] Initial commit pushed to GitHub

---

## Troubleshooting

### Common Issues:

1. **Port conflicts**: Change ports in docker-compose.yml if 8000, 8080, or 3306 are occupied
2. **Permission issues**: Run `sudo chown -R $USER:$USER .` in project directory
3. **Composer issues**: Clear composer cache: `composer clear-cache`
4. **Docker issues**: Restart Docker service and rebuild: `docker compose down && docker compose up -d --build`

### Useful Commands:
```bash
# View container logs
docker compose logs app
docker compose logs webserver
docker compose logs db

# Restart a specific service
docker compose restart app

# Access MySQL directly
docker compose exec db mysql -u laravel_user -p laravel_blog
```

---

## Next Steps
Tomorrow (Day 2) we'll implement:
- Laravel Sanctum authentication
- Database migrations for blog posts and comments
- Environment configuration for different stages