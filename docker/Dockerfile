FROM centos:centos7

RUN yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

RUN yum install -y https://rpms.remirepo.net/enterprise/remi-release-7.rpm

RUN yum install -y yum-utils nano which wget ksh zip unzip nodejs zsh redis git
RUN yum install -y gcc glibc glibc-common gd gd-devel
RUN yum-config-manager --disable 'remi-php*'
RUN yum-config-manager --enable   remi-php56

RUN yum repolist
RUN yum update -y

RUN yum install epel-release
RUN rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

RUN yum install -y php56w php56w-opcache php56w-cli php56w-common php56w-devel \
    php56w-gd php56w-intl php56w-mbstring php56w-mcrypt

RUN yum install -y php56w-mssql	php56w-mysql php56w-mysqlnd	php56w-odbc	php56w-opcache	\
    php56w-pdo	php56w-pear php56w-pecl-apcu php56w-pecl-geoip php56w-pecl-imagick \
    php56w-pecl-memcache php56w-pecl-xdebug php56w-pgsql php56w-soap

RUN yum install -y php56w-xml php56-php-json php56-php-phar php56-php-iconv php56-php-mbstring php56-php-redis php-redis

# Update Apache Configuration
RUN sed -E -i -e '/<Directory "\/var\/www\/html">/,/<\/Directory>/s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
RUN sed -E -i -e 's/DirectoryIndex (.*)$/DirectoryIndex index.php \1/g' /etc/httpd/conf/httpd.conf

#db2
RUN mkdir /opt/ibm \
    && cd /opt/ibm

ADD https://public.dhe.ibm.com/ibmdl/export/pub/software/data/db2/drivers/odbc_cli/linuxx64_odbc_cli.tar.gz /opt/ibm
ADD https://pecl.php.net/get/ibm_db2-1.9.7.tgz /opt/ibm

RUN tar xzf opt/ibm/linuxx64_odbc_cli.tar.gz -C /opt/ibm
RUN tar xzf opt/ibm/ibm_db2-1.9.7.tgz -C /opt/ibm

ENV IBM_DB_HOME=/opt/ibm/clidriver
ENV LD_LIBRARY_PATH=/opt/ibm/clidriver/lib


WORKDIR "/opt/ibm"

RUN  cd /opt/ibm/ibm_db2-1.9.7 \
    && phpize \
    && ./configure --with-IBM_DB2=/opt/ibm/clidriver \
    && make && make install
#    && ./configure  --with-PDO_IBM=/opt/ibm/clidriver \

COPY ./docker/composer-installer.sh /usr/local/bin/composer-installer

RUN chmod +x /usr/local/bin/composer-installer \
    && /usr/local/bin/composer-installer \
    && mv composer.phar /usr/local/bin/composer \
    && chmod +x /usr/local/bin/composer \
    && composer --version 

RUN curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-release.repo
RUN ACCEPT_EULA=Y yum install -y msodbcsql17
RUN ACCEPT_EULA=Y yum install -y mssql-tools
RUN echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
RUN source ~/.bashrc

RUN touch /var/log/xdebug.log
RUN /bin/bash -c "mkdir -p /var/log/php"
RUN touch /var/log/php/mono.log
RUN chmod -R +rwX  /var/log/
RUN chown -R apache:apache /var/log/

RUN #echo ' zend_extension =/usr/lib64/php/modules/xdebug.so '  >> /etc/php.ini
RUN echo ' xdebug.remote_host = host.docker.internal '  >> /etc/php.ini
RUN echo ' xdebug.remote_handler =dbgp '  >> /etc/php.ini
RUN echo ' xdebug.idekey =PHPSTORM '  >> /etc/php.ini
RUN echo ' xdebug.remote_enable = 1 '  >> /etc/php.ini
RUN echo ' xdebug.remote_connect_back = 0 '  >> /etc/php.ini
RUN echo ' xdebug.remote_port = 9000 '  >> /etc/php.ini
RUN echo ' xdebug.scream = 0 '  >> /etc/php.ini
RUN echo ' xdebug.cli_color = 1 '  >> /etc/php.ini
RUN echo ' xdebug.show_local_vars = 1 '  >> /etc/php.ini
RUN echo ' xdebug.max_nesting_level = 512 '  >> /etc/php.ini
RUN echo ' xdebug.mode =debug '  >> /etc/php.ini
RUN echo ' xdebug.log =/var/log/xdebug.log '  >> /etc/php.ini
RUN echo ' date.timezone = America/New_York '  >> /etc/php.ini

RUN #echo ' extension=redis.so '  >> /etc/php.ini
RUN echo ' extension=ibm_db2.so '  >> /etc/php.ini
RUN #echo ' extension=pdo_ibm.so '  >> /etc/php.ini

RUN echo 'alias vi="vim"' >> /root/.bash_profile
RUN echo 'alias ll="ls -la"' >> /root/.bash_profile
RUN echo 'alias cdp="cd /var/www/html"' >> /root/.bash_profile
RUN echo 'alias cdl="cd /var/log"' >> /root/.bash_profile
RUN echo 'alias pbcopy="xclip -selection clipboard"' >> /root/.bash_profile
RUN echo 'alias pbpaste="xclip -selection clipboard -o"' >> /root/.bash_profile
RUN echo 'alias phpinfo="echo  \"<?php phpinfo(); \" | php "' >> /root/.bash_profile
RUN source /root/.bash_profile

RUN yum install -y supervisor beanstalkd iproute xclip zsh net-tools

RUN mkdir -p /etc/httpd/sites-enabled
RUN chown -R apache:apache /var/www/html
COPY docker/httpd.conf /etc/httpd/conf/httpd.conf
COPY docker/apache.conf /etc/httpd/sites-enabled/000-default.conf
COPY docker/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY docker/start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

ENV APACHE_LOG_DIR=/var/log/httpd

EXPOSE 80
COPY . /var/www/html

CMD ["/bin/bash","start-container"]

