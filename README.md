# PHP alpine docker image
https://hub.docker.com/r/slymuzzle/php

This image works under unprivileged user

### Example of change uid and gid for dev environment
```
&& usermod --uid ${USER_UID} webuser \
    && groupmod --gid ${USER_GID} webgroup\
    && find / -user 9999 -exec chown webuser:webgroup {} \;
```

### Example of permissions set
```
RUN find /www -type d -exec chmod -R 555 {} \; \
    && find /www -type f -exec chmod -R 444 {} \; \
    && find /www/public -type d -exec chmod -R 755 {} \; \
    && find /www/public -type f -exec chmod -R 644 {} \;
```

### Overriding or extending the configuration

If you want to augment of replace the configuration of Nginx, PHP or FPM, there are multiple options:
- Place one or more configuration files in specific directories to augment the configuration
- If that does not suit your needs, you can also simply overwrite the configuration files altogether

These are the files to add or overwrite in order to configure the different parts of the webstack:

| Application               | Copy files into this directory | Overwrite this file if needed |
|---------------------------|--------------------------------|-------------------------------|
| PHP core directives (8.1) | /etc/php81/conf.d/             | /etc/php81/php.ini            |
| PHP-FPM (8.1)             | /etc/php81/php-fpm.d/          | /etc/php81/php-fpm.conf       |
| Nginx                     | /etc/nginx/conf.d/             | /etc/nginx/nginx.conf         |
| Supercronic               | -                              | /etc/supercronic/crontab      |