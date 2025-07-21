# BCOE&M Docker Setup (Nginx + PHP-FPM)

This Docker setup provides a containerized environment for running BCOE&M (Brew Competition Online Entry & Management) locally using Nginx and PHP-FPM.

## Prerequisites

- Docker and Docker Compose installed on your system
- The BCOE&M application files

## Setup Instructions

1. **Create project directory structure:**
   ```bash
   mkdir bcoem-docker
   cd bcoem-docker
   ```

2. **Download BCOE&M:**
   - Download the latest BCOE&M release from their website
   - Extract the files into a subdirectory called `bcoem`

3. **Add Docker files:**
   - Save the `Dockerfile` in the root directory
   - Save the `docker-compose.yml` in the root directory
   - Save the `docker-entrypoint.sh` in the root directory
   - Save the `nginx.conf` in the root directory

4. **Directory structure should look like:**
   ```
   bcoem-docker/
   ├── Dockerfile
   ├── docker-compose.yml
   ├── docker-entrypoint.sh
   ├── nginx.conf
   ├── README.md
   └── bcoem/
       └── (BCOE&M application files)
   ```

5. **Build and start the containers:**
   ```bash
   docker-compose up -d
   ```

6. **Access the application:**
   - BCOE&M: http://localhost:8080/bcoem
   - phpMyAdmin: http://localhost:8081
   - MySQL: localhost:3306

## Services

- **nginx**: Web server running on Alpine Linux
- **php**: PHP 7.3 FPM for processing PHP files
- **db**: MariaDB 10.5 database server
- **phpmyadmin**: Database management interface

## Database Setup

The MySQL container is configured with:
- Database name: `bcoem`
- Username: `bcoem_user`
- Password: `bcoem_password`
- Root password: `rootpassword`

## Initial Setup

1. Navigate to http://localhost:8080/bcoem/setup.php
2. Follow the browser-based installation process
3. **IMPORTANT**: After setup is complete, you MUST secure your installation:
   ```bash
   # Option 1: Use sed to automatically update the file
   docker exec bcoem-php sed -i 's/$setup_free_access = TRUE;/$setup_free_access = FALSE;/g' /var/www/html/bcoem/site/config.php
   
   # Option 2: Edit manually
   vi ./bcoem/site/config.php
   # Change: $setup_free_access = TRUE;
   # To:     $setup_free_access = FALSE;
   ```

## Manual Database Import (if needed)

If the browser-based setup fails, you can import the baseline database:

1. Copy the baseline SQL file to the sql directory:
   ```bash
   mkdir sql
   cp bcoem/sql/bcoem_baseline_*.sql sql/
   ```

2. Restart the containers:
   ```bash
   docker-compose down
   docker-compose up -d
   ```

The SQL file will be automatically imported when the MySQL container starts.

## Troubleshooting

### Internal Server Error (500)

1. **Check if BCOE&M files are present:**
   ```bash
   ls -la ./bcoem/
   ```
   Make sure you've downloaded and extracted BCOE&M files into the `bcoem` directory.

2. **Check nginx error logs:**
   ```bash
   docker-compose logs nginx
   docker exec bcoem-nginx tail -f /var/log/nginx/error.log
   ```

3. **Check PHP-FPM logs:**
   ```bash
   docker-compose logs php
   docker exec bcoem-php tail -f /var/log/php_errors.log
   ```

4. **Verify file permissions:**
   ```bash
   docker exec bcoem-php ls -la /var/www/html/bcoem/
   docker exec bcoem-php ls -la /var/www/html/bcoem/site/
   ```

5. **Test with phpinfo:**
   Navigate to http://localhost:8080/bcoem/
   If you see PHP info page, the server is working but BCOE&M files are missing.

6. **Rebuild the containers:**
   ```bash
   docker-compose down
   docker-compose build --no-cache
   docker-compose up -d
   ```

### White Screen of Death
Edit `bcoem/site/paths.php` and set:
```php
define('DEBUG', TRUE);
```

### Permission Issues
Run:
```bash
docker exec bcoem-php chmod -R 777 /var/www/html/bcoem/user_images
docker exec bcoem-php chmod -R 777 /var/www/html/bcoem/user_docs
```

### Database Connection Issues
Check that the database container is running:
```bash
docker-compose ps
docker-compose logs db
```

## Stopping the Application

```bash
docker-compose down
```

To also remove the database volume:
```bash
docker-compose down -v
```

## Notes

- The application runs on Nginx with PHP 7.3 FPM
- All required PHP extensions are installed
- File upload limits are set to 64MB
- Session timeout is set to 30 minutes by default
- The setup uses MariaDB 10.5 for database compatibility
- Nginx is configured to handle BCOE&M's URL rewriting needs
