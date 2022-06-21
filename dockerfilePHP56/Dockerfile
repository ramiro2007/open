FROM openshift/base-centos7

ENV PHP_VERSION=5.6

LABEL io.k8s.description="Platform for building and running PHP 5.6 applications" \
      io.k8s.display-name="Apache 2.4 with PHP 5.6" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,php,php56"


# Instala Apache
RUN yum -y install httpd httpd-tools
RUN yum install -y epel-release http://rpms.remirepo.net/enterprise/remi-release-7.rpm

# Instala php
RUN yum --enablerepo=remi-php56 install -y php-xml php-soap php-xmlrpc php-mbstring php-json php-gd php-mcrypt php php-devel \
                    php-ast php-bcmath php-cli php-common php-dba php-gd php-intl php-json php-mbstring \
                    php-mcrypt php-mysqlnd php-opcache php-pdo php-pear php-pecl-igbinary \
                    php-pecl-memcache php-pecl-mongodb php-pecl-redis php-pecl-scrypt php-pecl-uuid php-pecl-xdebug \
                    php-pgsql php-odbc.x86_64 php-ldap
                    #php-sqlsrv php-pdo_sqlsrv autoconf automake php-devel.x86_64

#RUN pecl install imagick
#RUN yum -y install ca-certificates.noarch

# Instala SQLServer driver
#RUN curl https://packages.microsoft.com/config/rhel/7/prod.repo --insecure > /etc/yum.repos.d/mssql-release.repo
#RUN ACCEPT_EULA=Y yum -y install msodbcsql17
#RUN ACCEPT_EULA=Y yum -y install unixODBC-devel
#RUN pecl install sqlsrv
#RUN pecl install pdo_sqlsrv
#RUN echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/30-pdo_sqlsrv.ini
#RUN echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/20-sqlsrv.ini

COPY --from=composer:2.2  /usr/bin/composer /usr/local/bin/composer

# Copia S2I
COPY ./dockerfilePHP56/s2i/bin/ /usr/libexec/s2i

# Copia Configuracion
COPY ./dockerfilePHP56/contrib/ /opt/app-root

#Lista lo instalado
RUN ls -l /opt/app-root/etc
RUN ls -l /etc/httpd
RUN php -v
RUN yum list installed *php*
#RUN yum list installed unixODBC*
#RUN yum list installed msodbcsql*

# Modifica los archivos de configuracion de apache
RUN sed -i -f /opt/app-root/etc/httpdconf.sed /etc/httpd/conf/httpd.conf
RUN echo "IncludeOptional /opt/app-root/etc/conf.d/*.conf" >> /etc/httpd/conf/httpd.conf
RUN head -n151 /etc/httpd/conf/httpd.conf | tail -n1 | grep "AllowOverride All" || exit
RUN sed -i '/php_value session.save_path/d' /etc/httpd/conf.d/php.conf

# Cambio de usuario y permisos
RUN chown -R 1001:0 /usr/libexec/s2i
RUN chmod +x /usr/libexec/s2i/assemble
RUN chmod +x /usr/libexec/s2i/usage
RUN chmod +x /usr/libexec/s2i/run

RUN mkdir /tmp/sessions
RUN touch /etc/php.ini

RUN chown -R 1001:0 /opt/app-root /tmp/sessions
RUN chown -R 1001:0 /etc/php.d

RUN chmod -R a+rwx /tmp/sessions
RUN chmod -R ug+rwx /opt/app-root
RUN chmod -R ug+rwx /etc/php.d
RUN chmod -R ug+rw /etc/php.ini
RUN chmod -R a+rwx /var/run/httpd

#Copia fuentes
COPY . /opt/app-root/src

WORKDIR /opt/app-root/src
RUN php /usr/local/bin/composer install --no-dev --no-interaction -o
RUN mkdir /storage
RUN mkdir /storage/logs
RUN chown -R 1001:0 /storage/logs
RUN touch /storage/logs/lumen.log
RUN chmod -R 775 /storage/logs

USER 1001

EXPOSE 8080

#RUN /usr/libexec/s2i/assemble

CMD /usr/libexec/s2i/run
