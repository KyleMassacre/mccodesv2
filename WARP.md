# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Development Environment

MCCodes v2 is a PHP-based browser game engine with MySQL database backend. This is an open-source mafia/crime game framework under MIT license.

### Docker Development Setup

The project includes Docker configuration for local development:

```bash
# Start the development environment
docker-compose up -d

# Access the application at http://localhost:4567
# phpMyAdmin available at http://localhost:4569
# MySQL on port 3306 (root password: rootpass1)

# If you encounter mysqli/GD module issues, exec into the container:
docker exec -it <container_name> bash
apt install libpng-dev -y && docker-php-ext-install mysqli gd
# Optionally install additional extensions:
docker-php-ext-install mbstring pdo pdo_mysql sockets sodium xsl
# Restart Apache after installation
```

### Installation Process

```bash
# Initial setup (first time only)
# Navigate to http://localhost:4567/installer.php
# Follow the web-based installation wizard
# Database: mccv2, User: root, Password: rootpass1, Host: db (for Docker)

# Set up cron jobs (production only)
# MCCodes requires 4 cron jobs for proper operation:
# */1 * * * * php /path/to/game/crons/CronHandler.php cron=minute-1 code=YOUR_CRON_CODE
# */5 * * * * php /path/to/game/crons/CronHandler.php cron=minute-5 code=YOUR_CRON_CODE  
# 0 * * * * php /path/to/game/crons/CronHandler.php cron=hour-1 code=YOUR_CRON_CODE
# 0 0 * * * php /path/to/game/crons/CronHandler.php cron=day-1 code=YOUR_CRON_CODE
```

### Database Management

```bash
# Import database schema (initial setup)
mysql -u root -p mccv2 < dbdata.sql

# Backup database
mysqldump -u root -p mccv2 > backup.sql

# Reset database (development)
mysql -u root -p -e "DROP DATABASE mccv2; CREATE DATABASE mccv2;"
mysql -u root -p mccv2 < dbdata.sql
```

## Architecture Overview

### Core Application Structure

- **Entry Points**: Each game feature is a separate PHP file in the root directory
- **Authentication**: `globals.php` handles user authentication and session management
- **Database Layer**: Custom database abstraction in `class/class_db_mysqli.php`
- **Global Functions**: Utility functions in `global_func.php`
- **Cron System**: Automated tasks managed by `CronHandler.php`

### Key Directories

- `class/` - Core classes (database, BBCode engine)
- `crons/` - Background task system and scheduled jobs
- `css/` - Stylesheets for different sections
- `lib/` - Error handlers and utility libraries

### Database Architecture

The game uses a MySQL database with the following key table categories:

- **User Management**: `users`, `userstats`, `users_roles`, `staff_roles`
- **Game Content**: `items`, `crimes`, `jobs`, `houses`, `cities`, `courses`
- **Game Systems**: `inventory`, `gangs`, `forums`, `events`, `mail`
- **Logging**: `attacklogs`, `cashxferlogs`, `bankxferlogs`, `stafflog`
- **Administration**: `settings`, `announcements`, `blacklist`

### Authentication & Security

- Session-based authentication with `MCCSID` session name
- CSRF protection using `request_csrf_code()` and `verify_csrf_code()`
- Password hashing with salt using `encode_password()` and `generate_pass_salt()`
- Role-based access control via `check_access()` function
- Input sanitization through `$db->escape()` method

### Key Game Systems

1. **User Stats System**: Tracks player progression with experience, levels, energy, health
2. **Item System**: Complex item management with effects, types, and inventory
3. **Crime System**: Crime mechanics with groups and success rates  
4. **Gang System**: Player organizations with hierarchy and wars
5. **Job System**: Employment with ranks and progression
6. **Housing System**: Property ownership affecting player stats
7. **Combat System**: Player vs player attacks with detailed logs
8. **Economy**: Multiple currencies (money, crystals, cyber bank)

### Cron Job System

The game relies heavily on scheduled tasks for:

- **minute-1**: Energy/health regeneration, basic maintenance
- **minute-5**: Intermediate game updates
- **hour-1**: Major game mechanics, daily resets
- **day-1**: Statistics, cleanup, major events

Cron jobs can be run via traditional cron or the newer timestamp-based system (`use_timestamps_over_crons` setting).

## Development Guidelines

### Code Style

- Uses PHP 8.3+ features including strict typing (`declare(strict_types=1)`)
- Modern PHP practices with type declarations and union types
- Database queries use the custom database class abstraction
- HTML generation mixed with PHP (traditional PHP web development style)

### Common Patterns

- Global variables: `$db` (database), `$ir` (user data), `$userid`, `$set` (settings)
- Page structure: Include `globals.php` → Process logic → Display HTML
- Database operations use `$db->query()`, `$db->fetch_row()`, `$db->escape()`
- User data type casting via `set_userdata_data_types($ir)`
- Event logging with `event_add($userid, $text)`
- Staff actions logged via `stafflog_add($text)`

### Key Functions for Development

- `money_formatter()` - Format currency display
- `item_add()`, `item_remove()` - Inventory management  
- `event_add()` - Send events to users
- `check_level()` - Handle user level ups
- `get_rank()` - Get user's ranking for stats
- Various dropdown functions for forms: `item_dropdown()`, `user_dropdown()`, etc.

### Security Considerations

- Always use `$db->escape()` for user input in SQL queries
- Use CSRF protection for state-changing operations
- Validate user permissions with `check_access()` for staff functions
- Sanitize output for HTML display to prevent XSS

### Testing Database Changes

```bash
# Backup before major changes
mysqldump -u root -p mccv2 > pre_change_backup.sql

# Test database modifications
mysql -u root -p mccv2 < your_changes.sql

# Restore if needed
mysql -u root -p -e "DROP DATABASE mccv2; CREATE DATABASE mccv2;"
mysql -u root -p mccv2 < pre_change_backup.sql
```

This is a legacy PHP codebase following traditional web development patterns. When making changes, maintain consistency with existing code style and architecture patterns.
