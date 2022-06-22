FROM php:5.4.45-apache
COPY . /var/www/html/
COPY ./dockerfilePHP56/s2i/bin/ /usr/libexec/s2i
RUN chown -R 1001:0 /usr/libexec/s2i
RUN chmod +x /usr/libexec/s2i/assemble
RUN chmod +x /usr/libexec/s2i/usage
RUN chmod +x /usr/libexec/s2i/run
USER 1001
EXPOSE 8080/tcp
CMD /usr/libexec/s2i/run
