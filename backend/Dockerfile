FROM yiisoftware/yii2-php:8.1-apache

# Set Apache Document Root to backend/web
RUN sed -i 's|/app/backend/web|/app/web|g' /etc/apache2/sites-available/000-default.conf

# Enable mod_rewrite and allow .htaccess
RUN a2enmod rewrite && \
    sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf

WORKDIR /app

# Copy your backend code
COPY . /app

# Set correct permissions
RUN chown -R www-data:www-data /app && \
    find /app -type d -exec chmod 755 {} \; && \
    find /app -type f -exec chmod 644 {} \;

# Expose Apache port
EXPOSE 80

# Start Apache in foreground
CMD ["apache2-foreground"]
