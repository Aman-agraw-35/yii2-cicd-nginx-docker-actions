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

# Fix autoloader and config paths in index.php for Docker structure
RUN if [ -f /app/web/index.php ]; then \
        echo "=== Original index.php content ==="; \
        cat /app/web/index.php; \
        echo "=== Fixing paths for Docker structure ==="; \
        # Fix all paths from ../../ to ../ since we're moving from backend/web/ to web/
        sed -i 's|__DIR__ \. \x27/\.\./\.\./|__DIR__ \. \x27/../|g' /app/web/index.php; \
        sed -i 's|__DIR__ \. "/\.\./\.\./|__DIR__ \. "/../|g' /app/web/index.php; \
        sed -i 's|/\.\./\.\./|/../|g' /app/web/index.php; \
        echo "=== Fixed index.php content ==="; \
        cat /app/web/index.php; \
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