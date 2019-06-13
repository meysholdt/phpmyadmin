FROM phpmyadmin/phpmyadmin:fpm-alpine

# install MariaDB and developer tools
RUN apk add --no-cache mariadb mariadb-client bash git composer yarn

# create a default user for Gitpod
RUN addgroup -g 33333 gitpod && \
    adduser -D -h /home/gitpod -s /bin/sh -u 33333 -G gitpod gitpod
ENV HOME=/home/gitpod

# pid-file of Supervisor sould be in /run, because there we have write permission
RUN replace ";pidfile=/run/supervisord.pid" "pidfile=/run/supervisord.pid" -- /etc/supervisord.conf

# listen on port 8080 and place pid-file in /run
RUN replace "listen 80" "listen 8080"                           -- /etc/nginx.conf && \
    replace "pid /nginx.pid" "pid /run/nginx.pid"               -- /etc/nginx.conf

# relax php config for easy development
RUN replace "open_basedir = /var/www/html:/tmp/:/etc/phpmyadmin/" "open_basedir = none" -- /usr/local/etc/php/conf.d/php-phpmyadmin.ini && \
    replace "allow_url_fopen = Off" "allow_url_fopen = On"                              -- /usr/local/etc/php/conf.d/php-phpmyadmin.ini

# launch MariaDB via Supervisord
COPY mariadb.ini /etc/supervisor.d/mariadb.ini

# remove the phpMyAdmin sources from the docker image; Gitpod will provide a fresh clone.
RUN rm -rf /usr/src/phpmyadmin/

# set file permissions so that user `gitpod` can work with MairaDB, Nginx, PHP and PHPMyAdmin
RUN mkdir /run/mysqld && \
    chown -R gitpod:gitpod \
        /etc/supervisor* \
        /etc/nginx.conf \
        /etc/mysql \
        /etc/phpmyadmin \
        /usr/local/etc/ \
        /var/lib/nginx \
        /var/tmp/nginx \
        /var/log \
        /run \
        /sessions

USER gitpod

# on launch, we configure Nginx to serve phpMyAdmin form the Gitpod-provided git-clone.
RUN echo 'sed s%/var/www/html%$GITPOD_REPO_ROOT%g /etc/nginx.conf > /tmp/tmp && cat /tmp/tmp > /etc/nginx.conf' >> ~/.profile

# We don't want an entrypoint/cmd here: Gitpod will launch an IDE in this container
ENTRYPOINT [ ]
CMD []
