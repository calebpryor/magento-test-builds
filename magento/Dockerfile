ARG BASE
FROM $BASE AS build
ARG COMPOSER_AUTH
ARG BRANCH
ARG GIT_REPO
RUN apk update && apk add git bash
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_HOME=/var/composer
ENV COMPOSER_AUTH=$COMPOSER_AUTH
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer self-update --1
RUN composer global require hirak/prestissimo
RUN git clone $GIT_REPO  -b $BRANCH  --depth 1 . && rm -rf .git
RUN composer install --no-dev --no-interaction --prefer-dist
RUN vendor/bin/ece-tools patch
# Workaround for the https://github.com/magento/magento2/issues/10041
ADD https://raw.githubusercontent.com/magento/magento2/2.4.2/dev/tests/integration/testsuite/Magento/Setup/Console/Command/_files/config/dump_config.php /app/app/etc/config.php
RUN bin/magento module:enable -vvv --all && bin/magento -vvv setup:di:compile
RUN composer dump-autoload --optimize
RUN bin/magento s:s:d -vvv -f
RUN bin/magento module:status -vvv
RUN rm -rf $COMPOSER_HOME /app/dev /app/phpserver  /app/var/* /tmp/*
FROM $BASE
COPY --from=build /app /app
EXPOSE 9000
CMD ["php-fpm", "-R"]
