FROM yiisoftware/yii2-php:8.1-apache

# Set Apache Document Root to web directory
RUN sed -i 's|/app/backend/web|/app/web|g' /etc/apache2/sites-available/000-default.conf

# Enable rewrite and allow .htaccess
RUN a2enmod rewrite && \
    sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf

# Set working directory
WORKDIR /app

# Copy app source
COPY . /app

# Create vendor directory and basic autoloader if composer.json doesn't exist
RUN if [ ! -f /app/composer.json ]; then \
        echo "Creating basic Yii2 setup without composer.json..."; \
        mkdir -p /app/vendor; \
        echo "<?php" > /app/vendor/autoload.php; \
        echo "// Basic autoloader for Yii2" >> /app/vendor/autoload.php; \
        echo "require_once('/usr/share/yii2/vendor/autoload.php');" >> /app/vendor/autoload.php; \
    else \
        echo "Installing composer dependencies..."; \
        composer install --no-dev --optimize-autoloader --no-interaction --prefer-dist; \
    fi

# Create necessary Yii2 directories
RUN mkdir -p /app/runtime /app/web/assets

# Fix permissions
RUN chown -R www-data:www-data /app && \
    find /app -type d -exec chmod 755 {} \; && \
    find /app -type f -exec chmod 644 {} \; && \
    chmod -R 777 /app/runtime /app/web/assets

# Fix autoloader path in index.php if it exists
RUN if [ -f /app/web/index.php ]; then \
        echo "Fixing autoloader path in index.php..."; \
        # Replace the autoloader path to point to the correct location
        sed -i "s|require(__DIR__ . '/\.\./\.\./vendor/autoload\.php')|require(__DIR__ . '/../vendor/autoload.php')|g" /app/web/index.php; \
        sed -i "s|require.*'/app/web/\.\./\.\./vendor/autoload\.php'|require(__DIR__ . '/../vendor/autoload.php')|g" /app/web/index.php; \
        # Also handle any other common patterns
        sed -i "s|require.*'\.\.\/\.\.\/vendor\/autoload\.php'|require(__DIR__ . '/../vendor/autoload.php')|g" /app/web/index.php; \
        # Show the fixed content for debugging
        echo "Contents of index.php after fix:"; \
        head -10 /app/web/index.php; \
    fi

EXPOSE 80
CMD ["apache2-foreground"]