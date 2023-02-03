FROM php:8.0-apache

MAINTAINER echo <echo.magento@gmail.com>

RUN apt-get update \
	&& DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
	software-properties-common \
	&& apt-get update \
	&& DEBIAN_FRONTEND=noninteractive apt-get install -y \
	nodejs \
	libfreetype6-dev \
	libicu-dev \
    libssl-dev \
	libjpeg62-turbo-dev \
	libmcrypt-dev \
	libedit-dev \
	libedit2 \
	libxslt1-dev \
	libzip-dev \
	apt-utils \
	gnupg \
	redis-tools \
	mariadb-client \
	zlib1g-dev \
	libpng-dev \
	libjpeg-dev \
	git \
	vim \
	wget \
	curl \
	lynx \
	psmisc \
	unzip \
	tar \
	cron \
    nano \
    openssl \
	bash-completion \
	&& apt-get clean


RUN docker-php-ext-configure \
  	gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/; \
  	docker-php-ext-install \
  	opcache \
  	bcmath \
  	gd \
  	intl \
  	mysqli \
  	pdo_mysql \
  	soap \
  	xsl \
  	zip \
  	sockets


RUN apt-get update \
  	&& apt-get install -y \
  	libpcre3 \
  	libpcre3-dev \
  	# php-pear \
  	&& pecl install oauth \
  	&& echo "extension=oauth.so" > /usr/local/etc/php/conf.d/docker-php-ext-oauth.ini


RUN	curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin/ --filename=composer

ENV PATH="/var/www/.composer/vendor/bin/:${PATH}"


RUN DEBIAN_FRONTEND=noninteractive apt-get -y install golang-go \
   && mkdir /opt/go \
   && export GOPATH=/opt/go \
   && go get github.com/mailhog/mhsendmail


ADD /.docker/config/php.ini /usr/local/etc/php/php.ini
COPY /.docker/bin/* /usr/local/bin/
COPY /.docker/users/* /var/www/
RUN chmod +x /usr/local/bin/*


RUN chmod 777 -Rf /var/www /var/www/.* \
	&& chown -Rf www-data:www-data /var/www /var/www/.* \
	&& usermod -u 1000 www-data \
	&& chsh -s /bin/bash www-data\
	&& a2enmod rewrite \
    && a2enmod expires \
    && a2enmod ssl \
    && a2enmod socache_shmcb \
	&& a2enmod headers

VOLUME /var/www/html
WORKDIR /var/www/html

RUN mkdir /etc/apache2/ssl
RUN openssl req -new -x509 -nodes -out /etc/apache2/ssl/server.pem -keyout /etc/apache2/ssl/server.key -days 3650 -subj '/CN=php80.localhost'

RUN apt-get update && apt-get install -y openssl && \
    openssl genrsa -des3 -passout pass:x -out server.pass.key 2048 && \
    openssl rsa -passin pass:x -in server.pass.key -out server.key && \
    rm server.pass.key && \
    openssl req -new -key server.key -out server.csr -subj "/C=UK/ST=Warwickshire/L=Leamington/O=OrgName/OU=IT Department/CN=php80.localhost" && \
    openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

RUN cp server.crt /etc/apache2/ssl/ssl.crt
RUN cp server.key /etc/apache2/ssl/ssl.key

RUN a2ensite default-ssl.conf
RUN echo "Mutex posixsem" >> /etc/apache2/apache2.conf
