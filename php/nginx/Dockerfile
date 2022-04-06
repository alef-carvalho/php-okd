# Define PHP Version
ARG PHP_VERSION=8

FROM php:${PHP_VERSION}-fpm-alpine3.15

# Set Image Labels
LABEL Maintainer="Alef Carvalho <alef.carvalho@inovedados.com.br>"
LABEL Description="Lightweight container with Nginx 1.20 & PHP 8.0 based on Alpine Linux."

# Define PHP Configuration Variables
ENV PHP_COMPOSER_VERSION=2.0.12 \
    PHP_DATE_TIMEZONE="America/Sao_Paulo" \
    PHP_POST_MAX_FILESIZE=256M \
    PHP_UPLOAD_MAX_FILESIZE=256M \
    PHP_MEMORY_LIMIT=256M \
    PHP_MAX_EXECUTION_TIME=90 \
    PHP_DISPLAY_ERRORS=Off \
    PHP_DISPLAY_STARTUP_ERRORS=Off \
    PHP_CONFIG_FILE=/usr/local/etc/php/conf.d/app.ini

# Instal Libraries
RUN apk add --no-cache curl zip unzip nginx nano supervisor wget

# Install PHP Extension Manager
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/php-ext-install
RUN chmod +x /usr/local/bin/php-ext-install && php-ext-install \
    bcmath \
    ctype \
    fileinfo \
    json \
    mbstring \
    openssl \
    pdo \
    tokenizer \
    xml \
    opcache \
    zip \
    curl \
    dom \
    redis \
    imagick  \
    pcntl \
    xdebug

# Install Composer
RUN php-ext-install @composer-2.0.2

# Configure nginx
COPY ./config/nginx/app.conf /etc/nginx/http.d/default.conf

# Configure PHP and FPM
COPY ./config/php/app.ini $PHP_CONFIG_FILE
COPY ./config/php-fpm/php-fpm.conf /usr/local/etc/php-fpm.conf
COPY ./config/php-fpm/pools/*.conf /etc/php-fpm/pools/

# Configure Supervisor
COPY ./config/supervisor/supervisord.conf /etc/supervisor/supervisord.conf
COPY ./config/supervisor/jobs/*.ini /etc/supervisor/conf.d/

# Apply PHP Configuration Variables
RUN sed -ri -e "s!;date.timezone =!date.timezone = "$PHP_DATE_TIMEZONE"!g" $PHP_CONFIG_FILE && \
    sed -i -e "s/^post_max_size = 8M/post_max_size = $PHP_POST_MAX_FILESIZE/" $PHP_CONFIG_FILE && \
    sed -i -e "s/^upload_max_filesize = 2M/upload_max_filesize = $PHP_UPLOAD_MAX_FILESIZE/" $PHP_CONFIG_FILE && \
    sed -i -e "s/^memory_limit = 128M/memory_limit = $PHP_MEMORY_LIMIT/" $PHP_CONFIG_FILE && \
    sed -i -e "s/^max_execution_time = 30/max_execution_time = $PHP_MAX_EXECUTION_TIME/" $PHP_CONFIG_FILE && \
    sed -i -e "s/^display_errors = On/display_errors = $PHP_DISPLAY_ERRORS/" $PHP_CONFIG_FILE && \
    sed -i -e "s/^display_startup_errors = On/display_startup_errors = $PHP_DISPLAY_STARTUP_ERRORS/" $PHP_CONFIG_FILE && \
    sed -i -e "s/^LogLevel warn/LogLevel Error/" $PHP_CONFIG_FILE && \
    sed -i "/;session.save_path/c\session.save_path=\/tmp" $PHP_CONFIG_FILE

# Create log files
RUN mkdir -p /var/log/supervisor /run/supervisor

# Go-To Application Workdir
WORKDIR /app

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]