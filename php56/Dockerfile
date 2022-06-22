FROM php:5.4.45-apache

ENV PHP_VERSION=5.6

LABEL io.k8s.description="Platform for building and running PHP 5.6 applications" \
      io.k8s.display-name="Apache 2.4 with PHP 5.6" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,php,php56"

COPY --from=composer:2.2  /usr/bin/composer /usr/local/bin/composer

# Copia S2I
COPY ./php56/s2i/bin/ /usr/libexec/s2i

# Copia Configuracion
COPY ./php56/contrib/ /opt/app-root
RUN ls -l /etc/apache2/conf-available

# Modifica los archivos de configuracion de apache
RUN sed -i -f /opt/app-root/etc/httpdconf.sed /etc/apache2/conf/httpd.conf
RUN echo "IncludeOptional /opt/app-root/etc/conf.d/*.conf" >> /etc/apache2/conf/httpd.conf
RUN head -n151 /etc/apache2/conf/httpd.conf | tail -n1 | grep "AllowOverride All" || exit
RUN sed -i '/php_value session.save_path/d' /etc/apache2/conf.d/php.conf

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
RUN mkdir /storage
RUN mkdir /storage/logs
RUN chown -R 1001:0 /storage/logs
RUN touch /storage/logs/lumen.log
RUN chmod -R 775 /storage/logs

USER 1001

EXPOSE 8080

#RUN /usr/libexec/s2i/assemble

CMD /usr/libexec/s2i/run
