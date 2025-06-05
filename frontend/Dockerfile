FROM yiisoftware/yii2-php:8.1-apache

# Set Apache Document Root to frontend
RUN sed -i 's|/app/frontend/web|/app/web|g' /etc/apache2/sites-available/000-default.conf

# Enable rewrite and allow .htaccess
RUN a2enmod rewrite && \
    sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf

WORKDIR /app

# Copy app source
COPY . /app

# Fix permissions
RUN chown -R www-data:www-data /app && \
    find /app -type d -exec chmod 755 {} \; && \
    find /app -type f -exec chmod 644 {} \;

EXPOSE 80
CMD ["apache2-foreground"]
