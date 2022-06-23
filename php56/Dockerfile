FROM php:5.6.9-apache

USER root
RUN chmod -R 777 /var/run/apache2
RUN chmod -R 777 /var/lock/apache2
COPY php56/config /etc/apache2
COPY php56/src /var/www/html

EXPOSE 8080
