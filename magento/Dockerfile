ARG BASE
FROM $BASE AS build
ARG BRANCH
ARG GIT_REPO
RUN php -i
ENV COMPOSER_AUTH='{"http-basic":{"repo.magento.com":{"username":"95dea79f51092c0a0e170b477812609a","password":"614a7ffdc6c260f04a5c67484e42c1f1"}}}'
RUN apk update && apk add git bash
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_HOME=/var/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=1.10.22 && composer global require hirak/prestissimo
RUN echo $GIT_REPO; git clone $GIT_REPO  -b $BRANCH  --depth 1 . && rm -rf .git
RUN composer install --no-dev --no-interaction --prefer-dist
RUN vendor/bin/ece-tools patch
# Workaround for the https://github.com/magento/magento2/issues/10041
ADD https://raw.githubusercontent.com/magento/magento2/2.3/dev/tests/integration/testsuite/Magento/Setup/Console/Command/_files/config/dump_config.php /app/app/etc/config.php
RUN bin/magento module:enable -vvv --all && php -dmemory_limit=-1  bin/magento -vvv setup:di:compile
RUN composer dump-autoload --optimize
RUN php -dmemory_limit=-1 bin/magento s:s:d -vvv -f
RUN bin/magento module:status -vvv
RUN rm -rf $COMPOSER_HOME /app/dev /app/phpserver  /app/var/* /tmp/*
FROM $BASE
COPY --from=build /app /app
EXPOSE 9000
CMD ["php-fpm", "-R"]
