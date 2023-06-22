# Helm WP

Helm is Bernskiold Media's WordPress development framework. It's meant to be loaded as a Composer dev depenency and
proves a set of tools to help you develop WordPress themes and plugins. Helm is heavily inspired by/a WordPress adapted fork of Laravel Sail.

## Installation

To install, your project must be using composer. Require the package to your project:

```bash
composer require --dev bernskioldmedia/helm-wp
```

After installing, place a `docker-compose.yml` file in the root of your project. You can use the following as a starting
point:

```yaml
version: '3.3'
services:

    wordpress.test:
        build:
            context: ./wp-content/vendor/bernskioldmedia/helm-wp/runtimes/8.1
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: helm-7.4/app
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '80:80'
        environment:
            WWWUSER: '${WWWUSER}'
            HELM_WP: 1
            XDEBUG_MODE: '${HELM_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${HELM_XDEBUG_CONFIG:-client_host=host.docker.internal}'
        volumes:
            - '.:/var/www/html'
        networks:
            - helm

    # We try to use the same version of MySQL as the one used in production.
    # Other recipes are available in Notion.
    mariadb:
        image: 'mariadb:10.5'
        ports:
            - '3306:3306'
        environment:
            MYSQL_ROOT_PASSWORD: 'wordpressroot'
            MYSQL_DATABASE: 'wordpress'
            MYSQL_USER: 'wordpress_user'
            MYSQL_PASSWORD: 'wordpress_password'
        volumes:
            - 'helm-mariadb:/var/lib/mysql'
        healthcheck:
            test: [ "CMD", "mysqladmin", "ping", "-pwordpress_password" ]
            retries: 3
            timeout: 5s
        networks:
            - helm

    # Mailpit is a fake SMTP server that catches all emails sent in the application.
    # It has a web interface to view the emails at: http://localhost:8025/
    mailpit:
        image: 'axllent/mailpit:latest'
        ports:
            - '1025:1025'
            - '8025:8025'
        networks:
            - helm

volumes:
    helm-mariadb:
    wordpress:

networks:
    helm:
        driver: bridge
```

## Running helm

Helm ships with its own `helm` command. This is a wrapper around the `docker-compose` command that makes it easier to
run commands inside the container.

Adapt the path in the command to match your project structure and where composer packages are installed in your project,
if you have customized your vendor folder.

For more information on the `helm` command, run `./vendor/bin/helm help`.

### Creating an alias command
This alias enables you to just type helm to reach our Helm WP environment instead of having to always type the full path to the `vendor/bin` directory.

```bash
alias helm='[ -f helm ] && sh helm || sh vendor/bin/helm'
```

### Starting the container

To start the container, run:

```bash
./vendor/bin/helm up
```

### Stopping the container

To stop the container, run:

```bash
./vendor/bin/helm down
```

### Running commands inside the container

Helm supports running commands from within the container. This is useful for running Composer commands, WP CLI commands
and more.

```bash
# Composer
./vendor/bin/helm composer ...

# Node
./vendor/bin/helm node ...

# NPM
./vendor/bin/helm npm ...

# PHP Commands
./vendor/bin/helm php ...

# WP-CLI Commands
./vendor/bin/helm wp ...

```

## Using a different PHP version

Helm ships with support for PHP versions from 7.4 to 8.2, although we recommend not using 7.4 for anything new as it has
been officially deprecated.

To use a different PHP version, change the `context` in the `wordpress.test` service to point to the correct runtime
folder. You should also change the `image` to match the runtime folder name.

For example, to use PHP 8.1, change it to:

```yaml
  wordpress.test:
      build:
          context: ./wp-content/vendor/bernskioldmedia/helm-wp/runtimes/8.1
          image: helm-8.1/app
```

The available runtimes are:

- `7.4`
- `8.0`
- `8.1`
- `8.2`

