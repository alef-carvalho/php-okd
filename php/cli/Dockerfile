FROM centos:8

# define php version
ARG PHP_VERSION=php74

# define system installation variables
ENV EPEL_REPOSITORY=https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm \
    REMI_REPOSITORY=https://rpms.remirepo.net/enterprise/remi-release-8.rpm

# define nano as default file editor
ENV VISUAL="nano"

# define php installation variables
ENV PHP_COMPOSER_VERSION=2.0.12 \
    PHP_DATE_TIMEZONE="America/Sao_Paulo" \
    PHP_POST_MAX_FILESIZE=256M \
    PHP_UPLOAD_MAX_FILESIZE=256M \
    PHP_MEMORY_LIMIT=256M \
    PHP_MAX_EXECUTION_TIME=90 \
    PHP_DISPLAY_ERRORS=Off \
    PHP_DISPLAY_STARTUP_ERRORS=Off

RUN echo "SELECTED PHP VERSION => $PHP_VERSION"

# add build labels
LABEL autor="Alef Carvalho <alef.carvalho@inovedados.com.br>" \
      io.k8s.description="PHP 7.4 CLI CentOS 8" \
      io.k8s.display-name="PHP 7.4 CLI CentOS 8" \
      io.openshift.tags="builder,php,openshift,okd" \
      io.openshift.expose-services="8080"

# install system packages
RUN yum update -y && yum install -y \
    glibc-langpack-en \
    glibc-langpack-es \
    glibc-all-langpacks \
    git \
    openssl \
    zip \
    unzip \
    curl \
    nc \
    which \
    wget \
    nano

# install php, nodejs and extensions
RUN yum install $EPEL_REPOSITORY yum-utils $REMI_REPOSITORY -y && \
    yum --enablerepo=remi && \
    curl -fsSL https://rpm.nodesource.com/setup_16.x | bash - && \
    curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | tee /etc/yum.repos.d/yarn.repo && \
    yum update -y && yum install -y \
        nodejs \
        yarn \
        supervisor \
        $PHP_VERSION-php \
        $PHP_VERSION-php-soap \
        $PHP_VERSION-php-tidy \
        $PHP_VERSION-php-imap \
        $PHP_VERSION-php-fpm \
        $PHP_VERSION-php-opcache \
        $PHP_VERSION-php-pdo \
        $PHP_VERSION-php-xml \
        $PHP_VERSION-php-json \
        $PHP_VERSION-php-cli \
        $PHP_VERSION-php-common \
        $PHP_VERSION-php-intl \
        $PHP_VERSION-php-pgsql \
        $PHP_VERSION-php-mysqlnd \
        $PHP_VERSION-php-process \
        $PHP_VERSION-php-mbstring \
        $PHP_VERSION-php-pecl-zip \
        $PHP_VERSION-php-gd \
        $PHP_VERSION-php-ldap \
        $PHP_VERSION-php-bcmath \
        $PHP_VERSION-php-mcrypt \
        $PHP_VERSION-php-xml \
        $PHP_VERSION-php-pear \
        $PHP_VERSION-php-curl

# change config files
RUN sed -ri -e 's!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g' /etc/httpd/conf/httpd.conf && \
    sed -i -e "s/^short_open_tag = Off/short_open_tag = On/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -ri -e "s!;date.timezone =!date.timezone = "$PHP_DATE_TIMEZONE"!g" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i -e "s/^post_max_size = 8M/post_max_size = $PHP_POST_MAX_FILESIZE/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i -e "s/^upload_max_filesize = 2M/upload_max_filesize = $PHP_UPLOAD_MAX_FILESIZE/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i -e "s/^memory_limit = 128M/memory_limit = $PHP_MEMORY_LIMIT/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i -e "s/^max_execution_time = 30/max_execution_time = $PHP_MAX_EXECUTION_TIME/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i -e "s/^display_errors = On/display_errors = $PHP_DISPLAY_ERRORS/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i -e "s/^display_startup_errors = On/display_startup_errors = $PHP_DISPLAY_STARTUP_ERRORS/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i -e "s/^LogLevel warn/LogLevel Error/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i "/;session.save_path/c\session.save_path=\/tmp" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i -e "s/^;curl.cainfo =/curl.cainfo='\/etc\/opt\/remi\/$PHP_VERSION\/cacert.pem'/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    sed -i -e "s/^;openssl.cafile=/openssl.cafile='\/etc\/opt\/remi\/$PHP_VERSION\/cacert.pem'/" /etc/opt/remi/$PHP_VERSION/php.ini && \
    curl -o /etc/opt/remi/$PHP_VERSION/cacert.pem https://curl.se/ca/cacert.pem

# configure path environment variable
ENV PATH=/opt/remi/$PHP_VERSION/root/usr/bin:/opt/remi/$PHP_VERSION/root/usr/sbin${PATH:+:${PATH}} \
    LD_LIBRARY_PATH=/opt/remi/$PHP_VERSION/root/usr/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}} \
    MANPATH=/opt/remi/$PHP_VERSION/root/usr/share/man:${MANPATH}

# install composer
RUN echo "INSTALLED PHP VERSION: $(php --version)" && echo "INSTALLING COMPOSER" && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php --install-dir=/usr/local/bin --filename=composer --version=$PHP_COMPOSER_VERSION && rm composer-setup.php && \
    yum autoremove -y &> /dev/null && yum clean all &> /dev/null && rm -rf /var/cache/yum &> /dev/null

# configure openshift user
RUN useradd inovedados && usermod -u 1001 inovedados && usermod -aG 0 inovedados && \
    mkdir /opt/composer_env && \
    mkdir /var/log/inovedados/ && \
    mkdir "${HOME}/.npm-packages" && \
    mkdir -p /usr/share/composer_install && \
    rm -rf /etc/localtime /var/www/html/* /etc/httpd/conf.d/welcome.conf && touch /etc/localtime /etc/timezone && \
    npm config set prefix "${HOME}/.npm-packages" && \
    chown -R 1001 /opt/composer_env /var/log/inovedados /var/opt/remi /etc/opt/remi /opt/remi /usr/share/composer_install /var/www /etc/timezone /etc/localtime && \
    chgrp -R 0 /opt/composer_env /var/log/inovedados /opt/remi /etc/opt/remi /var/opt/remi /var/www  /usr/share/composer_install /etc/timezone /etc/localtime && \
    chmod -R g=u /opt/composer_env /var/log/inovedados /var/opt/remi /etc/opt/remi /opt/remi /var/www /usr/share/composer_install /etc/timezone /etc/localtime

# set workdir to application files
WORKDIR /var/www

# switch to openshift default user
USER 1001:0

# expose port 8080 (needed by openshift)
EXPOSE 8080
