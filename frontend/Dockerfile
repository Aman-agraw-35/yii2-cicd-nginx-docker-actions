# FROM yiisoftware/yii2-php:8.1-apache

# # Change Apache doc root
# RUN sed -i 's|/app/web|/app/frontend/web|g' /etc/apache2/sites-available/000-default.conf

# WORKDIR /app
# # Copy frontend files to /app/frontend/
# COPY . /app/frontend/

FROM yiisoftware/yii2-php:8.1-apache

# Set document root to /app/frontend/web
RUN sed -i 's|/app/web|/app/frontend/web|g' /etc/apache2/sites-available/000-default.conf

# Set working directory
WORKDIR /app

# Copy entire Yii2 project
COPY . /app

# Enable Apache rewrite module (needed for Yii2 routing)
RUN a2enmod rewrite \
    && chown -R www-data:www-data /app

EXPOSE 80

CMD ["apache2-foreground"]
