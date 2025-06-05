# FROM yiisoftware/yii2-php:8.1-apache

# # Change Apache doc root  
# RUN sed -i 's|/app/web|/app/backend/web|g' /etc/apache2/sites-available/000-default.conf

# WORKDIR /app
# # Copy backend files to /app/backend/
# COPY . /app/backend/

FROM yiisoftware/yii2-php:8.1-apache

# Set document root to /app/backend/web
RUN sed -i 's|/app/web|/app/backend/web|g' /etc/apache2/sites-available/000-default.conf

# Set working directory
WORKDIR /app

# Copy your Yii2 app code
COPY . /app

# Set permissions (important for Yii)
RUN chown -R www-data:www-data /app \
    && a2enmod rewrite

EXPOSE 80

CMD ["apache2-foreground"]
