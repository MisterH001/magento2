# PHP 8.2 + Apache devcontainer base
FROM mcr.microsoft.com/devcontainers/php:8.2-apache


# Install OS deps and PHP extensions required by Magento 2
# (intl, bcmath, gd, mbstring, soap, xsl, zip, sockets, sodium, opcache, pdo_mysql)
RUN apt-get update && \
apt-get install -y \
libicu-dev libxml2-dev libxslt1-dev libzip-dev libonig-dev \
libpng-dev libjpeg-dev libfreetype6-dev libcurl4-openssl-dev \
unzip git && \
docker-php-ext-configure gd --with-freetype --with-jpeg && \
docker-php-ext-install -j"$(nproc)" \
intl bcmath gd mbstring soap xsl zip sockets sodium opcache pdo_mysql && \
rm -rf /var/lib/apt/lists/*


# Enable Apache rewrite and set DocumentRoot -> /workspaces/<repo>/pub
RUN a2enmod rewrite


# Tweak Apache to listen on 8080 in Codespaces
RUN sed -i 's/Listen 80/Listen 8080/' /etc/apache2/ports.conf && \
sed -i 's/:80>/:8080>/' /etc/apache2/sites-available/000-default.conf


# Update the vhost to allow overrides for Magento’s .htaccess
RUN sed -i 's#DocumentRoot /var/www/html#DocumentRoot ${APACHE_DOCUMENT_ROOT:-/var/www/html}#' /etc/apache2/sites-available/000-default.conf && \
sed -i 's#<Directory /var/www/>#<Directory ${APACHE_DOCUMENT_ROOT:-/var/www/html}>#' /etc/apache2/apache2.conf && \
sed -i 's/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf


# Install Composer (if not already in base image)
RUN if ! command -v composer >/dev/null 2>&1; then \
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
php composer-setup.php --install-dir=/usr/local/bin --filename=composer && \
rm composer-setup.php; \
fi


# Optional: Xdebug for step debugging
RUN pecl install xdebug && docker-php-ext-enable xdebug || true


# Recommended PHP settings for Magento
RUN echo "memory_limit=2G\nmax_execution_time=1800\nupload_max_filesize=64M\npost_max_size=64M\nerror_reporting=E_ALL\ndisplay_errors=On\nzend.assertions=1\n" > /usr/local/etc/php/conf.d/magento.ini


EXPOSE 8080
