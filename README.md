# Helm WP

Helm is Bernskiold Media's WordPress development framework. It's meant to be loaded as a Composer dev depenency and
proves a set of tools to help you develop WordPress themes and plugins.

- Docker container for local development

## Installation

To install, your project must be using composer. Require the package to your project:

```bash
composer require --dev bernskioldmedia/helm-wp
```

After installing, place a `docker-compose.yml` file in the root of your project. You can use the following as a starting point:

```yaml
version: '3.3'
services:

  wordpress.test:
    build:
      context: ./wp-content/vendor/bernskioldmedia/helm-wp/runtimes/7.4
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
