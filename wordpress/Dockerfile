FROM ubuntu:bionic
RUN apt-get update
RUN apt-get install -y nginx php-fpm php-mysql systemd
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
RUN sed -i -e "s/;\?daemonize\s*=\s*yes/daemonize = no/g" /etc/php/7.2/fpm/php-fpm.conf
RUN echo "\nclear_env = no" >> /etc/php/7.2/fpm/pool.d/www.conf
#COPY FILE
COPY wordpress/ /var/www/html/
# Nginx config
RUN rm /etc/nginx/sites-enabled/default
ADD wordpress/wordpress.conf /etc/nginx/sites-available/
RUN ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/wordpress.conf
# Expose ports.
EXPOSE 80
# RUN PHP and NGINX
CMD /etc/init.d/php7.2-fpm start && nginx
