# Laravel Sail dengan Laravel Octane

https://blog.devgenius.io/laravel-sail-with-https-swoole-ddab7f5303ec

```php
sail up -d
```

```php
sail composer require laravel/octane
```

```php
sail artisan octane:install
```

**Append/Edit** in `config/octane.php` file

```php
'swoole' => [
        'ssl' => true,
        'options' => [
            'ssl_cert_file' => '/etc/swoole/ssl/certs/sail-selfsigned.crt',
            'ssl_key_file' => '/etc/swoole/ssl/private/sail-selfsigned.key',
        ]
    ],
```

**Append** in `.env` file

```bash
OCTANE_SERVER=swoole
OCTANE_HTTPS=true
```

```php
sail artisan sail:publish
```

**update** `docker/8.0/supervisord.conf` (Line 8) as stated in [official documentation](https://laravel.com/docs/8.x/octane#swoole).

```php
command=/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port=8000 --watch
```

**Append** in `docker-compose.yml` file, `ports` section (~Line 13)

I need to map the port `8000` to public

```yaml
 ports:
            - '${APP_PORT:-80}:80'
            - 8000:8000
```

After sail user creation (~Line 47) —  
`RUN useradd -ms /bin/bash — no-user-group -g $WWWGROUP -u 1337 sail`

add these lines…

```
RUN mkdir -p /etc/swoole/ssl/certs/ /etc/swoole/ssl/private/
RUN openssl req -x509 -nodes -days 365 -subj "/C=CA/ST=QC/O=Artisan, Inc./CN=localhost" \
    -addext "subjectAltName=DNS:localhost" -newkey rsa:2048 \
    -keyout /etc/swoole/ssl/private/sail-selfsigned.key \
    -out /etc/swoole/ssl/certs/sail-selfsigned.crt;
RUN chmod 644 /etc/swoole/ssl/certs/*.crt
RUN chown -R root:sail /etc/swoole/ssl/private/
RUN chmod 640 /etc/swoole/ssl/private/*.key
```

```bash
sail build --no-cache
```

```bash
sail artisan octane:status
```
