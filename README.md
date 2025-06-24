# Laravel Blog Tutorial - 5-Day Project

A comprehensive Laravel blog tutorial project built with Laravel 12, Livewire 3, and Docker. This tutorial is designed for learning modern web development practices over 5 structured days.

## 📋 Project Overview

This tutorial project teaches you how to build a complete blog application with:
- Modern Laravel 12 features
- Interactive Livewire components
- Docker development environment
- User authentication with Sanctum
- Advanced comment system with nested replies
- Admin dashboard with CRUD operations
- Responsive design with Tailwind CSS

## 🚀 Quick Start

### Prerequisites
- Docker and Docker Compose installed
- Git installed
- Basic knowledge of PHP (recommended)

### Getting Started
```bash
# Clone the repository
git clone <your-repo-url>
cd practica2025

docker compose up -d --build

# Access your application
open http://localhost:8000

phpmyadmin: http://localhost:8080

```

## 📚 Tutorial Structure

### [Day 1: Foundation Setup](./day1/README.md)
**Duration**: 1-2 hours
- Git tutorial and GitHub setup
- Docker configuration for Laravel
- Clean Laravel 12 + Livewire 3 installation
- Initial project setup and deployment

**What you'll learn**:
- Git basics and version control
- Docker containers and development environment
- Laravel project structure
- Livewire basics

### [Day 2: Authentication & Database](./day2/README.md)
**Duration**: 1-2 hours
- Laravel Sanctum authentication setup
- Database migrations for blog posts and comments
- User roles and middleware configuration
- Environment configuration

**What you'll learn**:
- Modern Laravel authentication
- Database relationships and migrations
- Middleware and authorization
- Laravel Sanctum for API authentication

### [Day 3: Data & Frontend](./day3/README.md)
**Duration**: 1-2 hours
- Database seeders and factories
- Frontend controllers for blog display
- Tailwind CSS styling and responsive design
- Blog layout and navigation

**What you'll learn**:
- Database seeding and factories
- Laravel controllers and routing
- Tailwind CSS for styling
- Responsive web design principles

### [Day 4: Interactive Components](./day4/README.md)
**Duration**: 1-2 hours
- Livewire components for blog post CRUD
- Interactive admin dashboard
- Real-time form validation
- Modal forms and user interactions

**What you'll learn**:
- Advanced Livewire components
- Real-time interactions
- Form validation and error handling
- Component communication

### [Day 5: Advanced Features](./day5/README.md)
**Duration**: 1-2 hours
- Advanced comment system with nested replies
- Custom Livewire components
- Real-time comment management
- Admin approval workflow

**What you'll learn**:
- Complex component relationships
- Nested data structures
- Event-driven programming
- Admin interfaces and workflows

## 🛠 Tech Stack

- **Backend**: Laravel 12
- **Frontend**: Livewire 3, Alpine.js
- **Styling**: Tailwind CSS
- **Database**: MySQL 8.0
- **Authentication**: Laravel Sanctum
- **Development**: Docker, Docker Compose
- **Server**: Nginx, PHP 8.3-FPM

```

## 🎯 Learning Objectives

By the end of this tutorial, you will:

1. **Master Laravel 12**: Understand modern Laravel features and best practices
2. **Build Interactive UIs**: Create dynamic interfaces with Livewire 3
3. **Handle Authentication**: Implement secure user authentication with Sanctum
4. **Design Databases**: Create efficient database structures and relationships
5. **Style with Tailwind**: Build responsive, modern web interfaces
6. **Deploy with Docker**: Set up professional development environments
7. **Manage State**: Handle complex application state with Livewire
8. **Build Admin Interfaces**: Create powerful backend management tools

## 🌟 Key Features Implemented

### User Features
- ✅ User registration and authentication
- ✅ Browse blog posts with pagination
- ✅ Read individual blog posts
- ✅ Post comments and replies
- ✅ Edit and delete own comments
- ✅ Nested comment system
- ✅ Responsive design for mobile/desktop

### Admin Features
- ✅ Admin dashboard with statistics
- ✅ Full blog post CRUD operations
- ✅ Comment management and approval
- ✅ User management
- ✅ Real-time updates with Livewire
- ✅ Search and filter functionality

### Technical Features
- ✅ Docker development environment
- ✅ Laravel 12 with modern features
- ✅ Livewire 3 for interactivity
- ✅ Tailwind CSS for styling
- ✅ MySQL database with proper relationships
- ✅ Laravel Sanctum authentication
- ✅ Middleware for authorization
- ✅ Form validation and error handling

After completing all 5 days:
- Add your own features
- Customize the design
- Deploy to production
- Add additional functionality

## 🔧 Development Commands

### Docker Commands (run from project folder)
```bash
# Start services
docker compose up -d

# View logs
docker compose logs -f

# Access PHP container
docker compose exec app bash

# Stop services
docker compose down
```

### Laravel Commands (inside container)
```bash
# Install dependencies
composer install

# in day 3 an onwards
npm i && npm run build

# Run migrations
php artisan migrate

# Seed database
php artisan db:seed

# Clear caches
php artisan optimize:clear

# Create Livewire component
php artisan make:livewire ComponentName
```

## 🐛 Troubleshooting

### Common Issues

1. **Port conflicts**: If you need to change them, edit the docker-compose.yml in the folder
2. **Permission issues**: Run `sudo chown -R $USER:$USER .` from the folder
3. **Cache issues**: Run `php artisan optimize:clear` inside the container for the specific day
4. **Database connection**: Check .env file settings in the folder

### Getting Help
- Check individual day README files
- Review Laravel documentation
- Check Livewire documentation
- Look at troubleshooting sections in each day

## 📈 What's Next?

After completing this tutorial, consider:

### Immediate Extensions
- Add user profiles and avatars
- Implement post categories and tags
- Add email notifications
- Create rich text editor

### Advanced Features
- Build REST API endpoints
- Add search functionality with Elasticsearch
- Implement caching strategies
- Add automated testing

### Production Deployment
- Set up CI/CD pipeline
- Configure production Docker setup
- Implement monitoring and logging
- Set up backup strategies

## 📚 Additional Resources

### Documentation
- [Laravel 12 Documentation](https://laravel.com/docs/12.x)
- [Livewire 3 Documentation](https://livewire.laravel.com/docs/quickstart)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Docker Documentation](https://docs.docker.com/)

### Learning Materials
- Laravel from Scratch (Laracasts)
- Livewire Screencasts
- Tailwind CSS tutorials
- Docker for beginners

## 🤝 Contributing

This is a tutorial project, but improvements are welcome:
1. Fork the repository
2. Create a feature branch
3. Make your improvements
4. Submit a pull request

## 📄 License

This tutorial project is open-source and available under the MIT License.

## 🙋‍♂️ Support

If you encounter issues or have questions:
1. Check the troubleshooting sections in each day's README
2. Review the Laravel and Livewire documentation
3. Create an issue in the repository
4. Join the Laravel community forums

---

**Happy Learning!** 🎉

Start your journey with [Day 1](./day1/README.md) and build your first modern Laravel blog application.