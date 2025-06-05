FROM yiisoftware/yii2-php:8.1-apache

# Set Apache Document Root to web directory
RUN sed -i 's|/app/frontend/web|/app/web|g' /etc/apache2/sites-available/000-default.conf

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

# Fix autoloader path in index.php if it exists - Enhanced version
RUN if [ -f /app/web/index.php ]; then \
        echo "Fixing autoloader paths in index.php..."; \
        # Handle multiple common autoloader path patterns
        sed -i "s|require(__DIR__ . '/\.\./\.\./vendor/autoload\.php')|require(__DIR__ . '/../vendor/autoload.php')|g" /app/web/index.php; \
        sed -i "s|require_once(__DIR__ . '/\.\./\.\./vendor/autoload\.php')|require_once(__DIR__ . '/../vendor/autoload.php')|g" /app/web/index.php; \
        sed -i "s|require '\.\.\/\.\.\/vendor\/autoload\.php'|require __DIR__ . '/../vendor/autoload.php'|g" /app/web/index.php; \
        sed -i "s|require_once '\.\.\/\.\.\/vendor\/autoload\.php'|require_once __DIR__ . '/../vendor/autoload.php'|g" /app/web/index.php; \
        # If system Yii2 is available and local vendor doesn't exist, use system version
        if [ -f /usr/share/yii2/vendor/autoload.php ] && [ ! -f /app/vendor/autoload.php ]; then \
            sed -i "s|require.*vendor/autoload\.php.*|require('/usr/share/yii2/vendor/autoload.php');|g" /app/web/index.php; \
        fi; \
        echo "Autoloader paths fixed."; \
    fi

# Debug: Show final autoloader setup
RUN echo "=== Checking autoloader setup ===" && \
    if [ -f /app/vendor/autoload.php ]; then \
        echo "✓ Local vendor/autoload.php exists"; \
    else \
        echo "✗ Local vendor/autoload.php missing"; \
    fi && \
    if [ -f /usr/share/yii2/vendor/autoload.php ]; then \
        echo "✓ System Yii2 autoload.php exists"; \
    else \
        echo "✗ System Yii2 autoload.php missing"; \
    fi && \
    if [ -f /app/web/index.php ]; then \
        echo "=== index.php autoloader line ==="; \
        grep -n "require.*autoload" /app/web/index.php || echo "No autoloader require found"; \
    fi

EXPOSE 80
CMD ["apache2-foreground"]