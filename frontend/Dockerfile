FROM yiisoftware/yii2-php:8.1-apache

# Change Apache doc root
RUN sed -i 's|/app/web|/app/frontend/web|g' /etc/apache2/sites-available/000-default.conf

WORKDIR /app
COPY . /app
