#FROM openshift/base-centos7
FROM openshift/php-56-centos7
ENV PHP_VERSION=5.6

LABEL io.k8s.description="Platform for building and running PHP 5.6 applications" \
      io.k8s.display-name="Apache 2.4 with PHP 5.6" \
      io.openshift.expose-services="8080:http" \
      io.openshift.s2i.scripts-url="image:///usr/libexec/s2i" \
      io.s2i.scripts-url="image:///usr/libexec/s2i" \
      io.openshift.tags="builder,php,php56"


# Instala Apache
RUN yum -y install httpd httpd-tools
RUN yum -y install ca-certificates.noarch
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

# Copia S2I
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Copia Configuracion
COPY ./contrib/ /opt/app-root

#Lista lo instalado
RUN ls -l /opt/app-root/etc
RUN ls -l /etc/httpd
RUN php -v
RUN yum list installed *php*

# Modifica los archivos de configuracion de apache
RUN sed -i -f /opt/app-root/etc/httpdconf.sed /etc/httpd/conf/httpd.conf
RUN echo "IncludeOptional /opt/app-root/etc/conf.d/*.conf" >> /etc/httpd/conf/httpd.conf
RUN head -n151 /etc/httpd/conf/httpd.conf | tail -n1 | grep "AllowOverride All" || exit
RUN sed -i '/php_value session.save_path/d' /etc/httpd/conf.d/php.conf

# Cambio de usuario y permisos
RUN chown -R 1001:0 $STI_SCRIPTS_PATH
RUN chmod +x $STI_SCRIPTS_PATH/assemble
RUN chmod +x $STI_SCRIPTS_PATH/usage
RUN chmod +x $STI_SCRIPTS_PATH/run

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

CMD $STI_SCRIPTS_PATH/run
