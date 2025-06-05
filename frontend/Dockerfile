FROM yiisoftware/yii2-php:8.1-apache

# Install Composer (if not already available in the base image)
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set Apache Document Root to web directory (changed from frontend/web to web)
RUN sed -i 's|/app/frontend/web|/app/web|g' /etc/apache2/sites-available/000-default.conf

# Enable rewrite and allow .htaccess
RUN a2enmod rewrite && \
    sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf

# Set working directory
WORKDIR /app

# Copy composer files first for better Docker layer caching
COPY composer.json composer.lock* ./

# Install PHP dependencies
RUN composer install --no-dev --optimize-autoloader --no-interaction --prefer-dist

# Copy the rest of your application source
COPY . /app

# Ensure vendor directory exists and regenerate autoloader
RUN mkdir -p /app/vendor && \
    composer dump-autoload --optimize

# Create necessary Yii2 directories with proper permissions
RUN mkdir -p /app/runtime /app/web/assets && \
    chown -R www-data:www-data /app && \
    find /app -type d -exec chmod 755 {} \; && \
    find /app -type f -exec chmod 644 {} \; && \
    chmod -R 777 /app/runtime /app/web/assets

# Fix potential autoloader path issues in index.php
RUN if [ -f /app/web/index.php ]; then \
        # Common Yii2 autoloader path fixes
        sed -i "s|require(__DIR__ . '/\.\./\.\./vendor/autoload\.php')|require(__DIR__ . '/../vendor/autoload.php')|g" /app/web/index.php && \
        sed -i "s|require(__DIR__ . '/\.\./\.\./frontend/vendor/autoload\.php')|require(__DIR__ . '/../vendor/autoload.php')|g" /app/web/index.php && \
        sed -i "s|require(__DIR__ . '/\.\./frontend/vendor/autoload\.php')|require(__DIR__ . '/../vendor/autoload.php')|g" /app/web/index.php; \
    fi

# Verify autoloader exists
RUN if [ ! -f /app/vendor/autoload.php ]; then \
        echo "ERROR: Autoloader not found!" && exit 1; \
    fi

# Expose Apache port
EXPOSE 80

# Start Apache in foreground
CMD ["apache2-foreground"]